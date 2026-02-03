![[capa_arquitetura_urbana.jpeg]]

# O que é Arquitetura de Rede?

A palavra _arquitetura_, segundo o dicionário Cambridge Dictionary, significa:

> _“a arte e a prática de projetar e construir edifícios...”_

Podemos, ainda, compreender arquitetura como:

> _“Maneira pela qual são dispostas as partes ou elementos de um edifício ou de uma cidade.”_


Trazendo essa ideia para o mundo das redes de computadores, podemos pensar em _arquitetura_ como a forma pela qual os dispositivos estão dispostos fisicamente e como os dados são transmitidos entre eles. Essa estrutura compreende um conjunto de regras, protocolos, camadas e padrões que controlam a comunicação.

## Ok, mas por que a arquitetura é algo importante?

O envio de dados em uma rede é um processo complexo. Envolve diferentes tecnologias de hardware e software que precisam funcionar de forma coordenada — e ainda considerando as fronteiras geográficas e políticas.

Ao estabelecer regras, protocolos e padrões, a arquitetura de rede organiza essa complexidade, tornando a comunicação funcional e garantindo sua **eficiência, segurança, escalabilidade e confiabilidade**.

Mas como exatamente isso acontece? Vamos por partes.

## Modelos de comunicação

Primeiro, vamos entender os **modelos de comunicação** — mais especificamente, dois deles.

> Definição de modelo 
>
> Um modelo é algo que serve de referência, um padrão a ser seguido. Assim, _modelos de comunicação_ representam como a comunicação na rede deve ocorrer. Em outras palavras, a rede deve “imitar” o modelo.

Existem dois principais modelos para descrever as funções que precisam ocorrer para que uma comunicação seja bem-sucedida: o **modelo OSI** e o **modelo TCP/IP**.

O modelo _Open Systems Interconnection_ (OSI) é **conceitual**. Ele não é usado nas redes reais, mas serve como um guia para entender as funções na comunicação e seus respectivos processos. Esse modelo divide a comunicação em **sete camadas**, cada uma responsável por uma função específica na comunicação.

![[modelo_osi.jpg]]


O modelo _TCP/IP_, por sua vez, é **prático** — este sim é o que usamos na internet. Ele organiza a comunicação em **quatro camadas** (em algumas abordagens, cinco), que correspondem aproximadamente às funções do modelo OSI.

![[modelo_tcp.jpg]]

Cada camada, seja no OSI ou no TCP/IP, tem um papel específico na comunicação. E conhecer essas camadas é fundamental para a segurança: cada uma possui suas próprias vulnerabilidades e tipos de ataque, assim como suas técnicas de defesa.

> Por exemplo, ataques de _Denial of Service_ (DoS) podem afetar tanto a **Camada de Aplicação** quanto a **Camada de Transporte**, enquanto a **Camada Física** é vulnerável se o meio de transmissão — como cabos de cobre ou fibras ópticas — for comprometido.

## Projetos de rede

Além dos modelos de comunicação, o **design da rede** é essencial para garantir eficiência, segurança, escalabilidade e confiabilidade.

Ao projetar uma rede, devemos buscar quatro características básicas:

- **Tolerância a falhas**;
- **Escalabilidade**;
- **Qualidade de serviço (QoS)**;
- **Segurança**.

Por exemplo, imagine uma empresa com o seguinte projeto de rede:

![[bad_enterprise_project.png]]

Nesse cenário, há um roteador, dois switches e alguns PCs. Mas o que acontece se esse **único roteador** falhar? Ou se um dos **switches** parar de funcionar? Muitos dispositivos, ou todos, não conseguirão se conectar à internet!

Um projeto assim é completamente inviável, pois não garante disponibilidade contínua. É aqui que entram os **projetos de rede** bem elaborados. 

Eles buscam, por exemplo, eliminar _pontos únicos de falha_ — *single point of failure (SPOF)*, situações em que a falha de um único dispositivo compromete toda a comunicação.

## E o que mais?

Arquitetura de rede é um tema vasto. Além dos modelos de comunicação e do design de redes, ainda podemos falar sobre:

- Tipos de arquitetura existentes;
- Topologias físicas e lógicas;
- Meios de transmissão — cobre, fibra, radiofrequência;
- Dispositivos de rede, como roteadores, switches, firewalls, IPS, IDS e muitos outros.
