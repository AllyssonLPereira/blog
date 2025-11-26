
Quando falamos de aplicações modernas que precisam lidar com muitos usuários ao mesmo tempo — redes sociais, serviços de streaming, jogos online, sites de e-commerce — existe um componente que atua para manter tudo funcionando com rapidez e estabilidade: o **Load Balancer**, ou balanceador de carga.

Ele é o “maestro” do modelo Cliente-Servidor. É quem decide **para qual servidor cada cliente vai**, como o tráfego será distribuído e como manter tudo funcionando mesmo quando partes da infraestrutura falham.

Sem ele, aplicações de grande escala seriam simplesmente incapazes de lidar com a quantidade de acessos simultâneos que vemos hoje.

![[load_balancer.jpg]]

---
### O que é um Load Balancer e por que ele é tão importante?

O load balancer é um dispositivo (ou software) que fica entre o cliente e o conjunto de servidores responsáveis por processar suas requisições. Ele recebe cada solicitação e escolhe **qual servidor do conjunto — o _backend_ — deve atendê-la**.

Esse encaminhamento não é aleatório. O balanceador analisa critérios como:

- carga atual de cada servidor;
- disponibilidade;
- número de conexões simultâneas;
- tipo da requisição;
- políticas definidas pelo administrador.


Enquanto isso, o cliente nem percebe que existe mais de um servidor no sistema. Para ele, tudo parece uma única entidade. Essa “ilusão de unidade” é um dos pilares da escalabilidade moderna.

No modelo Cliente-Servidor tradicional, um único servidor atendia a todas as solicitações. Isso funciona bem para poucas requisições, mas quando a aplicação cresce, o servidor começa a sentir o aumento de latência, gargalos de CPU, limites de memória e conexões TCP, queda por sobrecarga e interrupções de serviço.

Com isso, vem o load balancer, resolvendo isso de maneira elegante **dividindo a carga entre vários servidores**, impedindo que um único nó tenha que lidar com tudo.

Além disso, ele cria uma camada de **alta disponibilidade**. Se um servidor falhar, o balanceador  simplesmente para de enviar requisições para ele. Para o usuário, nada muda.

Alguns balanceadores, especialmente os usados em nuvem, fazem até **checagens constantes de saúde** (_health checks_) nos servidores. Se um deles parar de responder, ele é removido automaticamente do pool e, quando volta ao normal, é reinserido.

---
### Algoritmos de distribuição

O que são algoritmos de distribuição? Eles são a base que determina como o load balancer vai escolher para qual servidor encaminhar cada requisição dentro do modelo cliente-servidor. A escolha do servidor não é aleatória. Os balanceadores usam algoritmos como:

- **Round Robin**, este é o mais simples. Ele funciona como uma fila circular que distribui as requisições de forma sequencial e igualitária a cada servidor disponível. Ele é eficaz quando os servidores têm capacidades equivalentes e as requisições têm cargas semelhantes.

- Já o **Least Connections** foca no estado atual dos servidores, enviando cada nova requisição para o servidor com o menor número de conexões ativas no momento. Isso permite um balanceamento mais dinâmico e mais adaptativo, ideal quando as sessões ficam abertas por tempo variável ou as cargas não são uniformes.

- O **IP Hash** mantem a persistência da sessão, garantindo que todas as requisições vindas de um mesmo endereço IP sejam encaminhadas para o mesmo servidor.

- Os algoritmos **Weighted Round Robin** e **Weighted Least Connections** são variações do Round Robin e Least Connection que levam em conta o poder computacional ou capacidade de cada servidor para distribuir trabalho proporcionalmente. Servidores mais potentes recebem mais requisições, enquanto servidores menos robustos recebem menos.


> *Cada algoritmo atende cenários específicos, e escolher o errado pode gerar gargalos inesperados.*

---
### Load Balancer no modelo OSI

Agora, quando colocamos o load balancer sob a lente do modelo OSI, percebemos que ele **não atua em todas as camadas**, isso não é o comum. O comum é o LB atuando nas camadas 4 e 7.

A verdade é que muitas organizações grandes usam dois (ou mais) load balancers em cascata, não um único que faz tudo:

``` 
Cliente
   |__ L4 LB (DNS, ataque de volume, conexões TCP)
       |__ L7 LB (HTTP routing, SSL, cookies, WAF)
           |__ Aplicação (servidor web, API, etc.)
```

