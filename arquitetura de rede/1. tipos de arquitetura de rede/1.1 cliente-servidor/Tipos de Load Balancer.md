---
tags:
  - node
---
Quando colocamos o load balancer sob a lente do modelo OSI, percebemos que ele **não atua em todas as camadas**, isso não é o comum de acontecer. O comum é o LB atuando nas camadas 4 (Layer 4/L4) e 7 (Layer 7/L7). 

E, aqui, vemos os primeiros dois tipos de load balancers, que são: *Balanceador de Carga de Rede (Network Load Balancer - NLB)*, que atua na camada 4, e *Balanceador de Carga de Aplicação (Application Load Balancer - ALB)*, que atua na camada 7.

A verdade é que muitas organizações grandes usam dois (ou mais) load balancers em cascata, não um único que faz tudo:

``` 
Cliente
   |__ L4 LB (DNS, ataque de volume, conexões TCP)
       |__ L7 LB (HTTP routing, SSL, cookies, WAF)
           |__ Aplicação (servidor web, API, etc.)
```

O **L4 LB é a primeira linha**, ultra-rápido e especializado em volume de requisições. Detecta SYN Flood, UDP Flood, TCP Exhaustion antes que chegue mais adiante e distribui tráfego legítimo para múltiplos L7 LBs.

O **L7 LB é a segunda linha**, mais inteligente. Ele entende HTTP, cookies, URLs. Faz roteamento baseado em conteúdo, SSL Termination e, também, detecção de ataque de aplicação.

Depois o tráfego chega aos servidores de aplicação, que fazem seu próprio trabalho.

Então, vamos entender melhor a atuação do load balancer nas camadas 4 e 7, o NLB e o ALB.

## Balanceamento de carga de rede (Network Load Balancer - NLB)

O balanceador, operando na camada 4 do modelo OSI, como vimos, é chamado de Balanceador de Carga de Rede (Network Load Balancer - NLB).

Na camada 4, o balanceador trabalha primariamente com dois protocolos de transporte: **TCP (Transmission Control Protocol)** e **UDP (User Datagram Protocol)**. O TCP é orientado a conexão, o que significa que antes de transmitir dados, ele estabelece uma conexão formal entre cliente e servidor através de um *handshake (aperto de mão)*. O UDP, por sua vez, é sem conexão — envia datagramas diretamente, sem ter criado uma conexão antes.​

![[tcp_vs_udp.jpeg]]

Para aplicações TCP (como HTTP, HTTPS, SSH), o balanceador mantém **estado de conexão**, rastreando cada fluxo de comunicação. Para UDP (DNS, VoIP, jogos online), o balanceador é mais "stateless" — apenas envia datagramas, embora possa ainda ter algum rastreamento básico.

### Handshake TCP 

Quando um cliente quer se conectar a um servidor através de um Network Load Balancer, ele inicia um handshake TCP de três vias (three-way handshake): **SYN, SYN-ACK, ACK**. O balanceador não é um mero tubo/túnel — ele intervém ativamente nesse processo.

1. O cliente envia um pacote **SYN** (sincronizar) para o VIP (Virtual IP) do balanceador, incluindo um número de sequência inicial (ISN). 

2. O balanceador recebe esse SYN e faz a primeira decisão: **qual servidor backend receberá essa conexão?** Aplica o algoritmo de distribuição escolhido (Round Robin, Least Connections, etc.) para decidir. Suponha que escolha o Servidor A. 

3. O balanceador então cria uma **segunda conexão TCP** entre si e o Servidor A, enviando um novo SYN para o Servidor A (com um ISN diferente). Isso cria uma duplicação de estado: duas conexões TCP, não uma.

4. Quando o Servidor A responde com **SYN-ACK**, o balanceador recebe e converte essa resposta para parecer que vem do balanceador, não do servidor. 

5. Retransmite um SYN-ACK para o cliente, usando números de sequência que parecem naturais do ponto de vista do cliente. 

6. O cliente então envia um **ACK** final, completando o handshake com o balanceador. 

7. O balanceador, simultaneamente, completa o handshake com o Servidor A.


