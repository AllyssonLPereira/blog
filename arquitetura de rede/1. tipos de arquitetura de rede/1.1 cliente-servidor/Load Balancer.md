---
tags:
  - node
---

Quando falamos de aplicações modernas que precisam lidar com muitos usuários ao mesmo tempo — redes sociais, serviços de streaming, jogos online, sites de e-commerce — existe um componente que atua para manter tudo funcionando com rapidez e estabilidade: o **Load Balancer**, ou balanceador de carga.

Ele é o “maestro” do modelo Cliente-Servidor. É quem decide **para qual servidor cada cliente vai**, como o tráfego será distribuído e como manter tudo funcionando mesmo quando partes da infraestrutura falham.

Sem ele, aplicações de grande escala seriam simplesmente incapazes de lidar com a quantidade de acessos simultâneos que vemos hoje.

![[load_balancer.jpg]]

---
## O que é um Load Balancer e por que ele é tão importante?


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
## Algoritmos de distribuição


O que são algoritmos de distribuição? Eles são a base que determina como o load balancer vai escolher para qual servidor encaminhar cada requisição dentro do modelo cliente-servidor. A escolha do servidor não é aleatória. Os balanceadores usam algoritmos como:

- **Round Robin**, este é o mais simples. Ele funciona como uma fila circular que distribui as requisições de forma sequencial e igualitária a cada servidor disponível. Ele é eficaz quando os servidores têm capacidades equivalentes e as requisições têm cargas semelhantes.

- Já o **Least Connections** foca no estado atual dos servidores, enviando cada nova requisição para o servidor com o menor número de conexões ativas no momento. Isso permite um balanceamento mais dinâmico e mais adaptativo, ideal quando as sessões ficam abertas por tempo variável ou as cargas não são uniformes.

- O **IP Hash** mantem a persistência da sessão, garantindo que todas as requisições vindas de um mesmo endereço IP sejam encaminhadas para o mesmo servidor.

- Os algoritmos **Weighted Round Robin** e **Weighted Least Connections** são variações do Round Robin e Least Connection que levam em conta o poder computacional ou capacidade de cada servidor para distribuir trabalho proporcionalmente. Servidores mais potentes recebem mais requisições, enquanto servidores menos robustos recebem menos.


> *Cada algoritmo atende cenários específicos, e escolher o errado pode gerar gargalos inesperados.*

---
## Tipos de balanceadores de carga


### Balanceamento de Carga de DNS (DNS Load Balancing)

O balanceamento de carga de DNS é talvez o tipo mais simples, operando no nível de resolução de nomes de domínio. 

Quando um usuário digita um URL no navegador, seu cliente DNS faz uma consulta para resolver o nome de domínio em um endereço IP. Um DNS load balancer intervém neste processo, retornando **diferentes endereços IP em respostas diferentes**, distribuindo tráfego sem qualquer dispositivo de balanceamento intermediário.​

Suponha que você tem dois servidores: `server1.example.com` (192.168.1.10) e `server2.example.com` (192.168.1.20). Um DNS LB tradicional pode ser configurado com múltiplos registros A para diferentes servidores:

``` text
example.com. 300 IN A 192.168.1.10 ; server1 
example.com. 300 IN A 192.168.1.20 ; server2
```
\**TTL 300 = 5 minutos de cache. Todos os registros têm o mesmo nome (`example.com`), mas IPs diferentes.*

Quando clientes digita `example.com` e faz a pesquisa, o servidor DNS retorna **ambos os IPs, em ordem alternada**. O cliente A recebe `[192.168.1.10, 192.168.1.20]` e conecta ao primeiro, enquanto o cliente B recebe `[192.168.1.20, 192.168.1.10]` (ordem revertida) e conecta ao também ao primeiro. Isso é o **DNS Round Robin**.​

Agora, balanceadores de carga DNS modernos (como CERN usa) são mais espertos. Eles monitoram a saúde dos servidores (health checks), a carga atual, localização geográfica dos clientes, e retornam IPs de forma dinâmica e inteligente. Então, um cliente em São Paulo é respondido com o IP do servidor mais próximo em São Paulo e, se um servidor está fora, DNS não retorna seu IP.​

### Balanceamento de Carga de Rede (Network Load Balancer - NLB)

O balanceador de carga de rede opera na **camada 4 (Transporte - TCP/UDP)**. É ultra-otimizado para velocidade, processando milhões de conexões por segundo com latência de microsegundos.​ Ou seja, trabalho bem rápido.

Aqui, um NLB recebe a requisição TCP/UDP, lê IP de origem, IP de destino, portas, e imediatamente decide para qual servidor backend encaminhar, sem olhar o conteúdo. Além disso, ele mantém tabelas de estado de conexão, aplica NAT bidirecional e rastreia conexões para persistência da sessão de comunicação.

### Balanceamento de Carga de Aplicação (Application Load Balancer - ALB)

O balanceador de carga de aplicação opera na **camada 7 (Aplicação - HTTP/HTTPS)**. É o oposto do NLB, oferecendo máxima inteligência e flexibilidade em troca de um pouco mais de latência.​

Para exemplificar, vamos considerar um ALB que recebe uma requisição HTTP completa, faz parsing (análise) de headers, URL, method, cookies, e toma decisões bem inteligentes, como: "requisições para `/api/*` vão para o pool de API; `/media/*` vão para o pool de mídia; requisições com header `Authorization: Premium` vão para servidores premium".​

### Balanceamento de Carga Global do Servidor (Global Server Load Balancing - GSLB)

O GSLB é um **meta-balanceador** que distribui tráfego entre múltiplas regiões geográficas ou data centers, frequentemente usando DNS como mecanismo principal. É o balanceamento "de balanceadores".​

E como isso funciona? Uma organização global como Netflix tem data centers em múltiplas regiões: São Paulo, Nova York, Tóquio, Frankfurt... Um cliente em São Paulo tentando acessar "netflix.com" faz uma requisição DNS, nisso, a GSLB intercepta essa requisição, verifica a localização do cliente (geolocation), verifica a saúde dos data centers, e retorna o IP do data center mais próximo ou mais saudável.

Se o data center de São Paulo está fora do ar, o GSLB retorna o IP do data center da região mais próxima (por exemplo, Miami).​