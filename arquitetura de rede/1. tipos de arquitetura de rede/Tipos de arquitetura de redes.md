---
tags:
  - root
---
No post "1° Post - O que é Arquitetura de Rede", foi feita uma introdução à arquitetura de redes. Nesse post, serão abordados os tipos de arquitetura de redes.

Primeiro, vamos relembrar o conceito de arquitetura de redes. 

Podemos pensar em _arquitetura_ como a forma pela qual os dispositivos estão organizados fisicamente e como os dados são transmitidos entre eles. Essa estrutura define um conjunto de regras, protocolos, camadas e padrões que controlam a comunicação.

Vejamos os tipos de arquitetura de redes:

---
## Cliente/Servidor


A comunicação acontece quando duas ou mais pessoas estão juntas e uma delas solicita algo ou comenta algo e a outra pessoa responde.

O modelo cliente/servidor, basicamente, vai dividir os papéis de solicitação e resposta. Aquele que irá pedir recursos ou serviços, é chamado de *cliente* (*client*), enquanto aquele que irá fornecer recursos ou serviços, é chamado de *servidor* (*server*).

> *Então, o cliente inicia a comunicação com o servidor solicitando algum recurso ou serviço, o servidor, então, fornece aquilo que foi solicitado pelo cliente.*

![[cliente-servidor.png]]

E quais são as vantagens e desvantagens que esse modelo traz? Vejamos:

- **Vantagens**:

	- *Um controle de acesso e segurança centralizada, num só lugar*. Aqui, todos os dados ficam nos servidores, permitindo controles de segurança superiores e garantindo que apenas clientes com credenciais válidas possam acessar e alterar dados.

	- Além disso, a *centralização do armazenamento* de dados facilita a administração e as atualizações, o que é mais simples do que em modelos descentralizados como P2P.


- **Desvantagens**:

	- Clientes solicitam serviços, mas não os oferecem a outros clientes, o que pode sobrecarregar o servidor e gerando uma falha conhecida como *Ponto Único de Falha - Single Point of Failure*.

		- Embora a falha de um servidor crítico comprometa a disponibilidade do serviço, essa desvantagem é ativamente combatida. Como? Através de *componentes redundantes*, *múltiplas conexões* e uso de estratégias como _clustering_ (agrupamento de vários dispositivos para parecerem um único dispositivo, garantindo que se um falhar, os outros continuam operando).


### Arquitetura Multicamadas

O modelo cliente/servidor é frequentemente caracterizado como uma _arquitetura multicamadas (multilayer architecture_), e por que isso?

Acontece que, há muito tempo atrás, numa galáxia muito, muito distante — entre os anos de 1950 e 1970, a maioria das aplicações era desenvolvida como um único sistema, um *sistema monolítico*, onde a interface de usuário, a lógica de negócio e o armazenamento de dados era executada em um único computador central, o *mainframe*. O mainframe tinha a responsabilidade de realizar todas as tarefas e processamento.

![[eniac.mp4]]

Depois disso, com a popularização dos computadores pessoais e a expansão das redes de computadores, os métodos de desenvolvimento de software foram aos poucos evoluindo para uma *arquitetura mais descentralizada*, surgindo a arquitetura de duas camadas, ou *cliente/servidor*.

Na arquitetura de duas camadas, o cliente lida tanto com a lógica de negócios quanto com a interface do usuário (UI), enquanto o servidor trata apenas do banco de dados.

Só que, a arquitetura de duas camadas ainda era difícil de gerenciar em larga escala. Mudanças na lógica de negócio exigiam atualizações no lado do cliente, e a sobrecarga de acessos diretos ao banco de dados afetava o desempenho.

Foi então que, para superar as limitações do modelo de duas camadas, foi desenvolvida a *arquitetura de três camadas*. 

A introdução de uma camada intermediária trouxe uma melhor *separação de responsabilidades*, onde a lógica de negócios é centralizada em um servidor de aplicações, separando a lógica da UI (cliente) e do armazenamento de dados (servidor de banco de dados).

> [!NOTE] Quatro Camadas
> Temos, ainda, a arquitetura de quatro camadas, com a camada de apresentação (UI) centralizada em um servidor (geralmente um _web server_), e o acesso à aplicação é feito via navegador.

---
## Peer to Peer