Resultado? Existem agora **duas conexões TCP independentes**, ambas estabelecidas e rastreadas pelo balanceador. Uma de cliente para balanceador, outra de balanceador para Servidor A. O balanceador sincroniza essas duas conexões, garantindo que os dados do cliente fluam para o servidor e as respostas do servidor fluam de volta para o cliente.​

### Connection State Tracking

Uma coisa muito interessante de um balanceador NLB é o **connection state tracking**, o que é isso?

É tabela interna no balanceador que registra cada conexão ativa, seu estado e para qual servidor backend ela está mapeada. Cada entrada nessa tabela contém informações críticas: 

- IP de origem (cliente); 
- IP de destino (VIP); 
- porta de origem; 
- porta de destino; 
- estado atual (SYN_SENT, ESTABLISHED, FIN_WAIT, TIME_WAIT, CLOSED);
- timestamps de última atividade; e
- o servidor backend associado.

Na medida que o cliente e servidor trocam pacotes, o balanceador monitora as **flags TCP** para entender quando a conexão muda de estado. Quando recebe um SYN, marca como SYN_SENT. Quando recebe dados sendo transmitidos, sabe que está em ESTABLISHED. Quando recebe um FIN (finalize), sabe que alguém quer encerrar. Quando ambos os lados enviaram FIN, a conexão vai para TIME_WAIT e eventualmente é purgada da tabela.

Esse rastreamento é essencial por várias razões. 

- Primeiro, **garante persistência**: toda requisição de um cliente sempre vai para o mesmo servidor backend, desde que a conexão esteja ativa. Se um novo SYN chegar antes da anterior ser encerrada, pode ser sincronizado com o mesmo backend, ou pode ir para um diferente (dependendo da configuração). 

- Segundo, **previne vazamentos de recursos**: conexões que ficam em estado TIME_WAIT por muito tempo são eventualmente fechadas e removidas, liberando memória. Se isso não ocorrer, o balanceador pode ficar sem recursos e começar a rejeitar novas conexões.​

### Tradução de Endereços - NAT Bidirecional

Durante toda a vida da conexão, o balanceador vai modifica continuamente os cabeçalhos dos pacotes para fazer a "ilusão" funcionar. 

Quando ele recebe um pacote do cliente, faz **DNAT (Destination NAT)**, reescrevendo o endereço IP de destino do pacote (que era o VIP do balanceador) para o IP real do servidor backend escolhido. Também pode reescrever a porta de destino se necessário.

Quando o servidor responde, o balanceador faz **SNAT (Source NAT)**, reescrevendo o endereço IP de origem (que era o servidor backend) para parecer que vem do VIP do balanceador. 

Do ponto de vista do cliente, toda a comunicação parece acontecer com o balanceador, não com nenhum servidor backend específico.

Existem **três modos operacionais** de NAT em balanceadores NLB moderno. 

- No modo **NAT Completo**, tanto IP de origem quanto destino são reescritos em ambas as direções, ocultando completamente a topologia interna. 

- No modo **Preservação de Origem**, o IP de origem do cliente é preservado ao enviar para o backend, permitindo que o servidor veja o IP real do cliente (útil para logging e auditoria). 

- No modo **Transparente (DSR — Direct Server Return)**, o balanceador não modifica nada — apenas encaminha pacotes, exigindo que o servidor esteja configurado para rotear respostas através do balanceador.

### Balanceador VS DDoS

Um dos maiores valores do NLB é a capacidade de **defender contra ataques de negação de serviço (DDoS)**. O balanceador pode detectar e mitigar vários tipos de ataques:


- **SYN Flood**

	O **SYN Flood** — "inundação de SYN" — é quando um atacante envia uma avalanche de pacotes SYN falsificados, sem nunca completar o handshake TCP. O servidor (ou balanceador) aloca memória para cada SYN, esperando que o cliente responda com ACK. Se ninguém responde, a memória fica marcada como "pending connection" e eventualmente esgota. 

	Um Network Load Balancer pode detectar padrões anormais (muitos SYNs vindos do mesmo IP, ou de IPs aparentemente aleatórios), e aplicar **SYN cookies** (uma técnica criptográfica que não aloca memória até o handshake estar completo), ou simplesmente descartar SYNs excessivos de IPs suspeitas.


