<!-- Source: https://raw.githubusercontent.com/wiki/ethereum/wiki/JSON-RPC.md -->

# API JSON RPC

[JSON](http://json.org/) é um formato leve de troca de dados. Ele pode representar números, *strings* sequências ordenadas de valores e coleções de pares nome/valor.

[JSON-RPC](http://www.jsonrpc.org/specification) é um protocolo leve e sem estado de chamada de procedimento remoto (*Remote Procedure Protocol*, RPC). Principalmente, essa especificação define várias estruturas de dados e as suas regras de processamento. Ele é agnóstico em termos de transporte no sentido de que os conceitos podem ser usados dentro do mesmo processo, através de sockets, por HTTP ou em outros variados ambientes de passagem de mensagens. Ele usa JSON ([RFC 4627](http://www.ietf.org/rfc/rfc4627.txt)) como formato de dado.

O Geth 1.4 tem suporte de pub/sub experimentos. Veja [esta](https://github.com/ethereum/go-ethereum/wiki/RPC-PUB-SUB) página para maiores informações.

## API de JavaScript

Para falar com um nó da ethereum de dentro de uma aplicação JavaScript use a biblioteca [web3.js](https://github.com/ethereum/web3.js), a qual dá uma interface conveniente para os métodos RPC
Veja a [API de JavaScript](https://github.com/ethereum/wiki/wiki/JavaScript-API) para saber mais.

## JSON-RPC Endpoint

*Endpoints* padrão do JSON-RPC:
```
C++: http://localhost:8545
Go: http://localhost:8545
Py: http://localhost:4000
```

### Go

Você pode iniciar o HTTP JSON-RPC com a flag `--rpc`
```bash
geth --rpc
```

Muda a porta *default* (8545) e o endereço de escuta (*localhost*) com:

```bash
geth --rpc --rpcaddr <ip> --rpcport <portnumber>
```

Se estiver acessando o RPC de um browser, o CORS precisar'a estar ativado com o domínio apropriado definido. Caso contrário, chamadas JavaScript são limitadas pela política de mesma origem e as requisições vão falhar:

```bash
geth --rpc --rpccorsdomain "http://localhost:3000"
```

O JSON RPC pode ser iniciado também do [console do geth](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) usando o comando `admin.startRPC(addr, port)`.


### C++

Você pode inciá-lo rodando a aplicação `eth`com a opção `-j`:
```bash
./eth -j
```

Você também pode especificar a porta JSON-RPC (o *default* é 8545):
```bash
./eth -j --json-rpc-port 8079
```

### Python
Em python, o servidor JSON-RPC é iniciado atualmente por *default* e escuta em `127.0.0.1:4000`

Você pode mudar a porta e o endereço de escuta dando uma opcão `config`.

`pyethapp -c jsonrpc.listen_port=4002 -c jsonrpc.listen_host=127.0.0.2 run`

## Suporte de JSON-RPC

| | cpp-ethereum | go-ethereum | py-ethereum|
|-------|:------------:|:-----------:|:-----------:|
| JSON-RPC 1.0 | &#x2713; | | |
| JSON-RPC 2.0 | &#x2713; | &#x2713; | &#x2713; |
| Batch requests | &#x2713; |  &#x2713; |  &#x2713; |
| HTTP | &#x2713; | &#x2713; | &#x2713; |
| IPC | &#x2713; | &#x2713; | |
| WS | | &#x2713; | |

## Valores de Output em HEX

Atualmente, existem dois tipos de dados chave que são passados através do JSON: *arrays* de bytes não formatados e quantidades. Ambos são passados com codificação em hex, no entanto, com diferentes requisitos para formatação:

Quando codificando **QUANTIDADES** (inteiros, números): codificar como hex, prefixo com "0x", a representação mais compacta (pequena exceção: o zero deve ser representado como "0x0). Exemplos:
- 0x41 (65 em decimal)
- 0x400 (1024 em decimal)
- ERRADO: 0x (deveria sempre ter pelo menos um dígito - zero é "0x0")
- ERRADO: 0x0400 (não é permitido iniciar com zeros)
- ERRADO: ff (precisa ter 0x como prefixo)

Quando codificando **DADOS NÃO FORMATADOS** (*arrays* de bytes, endereços de contas, *hashes*, *arrays* de *bytecode*): codificar como hex, prefixos com "0x", dois dígitos hex por byte. Exemplos:
- 0x41 (tamanho 1, "A")
- 0x004200 (tamanho 3, "\0B\0")
- 0x (tamanho 0, "")
- ERRADO: 0xf0f0f (deve ser um número par de dígitos)
- ERRADO: 004200 (precisa ter como prefixo 0x)

Atualmente, [cpp-ethereum](https://github.com/ethereum/cpp-ethereum) e [go-ethereum](https://github.com/ethereum/go-ethereum) fornecem comunicação JSON-RPC através de HTTP e IPC  (*Linux unix socket* e *OSX/named pipes* no Windows). Desde a versão 1.4 go-ethereum tem suporte de websocket.

## O parâmetro default de bloco

Os seguintes métodos têm um parâmetro *default* extra de bloco:

- [eth_getBalance](#eth_getbalance)
- [eth_getCode](#eth_getcode)
- [eth_getTransactionCount](#eth_gettransactioncount)
- [eth_getStorageAt](#eth_getstorageat)
- [eth_call](#eth_call)

Quando são feitas requisições que agem no estado da ethereum, o último parâmetro *default* de bloco determina a altura do bloco.

As seguintes opções são possíveis para o parâmetro *default *de bloco:

- `HEX String` - um número de bloco inteiro
- `String "earliest"` para o primeiro bloco/bloco gênesis
- `String "latest"` - para o último bloco minerado
- `String "pending"` - para o estado/transações pendentes

## Métodos JSON-RPC

* [web3_clientVersion](#web3_cientversion)
* [web3_sha3](#web3_sha3)
* [net_version](#net_version)
* [net_peerCount](#net_peercount)
* [net_listening](#net_listening)
* [eth_protocolVersion](#eth_protocolversion)
* [eth_syncing](#eth_syncing)
* [eth_coinbase](#eth_coinbase)
* [eth_mining](#eth_mining)
* [eth_hashrate](#eth_hashrate)
* [eth_gasPrice](#eth_gasprice)
* [eth_accounts](#eth_accounts)
* [eth_blockNumber](#eth_blocknumber)
* [eth_getBalance](#eth_getbalance)
* [eth_getStorageAt](#eth_getstorageat)
* [eth_getTransactionCount](#eth_gettransactioncount)
* [eth_getBlockTransactionCountByHash](#eth_getblocktransactioncountby*hash*)
* [eth_getBlockTransactionCountByNumber](#eth_getblocktransactioncountbynumber)
* [eth_getUncleCountByBlockHash](#eth_getunclecountbyblock*hash*)
* [eth_getUncleCountByBlockNumber](#eth_getunclecountbyblocknumber)
* [eth_getCode](#eth_getcode)
* [eth_sign](#eth_sign)
* [eth_sendTransaction](#eth_sendtransaction)
* [eth_sendRawTransaction](#eth_sendrawtransaction)
* [eth_call](#eth_call)
* [eth_estimateGas](#eth_estimategas)
* [eth_getBlockByHash](#eth_getblockby*hash*)
* [eth_getBlockByNumber](#eth_getblockbynumber)
* [eth_getTransactionByHash](#eth_gettransactionby*hash*)
* [eth_getTransactionByBlockHashAndIndex](#eth_gettransactionbyblock*hash*andindex)
* [eth_getTransactionByBlockNumberAndIndex](#eth_gettransactionbyblocknumberandindex)
* [eth_getTransactionReceipt](#eth_gettransactionreceipt)
* [eth_getUncleByBlockHashAndIndex](#eth_getunclebyblock*hash*andindex)
* [eth_getUncleByBlockNumberAndIndex](#eth_getunclebyblocknumberandindex)
* [eth_getCompilers](#eth_getcompilers)
* [eth_compileLLL](#eth_compilelll)
* [eth_compileSolidity](#eth_compilesolidity)
* [eth_compileSerpent](#eth_compileserpent)
* [eth_newFilter](#eth_newfilter)
* [eth_newBlockFilter](#eth_newblockfilter)
* [eth_newPendingTransactionFilter](#eth_newpendingtransactionfilter)
* [eth_uninstallFilter](#eth_uninstallfilter)
* [eth_getFilterChanges](#eth_getfilterchanges)
* [eth_getFilterLogs](#eth_getfilterlogs)
* [eth_getLogs](#eth_getlogs)
* [eth_getWork](#eth_getwork)
* [eth_submitWork](#eth_submitwork)
* [eth_submitHashrate](#eth_submit*hash*rate)
* [db_putString](#db_put*string*)
* [db_getString](#db_get*string*)
* [db_putHex](#db_puthex)
* [db_getHex](#db_gethex) 
* [shh_post](#shh_post)
* [shh_version](#shh_version)
* [shh_newIdentity](#shh_newidentity)
* [shh_hasIdentity](#shh_hasidentity)
* [shh_newGroup](#shh_newgroup)
* [shh_addToGroup](#shh_addtogroup)
* [shh_newFilter](#shh_newfilter)
* [shh_uninstallFilter](#shh_uninstallfilter)
* [shh_getFilterChanges](#shh_getfilterchanges)
* [shh_getMessages](#shh_getmessages)

## Referência da API JSON RPC

***

#### web3_clientVersion

Retorna a versão atual do *client*.

##### Parâmetros
nenhum

##### Retorna

`String` - A versão atual do *client*

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}'

// Resultado
{
  "id":67,
  "jsonrpc":"2.0",
  "result": "Mist/v0.9.3/darwin/go1.4.1"
}
```

***

#### web3_sha3

Retorna Keccak-256 (não o padronizado SHA3-256) dos dados fornecidos.

##### Parâmetros

1. `String` - o dado a ser convertido em um *hash* SHA3

```js
params: [
  '0x68656c6c6f20776f726c64'
]
```

##### Retorna

`DATA` - O resultado SHA3 da *string* dada.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"web3_sha3","params":["0x68656c6c6f20776f726c64"],"id":64}'

// Resultado
{
  "id":64,
  "jsonrpc": "2.0",
  "result": "0x47173285a8d7341e5e972fc677286384f802f8ef42a5ec5f03bbfa254cb01fad"
}
```

***

#### net_version

Retorna a versão atual do protocolo de rede.

##### Parâmetros
nenhum

##### Retorna

`String` - A versão atual do protocolo de rede.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":67}'

// Resultado
{
  "id":67,
  "jsonrpc": "2.0",
  "result": "59"
}
```

***

#### net_listening

Retorna `true` se o *client* está ativamente escutando por conexões de rede.

##### Parâmetros
nenhum

##### Retorna

`Boolean` - `true` quando escutando, caso contrário `false`.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"net_listening","params":[],"id":67}'

// Resultado
{
  "id":67,
  "jsonrpc":"2.0",
  "result":true
}
```

***

#### net_peerCount

Retorna o número de *peers* atualmente conectados ao *client*.

##### Parâmetros
nenhum

##### Retorna

`QUANTITY` - inteiro do número de *peers* conectados.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":74}'

// Resultado
{
  "id":74,
  "jsonrpc": "2.0",
  "result": "0x2" // 2
}
```

***

#### eth_protocolVersion

Retorna a versão atual do protocolo ethereum.

##### Parâmetros
nenhum

##### Retorna

`String` - A versão atual do protocolo ethereum.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_protocolVersion","params":[],"id":67}'

// Resultado
{
  "id":67,
  "jsonrpc": "2.0",
  "result": "54"
}
```

***

#### eth_syncing

Retorna um objeto com dados sobre o status de sincronização ou `false`.


##### Parâmetros
nenhum

##### Retorna

`Object|Boolean`, um objeto com dados sobre o status de sincronização ou `false` quando não está sincronizando:
  - `startingBlock`: `QUANTITY` - O bloco no qual começou a importação (será resetado apenas depois da sincronização alcançá-lo. ~~The block at which the import started (will only be reset, after the sync reached his head)~~
  - `currentBlock`: `QUANTITY` - O bloco atual, mesmo que eth_blockNumber
  - `highestBlock`: `QUANTITY` - O bloco mais alto estimado

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": {
    startingBlock: '0x384',
    currentBlock: '0x386',
    highestBlock: '0x454'
  }
}
// Ou, quando não está sincronizando
{
  "id":1,
  "jsonrpc": "2.0",
  "result": false
}
```

***

#### eth_coinbase

Retorna o endereço coinbase do *client*.


##### Parâmetros
nenhum

##### Retorna

`DATA`, 20 bytes - o atual endereço coinbase.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_coinbase","params":[],"id":64}'

// Resultado
{
  "id":64,
  "jsonrpc": "2.0",
  "result": "0x407d73d8a49eeb85d32cf465507dd71d507100c1"
}
```

***

#### eth_mining

Retorna `true` se o *client* está minerando ativamente novos blocos.

##### Parâmetros
nenhum

##### Retorna

`Boolean` - retorna `true` se o *client* está minerando, caso contrário, `false`.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_mining","params":[],"id":71}'

// Resultado
{
  "id":71,
  "jsonrpc": "2.0",
  "result": true
}

```

***

#### eth_hashrate

Retorna o número de *hashes* por segundo com o qual o nó está minerando.

##### Parâmetros
nenhum

##### Retorna

`QUANTITY` - número de *hashes* por segundo.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_hashrate","params":[],"id":71}'

// Resultado
{
  "id":71,
  "jsonrpc": "2.0",
  "result": "0x38a"
}

```

***

#### eth_gasPrice

Retorna o preço atual de gas em wei.

##### Parâmetros
nenhum

##### Retorna

`QUANTITY` - inteiro do preço atual de gas em wei.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":73}'

// Resultado
{
  "id":73,
  "jsonrpc": "2.0",
  "result": "0x09184e72a000" // 10000000000000
}
```

***

#### eth_accounts

Retorna uma lista de endereços sob controle do *client*.


##### Parâmetros
nenhum

##### Retorna

`Array of DATA`, 20 Bytes - endereços sob controle do *client*.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]
}
```

***

#### eth_blockNumber

Retorna o número do bloco mais recente.

##### Parâmetros
nenhum

##### Retorna

`QUANTITY` - inteiro do número do bloco atual no qual o *client* está.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}'

// Resultado
{
  "id":83,
  "jsonrpc": "2.0",
  "result": "0x4b7" // 1207
}
```

***

#### eth_getBalance

Retorna o balanço da conta de um dado endereço.

##### Parâmetros

1. `DATA`, 20 Bytes - endereço para checar o balanço.
2. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, veja o [parâmetro *default* do bloco](#the-default-block-parameter)

```js
params: [
   '0x407d73d8a49eeb85d32cf465507dd71d507100c1',
   'latest'
]
```

##### Retorna

`QUANTITY` - inteiro do balanço atual em wei.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "latest"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x0234c8a3397aab58" // 158972490234375000
}
```

***

#### eth_getStorageAt

Retorna o valor vindo de uma posição de armazenamento em um dado endereço.


##### Parâmetros

1. `DATA`, 20 Bytes - endereço do armazenamento.
2. `QUANTITY` - inteiro da posição no armazenamento.
3. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, Veja o [parâmetro *default* do bloco](#the-default-block-parameter)


```js
params: [
   '0x407d73d8a49eeb85d32cf465507dd71d507100c1',
   '0x0', // armazenamento em posição 0
   '0x2' // estado em número de bloco 2
]
```

##### Retorna

`DATA` - o valor nesta posição de armazenamento.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getStorageAt","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1", "0x0", "0x2"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x03"
}
```

***

#### eth_getTransactionCount

Retorna o número de transações enviadas de um endereço.

##### Parâmetros

1. `DATA`, 20 Bytes - endereço.
2. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, veja o [parâmetro *default* do bloco](#the-default-block-parâmetro)

```js
params: [
   '0x407d73d8a49eeb85d32cf465507dd71d507100c1',
   'latest' // estado no último bloco
]
```

##### Retorna

`QUANTITY` - Inteiro do número de transações enviadas deste endereço.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionCount","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1","latest"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

***

#### eth_getBlockTransactionCountByHash

Retorna o número de transações em um bloco vindas de um bloco correspondente ao dado *hash* de bloco.

##### Parâmetros

1. `DATA`, 32 Bytes - *hash* de um bloco

```js
params: [
   '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238'
]
```

##### Retorna

`QUANTITY` - inteiro do número de transações neste bloco.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xb" // 11
}
```

***

#### eth_getBlockTransactionCountByNumber

Retorna o número de transações em um bloco vindas de um bloco correspondente a um dado número de bloco.

##### Parâmetros

1. `QUANTITY|TAG` - inteiro de um número de bloco ou a *string* `"earliest"`, `"latest"` ou `"pending"`, como no [parâmetro *default* do bloco](#the-default-block-parameter
2. ).

```js
params: [
   '0xe8', // 232
]
```

##### Retorna

`QUANTITY` - inteiro do número de transações neste bloco.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockTransactionCountByNumber","params":["0xe8"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xa" // 10
}
```

***

#### eth_getUncleCountByBlockHash

Retorna o número de *uncles* em um bloco vindos de um bloco correspondente ao *hash* de bloco fornecido.

##### Parâmetros

1. `DATA`, 32 Bytes - *hash* de um bloco

```js
params: [
   '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238'
]
```

##### Retorna

`QUANTITY` - inteiro do número de *uncles* neste bloco.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id"Block:1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

***

#### eth_getUncleCountByBlockNumber

Retorna o número de *uncles* em um bloco vindos de um bloco correspondente ao número de bloco fornecido.


##### Parâmetros

1. `QUANTITY` - inteiro de um número de bloco ou a *string* "latest", "earliest" ou "pending", veja o [parâmetro *default*do bloco](#the-default-block-parameter)

```js
params: [
   '0xe8', // 232
]
```

##### Retorna

`QUANTITY` - inteiro do número de *uncles* neste bloco.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleCountByBlockNumber","params":["0xe8"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

***

#### eth_getCode

Retorna código em um dado endereço.


##### Parâmetros

1. `DATA`, 20 Bytes - endereço
2. `QUANTITY|TAG` - inteiro de um número de bloco ou a *string* "latest", "earliest" ou "pending", veja o [parâmetro *default*do bloco](#the-default-block-parameter)

```js
params: [
   '0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b',
   '0x2'  // 2
]
```

##### Retorna

`DATA` - o código do endereço .


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getCode","params":["0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b", "0x2"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b600060078202905091905056"
}
```

***

#### eth_sign

Assina dados com um determinado endereço.

**Nota** o endereço para assinar deve estar destrancado.

##### Parâmetros

1. `DATA`, 20 Bytes - endereço
2. `DATA`, Dados para assinar

##### Retorna

`DATA`: Dados assinados

##### Exemplo

```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sign","params":["0xd1ade25ccd3d550a7eb532ac759cac7be09c2719", "Schoolbus"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x2ac19db245478a06032e69cdbd2b54e648b78431d0a47bd1fbab18f79f820ba407466e37adbe9e84541cab97ab7d290f4a64a5825c876d22109f3bf813254e8601"
}
```



***

#### eth_sendTransaction

Cria uma nova transação de chamada de mensagem ou de criação de contrato, se o campo de dado contiver código.   

##### Parâmetros

1. `Object` - O objeto de transação
  - `from`: `DATA`, 20 Bytes - O endereço de onde a transação é enviada.
  - `to`: `DATA`, 20 Bytes - (Opcional quando criando um novo contrato) O endereço para onde a transação é dirigida.
  - `gas`: `QUANTITY`  - (opcional, *default*: 90000) Inteiro de gas fornecido para a execução da transação. Irá retornar gas não-utilizado.
  - `gasPrice`: `QUANTITY`  - (opcional, default: A ser determinado) Inteiro do preço de gas usado para cada gas pago.
  - `value`: `QUANTITY`  - (opcional) Inteiro do valor enviado com esta transação.
  - `data`: `DATA`  - O código compilado de um contrato OU o *hash* da assinatura do método chamado e parâmetros codificados. Para detalhes, veja [Contrato ABI Ethereum](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI)
  - `nonce`: `QUANTITY`  - (opcional)  Inteiro de um nonce. Isto permite sobrescrever suas próprias transações pendentes que usam o mesmo nonce.

```js
params: [{
  "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
  "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
  "gas": "0x76c0", // 30400,
  "gasPrice": "0x9184e72a000", // 10000000000000
  "value": "0x9184e72a", // 2441406250
  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
}]
```

##### Retorna

`DATA`, 32 Bytes - o *hash* da transação, ou o *hash* de zero se a transação ainda não estiver disponível.

Use [eth_getTransactionReceipt](#eth_gettransactionreceipt) para pegar o endereço do contrato, depois de a transação ter sido minerada, quando você criou um contrato.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{see above}],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```

***

#### eth_sendRawTransaction

Cria uma nova transação de chamada de mensagem ou de criação de contrato para transações assinadas.

##### Parâmetros

1. `DATA`, Os dados da transação assinada.

```js
params: ["0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"]
```

##### Retorna

`DATA`, 32 Bytes -  o *hash* da transação, ou o *hash* de zero se a transação ainda não estiver disponível.

Use [eth_getTransactionReceipt](#eth_gettransactionreceipt) para pegar o endereço do contrato, depois de a transação ter sido minerada, quando você criou um contrato.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":[{see above}],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
}
```

***

#### eth_call

Executa uma nova chamada de mensagem imediatamente sem criar uma transação na blockchain.


##### Parâmetros

1. `Object` - O objeto de chamada da transação
  - `from`: `DATA`, 20 Bytes - (opcional) O endereço de onde a transação é enviada.
  - `to`: `DATA`, 20 Bytes  -  O endereço para onde a transação é enviada..
  - `gas`: `QUANTITY`  - (opcional) Inteiro de gas fornecido para a execução da transação. eth_call consome zero gas, mas esse parâmetro pode ser necessário para algumas execuções.
  - `gasPrice`: `QUANTITY`  - (opcional)  Inteiro do preço de gas usado para cada gas pago.
  - `value`: `QUANTITY`  - (optional) Inteiro do valor enviado com essa transação.
  - `data`: `DATA`  - (opcional) O *hash* da assinatura do método e parâmetros codificados. Para detalhes, veja [Contrato ABI Ethereum](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI)
2. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, Veja o [parâmetro *default* do bloco](#the-default-block-parameter)

##### Retorna

`DATA` - o valor de retorno do contrato executado.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_call","params":[{see above}],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x0"
}
```

***

#### eth_estimateGas

Realiza uma chamada ou transação que não será adicionada à blockchain e retorna o gas usado, que pode ser usado para estimar o gas usado.

##### Parâmetros

Veja os parâmetros de [eth_call](#eth_call), espere que todas as propriedades sejam opcionais.

##### Retorna

`QUANTITY` - a quantidade de gas usado.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_estimateGas","params":[{see above}],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x5208" // 21000
}
```

***

#### eth_getBlockByHash

Retorna informação sobre um bloco pelo *hash*.


##### Parâmetros

1. `DATA`, 32 Bytes - *Hash* de um bloco.
2. `Boolean` - Se `true`,  retorna  os objetos completos de transação. Se `false` apenas os *hashes* das transações.

```js
params: [
   '0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331',
   true
]
```

##### Retorna

`Object` - Um objeto de bloco ou `null` quando nenhum bloco for achado:

  - `number`: `QUANTITY` - o número do bloco. `null` se o bloco estiver pendente.
  - `hash`: `DATA`, 32 Bytes - *hash* do bloco. `null` se o bloco estiver pendente.
  - `parentHash`: `DATA`, 32 Bytes - *hash* do bloco pai.
  - `nonce`: `DATA`, 8 Bytes - *hash* do *proof-of-work* gerado. `null` se o bloco estiver pendente.
  - `sha3Uncles`: `DATA`, 32 Bytes - SHA3 dos dados *uncles* no bloco.
  - `logsBloom`: `DATA`, 256 Bytes - o filtro *bloom* para os logs do bloco. `null` se o bloco estiver pendente.
  - `transactionsRoot`: `DATA`, 32 Bytes - a raiz da árvore de transação do bloco.
  - `stateRoot`: `DATA`, 32 Bytes - a raiz do último estado da árvore de transação do bloco.
  - `receiptsRoot`: `DATA`, 32 Bytes - a raiz da árvore de recibos do bloco.
  - `miner`: `DATA`, 20 Bytes - o endereço do beneficiário para quem as recompensas de transaÇão foram dadas.
  - `difficulty`: `QUANTITY` - inteiro da dificuldade deste bloco.
  - `totalDifficulty`: `QUANTITY` - inteiro da dificuldade total da *chain* até este bloco.
  - `extraData`: `DATA` - o campo de "dados extra" deste bloco.
  - `size`: `QUANTITY` - inteiro do tamanho deste bloco em bytes.
  - `gasLimit`: `QUANTITY` - o máximo de gas permitido neste bloco.
  - `gasUsed`: `QUANTITY` - o total de gas usado por todas as transações deste bloco.
  - `timestamp`: `QUANTITY` - o *timestamp* do unix de quando o blogo foi agrupado.
  - `transactions`: `Array` - *Array* de objetos de transação, ou *hashes* de transação de 32 Bytes dependendo do último parâmetro dado.
  - `uncles`: `Array` - *Array* de *hashes uncle*.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByHash","params":["0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331", true],"id":1}'

// Resultado
{
"id":1,
"jsonrpc":"2.0",
"result": {
    "number": "0x1b4", // 436
    "*hash*": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
    "parentHash": "0x9646252be9520f6e71339a8df9c55e4d7619deeb018d2a3f2d21fc165dde5eb5",
    "nonce": "0xe04d296d2460cfb8472af2c5fd05b5a214109c25688d3704aed5484f9a7792f2",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "logsBloom": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "stateRoot": "0xd5855eb08b3387c0af375e9cdb6acfc05eb8f519e419b874b6ff2ffda7ed1dff",
    "miner": "0x4e65fda2159562a496f9f3522f89122a3088497a",
    "difficulty": "0x027f07", // 163591
    "totalDifficulty":  "0x027f07", // 163591
    "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "size":  "0x027f07", // 163591
    "gasLimit": "0x9f759", // 653145
    "minGasPrice": "0x9f759", // 653145
    "gasUsed": "0x9f759", // 653145
    "timestamp": "0x54e34e8e" // 1424182926
    "transactions": [{...},{ ... }] 
    "uncles": ["0x1606e5...", "0xd5145a9..."]
  }
}
```

***

#### eth_getBlockByNumber

Retorna informação sobre um bloco pelo número do bloco.

##### Parâmetros

1. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, como no [parâmetro *default* do bloco](#the-default-block-parameter)
2. `Boolean` - Se `true`, retorna os objetos completos da transação, se `false` apenas os *hashes* das transações.

```js
params: [
   '0x1b4', // 436
   true
]
```

##### Retorna

Veja [eth_getBlockByHash](#eth_getblockby*hash*)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1b4", true],"id":1}'
```

Para o resultado veja [eth_getBlockByHash](#eth_getblockby*hash*)

***

#### eth_getTransactionByHash

Retorna a informação sobre a transação requerida pelo *hash* da transação.


##### Parâmetros

1. `DATA`, 32 Bytes - *hash* de uma transação.

```js
params: [
   "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"
]
```

##### Retorna

`Object` - Um objeto de transação, ou `null` quando nenhuma transação foi encontrada:

  - `*hash*`: `DATA`, 32 Bytes - *hash* da transação.
  - `nonce`: `QUANTITY` - o número de transações feito pelo *sender* (quem envia) anteriores a esta.
  - `blockHash`: `DATA`, 32 Bytes - *hash* do bloco onde esta transação está. `null` quando ela está pendente.
  - `blockNumber`: `QUANTITY` - número do bloco onde estava esta transação. `null` quando ela está pendente.
  - `transactionIndex`: `QUANTITY` - inteiro do índice de posição da transação no bloco. `null` quando ela está pendente.
  - `from`: `DATA`, 20 Bytes - endereço de quem envia.
  - `to`: `DATA`, 20 Bytes - endereço de quem recebe. `null` quando é uma transação de criação de contrato.
  - `value`: `QUANTITY` - valor transferido em Wei.
  - `gasPrice`: `QUANTITY` - preço de gas fornecido pelo *sender* em Wei.
  - `gas`: `QUANTITY` - gas fornecido pelo *sender*.
  - `input`: `DATA` - os dados enviados junto com a transação.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'

// Resultado
{
"id":1,
"jsonrpc":"2.0",
"result": {
    "*hash*":"0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b",
    "nonce":"0x",
    "blockHash": "0xbeab0aa2411b7ab17f30a99d3cb9c6ef2fc5426d6ad6fd9e2a26a6aed1d1055b",
    "blockNumber": "0x15df", // 5599
    "transactionIndex":  "0x1", // 1
    "from":"0x407d73d8a49eeb85d32cf465507dd71d507100c1",
    "to":"0x85h43d8a49eeb85d32cf465507dd71d507100c1",
    "value":"0x7f110" // 520464
    "gas": "0x7f110" // 520464
    "gasPrice":"0x09184e72a000",
    "input":"0x603880600c6000396000f300603880600c6000396000f3603880600c6000396000f360",
  }
}
```

***

#### eth_getTransactionByBlockHashAndIndex

Retorna informação sobre uma transação pelo *hash* do bloco e o índice de posição da transação.

##### Parâmetros

1. `DATA`, 32 Bytes - *hash* de um bloco.
2. `QUANTITY` - inteiro do índice de posição da transação.

```js
params: [
   '0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331',
   '0x0' // 0
]
```

##### Retorna

Veja [eth_getBlockByHash](#eth_gettransactionby*hash*)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockHashAndIndex","params":[0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b, "0x0"],"id":1}'
```

Para o resultado veja [eth_getTransactionByHash](#eth_gettransactionby*hash*)

***

#### eth_getTransactionByBlockNumberAndIndex

Retorna informação sobre a transação pelo número de bloco e índice de posição da transação.


##### Parâmetros

1. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, como no [parâmetro *default* do bloco](#the-default-block-parameter)
2. `QUANTITY` - o índice de posição da transação.

```js
params: [
   '0x29c', // 668
   '0x0' // 0
]
```

##### Retorna

Veja [eth_getBlockByHash](#eth_gettransactionby*hash*)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionByBlockNumberAndIndex","params":["0x29c", "0x0"],"id":1}'
```

Para o resultado veja [eth_getTransactionByHash](#eth_gettransactionby*hash*)

***

#### eth_getTransactionReceipt

Retorna o recibo de uma transação pelo *hash* da transação.

**Nota** O recibo não está disponível para transações pendentes.


##### Parâmetros

1. `DATA`, 32 Bytes - *hash* de uma transação.

```js
params: [
   '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238'
]
```

##### Retorna

`Object` - Um objeto de recibo de transação, ou `null` quando nenhum recibo foi encontrado:

  - `transactionHash `: `DATA`, 32 Bytes - *hash* da transação.
  - `transactionIndex`: `QUANTITY` - inteiro do índice de posição das transações no bloco.
  - `blockHash`: `DATA`, 32 Bytes - *hash* do bloco onde esta transação estava.
  - `blockNumber`: `QUANTITY` - número do bloco onde estava esta transação.
  - `cumulativeGasUsed `: `QUANTITY ` - A quantidade total de gas usada quando esta transação foi executada no bloco.
  - `gasUsed `: `QUANTITY ` - A quantidade de gas usada especificamente por esta transação sozinha.
  - `contractAddress `: `DATA`, 20 Bytes - O endereço de contrato criado, se a transação foi uma criação de contrato, caso contrário `null`.
  - `logs`: `Array` - *Array* de objetos do log, gerados por esta transação.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'

// Resultado
{
"id":1,
"jsonrpc":"2.0",
"result": {
     transactionHash: '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238',
     transactionIndex:  '0x1', // 1
     blockNumber: '0xb', // 11
     blockHash: '0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b',
     cumulativeGasUsed: '0x33bc', // 13244
     gasUsed: '0x4dc', // 1244
     contractAddress: '0xb60e8dd61c5d32be8058bb8eb970870f07233155' // or null, if none was created
     logs: [{
         // logs como retornados por getFilterLogs, etc.
     }, ...]
  }
}
```

***

#### eth_getUncleByBlockHashAndIndex

Retorna informação sobre o *uncle* de um bloco pelo *hash* e índice de posição do *uncle*.


##### Parâmetros


1. `DATA`, 32 Bytes - *hash* de um bloco.
2. `QUANTITY` - o índice de posição do *uncle*.

```js
params: [
   '0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b',
   '0x0' // 0
]
```

##### Retorna

Veja [eth_getBlockByHash](#eth_getblockby*hash*)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockHashAndIndex","params":["0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b", "0x0"],"id":1}'
```

Para o resultado veja [eth_getBlockByHash](#eth_getblockby*hash*)

**Nota**: Um *uncle* não contém transações individuais.

***

#### eth_getUncleByBlockNumberAndIndex

Retorna informação sobre um *uncle* de um bloco por número e índice de posição do *uncle*.


##### Parâmetros

1. `QUANTITY|TAG` - inteiro do número do bloco ou a *string* `"latest"`, `"earliest"` ou `"pending"`, como no [parâmetro *default* do bloco](#the-default-block-parameter)
2. `QUANTITY` - o índice de posição do *uncle*

```js
params: [
   '0x29c', // 668
   '0x0' // 0
]
```

##### Retorna

Veja [eth_getBlockByHash](#eth_getblockby*hash*)

**Nota**: Um *uncle* não contém transações individuais.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getUncleByBlockNumberAndIndex","params":["0x29c", "0x0"],"id":1}'
```

Para o resultado veja [eth_getBlockByHash](#eth_getblockby*hash*)

***

#### eth_getCompilers

Retorna uma lista de compiladores disponíveis no *client*.

##### Parâmetros
nenhum

##### Retorna

`Array` - *Array* de compiladores disponíveis.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getCompilers","params":[],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": ["solidity", "lll", "serpent"]
}
```

***

#### eth_compileSolidity

Retorna código de *solidity* compilado.

##### Parâmetros

1. `String` - O código fonte.

```js
params: [
   "contract test { function multiply(uint a) retorna(uint d) {   return a * 7;   } }",
]
```

##### Retorna

`DATA` - O código fonte compilado.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileSolidity","params":["contract test { function multiply(uint a) retorna(uint d) {   return a * 7;   } }"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": {
      "code": "0x605880600c6000396000f3006000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114602e57005b603d6004803590602001506047565b8060005260206000f35b60006007820290506053565b91905056",
      "info": {
        "source": "contract test {\n   function multiply(uint a) constant retorna(uint d) {\n       return a * 7;\n   }\n}\n",
        "language": "Solidity",
        "languageVersion": "0",
        "compilerVersion": "0.9.19",
        "abiDefinition": [
          {
            "constant": true,
            "inputs": [
              {
                "name": "a",
                "type": "uint256"
              }
            ],
            "name": "multiply",
            "outputs": [
              {
                "name": "d",
                "type": "uint256"
              }
            ],
            "type": "function"
          }
        ],
        "userDoc": {
          "methods": {}
        },
        "developerDoc": {
          "methods": {}
        }
      }

}
```

***

#### eth_compileLLL

Retorna código LLL compilado.

##### Parâmetros

1. `String` - O código fonte.

```js
params: [
   "(returnlll (suicide (caller)))",
]
```

##### Retorna

`DATA` - O código fonte compilado.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileLLL","params":["(returnlll (suicide (caller)))"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b6021600435602b565b8060005260206000f35b600081600702905091905056" // O código fonte compilado
}
```

***

#### eth_compileSerpent

Retorna código *serpent* compilado.

##### Parâmetros

1. `String` - O código fonte.

```js
params: [
   "/* some serpent */",
]
```

##### Retorna

`DATA` - O código fonte compilado.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileSerpent","params":["/* some serpent */"],"id":1}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b6021600435602b565b8060005260206000f35b600081600702905091905056" // O código fonte compilado
}
```

***

#### eth_newFilter

Cria um objeto de filtro, baseado em opções de filtro, para notificar quando houver mudança de estado (logs).
Para checar se houve mudança de estado, chame [eth_getFilterChanges](#eth_getfilterchanges).

##### Uma nota sobre a especificação de filtros de tópicos ~~A note on specifying topic filters:~~
Tópicos são dependentes de ordem. Uma transação com um log com tópicos [A, B] coincidirá com os seguintes filtros de tópicos:
* `[]` "qualquer coisa"
* `[A]` "A na primeira posição (e qualquer coisa após)"
* `[null, B]` "qualquer coisa na primeira posição E B na segunda posição (e qualquer coisa após)"
* `[A, B]` "A na primeira posição E B na segunda posição (e qualquer coisa após)"
* `[[A, B], [A, B]]` "(A OU B) na primeira posição E (A OU B) na segunda posição (e qualquer coisa após)"

##### Parâmetros

1. `Object` - As opções de filtro:
  - `fromBlock`: `QUANTITY|TAG` - (opcional, *default*: `"latest"`) Inteiro do número do bloco, ou `"latest"` para o último bloco minerado ou `"pending"`, `"earliest"` para transações ainda não mineradas.
  - `toBlock`: `QUANTITY|TAG` - (opcional, *default*: `"latest"`) Inteiro do número do bloco, ou `"latest"` para o último bloco minerado ou `"pending"`, `"earliest"` para transações ainda não mineradas.
  - `address`: `DATA|Array`, 20 Bytes - (opcional) Endereço do contrato ou uma lista de endereços de onde os logs devem se originar.
  - `topics`: `Array of DATA`,  - (opcional) *Array* de 32 Bytes de tópicos de `DATA`.  Tópicos são dependentes de ordem. Cada tópico pode ser também uma *array* de DADOS com opções "ou".

```js
params: [{
  "fromBlock": "0x1",
  "toBlock": "0x2",
  "address": "0x8888f1f195afa192cfee860698584c030f4c9db1",
  "topics": ["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b", null, [0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b, 0x000000000000000000000000aff3454fce5edbc8cca8697c15331677e6ebccc]]
}]
```

##### Retorna

`QUANTITY` - Uma id de filtro.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newFilter","params":[{"topics":["0x12341234"]}],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0x1" // 1
}
```

***

#### eth_newBlockFilter

Cria um filtro no nó, para notificar quando um novo bloco chegar.
Para checar se houve mudança de estado, chame [eth_getFilterChanges](#eth_getfilterchanges).

##### Parâmetros
Nenhum

##### Retorna

`QUANTITY` - Uma id de filtro.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newBlockFilter","params":[],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":  "2.0",
  "result": "0x1" // 1
}
```

***

#### eth_newPendingTransactionFilter

Cria um filtro no nó, para notificar quando novas transações pendentes chegarem.
Para checar se houve mudança de estado, chame [eth_getFilterChanges](#eth_getfilterchanges).

##### Parâmetros
Nenhum

##### Retorna

`QUANTITY` - Uma id de filtro.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_newPendingTransactionFilter","params":[],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":  "2.0",
  "result": "0x1" // 1
}
```

***

#### eth_uninstallFilter

Desinstala um filtro com uma dada id. Deve sempre ser chamada quando a observação não for mais necessária.
Adicionalmente, filtros expiram quando não são requisitados com [eth_getFilterChanges](#eth_getfilterchanges) por um período de tempo.


##### Parâmetros

1. `QUANTITY` - Uma id de filtro.

```js
params: [
  "0xb" // 11
]
```

##### Retorna

`Boolean` - `true` se o filtro foi desinstalado com sucesso, caso contrário `false`.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_uninstallFilter","params":["0xb"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": true
}
```

***

#### eth_getFilterChanges

Método de *polling* para um filtro, o qual retorna uma *array* de logs que ocorreram desde o último *poll*.


##### Parâmetros

1. `QUANTITY` - Uma id de filtro.

```js
params: [
  "0x16" // 22
]
```

##### Retorna

`Array` - *Array* de objetos do log, ou uma *array* vazia se nada tiver mudade desde o último *poll*.

- Para filtros criados com `eth_newBlockFilter` o retorno são *hashes* de blocos (`DATA`, 32 Bytes), e.g. `["0x3454645634534..."]`.
- Para filtros criados com `eth_newPendingTransactionFilter ` o retorno são *hashes* de transações (`DATA`, 32 Bytes), e.g. `["0x6345343454645..."]`.
- Para filtros criados com `eth_newFilter` logs são objetos com os seguintes params:

  - `type`: `TAG` - `pending` quando o log está pendente. `mined` se o log já está minerado.
  - `logIndex`: `QUANTITY` - inteiro do índice de posição do log no bloco. `null` quando o log está pendente.
  - `transactionIndex`: `QUANTITY` - inteiro do índice de posição das transações de onde o log foi criado. `null` quando o log está pendente.
  - `transactionHash`: `DATA`, 32 Bytes - *hash* das transações de onde foi criado esse log. `null` quando o log está pendente.
  - `blockHash`: `DATA`, 32 Bytes - *hash* do bloco onde estava esse log. `null` quando o log está pendente.
  - `blockNumber`: `QUANTITY` - o número do bloco onde este log estava. `null` quando o log está pendente.
  - `address`: `DATA`, 20 Bytes - endereço de onde este log se originou.
  - `data`: `DATA` - contém um ou mais argumentos não-indexados de 32 Bytes do log.
  - `topics`: `Array of DATA` - *Array* de 0 a 4 `DATA` de 32 Bytes de argumentos indexados do log. (Em *solidity*: o primeiro tópico é o **hash** da assinatura do evento (e.g. `Deposit(address,bytes32,uint256)`), exceto se você declarou o evento com o especificador `anonymous`.)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getFilterChanges","params":["0x16"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": [{
    "logIndex": "0x1", // 1
    "blockNumber":"0x1b4" // 436
    "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
    "transactionHash":  "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcf",
    "transactionIndex": "0x0", // 0
    "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
    "data":"0x0000000000000000000000000000000000000000000000000000000000000000",
    "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"]
    },{
      ...
    }]
}
```

***

#### eth_getFilterLogs

Retorna uma *array* de todos os logs coincidentes com a dada id.


##### Parâmetros

1. `QUANTITY` - A id do filtro.

```js
params: [
  "0x16" // 22
]
```

##### Retorna

Veja [eth_getFilterChanges](#eth_getfilterchanges)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getFilterLogs","params":["0x16"],"id":74}'
```

Para o resultado veja [eth_getFilterChanges](#eth_getfilterchanges)

***

#### eth_getLogs

Retorna uma *array* de todos os logs coincidentes com um dado objeto de filtro.

##### Parâmetros

1. `Object` - o objeto de filtro, veja [eth_newFilter parâmetros](#eth_newfilter).

```js
params: [{
  "topics": ["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]
}]
```

##### Retorna

Veja [eth_getFilterChanges](#eth_getfilterchanges)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getLogs","params":[{"topics":["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]}],"id":74}'
```

Para o resultado veja [eth_getFilterChanges](#eth_getfilterchanges)

***

#### eth_getWork

Retorna o *hash* do bloco atual, o seedHash, e a condição de contorna a ser satisfeita ("target").

##### Parâmetros
Nenhum

##### Retorna

`Array` - *Array*  com as seguintes propriedades:
  1. `DATA`, 32 Bytes - pow do cabeçalho do bloco atual ~~current block header pow~~-*hash*
  2. `DATA`, 32 Bytes - o *hash* semente usado para o DAG. ~~the seed *hash* used for the DAG.~~
  3. `DATA`, 32 Bytes - a condição de contorno ("target"), 2^256 / dificuldade.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getWork","params":[],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": [
      "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
      "0x5EED00000000000000000000000000005EED0000000000000000000000000000",
      "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
    ]
}
```

***

#### eth_submitWork

Usado para submeter uma solução de *proof-of-work*.


##### Parâmetros

1. `DATA`, 8 Bytes - O *nonce* encontrado (64 bits)
2. `DATA`, 32 Bytes - O pow do cabeçalho ~~The header's pow~~-*hash* (256 bits)
3. `DATA`, 32 Bytes - O resumo da mistura ~~The mix digest~~ (256 bits)

```js
params: [
  "0x0000000000000001",
  "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
  "0xD1FE5700000000000000000000000000D1FE5700000000000000000000000000"
]
```

##### Retorna

`Boolean` - retorna `true` se a solução provida for válida, caso contrário `false`.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0", "method":"eth_submitWork", "params":["0x0000000000000001", "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef", "0xD1GE5700000000000000000000000000D1GE5700000000000000000000000000"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": true
}
```

***

#### eth_submitHashrate

Usado para submeter o *hashrate* da mineração.


##### Parâmetros

1. `Hashrate`, uma represendação em *string* hexadecimal (32 bytes) da taxa de *hash*.
2. `ID`, *String* - Uma ID hexadecimal randômica (32 bytes) indentificando o *client*.

```js
params: [
  "0x0000000000000000000000000000000000000000000000000000000000500000",
  "0x59daa26581d0acd1fce254fb7e85952f4c09d0915afd33d3886cd914bc7d283c"
]
```

##### Retorna

`Boolean` - retorna `true` se a submissão foi completada com sucesso e `false`, caso contrário.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0", "method":"eth_submitHashrate", "params":["0x0000000000000000000000000000000000000000000000000000000000500000", "0x59daa26581d0acd1fce254fb7e85952f4c09d0915afd33d3886cd914bc7d283c"],"id":73}'

// Resultado
{
  "id":73,
  "jsonrpc":"2.0",
  "result": true
}
```

***

#### db_putString

Armazena uma *string* no banco de dados local.

**Nota** esta função está obsoleta e será removida no futuro.

##### Parâmetros

1. `String` - Nome do banco de dados.
2. `String` - Nome da chave.
3. `String` - *String* para armazenar.

```js
params: [
  "testDB",
  "myKey",
  "myString"
]
```

##### Retorna

`Boolean` - retorna `true` se o valor foi armazanado, caso contrário `false`.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"db_putString","params":["testDB","myKey","myString"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": true
}
```

***

#### db_getString

Retorna *string* do banco de dados local.

**Nota** esta função está obsoleta e será removida no futuro.

##### Parâmetros

1. `String` - Nome do banco de dados.
2. `String` - Nome da chave.

```js
params: [
  "testDB",
  "myKey",
]
```

##### Retorna

`String` - A *string* armazenada previamente.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"db_getString","params":["testDB","myKey"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": "myString"
}
```

***

#### db_putHex

Armazena dados binários no banco de dados local.

**Nota** esta função está obsoleta e será removida no futuro.

##### Parâmetros

1. `String` - Nome do banco de dados.
2. `String` - Nome da chave.
3. `DATA` - Os dados para armazenar.

```js
params: [
  "testDB",
  "myKey",
  "0x68656c6c6f20776f726c64"
]
```

##### Retorna

`Boolean` - retorna `true` se o valor foi armazenado, caso contrário `false`.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"db_putHex","params":["testDB","myKey","0x68656c6c6f20776f726c64"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": true
}
```

***

#### db_getHex

Retorna dados binários do banco de dados local.

**Nota** esta função está obsoleta e será removida no futuro.

##### Parâmetros

1. `String` - Nome do banco de dados.
2. `String` - Nome da chave.

```js
params: [
  "testDB",
  "myKey",
]
```

##### Retorna

`DATA` - Os dados armazenados previamente.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"db_getHex","params":["testDB","myKey"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": "0x68656c6c6f20776f726c64"
}
```

***

#### shh_version

Retorna a versão atual do protocolo whisper.

##### Parâmetros
Nenhum

##### Retorna

`String` - A versão atual do protocolo whisper.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_version","params":[],"id":67}'

// Resultado
{
  "id":67,
  "jsonrpc": "2.0",
  "result": "2"
}
```

***

#### shh_post

Envia uma mensagem whisper.

##### Parâmetros

1. `Object` - O objeto *post* do whisper:
  - `from`: `DATA`, 60 Bytes - (opcional) A identidade de quem envia.
  - `to`: `DATA`, 60 Bytes - (opcional) A identidade de quem recebe. Quando presente, whisper irá encriptar a mensagem para que apenas o destinatário possa desencriptá-la.
  - `topics`: `Array of DATA` - *Array* de tópicod de `DATA`, para o destinatário identificar as mensagens.
  - `payload`: `DATA` - O conteúdo da mensagem. ~~The payload of the message.~~
  - `priority`: `QUANTITY` - O inteiro da prioridade em um intervalo de ... (?). ~~The integer of the priority in a rang from ... (?).~~
  - `ttl`: `QUANTITY` - inteiro do tempo de vida em segundos.

```js
params: [{
  from: "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1",
  to: "0x3e245533f97284d442460f2998cd41858798ddf04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a0d4d661997d3940272b717b1",
  topics: ["0x776869737065722d636861742d636c69656e74", "0x4d5a695276454c39425154466b61693532"],
  payload: "0x7b2274797065223a226d6",
  priority: "0x64",
  ttl: "0x64",
}]
```

##### Retorna

`Boolean` - retorna `true` se a mensagem foi enviada, caso contrário `false`.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_post","params":[{"from":"0xc931d93e97ab07fe42d923478ba2465f2..","topics": ["0x68656c6c6f20776f726c64"],"payload":"0x68656c6c6f20776f726c64","ttl":0x64,"priority":0x64}],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": true
}
```

***

#### shh_newIdentity

Crria nova indentidade whisper no *client*.

##### Parâmetros
Nenhum

##### Retorna

`DATA`, 60 Bytes - o endereço da nova identidade.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_newIdentity","params":[],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xc931d93e97ab07fe42d923478ba2465f283f440fd6cabea4dd7a2c807108f651b7135d1d6ca9007d5b68aa497e4619ac10aa3b27726e1863c1fd9b570d99bbaf"
}
```

***

#### shh_hasIdentity

Checa se o *client* possui as chaves privadas para uma determinada identidade.


##### Parâmetros

1. `DATA`, 60 Bytes - O endereço da identidade para checar.

```js
params: [
  "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"
]
```

##### Retorna

`Boolean` - retorna `true` se o *client* possuir as chaves privadas para aquela identidade, caso contrário `false`.


##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_hasIdentity","params":["0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": true
}
```

***

#### shh_newGroup

(?)

##### Parâmetros
Nenhum

##### Retorna

`DATA`, 60 Bytes - O endereço do novo grupo. (?)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_newIdentity","params":[],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": "0xc65f283f440fd6cabea4dd7a2c807108f651b7135d1d6ca90931d93e97ab07fe42d923478ba2407d5b68aa497e4619ac10aa3b27726e1863c1fd9b570d99bbaf"
}
```

***

#### shh_addToGroup

(?)

##### Parâmetros

1. `DATA`, 60 Bytes - O endereço da identidade para adicionar a um grupo (?).

```js
params: [
  "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"
]
```

##### Retorna

`Boolean` - retorna `true` se a identidade foi adicionada com sucesso ao grupo, caso contrário `false` (?).

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_hasIdentity","params":["0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": true
}
```

***

#### shh_newFilter

Cria filtro para notificar quando o *client* recebe mensagens whisper coincidindo com as opções de filtro.


##### Parâmetros

1. `Object` - As opções de filtro:
  - `to`: `DATA`, 60 Bytes - (opcional) Identidade do destinatário. *Quando presente, irá tentar desencriptar qualquer mensagem que chegue se o *client* tiver as chaves privadas para essa identidade.
  - `topics`: `Array of DATA` - *Array* de tópicos `DATA` que os tópicos da mensagem que chegar devem coincidir. Você pode usar as seguintes combinações:
    - `[A, B] = A && B`
    - `[A, [B, C]] = A && (B || C)`
    - `[null, A, B] = ANYTHING && A && B` `null` funciona como um curinga

```js
params: [{
   "topics": ['0x12341234bf4b564f'],
   "to": "0x04f96a5e25610293e42a73908e93ccc8c4d4dc0edcfa9fa872f50cb214e08ebf61a03e245533f97284d442460f2998cd41858798ddfd4d661997d3940272b717b1"
}]
```

##### Retorna

`QUANTITY` - O filtro recém-criado.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_newFilter","params":[{"topics": ['0x12341234bf4b564f'],"to": "0x2341234bf4b2341234bf4b564f..."}],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": "0x7" // 7
}
```

***

#### shh_uninstallFilter

Desinstala um filtro com uma dada id. Deve sempre ser chamado quando a observação não for mais necessária. Adicionalmente, filtros expiram quando não são requisitados com [shh_getFilterChanges](#shh_getfilterchanges) por um período de tempo.

##### Parâmetros

1. `QUANTITY` - A id do filtro.

```js
params: [
  "0x7" // 7
]
```

##### Retorna

`Boolean` - `true` se o filtro foi desinstalado com sucesso, caso contrário `false`.

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_uninstallFilter","params":["0x7"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": true
}
```

***

#### shh_getFilterChanges

Método de *polling* para filtros whisper. Retorna novas mensagens desde a última chamada deste método.

**Nota** chamar o método [shh_getMessages](#shh_getmessages), irá resetar o *buffer* para este método, para que você não receba mensagens duplicadas.


##### Parâmetros

1. `QUANTITY` - A id do filtro.

```js
params: [
  "0x7" // 7
]
```

##### Retorna

`Array` - *Array* de mensagens recebidas desde o último *poll*:

  - `*hash*`: `DATA`, 32 Bytes (?) - O *hash* da mensagem.
  - `from`: `DATA`, 60 Bytes - O rementente da mensagem, se um remetente foi especificado.
  - `to`: `DATA`, 60 Bytes - O destinatário da mensagem, se um destinatário foi especificado.
  - `expiry`: `QUANTITY` - Inteiro do tempo em segundos quando esta mensagem deve expirar (?).
  - `ttl`: `QUANTITY` -  Inteiro no tempo que a mensagem deve flutuar no sistema em segundos (?).
  - `sent`: `QUANTITY` -  Inteiro do *timestamp* do unix de quando a mensagem foi enviada.
  - `topics`: `Array of DATA` - *Array* de tópicos `DATA` que a mensagem contém.
  - `payload`: `DATA` - O conteúdo da mensagem. ~~The payload of the message.~~
  - `workProved`: `QUANTITY` - Inteiro do trabalho que essa mensagem requeriu antes de ser enviada (?).

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_getFilterChanges","params":["0x7"],"id":73}'

// Resultado
{
  "id":1,
  "jsonrpc":"2.0",
  "result": [{
    "*hash*": "0x33eb2da77bf3527e28f8bf493650b1879b08c4f2a362beae4ba2f71bafcd91f9",
    "from": "0x3ec052fc33..",
    "to": "0x87gdf76g8d7fgdfg...",
    "expiry": "0x54caa50a", // 1422566666
    "sent": "0x54ca9ea2", // 1422565026
    "ttl": "0x64" // 100
    "topics": ["0x6578616d"],
    "payload": "0x7b2274797065223a226d657373616765222c2263686...",
    "workProved": "0x0"
    }]
}
```

***

#### shh_getMessages

Pega todas as mensagens correspondentes a um filtro. Diferente de `shh_getFilterChanges`, isso retorna todas as mensagens.

##### Parâmetros

1. `QUANTITY` - A id do filtro.

```js
params: [
  "0x7" // 7
]
```

##### Retorna

Veja [shh_getFilterChanges](#shh_getfilterchanges)

##### Exemplo
```js
// Requisição
curl -X POST --data '{"jsonrpc":"2.0","method":"shh_getMessages","params":["0x7"],"id":73}'
```

Para o resultado veja [shh_getFilterChanges](#shh_getfilterchanges)