No modelo cliente/servidor, temos o cliente como solicitante de informações, serviços ou recursos e o servidor como aquele que irá fornecer tudo isso. Então, basicamente, temos o cliente e o servidor com tarefas que são propriamente suas, cada um realizando algo distinto do outro.

Já no modelo Peer to Peer (P2P), os dispositivos farão essas duas tarefas, na qual eles tanto solicitam algo, como fornecem algo.

![[p2p.png]]

E quais são as vantagens e desvantagens que esse modelo traz? Vejamos:

- **Vantagens**:

	- *Disponibilidade/Tolerância a falhas*. Como não há um servidor central, a rede P2P continua funcionando mesmo que dispositivos específicos falhem.

	- *Maior escalabilidade*, pois a capacidade da rede não é limitada por um servidor central, escalando recursos à medida que mais usuários (peers) se conectam e contribuem com seu poder computacional e armazenamento.

- **Desvantagens**:

	- *Segurança reduzida*, isso porque, com a ausência de um controle de segurança centralizado, a disseminação de ameaças (como _malware_ ou vírus) fica mais fácil.

	- A *consistência de dados* aqui também é um desafio devido o uso de P2P para dados mutáveis, que mudam frequentemente, ser difícil. A falta de um controle central dos dados pode levar à corrupção ou conflito desses dados se houver modificações e versões distintas. O P2P é mais conveniente para dados _imutáveis_.

	- Além disso, a *sobrecarga de tráfego* é outra desvantagem. O processo de busca por uma informação pode causar grande tráfego na rede (_flooding_), tornando a busca lenta.


### Tipos de P2P

- *`P2P Centralizada`:* Utiliza um servidor central apenas para indexar informações e iniciar o sistema, mas as conexões diretas entre os _peers_ não são gerenciadas pelo algoritmo (ex: Napster original).

- *`P2P Pura/Não Estruturada`:* Toda a rede consiste em pares que se conectam diretamente. Não há servidor central. A localização de itens geralmente envolve inundar a rede com uma busca (ex: Gnutella).

- *`P2P Estruturada (DHT)`:* Utiliza um procedimento determinístico, como uma *Tabela Hash Distribuída (DHT - Distributed hash table)*, para mapear a localização de chaves e dados de forma eficiente através da rede sobreposta (_overlay network_).

---
## Arquitetura híbrida


Além das arquiteturas _Cliente/Servidor_ e _P2P_, existe um modelo que combina elementos dos dois: a **Arquitetura Híbrida**. Ela busca unir o melhor de cada abordagem — a **organização** do modelo cliente/servidor com a **flexibilidade** do ponto a ponto.

Nela, partes da rede seguem o modelo cliente/servidor — por exemplo, servidores dedicados para autenticação, controle de acesso ou armazenamento centralizado — enquanto outras partes permitem conexões ponto a ponto diretas entre os dispositivos.

> Um bom exemplo disso são as **aplicações modernas de compartilhamento de arquivos** e **jogos online**: elas usam servidores centrais para autenticação, gerenciamento de usuários e atualização de estado, mas permitem que os dados (como pacotes de jogo ou arquivos) sejam trocados diretamente entre os clientes, de forma P2P. Assim, combinam **eficiência, escalabilidade e resiliência**.

Essa arquitetura híbrida também aparece em **ambientes corporativos**. Imagine uma empresa em que os funcionários usam um servidor principal para acessar sistemas internos, mas trocam arquivos entre si diretamente, quando estão na mesma rede local. O resultado é um sistema mais **dinâmico**, com **melhor aproveitamento de recursos** e **redução de carga no servidor principal**.

Em resumo, a **Arquitetura Híbrida** aproveita:

- o **controle e a segurança** do modelo cliente/servidor;
- a **autonomia e a eficiência distribuída** do modelo ponto a ponto.

É uma solução inteligente para redes que precisam ser **escaláveis, tolerantes a falhas e versáteis** — características fundamentais nas redes modernas e em ambientes em nuvem.

---
## Cloud


A **arquitetura em nuvem**, ou simplesmente _Cloud_, é o modelo que revolucionou a forma como as redes e os serviços digitais funcionam.  

Se antes tudo precisava estar fisicamente dentro de uma empresa — servidores, cabos, armazenamento, segurança —, agora boa parte dessa estrutura pode estar em **ambientes virtuais**, acessíveis pela internet.

