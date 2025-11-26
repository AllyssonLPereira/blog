---
tags:
  - node
---
Quando falamos do modelo **Cliente-Servidor**, não tem como não abordar dois conceitos que influenciam diretamente o desempenho e a segurança das comunicações: **sessões persistentes** e **conexões simultâneas**.  

---
### Sessões Persistentes

As **sessões persistentes** representam a capacidade de **manter uma conexão ativa entre cliente e servidor** ao longo de várias requisições. Em vez de abrir e fechar uma nova conexão para cada solicitação, a mesma conexão é reaproveitada durante toda a interação.

Então, uma sessão persistente é uma comunicação que continua viva ao longo de várias requisições.

Isso acontece, por exemplo, com o recurso **Keep-Alive** do HTTP/1.1, ou com **WebSockets**, que mantêm um canal aberto entre o navegador e o servidor.

Nisso, temos algumas vantagens, como:

- **Menor latência:** elimina o tempo gasto em _handshake_ e renegociação TCP a cada requisição.

- **Redução de carga:** diminui a sobrecarga de criação e encerramento constante de conexões.

- **Manutenção de estado:** permite associar um cliente a uma sessão de autenticação ou contexto de uso.

- **Melhor experiência do usuário:** as respostas chegam mais rápido, pois a “ponte” já está construída.


Mas como tudo em segurança, o que facilita o uso também pode abrir brechas.

##### Questões de segurança em sessões persistentes

Sessões persistentes precisam ser **protegidas com extremo cuidado**, pois representam **um canal ativo e autenticado**.  

Alguns riscos comuns incluem:

- **Sequestro de sessão (Session Hijacking):**  

    Ocorre quando um atacante obtém o identificador de sessão (por exemplo, um _cookie de autenticação_), permitindo que ele se passe pelo usuário legítimo.  

    Isso pode acontecer via ataques como _XSS (Cross-Site Scripting)_, _Sniffing_ em conexões não criptografadas ou até vazamento de logs.

- **Fixação de sessão (Session Fixation):**  

    O atacante força a vítima a usar um ID de sessão conhecido, facilitando o roubo posterior dessa sessão.

- **Sessões sem expiração adequada:**  

    Manter sessões abertas indefinidamente é um convite a abusos. Uma boa prática é definir _timeouts_ curtos para inatividade e exigir renovação periódica do token.

- **Sessões não criptografadas:**  

    Todo o tráfego de sessão deve ocorrer sob **HTTPS (TLS)** — sem exceção. Sessões em texto claro são facilmente interceptadas.


Por isso, servidores modernos utilizam estratégias como:

- **Tokens temporários (JWT, OAuth2)**;
- **Rotação de chaves de sessão**;
- **Flags de segurança em cookies (HttpOnly, Secure, SameSite)**;
- **Limpeza automática de sessões inativas**.

Em resumo: uma sessão persistente é um atalho útil, mas precisa ser **curto e protegido** — como uma ponte levadiça que se fecha rápido após o uso.

---
### Conexões Simultâneas

Já as **conexões simultâneas** tratam da **quantidade de conexões que o servidor pode manter abertas ao mesmo tempo** — seja com diferentes clientes ou com o mesmo cliente enviando várias requisições paralelas (como quando você abre várias abas do navegador).

Em um servidor web, por exemplo, cada conexão ativa consome memória, CPU e _threads_.  
Gerenciar isso corretamente é vital para garantir que todos os usuários recebam respostas rápidas e que o sistema não entre em colapso.

**Principais desafios e implicações:**

- **Escalabilidade:** o servidor precisa lidar com centenas, milhares ou até milhões de conexões ativas.

- **Limitações físicas:** cada conexão usa recursos do sistema operacional, como descritores de arquivo e _sockets_.

- **Gerenciamento de fila:** quando o limite é atingido, novas conexões podem ser rejeitadas ou enfileiradas.


Ferramentas como **Apache**, **Nginx** e **IIS** possuem parâmetros específicos (como `MaxClients`, `worker_connections` ou `ThreadPoolSize`) que controlam esse limite e evitam sobrecarga.

##### Segurança e ataques relacionados

O gerenciamento inadequado de conexões simultâneas é terreno fértil para **ataques de negação de serviço (DoS e DDoS)**.

- **Ataques de exaustão de conexões:**  

    O invasor abre milhares de conexões TCP e as mantém sem enviar dados, consumindo todos os recursos disponíveis do servidor. 

    Exemplo clássico: **Slowloris Attack** — um ataque simples, porém eficaz, que mantém conexões HTTP abertas por longos períodos.

- **Exploração de timeouts mal configurados:**  

    Se o servidor demora muito para encerrar conexões inativas, ele pode atingir seu limite máximo, negando acesso a usuários legítimos.

- **Conexões simultâneas com autenticação fraca:**  

    Cada conexão autenticada representa um ponto de entrada. Se o controle de sessão for mal implementado, um atacante pode explorar conexões paralelas para realizar ações simultâneas (_session spamming_ ou _race conditions_).


Boas práticas incluem:

- **Definir limites máximos realistas de conexões por cliente/IP**;
- **Usar balanceadores de carga e proxies reversos**;
- **Aplicar _timeouts_ curtos para conexões ociosas**;
- **Implementar firewalls de aplicação (WAF) e rate limiting**;
- **Monitorar padrões de tráfego para detectar anomalias**.

---
### Sessões x Conexões: a relação entre desempenho e segurança

Esses dois conceitos estão intimamente ligados: uma sessão pode existir dentro de uma conexão persistente, e múltiplas sessões podem coexistir em conexões simultâneas. O desafio é equilibrar **eficiência e segurança**.

Sessões persistentes melhoram o desempenho, mas se mal protegidas tornam-se vulneráveis.  
Conexões simultâneas aumentam a capacidade do servidor, mas se não forem limitadas corretamente, abrem brechas para ataques de exaustão.

A arquitetura cliente-servidor moderna busca exatamente esse equilíbrio, usando técnicas como:

- **Pooling de conexões** (reutilizar conexões sem deixá-las abertas indefinidamente);
- **Gerenciamento inteligente de sessão com tokens curtos e rotativos**;
- **Monitoramento contínuo de conexões suspeitas**;
- **Segurança em camadas**, combinando autenticação forte, TLS e WAF.