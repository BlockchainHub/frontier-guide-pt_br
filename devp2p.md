<!--include "git+https://github.com/ethereum/wiki.wiki.git/ÐΞVp2p-Wire-Protocol.md" -->

Comunicações *peer-to-peer* entre nós rodando *clients* Ethereum/Whisper/&c. são feitas para serem governadas por um simples *wire-protocol* fazendo uso de tecnologias ÐΞV existentes e padrões como  [RLP](https://github.com/ethereum/wiki/wiki/RLP) sempre que for prático.

Este documento tem a intenção de especificar esse protocolo compreensivelmente.

### Baixo nível

Nós ÐΞVp2p se comunicam enviando mensagens usando RLPx, um protocolo de transporte encriptado e autenticado. *Peers* são livres para enviar e aceitar conexões em qualquer porta TCP que desejarem, no entanto, uma porta *default* na qual a conexão pode ser feita e escutada será a 30303.
Embora TCP forneça um meio orientado por conexão, nós ÐΞVp2p se comunicam em termos de pacotes.
RLPx provê facilidades para enviar e receber pacotes. Para mais informação sobre RLPx, se refira a [especificação do protocolo](https://github.com/ethereum/devp2p/tree/master/rlpx.md). 

Nós ÐΞVp2p encontram *peers* através do protocolo de descobrimento do RLPx, DHT. Conexões entre *peers* também podem ser iniciadas fornecendo o *endpoint* de um *peer* a uma API RPC específica do *client*.

### *Payload* de conteúdo

Há um número de diferentes tipos de *payloads* que podem ser codificados dentro do RLP. Esse ''tipo'' é sempre determinado pela primeira entrada do RLP, interpretado como um inteiro.

ÐΞVp2p é feito para dar suporte a sub-protocolos arbitrários (ou seja, _capabilities_) sobre o protocolo *wire* básico. Cada sub-protocolo é dado tanto espaço da *message-ID* quanto precisa (todos esse protocolos devem especificar estaticamente quantas IDs de mensagens eles necessitam). Em conexão e recepção da mensagem `Hello`, ambos os *peers* têm informação equivalente sobre quais sub-protocolos eles compartilham (incluindo versões) e são capazes de formar consenso sobre a composição do espaço de ID de mensagem.

IDs de mensagens são assumidas como compactas da 0x10 para frente (0x00-0x10 é reservada para mensagens ÐΞVp2p) e dadas para cada sub-protocolo compartilhado (versão igual, nome igual) em ordem alfabética. Sub-protocolos que não são compartilhados são ignorados. Se múltiplas versões são compartilhadas do mesmo (nome igual) sub-protocolo, a versão numericamente maior ganha, outras são ignoradas.

### P2P

**Hello**
`0x00` [`p2pVersion`: `P`, `clientId`: `B`, [[`cap1`: `B_3`, `capVersion1`: `P`], [`cap2`: `B_3`, `capVersion2`: `P`], `...`], `listenPort`: `P`, `nodeId`: `B_64`] Primeiro pacote enviado pela conexão e enviado por ambos os lados. Nenhuma outra mensagem pode ser mandada até que um "Hello" seja recebido.
* `p2pVersion` Especifica a versão implementada do protocolo P2P. Agora deve ser 1.
* `clientId` Especifica a identidade do software do *client*, como uma *string* legível para humanos (por exemplo, "Ethereum(++)/1.0.0").
* `cap` Especifica o nome de uma capacidade do *peer* como uma *string* ASCII de comprimento 3. As capacidades atualmente suportadas são `eth`, `shh`.
* `capVersion` Especifica a versão de uma capacidade de um *peer* como um inteiro positivo. As versões suportadas atualmente são 34 para `eth`, e 1 para `shh`.
* `listenPort` especifica a porta que o *client* está escutando (na interface onde a presente conexão atravessa). Se for 0, indica que o *client* não está escutando.
* `nodeId` é uma Identidade Única do nó e especifica um *hash* de 512-bit que identifica este nó.

**Disconnect**
`0x01` [`reason`: `P`] Informa o *peer* que uma desconexão é iminente; se recebido, um *peer* deve desconectar imediatamente. Quando enviando, *hosts* de bom comportamento dão a seus *peers* uma chance de lutar (leia: espere 2 segundos) para desconectar antes de se desconectarem. 
* `reason` é um inteiro opcional especificando uma de um número de razões para desconectar:
  * `0x00` Desconexão requerida;
  * `0x01` Erro do sub-sistema TCP;
  * `0x02` Quebra de protocolo, ex. uma mensagem mal-formada, RLP ruim, número mágico incorreto, etc.;
  * `0x03` *Peer* inútil;
  * `0x04` *Peers* em excesso;
  * `0x05` Já conectado;
  * `0x06` Versão incompatível do protocolo P2P;
  * `0x07` Identidade nula de nó recebida - isto é automaticamente inválido;
  * `0x08` *Client* fechando;
  * `0x09` Identidade inesperada (isto é, uma identidade diferente de uma conexão anterior/o que uma *peer* confiado nos disse).
  * `0x0a` Identidade é mesma desse nó (isto é, conectado a si mesmo);
  * `0x0b` Tempo esgotado para receber uma mensagem (isto é, nada recbido desde o último *ping* enviado);
  * `0x10` Alguma outra razão específica para um sub-protocolo.

**Ping**
`0x02` [] Solicita uma resposta imediata de `Pong` do *peer*.

**Pong**
`0x03` [] Responde ao pacote `Ping` do *peer*.

**NotImplemented (was GetPeers)**
`0x04`

**NotImplemented (was Peers)**
`0x05`

### Identidade do nó e reputação

A identidade de um nó ÐΞVp2p é uma chave pública secp256k1.

Nós são livres para guardar *ratings* para dadas IDs (quão útil o nó foi no passado) e dar preferência de acordo com elas. Nós também podem seguir IDs de nós (e a proveniência delas) para ajudar a determinar potenciais ataques de *man-in-the-middle*.
*Clients* são livres para marcar novos nós e usar a ID do nó como forma de determinar a reputação de um nó.

###Pacotes Exemplo

0x22400891000000088400000043414243

Um pacote "Hello" especificando que a id do *client* é "ABC".

Peer 1: 0x22400891000000028102

Peer 2: 0x22400891000000028103

Um *Ping* e o *Pong* retornado.

### Gerenciamento de Sessão

Ao conectar, todos os *clients* (isto é, ambos os lados da conexão) devem enviar uma mensagem de `Hello`. Ao receber essa mensagem e verificar a compatibilidade da rede e versões, uma sessão se torna ativa e quaisquer outras mensagens P2P podem ser enviadas.

A qualquer momento, uma mensagem de desconexão pode ser enviada.