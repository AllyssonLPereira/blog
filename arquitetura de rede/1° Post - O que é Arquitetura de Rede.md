---
tags:
  - root
---
## O que é Arquitetura de Rede?

A palavra _arquitetura_ vem do latim _architectūra, -ae_, que significa “arte de edificar”. Segundo o dicionário _Oxford Languages_, arquitetura é:

> _“Arte e técnica de organizar espaços e criar ambientes para abrigar os diversos tipos de atividades humanas...”_

Ou ainda:

> _“Maneira pela qual são dispostas as partes ou elementos de um edifício ou de uma cidade.”_


![[arquitetura_urbana.jpg]]
*`Fonte: https://www.weg.net/weghome/blog/arquitetura/arquitetura-e-urbanismo/`*

Trazendo essa ideia para o mundo das redes de computadores, podemos pensar em _arquitetura_ como a forma pela qual os dispositivos estão organizados fisicamente e como os dados são transmitidos entre eles. Essa estrutura define um conjunto de regras, protocolos, camadas e padrões que controlam a comunicação.

## Ok, mas por que a arquitetura é importante?

O envio de dados em uma rede é um processo complexo. Envolve diferentes tecnologias de hardware e software que precisam funcionar de forma coordenada — e ainda considerando as fronteiras geográficas e políticas.

Ao estabelecer regras, protocolos e padrões, a arquitetura de rede organiza essa complexidade, tornando a comunicação funcional e garantindo sua **eficiência, segurança, escalabilidade e confiabilidade**.

Mas como exatamente isso acontece? Vamos por partes.

#### Modelos de comunicação

Primeiro, é importante entender os **modelos de comunicação** — mais especificamente, dois deles.

> [!NOTE] Definição de modelo 
> Um modelo é algo que serve de referência, um padrão a ser seguido. Assim, _modelos de comunicação_ representam como a comunicação na rede deve ocorrer. Em outras palavras, a rede deve “imitar” o modelo.

Existem dois principais modelos de referência para descrever as funções que precisam ocorrer para que uma comunicação seja bem-sucedida: o **modelo OSI** e o **modelo TCP/IP**.

O modelo _Open Systems Interconnection_ (OSI) é **conceitual**. Ele não é usado diretamente nas redes reais, mas serve como guia para entender as funções e processos da comunicação. Esse modelo divide a comunicação em **sete camadas**, cada uma responsável por uma tarefa específica.

![[modelo_osi.jpg]]


O modelo _TCP/IP_, por sua vez, é **prático** — este sim é o que usamos na internet. Ele organiza a comunicação em **quatro camadas** (em algumas abordagens, cinco), que correspondem aproximadamente às funções do modelo OSI.

![[modelo_tcp.jpg]]

Cada camada, seja no OSI ou no TCP/IP, tem um papel específico na comunicação. E conhecer essas camadas é fundamental para a segurança: cada uma possui suas próprias vulnerabilidades e tipos de ataque.

> Por exemplo, ataques de _Denial of Service_ (DoS) podem afetar tanto a **Camada de Aplicação** quanto a **Camada de Transporte**, enquanto a **Camada Física** é vulnerável se o meio de transmissão — como cabos de cobre ou fibras ópticas — for comprometido.

#### Projetos de rede

Além dos modelos de comunicação, o **design da rede** é essencial para garantir eficiência, segurança, escalabilidade e confiabilidade.

Ao projetar uma rede, devemos buscar quatro características básicas:

- **Tolerância a falhas**;
- **Escalabilidade**;
- **Qualidade de serviço (QoS)**;
- **Segurança**.

Por exemplo, imagine uma empresa com o seguinte projeto de rede:

![[bad_enterprise_project.png]]

Nesse cenário, há um roteador, dois switches, alguns PCs e a conexão com a internet.  
Mas o que acontece se esse **único roteador** falhar? Ou se um dos **switches** parar de funcionar? Nenhum dispositivo conseguiria se conectar à internet!

Um projeto assim é completamente inviável, pois não garante disponibilidade contínua.

É aqui que entram os **projetos de rede** bem elaborados. Eles buscam eliminar _pontos únicos de falha_ — *single point of failure (SPOF)*, situações em que a falha de um único dispositivo compromete toda a comunicação.

O segredo está na **redundância**: criar caminhos alternativos e dispositivos de backup para garantir que, mesmo diante de falhas, a rede continue funcionando. Uma rede tolerante a falhas é construída com múltiplas rotas entre a origem e o destino das mensagens, permitindo uma recuperação rápida e automática.

## E o que mais?

Arquitetura de rede é um tema vasto. Além dos modelos e do design, ainda podemos falar sobre:

- Tipos de arquitetura existentes;
- Topologias físicas e lógicas;
- Meios de transmissão — cobre, fibra, radiofrequência;
- Dispositivos de rede, como roteadores, switches, firewalls, IPS, IDS e muitos outros.

Entender arquitetura de rede é compreender a espinha dorsal da comunicação digital moderna.  
Nos próximos posts, exploraremos cada um desses elementos em detalhes — para que você veja como eles tornam possível (e segura) a troca de dados que move o mundo.

---
### Links

[[1° Post - Tipos de arquitetura de redes]]
[[1° Post - Topologias lógicas]]
[[1° Post - Topologias físicas]]
