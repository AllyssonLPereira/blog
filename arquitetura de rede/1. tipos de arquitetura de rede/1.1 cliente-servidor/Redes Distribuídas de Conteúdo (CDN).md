---
tags:
  - node
---
## O que é uma Rede Distribuída de Conteúdo?

Imagine alguém do estado do Rio Grande do Sul solicitando um site que está armazenado nos servidores da Espanha. Por mais que a velocidade de entrega hoje seja bem rápida, se a página tiver 100 imagens, a latência se acumula, tornando o site lento.

E é justamente esse problema que as CDNs resolvem, a **latência causada pela distância física**.

A CDN espalha milhares de servidores (chamados de **Edge Servers** ou Servidores de Borda) em centenas de locais ao redor do mundo (chamados de **PoPs** - Points of Presence).

![[point_of_presence_azure.webp]]
fonte: `https://azure.microsoft.com/en-us/blog/new-locations-for-azure-cdn-now-available/`

Com isso, as CDNs trazem consigo uma série de benefícios, como: redução do tempo de carregamento de uma página, redução dos custos de largura de banda, aumento da disponibilidade do conteúdo, além de melhorar a segurança do site.


## Princípios

### Armazenamento em cache

Armazenamento em cache é o processo de armazenar cópias do mesmo dado para o acesso mais rápido ao dado. Por exemplo:

1. Um usuário solicita uma informação que não está presente no CDN POP geograficamente mais próximo do usuário.

2. A requisição, então, vai para o servidor de origem. O servidor de origem envia o que o usuário solicitou, mas, ao mesmo tempo, ele envia uma cópia da resposta para o CDN POP geograficamente mais próximo do usuário.

3. O servidor CDN POP armazena a cópia como um arquivo de cache.

4. Assim, quando esse usuário ou qualquer outro for solicitar essa mesma informação, ele será respondido pelo servidor CDN POP próximo, e não pelo servidor de origem.

### Aceleração dinâmica (DSA - Dynamic Site Acceleration)

Os servidores CDN fazem cache de conteúdos estáticos (imagens, CSS, JavaScript, fontes, vídeos). Eles praticamente "moram" nos Edge Servers.

Já para conteúdo dinâmico (carrinho de compras, feed personalizado e muito mais), eles não fazem cache, pois esse tipo de conteúdo muda toda hora. Se as CDNs fizessem o cache dessas informações, elas teriam que reconectar com o servidor de origem a cada requisição, o que é custoso.

Em vez disso, a rede privada da CDN é usada para encurtar o caminho até o servidor de origem. Poderia ser usada a Internet até o servidor de origem, mas ela é mais lente e instável que a rede privada da CDN.

Com isso, otimiza-se a conexão entre os solicitantes e o servidor de origem.

### Lógica computacional de borda

É possível fazer com que os servidores de borda executam parte da lógica da aplicação, como: a autenticação do usuário, validar e lidar com as requisições incorretas do usuário... sem que a requisição precise ir até o servidor principal.

Isso ajuda os desenvolvedores a aliviar a carga nos requisitos computacionais dos servidores de origem e melhorar a performance do site.

## Segurança nas CDNs