- **UDP Flood**

	O **UDP Flood** é semelhante, mas aqui há um envio massivo de datagramas para sobrecarregar o servidor. O balanceador pode aplicar **rate limiting**, limitando quantos datagramas por segundo aceitará de um IP específico ou em total.


- **TCP Exhaustion**

	O **TCP Exhaustion** é quando um atacante abre muitas conexões TCP válidas e as mantém abertas, exaurindo o limite de conexões simultâneas. O balanceador pode detectar que um IP específico tem anormalmente muitas conexões abertas (centenas, enquanto outros têm dezenas) e limitar ou descartar novas conexões daquele IP.


Essas proteções ocorrem **abaixo do nível de aplicação**, então não importa o que a aplicação faz — se alguém está bombardeando o servidor, o balanceador NLB pode bloquear antes que chegue ao aplicativo.​

### Limitações de NLB


Apesar de sua sofisticação, o NLB tem limitações. Primeiro, **sem inspeção de conteúdo**, o balanceador não consegue distinguir entre tráfego HTTP legítimo e um **HTTP Flood** (muitas requisições HTTP coordenadas). No nível de TCP, tudo parece normal — apenas muitas conexões legítimas. Um HTTP Flood só seria detectado em L7.

Além disso, o **NLB não entende a aplicação**. Se há um bug no Servidor A que causa loop infinito, por exemplo, o Network Load Balancer puro continuará mandando tráfego para A indefinidamente, porque ao nível de TCP, a conexão tecnicamente está "viva". Alguns *health checks* simples, como ICMP ping ou TCP port check, não conseguem detectar falhas a nível de aplicação. Um servidor pode estar "respondendo" em TCP mas servindo erro HTTP 500 para todas as requisições.

Outra questão é a **falta de inteligência de roteamento**. O balanceador não consegue dizer "URLs contendo `/api/*` vão para um pool de servidores especializados, enquanto `/media/*` vão para outro". Tudo é apenas "distribua para o próximo servidor na lista". Isso é onde **L7** brilha.

### Performance e Eficiência

Mas, também devemos reconhecer um grande diferencial do NLB, que é a sua **velocidade**. Como decisões são tomadas apenas em cima de IP e porta (informações presentes no primeiro pacote), o balanceador pode processar milhões de conexões por segundo com latência mínima.

Balanceadores NLB geralmente operam a **kernel level** em Linux (usando tecnologias como IPVS/LVS ou nftables), permitindo que a decisão de roteamento seja tomada pelo próprio kernel, contornando context switches para userspace.

Essa eficiência permite que um único balanceador NLB gerencie tráfego de centenas de gigabits por segundo, enquanto um L7 comparável pode ficar para trás porque tem que inspecionar cada byte do payload.​

---
## Balanceamento de Carga de Aplicação (Application Load Balancer - ALB)


Agora, o load balancer na camada 7, chamado de Balanceador de Carga de Aplicação (Application Load Balancer - ALB). A diferença fundamental entre NLB e ALB é que o balanceador na camada 4 trabalha com *streams (fluxos)* enquanto o balanceador na camada 7 trabalha com *mensagens (requisições)*. 

No NLB, o balanceador não sabe e nem se importa em saber qual é o conteúdo de um pacote TCP — ele só quer saber se o TCP é legítimo. Já no ALB, o balanceador **lê e entende o conteúdo da requisição HTTP**, dissecando cada elemento: URL, método HTTP, headers, query strings, cookies, corpo da mensagem.

Isso requer que o Application Load Balancer atuante como um **proxy reverso completo** (reverse proxy). Diferente de um NLB que pode passar pacotes praticamente sem modificação após redirecionar IP:porta, um ALB **termina** a conexão HTTP do cliente, lê a requisição inteira, toma uma decisão sobre para qual backend enviar, cria uma **nova requisição HTTP** para o backend (potencialmente modificada), envia essa requisição, recebe a resposta do backend, modifica essa resposta se necessário, e a envia de volta ao cliente. Tudo isso acontece dentro de uma única sessão TCP lógica do cliente.​