O **L4 LB é a primeira linha**, ultra-rápido e especializado em volume. Detecta SYN Flood, UDP Flood, TCP Exhaustion antes que chegue mais adiante. Distribui tráfego legítimo entre múltiplos L7 LBs.

O **L7 LB é a segunda linha**, mais inteligente. Entende HTTP, cookies, URLs. Faz roteamento baseado em conteúdo, SSL Termination, detecção de ataque de aplicação.

Depois o tráfego chega aos **servidores aplicacionais**, que fazem seu próprio trabalho.

Então, vamos entender melhor a atuação do load balancer nas camadas 4 e 7.

#### Camada 4 – Transporte (TCP/UDP)

Na camada 4, o balanceador trabalha primariamente com dois protocolos de transporte: **TCP (Transmission Control Protocol)** e **UDP (User Datagram Protocol)**. O TCP é orientado a conexão, o que significa que antes de transmitir dados, ele estabelece uma conexão formal entre cliente e servidor através de um *handshake (aperto de mão)*. O UDP, por sua vez, é sem conexão — envia datagramas diretamente, sem ter criado uma conexão antes.​

Para aplicações TCP (como HTTP, HTTPS, SSH), o balanceador mantém **estado de conexão**, rastreando cada fluxo de comunicação. Para UDP (DNS, VoIP, jogos online), o balanceador é mais "stateless" — apenas envia datagramas, embora possa ainda ter algum rastreamento básico.

##### O Handshake TCP e a Intervenção do Balanceador

Quando um cliente quer se conectar a um servidor através de um balanceador L4, ele inicia um handshake TCP de três vias (three-way handshake): **SYN, SYN-ACK, ACK**. O balanceador não é um mero tubo/túnel — ele intervém ativamente nesse processo.

1. O cliente envia um pacote **SYN** (sincronizar) para o VIP (Virtual IP) do balanceador, incluindo um número de sequência inicial (ISN). 

2. O balanceador recebe esse SYN e faz a primeira decisão: **qual servidor backend receberá essa conexão?** Aplica o algoritmo de distribuição escolhido (Round Robin, Least Connections, etc.) para decidir. Suponha que escolha o Servidor A. 

3. O balanceador então cria uma **segunda conexão TCP** entre si e o Servidor A, enviando um novo SYN para o Servidor A (com um ISN diferente). Isso cria uma duplicação de estado: duas conexões TCP, não uma.

4. Quando o Servidor A responde com **SYN-ACK**, o balanceador recebe e converte essa resposta para parecer que vem do balanceador, não do servidor. 

5. Retransmite um SYN-ACK para o cliente, usando números de sequência que parecem naturais do ponto de vista do cliente. 

6. O cliente então envia um **ACK** final, completando o handshake com o balanceador. 

7. O balanceador, simultaneamente, completa o handshake com o Servidor A.


Resultado: existem agora **duas conexões TCP independentes**, ambas estabelecidas e rastreadas pelo balanceador. Uma de cliente para balanceador, outra de balanceador para Servidor A. O balanceador sincroniza essas duas conexões, garantindo que os dados do cliente fluam para o servidor e as respostas do servidor fluam de volta para o cliente.​

##### Connection State Tracking: O Coração do L4

Uma coisa muito interessante de um balanceador L4 é o **connection state tracking**, o que é isso?

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

##### Tradução de Endereços: NAT Bidirecional

Durante toda a vida da conexão, o balanceador vai modifica continuamente os cabeçalhos dos pacotes para fazer a "ilusão" funcionar. 

Quando ele recebe um pacote do cliente, faz **DNAT (Destination NAT)**, reescrevendo o endereço IP de destino do pacote (que era o VIP do balanceador) para o IP real do servidor backend escolhido. Também pode reescrever a porta de destino se necessário.

Quando o servidor responde, o balanceador faz **SNAT (Source NAT)**, reescrevendo o endereço IP de origem (que era o servidor backend) para parecer que vem do VIP do balanceador. 

Do ponto de vista do cliente, toda a comunicação parece acontecer com o balanceador, não com nenhum servidor backend específico.

Existem **três modos operacionais** de NAT em balanceadores L4 moderno. 

- No modo **NAT Completo**, tanto IP de origem quanto destino são reescritos em ambas as direções, ocultando completamente a topologia interna. 