Em outras palavras, a _cloud computing_ permite que recursos de computação — como servidores, bancos de dados, redes e aplicativos — sejam disponibilizados **sob demanda**, sem que seja necessário ter toda a infraestrutura localmente.  

O usuário consome recursos conforme precisa, e quem cuida da parte física é o **provedor de nuvem**, como AWS, Microsoft Azure, Google Cloud, entre outros.

![[toy_story_cloud.png]]
##### Como a cloud funciona?

A ideia central é dividir os recursos em **camadas e serviços**, de modo que o usuário possa escolher o nível de controle e responsabilidade que deseja. Vamos exemplificar!

De forma geral, a arquitetura cloud é composta por três camadas principais:

1. **Infraestrutura como Serviço (IaaS — Infrastructure as a Service):**  

    O provedor oferece recursos básicos, como máquinas virtuais, redes, armazenamento e servidores. Aqui, o usuário tem liberdade para configurar o sistema operacional e os aplicativos, mas não precisa se preocupar com o hardware físico.

2. **Plataforma como Serviço (PaaS — Platform as a Service):**  

    Já no PaaS, o foco é o ambiente de desenvolvimento. O provedor cuida da infraestrutura e oferece ferramentas para criar, testar e implantar aplicações. É como ter um “laboratório pronto” para desenvolvimento, sem se preocupar com servidores ou rede.

3. **Software como Serviço (SaaS — Software as a Service):**  

    É o modelo mais conhecido pelo usuário final. O software já está pronto e acessível via navegador, sem precisar ser instalado ou gerenciado localmente — exemplos: Gmail, Microsoft 365, Google Drive.


##### Vantagens e importância

A adoção da nuvem traz vantagens claras para empresas e usuários:

- **Escalabilidade:** recursos podem ser aumentados ou reduzidos rapidamente conforme a demanda.

- **Custo reduzido:** paga-se apenas pelo que é usado, sem a necessidade de manter grandes data centers.

- **Disponibilidade:** serviços podem ser acessados de qualquer lugar, a qualquer hora.

- **Resiliência:** os provedores oferecem redundância e backup distribuído, reduzindo falhas.


Em termos de **arquitetura de rede**, a nuvem mudou completamente a forma como projetamos sistemas. 

Hoje, parte da infraestrutura está **on-premise** (local) e parte na **cloud**, formando ambientes **híbridos** ou **multicloud**, onde diferentes provedores coexistem. Essa combinação oferece flexibilidade e alta disponibilidade, mas também exige novas estratégias de segurança, controle de acesso e gerenciamento de tráfego.

---
## Arquitetura hierárquica


A **Arquitetura Hierárquica** é um dos modelos mais utilizados em redes corporativas e em ambientes de médio a grande porte.  

Seu principal objetivo é **organizar a rede em camadas**, de forma que cada uma desempenhe funções específicas, tornando o sistema mais **eficiente, escalável e fácil de gerenciar**.

De modo geral, essa arquitetura divide a rede em **três camadas principais**:

1. **Camada de Acesso (Access Layer)**  

    É onde os dispositivos finais — como computadores, impressoras e pontos de acesso — se conectam à rede.  

    Nessa camada, ocorre o controle de quem pode ou não acessar a rede. Aqui normalmente estão os _switches de acesso_, que conectam diretamente os dispositivos dos usuários e, por ela fornecer acesso à rede, ela é o lugar ideal para executar *autenticação de usuários* e *segurança de porta*.

2. **Camada de Distribuição (Distribution Layer)**  

    Atua como uma ponte entre a camada de acesso e a camada de núcleo.  

    É responsável por **agregar o tráfego** proveniente da camada de acesso, aplicar **políticas de segurança**, **controle de VLANs**, **filtragem de pacotes** e **balanceamento de carga**.  

    É nesta camada que a rede começa a se comportar como um sistema mais inteligente, com decisões sobre o tráfego e a segmentação da rede.

3. **Camada de Núcleo (Core Layer)**  

    É o coração da rede. Sua função é garantir **alta velocidade e disponibilidade** no transporte de dados.  

    Aqui o foco é o desempenho — por isso, costuma-se evitar políticas complexas ou filtragens que possam impactar a latência.
  
    Os dispositivos dessa camada são roteadores e switches de alta capacidade, projetados para lidar com grandes volumes de tráfego e prover redundância. Um exemplo é o *Cisco Catalyst 9600*.