### O que um ALB enxerga?

Quando um cliente envia uma requisição HTTP para um balanceador ALB, o primeiro pacote chega com dados que parecem simples, mas contêm uma riqueza de informações. Uma requisição HTTP típica se parece com isso:

``` http
GET /api/users?id=42&sort=name HTTP/1.1 
Host: example.com User-Agent: 
Mozilla/5.0 Authorization: Bearer eyJhbGc... 
Cookie: session_id=abc123; user_pref=dark_mode 
Accept: application/json 
Content-Type: application/json 
Custom-Header: special-value 

{   
	"search": "john",  
	"filters": ["active", "verified"] 
}
```

Um balanceador ALB pode extrair cada uma dessas linhas, inspecionar elas e tomar decisões baseadas nelas. Nada disso seria visível em NLB — o NLB apenas veria "é uma conexão TCP na porta 80/443, então, envie para o próximo servidor".

### Roteamento baseado em conteúdo

A coisa que o ALB vai trazer é o **roteamento inteligente baseado em conteúdo**. E existem várias estratégias que podem ser adotadas:

- O **Roteamento baseado em URL Path** permite que diferentes caminhos vão para diferentes backends. Por exemplo, todas as requisições para `/api/*` vão para um pool de servidores especializados em APIs, enquanto `/media/*` vão para um pool otimizado para servir arquivos estáticos, e `/admin/*` vão para um pool de servidores administrativos com acesso reforçado. E, como sabemos até esse ponto, o cliente não sabe disso — para ele, é um único servidor.​

- Temos, também, o **Roteamento baseado em hostnames virtuais**, na qual ele permite que uma mesma infraestrutura sirva múltiplos sites. Se alguém acessa `api.example.com`, vai para o pool de API. Se acessa `media.example.com`, vai para o pool de mídia. Tudo por trás do mesmo VIP, diferenciado pelo header HTTP `Host`.

- O **Roteamento baseado em HTTP headers** permite decisões arbitrariamente sofisticadas. Um header `X-Client-Type: mobile` pode rotear para um pool otimizado para mobile, enquanto `X-Client-Type: desktop` vai para um diferente. Um header `Authorization: Premium-token-xyz` pode rotear para servidores premium com mais recursos, enquanto `Authorization: free-token-abc` vai para servidores de recursos gratuitos.

- Já o **Roteamento baseado em query strings** permite que parâmetros de URL determinem o destino. `/search?service=email` vai para o backend de email, enquanto `/search?service=files` vai para o backend de arquivos.

- Há, ainda, o **Roteamento baseado em métodos HTTP**, que diferencia GET de POST. Todas as requisições GET podem ir para um pool read-optimized com cache, enquanto POST/PUT/DELETE vão para um pool transactional com garantias de integridade.

- Por fim, temos o **Roteamento baseado em IP de origem**, que permite que clientes internos sejam roteados para um pool diferente de clientes externos. Um IP de origem dentro de uma subnet corporativa pode acessar um backend administrativo, enquanto IPs externos não conseguem.​

Essas estratégias podem ser **combinadas em regras complexas**: "Se o path é `/api/*` AND o método é POST AND o header `Authorization` existe, envie para o backend de escrita segura. Caso contrário, se o path é `/api/*` AND o método é GET, envie para o backend de leitura em cache".​

### SSL/TLS Termination

Uma das coisas mais críticos que o ALB opera é sobre **TLS Termination**. O cliente estabelece uma conexão SSL/TLS criptografada com o balanceador ALB (não com o servidor). O balanceador **descriptografa a requisição** recebida, inspeciona-a completamente, depois encripta uma **nova requisição** para enviar ao backend.

Isso cria três benefícios:

Primeiro, **centraliza o gerenciamento de certificados**. Os certificados SSL/TLS ficam no balanceador, não em cada um dos servidores backend. Atualizar um certificado expirado acontece em um único lugar, não em centenas de servidores.