- No modo **Preservação de Origem**, o IP de origem do cliente é preservado ao enviar para o backend, permitindo que o servidor veja o IP real do cliente (útil para logging e auditoria). 

- No modo **Transparente (DSR — Direct Server Return)**, o balanceador não modifica nada — apenas encaminha pacotes, exigindo que o servidor esteja configurado para rotear respostas através do balanceador.

##### Balanceador VS DDoS

Um dos maiores valores do L4 é a capacidade de **defender contra ataques de negação de serviço (DDoS)**. O balanceador pode detectar e mitigar vários tipos de ataques:


- **SYN Flood**

	O **SYN Flood** — "inundação de SYN" — é quando um atacante envia uma avalanche de pacotes SYN falsificados, sem nunca completar o handshake TCP. O servidor (ou balanceador) aloca memória para cada SYN, esperando que o cliente responda com ACK. Se ninguém responde, a memória fica marcada como "pending connection" e eventualmente esgota. 

	Um balanceador L4 pode detectar padrões anormais (muitos SYNs vindos do mesmo IP, ou de IPs aparentemente aleatórios), e aplicar **SYN cookies** (uma técnica criptográfica que não aloca memória até o handshake estar completo), ou simplesmente descartar SYNs excessivos de IPs suspeitas.


- **UDP Flood**

	O **UDP Flood** é semelhante, mas aqui há um envio massivo de datagramas para sobrecarregar o servidor. O balanceador pode aplicar **rate limiting**, limitando quantos datagramas por segundo aceitará de um IP específico ou em total.


- **TCP Exhaustion**

	O **TCP Exhaustion** é quando um atacante abre muitas conexões TCP válidas e as mantém abertas, exaurindo o limite de conexões simultâneas. O balanceador pode detectar que um IP específico tem anormalmente muitas conexões abertas (centenas, enquanto outros têm dezenas) e limitar ou descartar novas conexões daquele IP.


Essas proteções ocorrem **abaixo do nível de aplicação**, então não importa o que a aplicação faz — se alguém está bombardeando o servidor, o balanceador L4 pode bloquear antes que chegue ao aplicativo.​

##### Limitações de L4

Apesar de sua sofisticação, o L4 tem limitações. Primeiro, **sem inspeção de conteúdo**, o balanceador não consegue distinguir entre tráfego HTTP legítimo e um **HTTP Flood** (muitas requisições HTTP coordenadas). No nível de TCP, tudo parece normal — apenas muitas conexões legítimas. Um HTTP Flood só seria detectado em L7.

Além disso, o **L4 não entende a aplicação**. Se há um bug no Servidor A que causa loop infinito, por exemplo, o balanceador L4 puro continuará mandando tráfego para A indefinidamente, porque ao nível de TCP, a conexão tecnicamente está "viva". Alguns *health checks* simples, como ICMP ping ou TCP port check, não conseguem detectar falhas a nível de aplicação. Um servidor pode estar "respondendo" em TCP mas servindo erro HTTP 500 para todas as requisições.

Outra questão é a **falta de inteligência de roteamento**. O balanceador não consegue dizer "URLs contendo `/api/*` vão para um pool de servidores especializados, enquanto `/media/*` vão para outro". Tudo é apenas "distribua para o próximo servidor na lista". Isso é onde **L7** brilha.

##### Performance e Eficiência

Mas, também devemos reconhecer um grande diferencial do L4, que é a sua **velocidade**. Como decisões são tomadas apenas em cima de IP e porta (informações presentes no primeiro pacote), o balanceador pode processar milhões de conexões por segundo com latência mínima.

Balanceadores L4 geralmente operam a **kernel level** em Linux (usando tecnologias como IPVS/LVS ou nftables), permitindo que a decisão de roteamento seja tomada pelo próprio kernel, contornando context switches para userspace.

Essa eficiência permite que um único balanceador L4 gerencie tráfego de centenas de gigabits por segundo, enquanto um L7 comparável pode ficar para trás porque tem que inspecionar cada byte do payload.​

##### Camada 7 – Aplicação: A Inteligência

Agora, o load balancer na camada 7. A diferença fundamental entre L4 e L7 é que **L4 trabalha com streams (fluxos)** enquanto **L7 trabalha com mensagens (requisições)**. 