Ai está um exemplo dessa arquitetura

![[cisco_three-tier_network_design_model.png]]

### Por que usar uma arquitetura hierárquica?

Em redes corporativas, dividir a estrutura em camadas traz benefícios claros:

- **Escalabilidade:** permite expandir a rede facilmente, adicionando novos dispositivos sem comprometer o desempenho.

- **Resiliência:** facilita a implementação de redundância em pontos críticos, reduzindo o impacto de falhas.

- **Gerenciabilidade:** a segmentação por funções torna a manutenção e o monitoramento mais simples e previsíveis.

- **Segurança:** políticas e controles podem ser aplicados em pontos estratégicos — por exemplo, autenticação no acesso e filtragem na distribuição.


> [!NOTE] Pra saber mais
> Segue um vídeo do youtube do *"NetworkChuck"*, na qual ele explica essa arquitetura de uma maneira excelente: https://www.youtube.com/watch?v=wwwAXlE4OtU&list=PLIhvC56v63IJVXv0GJcl9vO5Z6znCVb1P&index=7

---
## Arquitetura de Rede Definida por Software SDN


Com o crescimento das redes corporativas e dos serviços em nuvem, os administradores passaram a enfrentar um desafio cada vez maior: **gerenciar redes complexas de forma ágil, segura e escalável**. Foi dessa necessidade que surgiu a **SDN — Software Defined Networking**, ou **Rede Definida por Software**.

A SDN representa uma mudança profunda na forma como as redes são projetadas, configuradas e administradas. Seu princípio básico é **separar o plano de controle do plano de dados**, permitindo que a inteligência da rede seja centralizada e controlada por software.

### Entendendo os planos

Tradicionalmente, em uma rede convencional, cada dispositivo — como roteadores e switches — tomavam suas próprias decisões de encaminhamento. Ou seja, o **plano de controle** (as regras que determinam para onde os pacotes vão) e o **plano de dados** (o envio real desses pacotes) estão integrados no mesmo equipamento.

Na SDN, o jogo mudou:

- O **Plano de Controle** é **centralizado** em um software controlador — o _SDN Controller_.

- O **Plano de Dados** continua nos dispositivos de rede, mas agora eles obedecem às instruções enviadas pelo controlador.


Esse controlador atua como o **“cérebro” da rede**, tomando decisões globais e programando dinamicamente os dispositivos. Em vez de configurar cada roteador ou switch manualmente, o administrador configura o controlador, que aplica as políticas de forma automática e coerente em toda a infraestrutura.

### Principais componentes

1. **Plano de Dados (Data Plane):**  

    Aqui estão os switches e roteadores que apenas seguem as regras definidas pelo controlador, sendo responsáveis pelo encaminhamento dos pacotes. Só obedecem, só seguem as regras do controlador.

2. **Plano de Controle (Control Plane):**  

    Onde fica o **controlador SDN**, que decide o caminho que os dados devem seguir e envia essas instruções aos dispositivos da rede.

3. **Plano de Aplicação (Application Plane):**  

    Camada onde residem as aplicações que utilizam a rede — como sistemas de monitoramento, políticas de segurança e controle de QoS.  

    Essas aplicações se comunicam com o controlador por meio de **APIs** (_Application Programming Interfaces_), permitindo automação e integração com outros sistemas.

### Vantagens da SDN

- **Centralização da gestão:**  

    Todas as decisões de controle são tomadas a partir de um ponto central, simplificando a configuração e reduzindo erros humanos.

- **Automação e agilidade:**  

    Mudanças na rede podem ser aplicadas via software em questão de segundos — algo que, em redes tradicionais, exigiria reconfiguração manual de vários dispositivos.

- **Escalabilidade e flexibilidade:**  

    É possível criar, modificar ou remover políticas de tráfego de acordo com a demanda, sem interromper a operação da rede.

- **Melhor visibilidade e segurança:**  

    O controlador tem uma visão completa da topologia e do tráfego, o que facilita a detecção de anomalias e o isolamento de ameaças.

---
## Links 

[[0_link_p2p|0_link_p2p]]
[[0_link_cliente-servidor|0_link_cliente-servidor]]