Segundo, **offload de computação**. Criptografia é computacionalmente cara, é custoso. Fazer em um balanceador centralizado (que pode usar hardware acelerador de criptografia) economiza recursos dos servidores de aplicação. Cada servidor não precisa pagar o custo de criptografia — apenas recebe dados já descriptografados.

Terceiro, mas não menos importante, **visibilidade da requisição**. Sem TLS Termination, um balanceador não conseguiria ler URLs ou headers HTTPS (estariam criptografados). Com TLS Termination, o balanceador consegue inspecionar requisições HTTPS de forma inteligente.

Mas nem tudo são flores, há também um risco: **a quebra da criptografia end-to-end**. Com TLS Termination, há dois canais: cliente-balanceador (criptografado), balanceador-servidor (geralmente não criptografado internamente). Se alguém conseguir acessar o balanceador ou a rede interna, pode interceptar dados em texto plano.

### Persistência de sessão e cookies

Um ALB é excelente em manter **persistência de sessão** usando mecanismos bem sofisticados. Ele pode inspecionar cookies de sessão, extrair IDs de sessão e garantir que todas as requisições de uma mesma sessão vão para o mesmo backend.

Suponha que um usuário faça login em um servidor backend chamado "C3PO". O servidor C3PO cria uma sessão em memória local e retorna um cookie `session_id=xyz`. Um balanceador NLB não conseguiria rastrear essa sessão — só veria conexões TCP. Mas um ALB lê o cookie, extrai o ID, e **mapeia** internamente "toda requisição com este cookie deve ir para o servidor que criou a sessão". Se aquele servidor sair do pool ou morrer, o ALB pode até **migrar a sessão** para outro servidor (dependendo da configuração), ou informar ao cliente que a sessão expirou.

Além disso, um ALB pode *reescrever cookies*, pode *adicionar cookies próprios* (como `X-Forwarded-For` para preservar o IP original do cliente), *remover cookies sensíveis* antes de enviar ao backend, ou *modificar atributos de cookie* como `HttpOnly`, `Secure`, `SameSite` para melhorar a segurança.

### Proteção contra ataques de aplicação

Um balanceador ALB integrado com proteção de segurança funciona como um **Firewall de Aplicação Web (WAF) miniaturizado**. Ele consegue detectar e bloquear ataques que um NLB jamais veria:

- **SQL Injection** é quando um atacante tenta enviar comandos SQL maliciosos através de parâmetros de URL ou corpo de requisição.

- **Cross-Site Scripting (XSS)** tenta injetar JavaScript malicioso em respostas.

- **Path Traversal** tenta acessar arquivos fora do diretório esperado usando `../`. 

- **HTTP Flood** é quando um atacante envia um volume massivo de requisições HTTP legítimas. Diferente de um SYN Flood (bloqueável em NLB), um HTTP Flood é tecnicamente requisições HTTP válidas. Um ALB pode detectar padrões anormais, como: "este IP enviou 10.000 requisições em 1 segundo", "este User-Agent está enviando requisições muito rapidamente sem intervalo natural", "este cookie está sendo usado de múltiplos IPs simultaneamente (sessão hijacking)".

- **Rate Limiting por requisição** é possível apenas em ALB. Um NLB pode limitar conexões TCP, mas não requisições HTTP. Um ALB pode dizer "máximo de 100 requisições GET por minuto, máximo de 10 requisições POST por minuto", oferecendo um controle rigoroso.​

### Reescrita de headers e URLs

Um ALB pode modificar requisições de forma sofisticada. Pode **injetar headers** que informam ao backend dados úteis: adicionar `X-Forwarded-For` com o IP real do cliente (porque o servidor veria apenas o IP do balanceador se não fosse informado), adicionar `X-Forwarded-Proto` para indicar se foi HTTP ou HTTPS original, adicionar `X-Real-IP`, etc.

Pode **remover headers sensíveis** antes de enviar ao backend, como `Authorization` de cliente, removendo Headers internos que não devem vazar para o backend.