No L4, o balanceador não sabe e nem se importa em saber qual é o conteúdo de um pacote TCP — ele só quer saber se é TCP legítimo. Já no L7, o balanceador **lê e entende o conteúdo da requisição HTTP**, dissecando cada elemento: URL, método HTTP, headers, query strings, cookies, corpo da mensagem.

---

### Load Balancer de Hardware vs Software

Você encontrará load balancers em duas formas:

- **Hardware**: equipamentos robustos usados em data centers (F5, Citrix Netscaler).

- **Software**: executados em servidores comuns, VMs ou containers (Nginx, HAProxy, Traefik, Envoy).


Em nuvem, existem também balanceadores totalmente gerenciados, como:

- AWS ELB/ALB,
- Google Cloud Load Balancing,
- Azure Load Balancer.

Cada abordagem tem seu lugar, dependendo do orçamento, da escala e do nível de controle necessário.

---

## Segurança: o lado crítico do Load Balancer

Como o load balancer fica **exatamente na linha de frente**, ele se torna automaticamente um dos pontos mais sensíveis da infraestrutura.  
Um invasor que compromete o balanceador compromete **toda a aplicação**.

Por isso, é indispensável tratá-lo como um elemento crítico de segurança. Vamos aos principais riscos:

---

### 1. Ponto único de ataque

Se o load balancer cair, toda a aplicação pode ficar inacessível.  
Por isso, usa-se normalmente pares de balanceadores em **alta disponibilidade (HA)**.

Um substituto entra automaticamente se o principal falhar.

---

### 2. Alvo de ataques DDoS

Ataques de negação de serviço muitas vezes miram diretamente o balanceador.  
Se ele ficar sobrecarregado:

- não consegue encaminhar conexões,
    
- começa a rejeitar requisições legítimas,
    
- a aplicação inteira parece estar offline.
    

Mitigação:

- rate limiting,
    
- firewalls de aplicação (WAF),
    
- proteção anti-DDoS da nuvem,
    
- isolamento de planos (controle vs dados).
    

---

### 3. Vulnerabilidades de TLS/SSL

Quando o balanceador faz _TLS termination_ (descriptografando tráfego HTTPS para inspeção):

- ele se torna responsável por manter chaves seguras,
    
- deve estar atualizado contra CVEs,
    
- deve seguir práticas modernas de criptografia.
    

Uma configuração fraca pode permitir **ataques man-in-the-middle** ou vazamento de sessões.

---

### 4. Exposição de APIs internas

Balanceadores modernos possuem APIs (REST, gRPC) para automação.  
Se mal protegidas, essas APIs se tornam portas traseiras valiosas para invasores.

---

### 5. Configurações incorretas

Um balanceador é poderoso — mas isso significa que qualquer erro de configuração pode gerar:

- vazamento de dados entre sessões,
    
- redirecionamento incorreto de tráfego,
    
- falhas no health check que derrubam servidores saudáveis,
    
- bypass de políticas de segurança.
    

Boa parte das falhas em produção não é causada por ataque, e sim por **erro humano no balanceador**.

---

### 6. Inspeção de tráfego criptografado

Se o tráfego HTTPS não é inspecionado, malwares ou payloads maliciosos podem passar “dentro” da criptografia.  
Se ele é inspecionado, abre-se a necessidade de **TLS termination**, que traz novos riscos e responsabilidades.

É sempre um equilíbrio delicado.

---

## Load Balancer e Sessões Persistentes

Lembra das **sessões persistentes**?  
Pois é — o load balancer precisa lidar com elas também.

Imagine uma aplicação onde o usuário precisa continuar sendo atendido pelo mesmo servidor após o login.  
Para isso, usamos:

- **Sticky Sessions** (sessões fixas),
    
- cookies de afinidade,
    
- _IP Hash_,
    
- caches de sessão distribuída.
    

Configurar isso errado pode:

- quebrar logins,
    
- causar inconsistências,
    
- gerar perdas de sessão,
    
- expor tokens se mal tratados.
    

---

## Load Balancer como peça de defesa

Curiosamente, ele não é só um alvo — também pode ser uma **camada de defesa**.

Um balanceador bem configurado pode:

- bloquear tráfego suspeito,
    
- filtrar IPs maliciosos,
    
- limitar taxas de requisições,
    
- inserir cabeçalhos de segurança,
    
- proteger diretamente contra ataques de camada 7 (HTTP floods).
    

É um ponto estratégico no desenho de redes seguras.