Pode **reescrever URLs**. Se um cliente pede `/old-path/resource`, o balanceador pode redirecionar ou reescrever internamente para `/new-path/resource` antes de enviar ao backend, permitindo que migrations de URL aconteçam sem mudanças no backend.

Pode **redirecionar requisições**. Se a requisição for para HTTP (não segura), pode redirecionar o cliente para HTTPS, adicionando camadas de segurança.

### Performance

O grande trade-off do ALB é **latência aumentada**. Enquanto um NLB pode rotear um pacote em microsegundos, um ALB precisa:

1. Receber o pacote inteiro (ou pelo menos headers suficientes);
2. Fazer parsing da requisição HTTP;
3. Executar regras de roteamento (potencialmente complexas);
4. Criar uma nova requisição para o backend;
5. Esperar a resposta do backend;
6. Fazer parsing da resposta;
7. Modificá-la se necessário;
8. Enviá-la de volta.

Isso adiciona um nível de latência considerável  — tipicamente alguns milissegundos por requisição, comparado a microsegundos em NLB. 

Para aplicações que precisam de ultra-baixa latência (trading de alta frequência, jogos competitivos), ALB pode ser inaceitável. Já para aplicações web típicas, essa latência é negligenciável considerando os benefícios do roteamento inteligente.

Além disso, a carga de processamento é muito maior. Um ALB consome muitos mais recursos CPU que um NLB. Um Network Load Balancer de alto desempenho pode rotear terabits por segundo, em comparação, um ALB robusto tipicamente roteia gigabits por segundo antes de ficar sobrecarregado. 

É a razão pela qual arquiteturas modernas usam NLB como primeira linha (para volume) e ALB como segunda linha (para inteligência).

---

Mas, além dos Balanceadores de Carga de Rede (Network Load Balancer - NLB) e Balanceadores de Carga de Aplicação (Application Load Balancer - ALB), temos o *DNS Load Balancer* e o *Global Server Load Balancer (GSLB)*.

Em termos simples, o **DNS Load Balancer (DNS LB)** vai distribui o tráfego apenas manipulando as respostas DNS (qual IP devolver para um nome). Já o **Global Server Load Balancer (GSLB)** é um “load balancer de data centers”, normalmente ele também é baseado em DNS, que decide **em qual região / data center** o usuário vai cair, antes do balanceamento local por L4/L7.

Vamos conhecer melhor o DNS Load Balancer!

## DNS Load Balancer

O balanceamento de carga de DNS é talvez o tipo mais simples, operando no nível de resolução de nomes de domínio. 

Quando um usuário digita um URL no navegador, seu cliente DNS faz uma consulta para resolver o nome de domínio em um endereço IP. Um DNS load balancer intervém neste processo, retornando **diferentes endereços IP em respostas diferentes**, distribuindo tráfego sem qualquer dispositivo de balanceamento intermediário.​

Suponha que você tem dois servidores: `server1.example.com` (192.168.1.10) e `server2.example.com` (192.168.1.20). Um DNS LB tradicional pode ser configurado com múltiplos registros A para diferentes servidores:

``` text
example.com. 300 IN A 192.168.1.10 ; server1 
example.com. 300 IN A 192.168.1.20 ; server2
```
\**TTL 300 = 5 minutos de cache. Todos os registros têm o mesmo nome (`example.com`), mas IPs diferentes.*

Quando clientes digita `example.com` e faz a pesquisa, o servidor DNS retorna **ambos os IPs, em ordem alternada**. O cliente A recebe `[192.168.1.10, 192.168.1.20]` e conecta ao primeiro, enquanto o cliente B recebe `[192.168.1.20, 192.168.1.10]` (ordem revertida) e conecta ao também ao primeiro. Isso é o **DNS Round Robin**.​

Há, ainda, o algoritmo **Round-robin ponderado (weighted)**, na qual o DNS devolve IPs com pesos diferentes (às vezes repetindo o IP mais “pesado” mais vezes) para que um servidor mais forte receba mais tráfego.

Outro meio usado é o **GeoDNS**. Aqui, o DNS autoritativo olha o IP do resolvedor, mais especificamente a aproximação da localização dele com o cliente, e devolve o IP do servidor/região mais próxima.​

Por fim, temos o algoritmo de **Latência / health-aware**, em que o DNS LB integra health checks e métricas de latência. Se um backend está fora do ar ou lento, ele é removido ou menos utilizado nas respostas.

### Ameaças e vulnerabilidades específicas de DNS LB

Muitas vulnerabilidades não são do _load balancing_ em si, mas do **ecossistema DNS que ele depende**. As principais são:

#### a) Comprometimento do painel DNS / registrador

Se alguém tomar o controle do painel do registrador ou do provedor de DNS, ele pode fazer duas coisas. Primeiro, ele pode **alterar os IPs** usados no load balancing para apontar para servidores maliciosos (phishing, MITM, roubo de credenciais). Segundo, ele pode **alterar pesos/geo-routes** para que apenas parte do tráfego seja desviada, tornando o ataque sutil e difícil de detectar.​

Este é, na prática, um dos maiores riscos: **seu domínio inteiro passa pela confiança no provedor de DNS**.

#### b) DNS cache poisoning / spoofing

Ataques de **envenenamento de cache DNS** tentam fazer resolvers armazenarem respostas falsas (nome do domínio → IP malicioso). Mesmo que o autoritativo esteja correto, o usuário é enganado na borda.​

Em contexto de DNS LB:

- O atacante pode “fixar” um IP malicioso ou um IP específico do pool, anulando o balanceamento.

- TTLs muito altos fazem com que um cache envenenado dure mais tempo.​

Medidas como **DNSSEC** ajudam a mitigar a adulteração de respostas, mas ainda há ataques sofisticados (fragmentation-based, NXNSAttack etc.).​

#### c) DDoS contra a infraestrutura DNS

DNS LB depende de:

- Autoritativos disponíveis
    
- Resolvedores funcionando corretamente
    

Ataques DDoS podem mirar:

- Os servidores autoritativos do domínio (como no ataque à Dyn em 2016).
    
- Resolvedores públicos (Google, Cloudflare, etc.), impactando resolução global.​
    

Mesmo que seus servidores de aplicação estejam de pé, **se o DNS cair, seu site “deixa de existir” na prática**.

DNS LB pode ajudar um pouco (anycast, múltiplos autoritativos, distribuição global), mas a camada DNS continua sendo um alvo crítico.​

#### d) Ataques à lógica de balanceamento de autoritativos (Disablance)

Trabalhos recentes mostraram que **a própria lógica de balanceamento entre autoritativos pode ser explorada**. O ataque Disablance demonstra que, explorando decisões de certos resolvers e implementações de servidores DNS (BIND, PowerDNS, Microsoft DNS), um atacante consegue forçar que a maior parte do tráfego vá sempre para um autoritativo específico, quebrando o balanceamento previsto e criando um novo ponto único de falha para DDoS ou envenenamento.​

Ou seja, **até o load balancing entre nameservers autoritativos pode ser sabotado**.

#### e) Vulnerabilidades em software de DNS LB

Quando o DNS LB é implementado via **proxy/balancer DNS dedicado**, ele passa a ter as mesmas classes de vulnerabilidades de qualquer software de rede:

- Exemplo recente: vulnerabilidade de DoS em DNSdist (PowerDNS), explorável com requisições DoH malformadas, que derruba o serviço e, portanto, interrompe total ou parcialmente a resolução DNS.​

- Ataques de complexidade algorítmica (como KeyTrap, em DNSSEC) que fazem o resolver consumir CPU absurdamente, causando DoS com poucos pacotes.​


#### f) Problemas de TTL, cache e visibilidade

- **TTL alto**: um erro ou envenenamento perdura por horas ou dias.
    
- **TTL muito baixo sem monitoramento**: flutuações constantes podem esconder anomalias (ex.: trânsito súbito para IP estranho, mas “parece só mais uma mudança normal”).​
    
- Logs incompletos de consultas e mudanças em registros dificultam **forense** após incidente.​
    

#### g) Configuração inconsistente entre backends

DNS LB **não garante** que os servidores por trás estejam:

- Com o mesmo certificado TLS
    
- Com os mesmos níveis de patch, WAF, autenticação, etc.
    

Se um dos IPs do pool estiver com TLS fraco, app desatualizada ou configurações permissivas, o atacante foca nele. Em load balancing, **o nó mais fraco define seu nível real de segurança**.​


## Global Server Load Balancer (GSLB)

O GSLB é um **meta-balanceador** que distribui tráfego entre múltiplas regiões geográficas ou data centers, frequentemente usando DNS como mecanismo principal. É o balanceamento "de balanceadores".​

Pense em três camadas:

1. O **GSLB (global)** decide “vai para a região Brasil, EUA ou Europa?” — normalmente via DNS.

2. **Load balancer local (L4/L7)**, dentro da região escolhida, decide para qual servidor específico mandar.

3. **Servidor de aplicação**: processa a requisição.

### Ameaças e vulnerabilidades específicas de GSLB

Como GSLB é quase sempre baseado em DNS, **herda todos os riscos de DNS LB** já citados. Além disso, traz desafios próprios:

#### a) “Single point of global truth”

Seu GSLB (seja um serviço de DNS avançado, seja um cluster de appliances) se torna o **ponto central de decisão global**:

- Um erro de configuração global → tráfego do mundo inteiro aponta para data center errado, ou para IP inexistente.
    
- Comprometimento do painel/API do GSLB → atacante pode redirecionar usuários de regiões específicas para infraestrutura maliciosa, mantendo outras regiões intactas (ataque furtivo).​


Ou seja: ele elimina pontos únicos regionais, mas cria um **plano de controle global altamente sensível**.

#### b) Inconsistências de DNS e “split-brain”

Se diferentes provedores ou clusters GSLB não estiverem perfeitamente sincronizados, podem surgir:

- Resoluções diferentes para o mesmo FQDN dependendo do resolvedor, violando a política de roteamento pretendida.
    
- Cenários em que uma região deveria estar em “failover” mas alguns resolvers continuam recebendo IPs daquela região por TTL/caches diferentes.​
    

Isso complica muito troubleshooting e resposta a incidentes.

#### c) Dependência de dados de geolocalização

GSLB muitas vezes decide com base em GeoIP ou similar:

- Bases GeoIP desatualizadas podem mandar usuários para regiões distantes ou proibidas (compliance).
    
- IPs de VPN, proxies ou atacantes podem enganar o roteamento (“parecer” de outra região).​
    

Em cenários de ataque, isso pode ser explorado para:

- Concentrar tráfego malicioso em regiões menos protegidas.
    
- Contornar geo-blocks aplicados a regiões específicas.
    

#### d) DDoS e ataques estruturais contra a camada de decisão global

Além dos ataques DNS genéricos (amplificação, NXNSAttack, KeyTrap etc.), existem cenários específicos:

- DDoS volumétrico focado nos **autoritativos de GSLB**, derrubando a resolução global.​
    
- Ataques como Disablance, que visam explorar falhas na forma como resolvers e autoritativos distribuem consultas entre múltiplos NS, para sobrecarregar ou isolar um único nó da infraestrutura DNS global.​
    
- Exploração de vulnerabilidades de software no próprio GSLB (por exemplo, em appliances ou proxies DNS usados para as decisões).​
    

#### e) Coerência de segurança entre regiões

GSLB assume que **todas as regiões atendem o mesmo nível de segurança**:

- Mesma versão da aplicação
    
- Mesmo hardening de SO
    
- Mesmos certificados TLS, chaves e políticas
    
- Mesmo WAF e regras
    

Na prática, é comum uma região “secundária” estar menos atualizada ou menos vigiada. O atacante pode então:

- Forçar ou enganar o roteamento para essa região mais fraca.
    
- Explorar vulnerabilidades ali que já foram corrigidas na região principal.​
    

#### f) Implicações de compliance e soberania de dados

Se o GSLB for mal configurado:

- Usuários da UE podem ser roteados para data centers fora da UE, violando GDPR.
    
- Dados sensíveis podem replicar para regiões onde a legislação é menos rígida, expondo riscos legais e de privacidade.​