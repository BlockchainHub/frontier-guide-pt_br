<--! Source: https://raw.githubusercontent.com/wiki/ethereum/wiki/JavaScript-API.md-->

# API de Web3 JavaScript para Ðapp

Para fazer a sua  Ðapp funcionar na Ethereum, você pode usar o objeto `web3` provido pela biblioteca [web3.js library](https://github.com/ethereum/web3.js). Por baixo dos panos, ela se comunica a um nó local através de [Chamadas RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC). web3.js funciona com qualquer nó Ethereum que exponha uma camada RPC.

`web3` contém o objeto `eth` - `web3.eth` (especificamente para interações da blockchain da Ethereum) e o objeto `shh` - `web3.shh` (para interações de Whisper). Ao longo do tempo, nós introduziremos outros objetos para cada um dos outros protocolos web3. [Exemplos em funcionamento podem ser encontrados aqui](https://github.com/ethereum/web3.js/tree/master/example).

Se você quiser ver alguns exemplos mais sofisticados usando web3.js, veja esses [padrões de app úteis](https://github.com/ethereum/wiki/wiki/Useful-Ðapp-Patterns).

## Começando

* [Adicionando web3](#adding-web3)
* [Usando Callbacks](#using-callbacks)
* [Requisições em *Batch*](#batch-requests)
* [Uma nota sobre grandes números em web3.js](#a-note-on-big-numbers-in-web3js)
* [-> Referência da API](#web3js-api-reference)

### Adicionando web3

Primeiro, você precisa colocar web3.js nos seus projetos. Isso pode ser feito usando os seguintes métodos:

- npm: `npm install web3`
- bower: `bower install web3`
- meteor: `meteor add ethereum:web3`
- vanilla: link the `dist./web3.min.js`

Depois, você precisa criar uma instância de web3, definindo um provedor.
Para garantir que você não vai sobrescrever o provedor já definido no *mist*, cheque primeiro se a web3 está disponível:

```js
if (typeof web3 !== 'undefined') {
  web3 = new Web3(web3.currentProvider);
} else {
  // define o provedor que você quer de Web3.providers
  web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
}
```

Depois disso, você pode usar a [API](web3js-api-reference) do objeto `web3`.

### Usando callbacks

Como essa API é feita para funcionar com um nó RPC local, todas as suas funções usam requisições síncronas de HTTP por *default*.

Se você quer fazer uma requisição assíncrona, você pode passar um *callback* opçional como último parâmetro para a maior parte das funções.
Todos *callbacks* estão usando um estilo de [callback de erro antes](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/):

```js
web3.eth.getBlock(48, function(error, result){
    if(!error)
        console.log(result)
    else
        console.error(error);
})
```

### Requisições em *batch*

Requisições em batelada (*batch*) permitem enfileirar requisições e as processar de uma vez.

**Nota** Requisições em batelada não são mais rápidas! De fato, fazer várias requisições de uma vez será mais rápido em alguns casos, já que requisições são processadas assincronamente. Requisições em batelada são úteis, principalmente, para garantir o processamento em série das requisições.

```js
var batch = web3.createBatch();
batch.add(web3.eth.getBalance.request('0x0000000000000000000000000000000000000000', 'latest', callback));
batch.add(web3.eth.contract(abi).at(address).balance.request(address, callback2));
batch.execute();
```

### Uma nota sobre grandes números em web3.js

Você sempre terá um objeto BigNumber para valores numéricos já que o JavaScript não é capaz de lidar com grandes números corretamente.
Veja os seguintes exemplos:

```js
"101010100324325345346456456456456456456"
// "101010100324325345346456456456456456456"
101010100324325345346456456456456456456
// 1.0101010032432535e+38
```

web3.js depende da [Biblioteca BigNumber](https://github.com/MikeMcl/bignumber.js/) e a adiciona automaticamente.

```js
var balance = new BigNumber('131242344353464564564574574567456');
// ou var balance = web3.eth.getBalance(someAddress);

balance.plus(21).toString(10); // toString(10) converte para uma *string* de número
// "131242344353464564564574574567477"
```

O próximo exemplo não funcionaria já que temos mais de 20 pontos flutuantes, por isso, é recomendado manter seu balanço sempre em *wei* e apenas transformá-lo para outras unidades quando for apresentar ao usuário:

```js
var balance = new BigNumber('13124.234435346456466666457455567456');

balance.plus(21).toString(10); // toString(10) converte para uma *string* de número, mas pode mostrar apenas um máximo de 20 pontos flutuantes
// "13145.23443534645646666646" // seu número seria cortado após 20 pontos flutuantes
```

## Referência da API Web3.js

* [web3](#web3)
  * [versão](#web3versionapi)
     * [api](#web3versionapi)
     * [node/getNode](#web3versionnode)
     * [network/getNetwork](#web3versionnetwork)
     * [ethereum/getEthereum](#web3versionethereum)
     * [whisper/getWhisper](#web3versionwhisper)
  * [isConnected()](#web3isconnected)
  * [setProvider(provider)](#web3setprovider)
  * [currentProvider](#web3currentprovider)
  * [reset()](#web3reset)
  * [sha3(*string*, options)](#web3sha3)
  * [toHex(*string*OrNumber)](#web3tohex)
  * [toAscii(hexString)](#web3toascii)
  * [fromAscii(textString, [padding])](#web3fromascii)
  * [toDecimal(hexString)](#web3todecimal)
  * [fromDecimal(number)](#web3fromdecimal)
  * [fromWei(numberStringOrBigNumber, unit)](#web3fromwei)
  * [toWei(numberStringOrBigNumber, unit)](#web3towei)
  * [toBigNumber(numberOrHexString)](#web3tobignumber)
  * [isAddress(hexString)](#web3isAddress)
  * [net](#web3net)
    * [listening/getListening](#web3netlistening)
    * [*peer*Count/getPeerCount](#web3eth*peer*count)
  * [eth](#web3eth)
    * [defaultAccount](#web3ethdefaultaccount)
    * [defaultBlock](#web3ethdefaultblock)
    * [syncing/getSyncing](#web3ethsyncing)
    * [isSyncing](#web3ethissyncing)
    * [coinbase/getCoinbase](#web3ethcoinbase)
    * [*hash*rate/getHashrate](#web3eth*hash*rate)
    * [gasPrice/getGasPrice](#web3ethgasprice)
    * [accounts/getAccounts](#web3ethaccounts)
    * [mining/getMining](#web3ethmining)
    * [blockNumber/getBlockNumber](#web3ethblocknumber)
    * [register(hexString)](#web3ethregister) (Ainda não implementado)
    * [unRegister(hexString)](#web3ethunregister) (Ainda não implementado)
    * [getBalance(address)](#web3ethgetbalance)
    * [getStorageAt(address, position)](#web3ethgetstorageat)
    * [getCode(address)](#web3ethgetcode)
    * [getBlock(*hash*/number)](#web3ethgetblock)
    * [getBlockTransactionCount(*hash*/number)](#web3ethgetblocktransactioncount)
    * [getUncle(*hash*/number)](#web3ethgetuncle)
    * [getBlockUncleCount(*hash*/number)](#web3ethgetblockunclecount)
    * [getTransaction(*hash*)](#web3ethgettransaction)
    * [getTransactionFromBlock(*hash*OrNumber, indexNumber)](#web3ethgettransactionfromblock)
    * [getTransactionReceipt(*hash*)](#web3ethgettransactionreceipt)
    * [getTransactionCount(address)](#web3ethgettransactioncount)
    * [sendTransaction(object)](#web3ethsendtransaction)
    * [sendRawTransaction(object)](#web3ethsendrawtransaction)
    * [sign(object)](#web3ethsign)
    * [call(object)](#web3ethcall)
    * [estimateGas(object)](#web3ethestimategas)
    * [filter(array (, options) )](#web3ethfilter)
        - [watch(callback)](#web3ethfilter)
        - [stopWatching(callback)](#web3ethfilter)
        - [get()](#web3ethfilter)
    * [contract(abiArray)](#web3ethcontract)
    * [contract.myMethod()](#contract-methods)
    * [contract.myEvent()](#contract-events)
    * [contract.allEvents()](#contract-allevents)
    * [getCompilers()](#web3ethgetcompilers)
    * [compile.lll(*string*)](#web3ethcompilelll)
    * [compile.solidity(*string*)](#web3ethcompilesolidity)
    * [compile.serpent(*string*)](#web3ethcompileserpent)
    * [namereg](#web3ethnamereg)
    * [sendIBANTransaction](#web3ethsendibantransaction)
    * [iban](#web3ethiban)
      * [fromAddress](#web3ethibanfromaddress)
      * [fromBban](#web3ethibanfrombban)
      * [createIndirect](#web3ethibancreateindirect)
      * [isValid](#web3ethibanisvalid)
      * [isDirect](#web3ethibanisdirect)
      * [isIndirect](#web3ethibanisindirect)
      * [checksum](#web3ethibanchecksum)
      * [institution](#web3ethibaninstitution)
      * [*client*](#web3ethiban*client*)
      * [address](#web3ethibanaddress)
      * [toString](#web3ethibanto*string*)
  * [db](#web3db)
    * [putString(name, key, value)](#web3dbput*string*)
    * [getString(name, key)](#web3dbget*string*)
    * [putHex(name, key, value)](#web3dbputhex)
    * [getHex(name, key)](#web3dbgethex)
  * [shh](#web3shh)
    * [post(postObject)](#web3shhpost)
    * [newIdentity()](#web3shhnewidentity)
    * [hasIdentity(hexString)](#web3shhhaveidentity)
    * [newGroup(_id, _who)](#web3shhnewgroup)
    * [addToGroup(_id, _who)](#web3shhaddtogroup)
    * [filter(object/*string*)](#web3shhfilter)
      * [watch(callback)](#web3shhfilter)
      * [stopWatching(callback)](#web3shhfilter)
      * [get(callback)](#web3shhfilter)

### Uso

#### web3
O objeto `web3` fornece todos os métodos.

##### Exemplo

```js
var Web3 = require('web3');
// cria uma instância de web3 usando o provedor de HTTP.
// NOTA no mist web3 já está disponível, então cheque sua disponibilidade antes de criar uma instância.
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
```

***

#### web3.version.api

```js
web3.version.api
```

##### Retorna

`String` - A versão  da api js da Ethereum.

##### Exemplo

```js
var version = web3.version.api;
console.log(version); // "0.2.0"
```

***

#### web3.version.node

    web3.version.node
    // or async
    web3.version.getNode(callback(error, result){ ... })


##### Retorna

`String` - A versão do *client*/nó.

##### Exemplo

```js
var version = web3.version.node;
console.log(version); // "Mist/v0.9.3/darwin/go1.4.1"
```

***

#### web3.version.network

    web3.version.network
    // or async
    web3.version.getNetwork(callback(error, result){ ... })


##### Retorna

`String` - A versão do protocolo de rede.

##### Exemplo

```js
var version = web3.version.network;
console.log(version); // 54
```

***

#### web3.version.ethereum

    web3.version.ethereum
    // or async
    web3.version.getEthereum(callback(error, result){ ... })


##### Retorna

`String` - A versão do protocolo Ethereum.

##### Exemplo

```js
var version = web3.version.ethereum;
console.log(version); // 60
```

***

#### web3.version.whisper

    web3.version.whisper
    // or async
    web3.version.getWhisper(callback(error, result){ ... })


##### Retorna

`String` - The whisper protocol version.

##### Exemplo

```js
var version = web3.version.whisper;
console.log(version); // 20
```

***
#### web3.isConnected

    web3.isConnected()

Deve ser chamado para checar se uma conexão com um nó existe.

##### Parâmetros
Nenhum

##### Retorna

`Boolean`

##### Exemplo

```js
if(!web3.isConnected()) {
  
   // mostra um diálogo para perguntar se o usuário quer iniciar um nó

} else {
 
   // inicia filtros web3, chamadas web3, etc.
  
}
```

***

#### web3.setProvider

    web3.setProvider(provider)

Deve ser chamado para definir o provedor.

##### Parâmetros
Nenhum

##### Retorna

`undefined`

##### Exemplo

```js
web3.setProvider(new web3.providers.HttpProvider('http://localhost:8545')); // 8080 para cpp/AZ, 8545 para go/mist
```

***

#### web3.currentProvider

    web3.currentProvider

Conterá o atual provedor, se um estiver definido. Isso pode ser usado para checar se o mist já possui um provedor definido.

##### Retorna

`Object` - O provedor definido ou `null`;

##### Exemplo

```js
// Checa se o mist etc. já definiu um provedor. ~~Check if mist etc. already set a provider~~
if(!web3.currentProvider)
    web3.setProvider(new web3.providers.HttpProvider("http://localhost:8545"));

```

***

#### web3.reset

    web3.reset(keepIsSyncing)

Deve ser chamado para resetar o estado de web3. Reseta tudo exceto o *manager*. Desinstala todos os filtros. Pára o *polling*.

##### Parâmetros

1. `Boolean` - Se `true` irá desinstalar todos os filtros, mas manterá os *polls* de [web3.eth.isSyncing()](#web3ethissyncing).

##### Retorna

`undefined`

##### Exemplo

```js
web3.reset();
```

***

#### web3.sha3

    web3.sha3(*string*, options)

##### Parâmetros

1. `String` - A *string* para fezer *hash* usando o algoritmo Keccak-256 SHA3.
1. `Object` - Define `encoding` para `hex` se a *string* para fazer *hash* estiver codificada em hex. Um `0x` inicial será automaticamente ignorado.

##### Retorna

`String` - O Keccak-256 SHA3 dos dados em questão.

##### Exemplo

```js
var *hash* = web3.sha3("Some *string* to be *hash*ed");
console.log(*hash*); // "0xed973b234cf2238052c9ac87072c71bcf33abc1bbd721018e0cca448ef79b379"

var *hash*OfHash = web3.sha3(*hash*, {encoding: 'hex'});
console.log(*hash*OfHash); // "0x85dd39c91a64167ba20732b228251e67caed1462d4bcf036af88dc6856d0fdcc"
```

***

#### web3.toHex

    web3.toHex(mixed);
 
Converte qualquer valor em HEX.

##### Parâmetros

1. `String|Number|Object|Array|BigNumber` - O valor para converter em HEX. Se for um objeto ou uma *array* será `JSON.stringify` primeiro. Se for um BigNumber irá transformá-lo no valor HEX de um número.

##### Retorna

`String` - A *string* em HEX de `mixed`.

##### Exemplo

```js
var str = web3.toHex({test: 'test'});
console.log(str); // '0x7b2274657374223a2274657374227d'
```

***

#### web3.toAscii

    web3.toAscii(hexString);

Converte uma *string* em HEX para uma *string* em ASCII.

##### Parâmetros

1. `String` - Uma *string* em HEX para ser convertida em ascii.

##### Retorna

`String` - Uma *string* em ASCII feita da dada `hexString`.

##### Exemplo

```js
var str = web3.toAscii("0x657468657265756d000000000000000000000000000000000000000000000000");
console.log(str); // "ethereum"
```

***

#### web3.fromAscii

    web3.fromAscii(*string* [, padding]);

Converte qualquer *string* em ASCII para uma *string* em HEX.

##### Parâmetros

1. `String` - Uma *string* em ASCII a ser convertida em HEX.
2. `Number` - O número de bytes que a *string* em HEX retornada deve ter.

##### Retorna

`String` - A *string* em HEX convertida.

##### Exemplo

```js
var str = web3.fromAscii('ethereum');
console.log(str); // "0x657468657265756d"

var str2 = web3.fromAscii('ethereum', 32);
console.log(str2); // "0x657468657265756d000000000000000000000000000000000000000000000000"
```

***

#### web3.toDecimal

    web3.toDecimal(hexString);

Converte uma *string* em HEX para sua representação numérica.

##### Parâmetros

1. `String` - Uma *string* em HEX a ser convertida em número.


##### Retorna

`Number` - O número representando os dados `hexString`.

##### Exemplo

```js
var number = web3.toDecimal('0x15');
console.log(number); // 21
```

***

#### web3.fromDecimal

    web3.fromDecimal(number);

Converte um número ou *string* de número em sua representação em HEX.

##### Parâmetros

1. `Number|String` - Um número a ser convertido a uma *string* em HEX.

##### Retorna

`String` - A *string* em HEX representando o dado `number`.

##### Exemplo

```js
var value = web3.fromDecimal('21');
console.log(value); // "0x15"
```

***

#### web3.fromWei

    web3.fromWei(number, unit)

Converte uma quantidade de wei nas seguintes unidades da ethereum:

- `kwei`/`ada`
- `mwei`/`babbage`
- `gwei`/`shannon`
- `szabo`
- `finney`
- `ether`
- `kether`/`grand`/`einstein`
- `mether`
- `gether`
- `tether`

##### Parâmetros

1. `Number|String|BigNumber` - Um número ou uma instância BigNumber.
2. `String` - Umas das unidades de ether acima.


##### Retorna

`String|BigNumber` - Uma *string* de número, ou uma instância BigNumber, dependendo do parâmetro `number` dado.

##### Exemplo

```js
var value = web3.fromWei('21000000000000', 'finney');
console.log(value); // "0.021"
```

***

#### web3.toWei

    web3.toWei(number, unit)

Converte uma unidade ethereum em wei. Possíveis unidades são:

- `kwei`/`ada`
- `mwei`/`babbage`
- `gwei`/`shannon`
- `szabo`
- `finney`
- `ether`
- `kether`/`grand`/`einstein`
- `mether`
- `gether`
- `tether`

##### Parâmetros

1. `Number|String|BigNumber` - Um número ou uma instância BigNumber.
2. `String` - Uma das unidades de ether acima.

##### Retorna

`String|BigNumber` - Uma *string* de número, ou uma instância BigNumber, dependendo do parâmetro `number` dado.

##### Exemplo

```js
var value = web3.toWei('1', 'ether');
console.log(value); // "1000000000000000000"
```

***

#### web3.toBigNumber

    web3.toBigNumber(numberOrHexString);

Converte um dado número em uma instância BigNumber.

Veja a [nota sobre BigNumber](#a-note-on-big-numbers-in-web3js).

##### Parâmetros

1. `Number|String` - Um número, *string* de número ou *string* em HEX de um número.


##### Retorna

`BigNumber` - Uma instância BigNumber representando o dado valor.


##### Exemplo

```js
var value = web3.toBigNumber('200000000000000000000001');
console.log(value); // instanceOf BigNumber
console.log(value.toNumber()); // 2.0000000000000002e+23
console.log(value.toString(10)); // '200000000000000000000001'
```

***

### web3.net

#### web3.net.listening

    web3.net.listening
    // or async
    web3.net.getListening(callback(error, result){ ... })

Essa propriedade só pode ser lida (*read only*) e diz se um nó está escutando ativamente por conexões de rede ou não.

##### Retorna

`Boolean` - `true` se o *client* está ativamente escutando por conexões de rede, caso contrário `false`.

##### Exemplo

```js
var listening = web3.net.listening;
console.log(listening); // true ou false
```

***

#### web3.net.peerCount

    web3.net.peerCount
    // or async
    web3.net.getPeerCount(callback(error, result){ ... })

Essa propriedade só pode ser lida (*read only*) e retorna o número de *peers* conectados.

##### Retorna

`Number` - O número de *peers* conectados atualmente ao *client*.

##### Exemplo

```js
var peerCount = web3.net.peerCount;
console.log(*peer*Count); // 4
```

***

### web3.eth

Contém os métodos relacionados à blockchain da ethereum.

##### Exemplo

```js
var eth = web3.eth;
```

***

#### web3.eth.defaultAccount

    web3.eth.defaultAccount

Esse endereço *default* é usado para os seguintes métodos (opcionalmente você pode sobrescrevê-lo especificando a propriedade `from`):

- [web3.eth.sendTransaction()](#web3ethsendtransaction)
- [web3.eth.call()](#web3ethcall)

##### Valores

`String`, 20 Bytes - Qualquer endereço que você possua, ou que você tenha a chave privada. ~~Any address you own, or where you have the private key for.~~

*Default é* `undefined`.

##### Retorna

`String`, 20 Bytes - O endereço definido atualmente como *default*.

##### Exemplo

```js
var defaultAccount = web3.eth.defaultAccount;
console.log(defaultAccount); // ''

// define a conta default
web3.eth.defaultAccount = '0x8888f1f195afa192cfee860698584c030f4c9db1';
```

***

#### web3.eth.defaultBlock

    web3.eth.defaultBlock

Esse bloco *default* é usado para os seguintes métodos (opcionalmente você pode sobrescrevê-lo passando o parâmetro defaultBlock):

- [web3.eth.getBalance()](#web3ethgetbalance)
- [web3.eth.getCode()](#web3ethgetcode)
- [web3.eth.getTransactionCount()](#web3ethgettransactioncount)
- [web3.eth.getStorageAt()](#web3ethgetstorageat)
- [web3.eth.call()](#web3ethcall)
- [contract.myMethod.call()](#contract-methods)
- [contract.myMethod.estimateGas()](#contract-methods)

##### Valores

Os parâmetros *default* do bloco podem ser um dos seguintes:

- `Number` - um número de bloco
- `String` - `"earliest"`, o bloco gênesis
- `String` - `"latest"`, o último bloco (atual cabeçalho da blockchain)
- `String` - `"pending"`, o bloco recentemente minerado (incluindo transações pendentes)

*Default é* `latest`

##### Retorna

`Number|String` - O número de bloco *default* quando consultando um estado.

##### Exemplo

```js
var defaultBlock = web3.eth.defaultBlock;
console.log(defaultBlock); // 'latest'

// set the default block
web3.eth.defaultBlock = 231;
```

***

#### web3.eth.syncing

    web3.eth.syncing
    // or async
    web3.eth.getSyncing(callback(error, result){ ... })

Essa propriedade só pode ser lida (*read only*) e retorna um objeto sync quando o nó está sincronizando ou `false`.

##### Retorna

`Object|Boolean` - Um objeto sync como, como segue, quando o nó está atualmente sincronizando, ou `false`:
   - `startingBlock`: `Number` - O número de bloco onde a sincronização começou.
   - `currentBlock`: `Number` - O número de bloco até onde o nó sincronizou até o momento.
   - `highestBlock`: `Number` - O número de bloco estimado até onde sincronizará.

##### Exemplo

```js
var sync = web3.eth.syncing;
console.log(sync);
/*
{
   startingBlock: 300,
   currentBlock: 312,
   highestBlock: 512
}
*/
```

***

#### web3.eth.isSyncing

    web3.eth.isSyncing(callback);

Essa função de conveniência chama o `callback` todas as vezes que uma sincronização começa, atualiza e pára.

##### Retorna

`Object` - um objeto isSyncing com os seguintes métodos:

  * `syncing.addCallback()`: Adiciona outro callback, que será inciado quando o nó começar ou parar de sincronizar.
  * `syncing.stopWatching()`: Pára os callback de sincronização.

##### Valor de retorno do callback

- `Boolean` - O callback será ativado com `true` quando a sincronização começar e com `false`quando ela houver terminado.
- `Object` - Enquanto sincronizando, irá retornar o objeto de sincronização:
   - `startingBlock`: `Number` - O número de bloco onde a sincronização começou.
   - `currentBlock`: `Number` - O número de bloco até onde o nó sincronizou até o momento.
   - `highestBlock`: `Number` - O número de bloco estimado até onde sincronizará.


##### Exemplo

```js
web3.eth.isSyncing(function(error, sync){
    if(!error) {
        // pára todas as atividades de app
        if(sync === true) {
           // nós usamos `true`, então pára todos os filtros, mas não o polling web3.eth.syncing
           web3.reset(true);
        
        // mostra informações da sincronização
        } else if(sync) {
           console.log(sync.currentBlock);
        
        // retoma as opações de app
        } else {
            // roda sua função init do app...
        }
    }
});
```

***

#### web3.eth.coinbase

    web3.eth.coinbase
    // or async
    web3.eth.getCoinbase(callback(error, result){ ... })

Essa propriedade é apenas para leitura (*read only*) e retorno o endereço *coinbase* para onde as recompensas de mineração vão.

##### Retorna

`String` - O endereço *coinbase* do *client*.

##### Exemplo

```js
var coinbase = web3.eth.coinbase;
console.log(coinbase); // "0x407d73d8a49eeb85d32cf465507dd71d507100c1"
```

***

#### web3.eth.mining

    web3.eth.mining
    // or async
    web3.eth.getMining(callback(error, result){ ... })

Essa propriedade é apenas para leitura (*read only*) e diz se o nó está minerando ou não.

##### Retorna

`Boolean` - `true` se o *client* está minerando, caso contrário `false`.

##### Exemplo

```js
var mining = web3.eth.mining;
console.log(mining); // true ou false
```

***

#### web3.eth.hashrate

    web3.eth.hashrate
    // or async
    web3.eth.getHashrate(callback(error, result){ ... })

Essa propriedade é apenas para leitura (*read only*) e retorna o número de *hashes* por segundo com os quais o nó está minerando.

##### Retorna

`Number` - número de *hashes* por segundo.

##### Exemplo

```js
var hashrate = web3.eth.hashrate;
console.log(*hash*rate); // 493736
```

***

#### web3.eth.gasPrice

    web3.eth.gasPrice
    // or async
    web3.eth.getGasPrice(callback(error, result){ ... })

Essa propriedade é apenas para leitura (*read only*) e retorna o preço atual de gas.
O preço de gas é determinado pelo preço médio nos últimos últimos x blocos.

##### Retorna

`BigNumber` - Uma instância de BigNumber do preço atual de gas em wei.

Veja a [nota sobre BigNumber](#a-note-on-big-numbers-in-javascript).

##### Exemplo

```js
var gasPrice = web3.eth.gasPrice;
console.log(gasPrice.toString(10)); // "10000000000000"
```

***

#### web3.eth.accounts

    web3.eth.accounts
    // or async
    web3.eth.getAccounts(callback(error, result){ ... })

Essa propriedade é apenas para leitura (*read only*) e retorna uma lista de contas que o nó controla.

##### Retorna

`Array` - Uma *array* de endereços controlados pelo *client*.

##### Exemplo

```js
var accounts = web3.eth.accounts;
console.log(accounts); // ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"] 
```

***

#### web3.eth.blockNumber

    web3.eth.blockNumber
    // or async
    web3.eth.getBlockNumber(callback(error, result){ ... })

Essa propriedade é apenas para leitura (*read only*) e retorna o atual número de bloco.

##### Retorna

`Number` - O número do bloco mais recente.

##### Exemplo

```js
var number = web3.eth.blockNumber;
console.log(number); // 2744
```

***

#### web3.eth.register

    web3.eth.register(addressHexString [, callback])

(Ainda não implementada)
Registra o dado endereço a ser incluído em `web3.eth.accounts`. Isso permite contas que não possuem a chave privada serem associadas como contas possuídas. (e.g., carteiras de contrato).

##### Parâmetros

1. `String` - O endereço para registrar.
2. `Function` - (opcional) Se você passar um callback a requisição HTTP se transforma em assíncrona. Veja [esta nota](#using-callbacks) para detalhes.


##### Retorna

?


##### Exemplo

```js
web3.eth.register("0x407d73d8a49eeb85d32cf465507dd71d507100ca")
```

***

#### web3.eth.unRegister

     web3.eth.unRegister(addressHexString [, callback])


(Ainda não implementada)
Anula o registro de um dado endereço.

##### Parâmetros

1. `String` - O endereço a ser desregistrado.
2. `Function` - (opcional) Se você passar um callback a requisição HTTP se transforma em assíncrona. Veja [esta nota](#using-callbacks) para detalhes.


##### Retorna

?


##### Exemplo

```js
web3.eth.unregister("0x407d73d8a49eeb85d32cf465507dd71d507100ca")
```

***

#### web3.eth.getBalance

    web3.eth.getBalance(addressHexString [, defaultBlock] [, callback])

Pega o balanço de um endereo em um dado bloco.

##### Parâmetros

1. `String` - O endereço de onde o balanço será pego.
2. `Number|String` - (opcional) Se você passar esse parâmetro, ele não usará o bloco *default* definids por [web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (opcional) Se você passar um *callback*, a requisição HTTP será assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - Uma instância BigNumber do atual balanço para o dado endereço em wei.

Veja a [nota sobre BigNumber](#a-note-on-big-numbers-in-javascript).

##### Exemplo

```js
var balance = web3.eth.getBalance("0x407d73d8a49eeb85d32cf465507dd71d507100c1");
console.log(balance); // instanceof BigNumber
console.log(balance.toString(10)); // '1000000000000'
console.log(balance.toNumber()); // 1000000000000
```

***

#### web3.eth.getStorageAt

    web3.eth.getStorageAt(addressHexString, position [, defaultBlock] [, callback])

Pega o armazenamento em uma posição específica de um endereço.

##### Parâmetros

1. `String` - O endereço de onde pegar o armazenamento.
2. `Number` - O índice de posição do armazenamento.
3. `Number|String` - (opcional) Se você passar esse parâmetro, ele não usará o bloco *default* definids por [web3.eth.defaultBlock](#web3ethdefaultblock).
4. `Function` - (opcional) Se você passar um *callback*, a requisição HTTP será assíncrona. Veja [esta nota](#using-callbacks) para detalhes.


##### Retorna

`String` - O valor armazenado em uma dada posição.

##### Exemplo

```js
var state = web3.eth.getStorageAt("0x407d73d8a49eeb85d32cf465507dd71d507100c1", 0);
console.log(state); // "0x03"
```

***

#### web3.eth.getCode

    web3.eth.getCode(addressHexString [, defaultBlock] [, callback])

Pega o código em um endereço específico.

##### Parâmetros

1. `String` - o endereço de onde pegar o código.
2. `Number|String` - (opcional) Se você passar esse parâmetro, ele não usará o bloco *default* definids por [web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (opcional) Se você passar um *callback*, a requisição HTTP será assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - Os dados em um dado endereço `addressHexString`.

##### Exemplo

```js
var code = web3.eth.getCode("0xd5677cf67b5aa051bb40496e68ad359eb97cfbf8");
console.log(code); // "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b600060078202905091905056"
```

***

#### web3.eth.getBlock

     web3.eth.getBlock(blockHashOrBlockNumber [, returnTransactionObjects] [, callback])

Retorna um bloco coincidindo com o número de bloco ou o *hash* do bloco.

##### Parâmetros

1. `String|Number` - O número de bloco ou *hash*. Ou a *string* `"earliest"`, `"latest"` ou `"pending"` como no [parâmetro default do bloco](#web3ethdefaultblock).
2. `Boolean` - (opcional, *default *`false`) Se `true`, o bloco retornado conterá todas as transações como objetos, se `false` ele conterá apenas os *hashes* de transação.
3. `Function` - (opcional) Se você passar um *callback*, a requisição HTTP será assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Object` - O objeto de bloco:

  - `number`: `Number` - o número de bloco. `null` quando o bloco está pendente.
  - `*hash*`: `String`, 32 Bytes - *hash* do bloco.`null` quando o bloco está pendente.
  - `parentHash`: `String`, 32 Bytes - *hash* do bloco anterior (bloco pai).
  - `nonce`: `String`, 8 Bytes - *hash* do *proof-of-work* gerado. `null` quando o bloco está pendente
  - `sha3Uncles`: `String`, 32 Bytes - SHA3 dos dados nos blocos *uncle*.
  - `logsBloom`: `String`, 256 Bytes - o filtro de bloom para os logs do bloco. `null` quando o bloco esta pendente
  - `transactionsRoot`: `String`, 32 Bytes - a raiz da árvore de transações do bloco.
  - `stateRoot`: `String`, 32 Bytes - a raiz do estado final da árvore do bloco.
  - `miner`: `String`, 20 Bytes - o endereço do benefeciário a quem a recompensa da mineração foi dada.
  - `difficulty`: `BigNumber` - *integer* da dificuldade para este bloco.
  - `totalDifficulty`: `BigNumber` - integer da dificuldade  total desta corrente (*chain*) até este bloco.
  - `extraData`: `String` - o campo "extra data" deste bloco.
  - `size`: `Number` - *integer* com o tamanho deste bloco em *bytes*.
  - `gasLimit`: `Number` - o máximo gas permitido neste bloco.
  - `gasUsed`: `Number` - o gas total utilizado por todas as transações até este bloco.
  - `timestamp`: `Number` - o *timestamp* unix para quando o bloco foi coligido.
  - `transactions`: `Array` - *Array* dos objetos da transação, ou *hashes* de 32 *bytes* da transação dependendo do último parâmetro passado.
  - `uncles`: `Array` - *Array* de *hashes* de blocos *uncles*.

##### Exemplo

```js
var info = web3.eth.getBlock(3150);
console.log(info);
/*
{
  "number": 3,
  "*hash*": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
  "parentHash": "0x2302e1c0b972d00932deb5dab9eb2982f570597d9d42504c05d9c2147eaf9c88",
  "nonce": "0xfb6e1a62d119228b",
  "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "transactionsRoot": "0x3a1b03875115b79539e5bd33fb00d8f7b7cd61929d5a3c574f507b8acf415bee",
  "stateRoot": "0xf1133199d44695dfa8fd1bcfe424d82854b5cebef75bddd7e40ea94cda515bcb",
  "miner": "0x8888f1f195afa192cfee860698584c030f4c9db1",
  "difficulty": BigNumber,
  "totalDifficulty": BigNumber,
  "size": 616,
  "extraData": "0x",
  "gasLimit": 3141592,
  "gasUsed": 21662,
  "timestamp": 1429287689,
  "transactions": [
    "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b"
  ],
  "uncles": []
}
*/
```

***

#### web3.eth.getBlockTransactionCount

    web3.eth.getBlockTransactionCount(*hash*StringOrBlockNumber [, callback])

Retorna o número de transações em um dado bloco.

##### Parâmetros

1. `String|Number` - O número do bloco ou *hash*. Ou a *string* `"earliest"`, `"latest"` ou `"pending"` como no [parâmetro default block](#web3ethdefaultblock).
2. `Function` - (opcional) Se você passa um *callback*, a requisição HTTP é feita assíncronamente. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Number` - O número de transações no dado bloco.

##### Exemplo

```js
var number = web3.eth.getBlockTransactionCount("0x407d73d8a49eeb85d32cf465507dd71d507100c1");
console.log(number); // 1
```

***

#### web3.eth.getUncle

    web3.eth.getUncle(blockHashStringOrNumber, uncleNumber [, returnTransactionObjects] [, callback])

Retorna um uncle de blocos de acodo com a o índice de posição.

##### Parâmetros

  1. `String|Number` - O númeo do bloco ou *hash*. Ou a *string* `"earliest"`, `"latest"` ou `"pending"` como no [parâmetro default block](#web3ethdefaultblock).
2. `Number` - A posição índice do uncle.
3. `Boolean` - (opcional, padrão `false`) Se `true`, o bloco retornado conterá todas transações como objetos, se `false` conterá os *hashes* da transação.
4. `Function` - (opcional) Se você passa um *callback*, a requisição HTTP é feita assíncronamente. Veja [esta nota](#using-callbacks) para detalhes.


##### Retorna

`Object` - O uncle retornado. Para um valor do retorno, veja [web3.eth.getBlock()](#web3ethgetblock).

**Nota**: Um uncle não contém transações individuais.

##### Exemplo

```js
var uncle = web3.eth.getUncle(500, 0);
console.log(uncle); // see web3.eth.getBlock

```

***

##### web3.eth.getTransaction

    web3.eth.getTransaction(transactionHash [, callback])

Retorna uma transação de acordo com o *hash* de transação dado.

##### Parâmetros

1. `String` - O *hash* da transação.
2. `Function` - (opcional) Se você passa um *callback*, a requisição HTTP é feita assíncronamente. Veja [esta nota](#using-callbacks) para detalhes.


##### Retorna

`Object` - Um objeto transação:

  - `*hash*`: `String`, 32 *Bytes* - *hash* da transação.
  - `nonce`: `Number` - o número de transações anteriores a esta feitas pelo enviador desta transação.
  - `blockHash`: `String`, 32 Bytes - *hash* do bloco onde esta transação estava. `null` quando está pendente.
  - `blockNumber`: `Number` - número do bloco onde esta transação estava. `null` quando está pendente.
  - `transactionIndex`: `Number` - *integer* da com o índice da posição da transação no bloco. `null` quando está pendente.
  - `from`: `String`, 20 Bytes - endereço do enviador da transação.
  - `to`: `String`, 20 Bytes - endereço do recebedor da transação. `null` quando é a criação de um contrato.
  - `value`: `BigNumber` - valor transferido em Wei.
  - `gasPrice`: `BigNumber` - preço do gas provido pelo enviador em Wei.
  - `gas`: `Number` - gas provido pelo enviador.
  - `input`: `String` - os dados enviados junto com a transação.


##### Exemplo

```js
var blockNumber = 668;
var indexOfTransaction = 0

var transaction = web3.eth.getTransaction(blockNumber, indexOfTransaction);
console.log(transaction);
/*
{
  "*hash*": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
  "nonce": 2,
  "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
  "blockNumber": 3,
  "transactionIndex": 0,
  "from": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
  "to": "0x6295ee1b4f6dd65047762f924ecd367c17eabf8f",
  "value": BigNumber,
  "gas": 314159,
  "gasPrice": BigNumber,
  "input": "0x57cb2fc4"
}
*/

```

***

#### web3.eth.getTransactionFromBlock

    getTransactionFromBlock(*hash*StringOrNumber, indexNumber [, callback])

Retorna uma transação baseada no *hash* ou número do bloco e o indíce da posição das transações.

##### Parâmetros

1. `String` - Um número ou *hash* de bloco. O a *string* `"earlisest"`, `"latest"` ou `"pending"` como no [parâmetro default block](#web3ethdefaultblock).
1. `Number` - O indíce da posição das transações.
2. `Number` - The transactions index position.
3. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona.

##### Retorna

`Object` - Um objeto de transação, veja [web3.eth.getTransaction](#web3ethgettransaction):


##### Exemplo

```js
var transaction = web3.eth.getTransactionFromBlock('0x4534534534', 2);
console.log(transaction); // see web3.eth.getTransaction

```

***

#### web3.eth.getTransactionReceipt

    web3.eth.getTransactionReceipt(*hash*String [, callback])

Retorna o recibo da transação de acordo com o *hash* da transação.

**Note** Recibo não disponível para transações pendentes.


##### Parâmetros

1. `String` - O *hash* da transação.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona.

##### Retorna

`Object` - Um objeto de recibo de transação, ou `null` quando nenhum recibo for achado:

  - `blockHash`: `String`, 32 Bytes - *hash* do bloco onde esta transação está.
  - `blockNumber`: `Number` - número do bloco onde está transação está.
  - `transactionHash`: `String`, 32 Bytes - *hash* da transação.
  - `transactionIndex`: `Number` - integer do índice de posição das transações no bloco.
  - `from`: `String`, 20 Bytes - endereço de quem enviou a transação.
  - `to`: `String`, 20 Bytes - endereço de quem recebe a transação. `null` em caso de ser uma t transação de criação de contrato.
  - `cumulativeGasUsed `: `Number ` - A quantidade total de gas usado quando esta transação foi executada no bloco.
  - `gasUsed `: `Number ` -  A quantidade de gas usado por esta transação específica sozinha.
  - `contractAddress `: `String` - 20 Bytes - O endereço do contrato criado, se a transação foi uma criação de contrato, em outros casos `null`
  - `logs `:  `Array` - Array de objetos de log, que esta transação gerou.

##### Exemplo
```js
var receipt = web3.eth.getTransactionReceipt('0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b');
console.log(receipt);
{
  "transactionHash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
  "transactionIndex": 0,
  "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
  "blockNumber": 3,
  "contractAddress": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
  "cumulativeGasUsed": 314159,
  "gasUsed": 30234,
  "logs": [{
         // logs as returned by getFilterLogs, etc.
     }, ...]
}
```

***

#### web3.eth.getTransactionCount

    web3.eth.getTransactionCount(addressHexString [, defaultBlock] [, callback])

Pega o número de transações enviadas por este endereço.

##### Parâmetros

1. `String` - O endereço de qual as transações devem ser pegas.
2. `Number|String` - (opcional) Se você passar este parâmetro, o bloco default não será configurado com web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Number` - O número de transações enviadas pelo endereço informado.

##### Exemplo

```js
var number = web3.eth.getTransactionCount("0x407d73d8a49eeb85d32cf465507dd71d507100c1");
console.log(number); // 1
```

***

#### web3.eth.sendTransaction

    web3.eth.sendTransaction(transactionObject [, callback])

Envia uma transação para a rede.

##### Parâmetros

1. `Object` - O objeto da transação a ser enviada:
  - `from`: `String` - O endereço para a conta enviando. Usa a propriedadeThe [web3.eth.defaultAccount](#web3ethdefaultaccount), se não especificada.
  - `to`: `String` - (opcional) O endereço de destino da mensagem, deixado indefinido para uma transação de criação de contrato.
  - `value`: `Number|String|BigNumber` - (opcional) O valor transferido para a transação em Wei, também a doação se esta for uma transação de criação de contrato.
  - `gas`: `Number|String|BigNumber` - (opcional, padrão: *To-Be-Determined*) A quantidade de gas para uso pela transação (gas não usado é restituído).
  - `gasPrice`: `Number|String|BigNumber` - (opcional, padrão: *To-Be-Determined*) O preço do gas para esta transação em  Wei, padrão significará o preço de gas da rede.
  - `data`: `String` - (opcional) Ou uma [*byte string*](https://github.com/ethereum/wiki/wiki/Solidity,-Docs-and-ABI) (optional) Either a [byte *string*](https://github.com/ethereum/wiki/wiki/Solidity,-Docs-and-ABI) contendo o dado associado da mensagem, ou em caso de uma transação de criação de contrato, a inicialização do código.
  - `nonce`: `Number`  - (opcional) Integer de um *nonce*. Isso permite sobrescrever suas próprias transações pendentes que usam o mesmo *nonce*.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - The 32 Bytes transaction *hash* as HEX *string*.

Se a transação foi uma criação de contrato use [web3.eth.getTransactionReceipt()](#web3gettransactionreceipt) para pegar o enderço do contato, após a transação ter sido minerada.

##### Exemplo

```js

// código-fonte solidity compilado usando https://chriseth.github.io/cpp-ethereum/
var code = "603d80600c6000396000f3007c01000000000000000000000000000000000000000000000000000000006000350463c6888fa18114602d57005b600760043502
8060005260206000f3";

web3.eth.sendTransaction({data: code}, function(err, address) {
  if (!err)
    console.log(address); // "0x7f9fade1c0d57a7af66ab4ead7c2eb7b11a91385"
});
```

***

#### web3.eth.sendRawTransaction

    web3.eth.sendRawTransaction(signedTransactionData [, callback])

Envia uma transação já assinada. Por exemplo, pode ser assinada usando: https://github.com/SilentCicero/ethereumjs-accounts

##### Parâmetros

1. `String` - Transação assinada em formato HEX. 
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - Os 32 Bytes do *hash* da transação como uma HEX *string*.

Se a transação foi uma criação de contrato use [web3.eth.getTransactionReceipt()](#web3gettransactionreceipt) para pegar o endereço do contrato, após a transação ter sido minerada.

##### Exemplo

```js
var Tx = require('ethereumjs-tx');
var privateKey = new Buffer('e331b6d69882b4cb4ea581d88e0b604039a3de5967688d3dcffdd2270c0fd109', 'hex')

var rawTx = {
  nonce: '0x00',
  gasPrice: '0x09184e72a000', 
  gasLimit: '0x2710',
  to: '0x0000000000000000000000000000000000000000', 
  value: '0x00', 
  data: '0x7f7465737432000000000000000000000000000000000000000000000000000000600057'
}

var tx = new Tx(rawTx);
tx.sign(privateKey);

var serializedTx = tx.serialize();

//console.log(serializedTx.toString('hex'));
//0xf889808609184e72a00082271094000000000000000000000000000000000000000080a47f74657374320000000000000000000000000000000000000000000000000000006000571ca08a8bbf888cfa37bbf0bb965423625641fc956967b81d12e23709cead01446075a01ce999b56a8a88504be365442ea61239198e23d1fce7d00fcfc5cd3b44b7215f

web3.eth.sendRawTransaction(serializedTx.toString('hex'), function(err, *hash*) {
  if (!err)
    console.log(*hash*); // "0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385"
});
```

***


#### web3.eth.sign

    web3.eth.sign(address, dataToSign, [, callback])

Assina os dados de uma conta específica. Esta conta precisa estar destrancada.

##### Parâmetros

1. `String` - Endereço que será usado para assinatura.
2. `String` - Dados para assinar.
3. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - Os dados assinados.

Após o prefixo hex, carácteres correspondem aos valores ECDSA assim:
```
r = signature[0:64]
s = signature[64:128]
v = signature[128:130]
```

##### Exemplo

```js
var result = web3.eth.sign("0x135a7de83802408321b74c322f8558db1679ac20",
    "0x9dd2c369a187b4e6b9c402f030e50743e619301ea62aa4c0737d4ef7e10a3d49"); // segundo argumento é web3.sha3("xyz")
console.log(result); // "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400"
```

***

#### web3.eth.call

    web3.eth.call(callObject [, defaultBlock] [, callback])

Executa uma transação de chamada de mensagem, que é diretamente executada na VM do nó, mas nunca minerada para dentro da blockchain.

##### Parâmetros

1. `Object` - Um objeto de transação ,veja [web3.eth.sendTransaction](#web3ethsendtransaction), com a diferença de que para chamadas a propriedade `from` também é opcional.
2. `Number|String` - (opcional) Se você passar este parâmetro, o bloco default não será confiurado com [web3.eth.defaultBlock](#web3ethdefaultblock).
3. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - Os dados retornados da chamada, e.g. o valor de retorno de uma funcão de códigos. 

##### Exemplo

```js
var result = web3.eth.call({
    to: "0xc4abd0339eb8d57087278718986382264244252f", 
    data: "0xc6888fa10000000000000000000000000000000000000000000000000000000000000003"
});
console.log(result); // "0x0000000000000000000000000000000000000000000000000000000000000015"
```

***

#### web3.eth.estimateGas

    web3.eth.estimateGas(callObject [, callback])

Executa uma chamada de mensagem ou transação, que é diretamente executada na VM do nó, mas nunca minerada para dentro da blockchain e retorna a quantidade de gas utilizada.

##### Parâmetros

Veja [web3.eth.sendTransaction](#web3ethsendtransaction), exceto que todas as propriedades são opcionais.

##### Retorna

`Number` - O gas usado para a chamada/transação simulada.

##### Exemplo

```js
var result = web3.eth.estimateGas({
    to: "0xc4abd0339eb8d57087278718986382264244252f", 
    data: "0xc6888fa10000000000000000000000000000000000000000000000000000000000000003"
});
console.log(result); // "0x0000000000000000000000000000000000000000000000000000000000000015"
```

***

#### web3.eth.filter

```js
// pode ser 'latest' ou 'pending'
var filter = web3.eth.filter(filterString);
// OU objetos são opções de filtros de log
var filter = web3.eth.filter(options);

// assista por mudanças
filter.watch(function(error, result){
  if (!error)
    console.log(result);
});

// Adicionalmente você pode começar a assistir na mesma hora, passando um callback:
web3.eth.filter(options, function(error, result){
  if (!error)
    console.log(result);
});
```

##### Parâmetros

1. `String|Object` - A *string* `"latest"` ou `"pending"` para assistir por mudanças no último bloco ou transação pendente respectivamente. Ou um objeto de opções de filtros de log como a seguir:
  * `fromBlock`: `Number|String` - O número do bloco menos recente (`latest` pode ser passado para representar o bloco mais recente e `pending` para um bloco sendo minerado).
  * `toBlock`: `Number|String` - O número do último bloco. (`latest` pode ser passado para representar o bloco mais recente e `pending` para um bloco sendo minerado).Por padrão: `latest`.
  * `address`: `String` - Um endereço ou lista de endereços para pegar apenas logs de conta(s) em particular.
  * `topics`: `Array of Strings` - Uma *array* de valores que devem aparecer nas entradas de log. A ordem é importante, se você quiser deixar os `topics` de fora use `null`, e.g. `[null, '0x00...']`. Você também pode pasar uma outra *array* para cada tópico com opções para aquele tópico e.g. `[null, ['option1', 'option2']]`

##### Retorna

`Object` - Um objeto de filtro com os seguintes métodos:

  * `filter.get(callback)`: Retorna todas as entradas de log que caírem no filtro.
  * `filter.watch(callback)`: Assiste pelas mudanças de estato que caírem no filtro e as chamadas do *callback*. Veja [esta nota](#using-callbacks) para detalhes.
  * `filter.stopWatching()`: Para de assistir e desinstala o filtro no nó. Deve sempre ser chamada quando terminado.

##### Watch callback return value

- `String` - Quando estiver usando o parâmetro `"latest"`, retorna o *hash* do bloco do último bloco entrando.
- `String` - Quando estiver usando o parâmetro `"pending"`, retorna o *hash* da transação da última transação pendente.
- `Object` - Quando estiver usando opções de filtro manuais, retorna o objeto do log como a seguir:
    - `logIndex`: `Number` - integer do indíce da posição do log. `null` quando o log está pendente.
    - `transactionIndex`: `Number` - integer do índice de posição da transação de onde o log foi criado. `null` quando o log está pendente.
    - `transactionHash`: `String`, 32 Bytes - *hash* das transações de onde este log foi criado. `null` quando o log está pendente.
    - `blockHash`: `String`, 32 Bytes - *hash* do bloco onde o log está, `null` quando está pendente. `null` quando o log está pendente.
    - `blockNumber`: `Number` - the block number where this log was in. `null` when its pending. `null` when its pending log.
    - `address`: `String`, 32 Bytes - endreço de onde o log foi originado.
    - `data`: `String` - contém um ou mais argumentos não indexados de 32 *Bytes* do log.
    - `topics`: `Array of Strings` - *Array* de 0 a 4 32 *bytes* `DATA` de argumentos de log indexados. (Em *solidity*: O primeiro tópico é o *hash* da assinatura do evento (e.g. `Deposit(address,bytes32,uint256)`), exceto se você tiver declarado o evento com o especificador `anonymous`.)

**Nota** Para o retorno de filtro de evento veja [Eventos de Contrato](#contract-events)

##### Exemplo

```js
var filter = web3.eth.filter('pending');

filter.watch(function (error, log) {
  console.log(log); //  {"address":"0x0000000000000000000000000000000000000000", "data":"0x0000000000000000000000000000000000000000000000000000000000000000", ...}
});

// pega todos os logs passados novamente.
var myResultados = filter.get(function(error, logs){ ... });

...

// para e desinstala o filtro.
filter.stopWatching();

```

***

#### web3.eth.contract

    web3.eth.contract(abiArray)

Cria um objeto de contrato para um contrato solidity, que pode ser usado para iniciar contratos no endereço.
Você pode ler mais sobre eventos [aqui](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#exemplo-javascript-usage).

##### Parâmetros

1. `Array` - array ABI com descrição das funções e eventos do contrato.

##### Retorna

`Object` - Um ojeto de contrato, que pode ser iniciado como a seguir:

```js
var MyContract = web3.eth.contract(abiArray);

// instanciado pelo endereço
var contractInstance = MyContract.at([address]);

// deploy novo contrato
var contractInstance = MyContract.new([contructorParam1] [, contructorParam2], {data: '0x12345...', from: myAccount, gas: 1000000});

// Pega os dados manualmene para deployar o contrato manualmente
var contractData = MyContract.new.getData([contructorParam1] [, contructorParam2], {data: '0x12345...'});
// contractData = '0x12345643213456000000000023434234'
```

E então você pode tanto iniciar um contrato existente num endereço, ou implantar o contrato usando o *byte code* compilado:

```js
// Instancia a partir de um endereço existente:
var myContractInstance = MyContract.at(myContractAddress);


// Ou implanta um novo contrato:

// Implanta o contrato de forma assíncrona:
var myContractReturned = MyContract.new(param1, param2, {
   data: myContractCode,
   gas: 300000,
   from: mySenderAddress}, function(err, myContract){
    if(!err) {
       // NOTE: The callback will fire twice!
       // NOTA: O callback irá ser ativado duas vezes!
       // Assim que o contrato tiver a propriedade transactionHash configurada e assim que for implantado num endereço.
       // e.g. cheque tx *hash* na primeira chamada (transaction send)
       if(!myContract.address) {
           console.log(myContract.transactionHash) // The *hash* of the transaction, which deploys the contract
       
       // cheque endereço em segunda chamada (contrato implantado)
       } else {
           console.log(myContract.address) // o endereço do contrato
       }

       // Note que o "myContractReturned" retornado === "myContract",
       // Então o objeto "myContractReturned" retornado também terá o endereço configurado.
    }
  });

// Implant contrato síncrono: O endereço será adicionado assim que o contrato for minerado.
// Adicionalmente você pode assistir a transação usando a propriedade "transactionHash"
var myContractInstance = MyContract.new(param1, param2, {data: myContractCode, gas: 300000, from: mySenderAddress});
myContractInstance.transactionHash // O *hash* da transação que criou o contrato
myContractInstance.address // undefined no começo, mas será auto-preenchida mais tarde
```

**Note** Quando você implanta um novo contrato, você deveria cehcar pelos próximos 12 blocos aproximadamente se o código do contrato ainda está no endereço (usando [web3.eth.getCode()](#web3ethgetcode)), para ter certeza de que um fork não mudou isso.

##### Exemplo

```js
// abi do contrato
var abi = [{
     name: 'myConstantMethod',
     type: 'function',
     constant: true,
     inputs: [{ name: 'a', type: '*string*' }],
     outputs: [{name: 'd', type: '*string*' }]
}, {
     name: 'myStateChangingMethod',
     type: 'function',
     constant: false,
     inputs: [{ name: 'a', type: '*string*' }, { name: 'b', type: 'int' }],
     outputs: []
}, {
     name: 'myEvent',
     type: 'event',
     inputs: [{name: 'a', type: 'int', indexed: true},{name: 'b', type: 'bool', indexed: false}]
}];

// criação do objeto de contrato
var MyContract = web3.eth.contract(abi);

// inicia o contrato com um endereço
var myContractInstance = MyContract.at('0xc4abd0339eb8d57087278718986382264244252f');

// chama a função constante
var result = myContractInstance.myConstantMethod('myParam');
console.log(result) // '0x25434534534'

// envia uma transação para uma função
myContractInstance.myStateChangingMethod('someParam1', 23, {value: 200, gas: 2000});

// estilo *short hand*
web3.eth.contract(abi).at(address).myAwesomeMethod(...);

// cria filtro
var filter = myContractInstance.myEvent({a: 5}, function (error, result) {
  if (!error)
    console.log(result);
    /*
    {
        address: '0x8718986382264244252fc4abd0339eb8d5708727',
        topics: "0x12345678901234567890123456789012", "0x0000000000000000000000000000000000000000000000000000000000000005",
        data: "0x0000000000000000000000000000000000000000000000000000000000000001",
        ...
    }
    */
});
```

***

#### Métodos de Contratos

```js
// Automaticamente determina o uso da chamada ou sendTransaction basado no tipo de método myContractInstance.myMethod(param1 [, param2, ...] [, transactionObject] [, defaultBlock] [, callback]);

// Explicitamente chamando este método
myContractInstance.myMethod.call(param1 [, param2, ...] [, transactionObject] [, defaultBlock] [, callback]);

// Explicitamente enviando a transação para este método
myContractInstance.myMethod.sendTransaction(param1 [, param2, ...] [, transactionObject] [, callback]);

// Pegue os dados da chamada, assim você pode chamar o contrto de outras formas
var myCallData = myContractInstance.myMethod.getData(param1 [, param2, ...]);
// myCallData = '0x45ff3ff6000000000004545345345345..'
```

The contract object exposes the contract's methods, which can be called using parâmetros and a transaction object.
O objeto de contrato expoem os métodos do contrato, que podem ser chamados usando parâmetros e um objeto de transação.

##### Parâmetros

- `String|Number` - (opcional) Zero ou mais parâmetros da função.
- `Object` - (opcional) O (anterior) último parâmetro pode ser um objeto de transação, veja o parâmetro [web3.eth.sendTransaction](#web3ethsendtransaction) 1 ou mais. **Note**: propriedades `data` e `to` não serão levadas em conta.
- `Number|String` - (opcional) Se você passar este parâmetro, o default block configurado com [web3.eth.defaultBlock](#web3ethdefaultblock) não será usado.
- `Function` - (opcional) Se você passar um *callback* como o último parâmetro a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Sting` - Se for uma chamada será os dados do resultado, se for um envio de transação retornará o endereço de contrao, ou o *hash* da transação, veja [web3.eth.sendTransaction](#web3ethsendtransaction) para detalhes.

##### Exemplo

```js
// criação do contrato do objeto
var MyContract = web3.eth.contract(abi);

// inicia contrato para um endereço
var myContractInstance = MyContract.at('0x78e97bcc5b5dd9ed228fed7a4887c0d7287344a9');

var result = myContractInstance.myConstantMethod('myParam');
console.log(result) // '0x25434534534'

myContractInstance.myStateChangingMethod('someParam1', 23, {value: 200, gas: 2000}, function(err, result){ ... });
```

***


#### Eventos de Contratos

```js
var event = myContractInstance.MyEvent({valueA: 23} [, additionalFilterObject])

// assiste por mudanças
event.watch(function(error, result){
  if (!error)
    console.log(result);
});

// Ou passa um callback para começar a assistir imediatamente
var event = myContractInstance.MyEvent([{valueA: 23}] [, additionalFilterObject] , function(error, result){
  if (!error)
    console.log(result);
});

```

Você pode usar eventos como [filters](#web3ethfilter) e eles tem os mesmos métodos, mas você deve passar objetos diferente pra criar o evento de filtro.

##### Parâmetros

1. `Object` - Valores retornados indexados pelos quais você quer filtrar os logs, e.g. `{'valueA': 1, 'valueB': [myFirstAddress, mySecondAddress]}`. Por padrão todos valores de filtro são configurados como `null`.
2. `Object` - Opcões adcionais de filtro, veja [filtros](#web3ethfilter) parâmetro 1 para mais. Por padrão o campo filterObject tem campo 'address' configurado como o endereco do contato. Também, primeiro tópico é a assinatura do evento.
3. `Function` - (opcional) Se você passar um *callback* como último parâmetro, comecará a assistir imediatamente e você não precisará chamar `myEvent.watch(function(){})`. Veja [esta nota](#using-callbacks) para detalhes.

##### Callback return


`Object` - Um objeto event como segue:

- `args`: `Object` - Os argumentos chegando do evento.
- `event`: `String` - O nome do evento.
- `logIndex`: `Number` - *integer* do índice de posicão do log no bloco.
- `transactionIndex`: `Number` - *integer* do indíce de posicão das transacões de onde o log foi criado.
- `transactionHash`: `String`, 32 Bytes - *hash* das transacões de onde este log foi criado.
- `address`: `String`, 32 Bytes - endereco de onde este log foi originado.
- `blockHash`: `String`, 32 Bytes - *hash* do bloco onde este log estava. `null`quando pendente.
- `blockNumber`: `Number` - O numero do bloco onde este log estava. `null` quando pendente.


##### Exemplo

```js
var MyContract = web3.eth.contract(abi);
var myContractInstance = MyContract.at('0x78e97bcc5b5dd9ed228fed7a4887c0d7287344a9');

// assista por um evento com {some: 'args}
var myEvent = myContractInstance.MyEvent({some: 'args'}, {fromBlock: 0, toBlock: 'latest'});
myEvent.watch(function(error, result){
   ...
});

// pegaria todos os logs passados
var myResultados = myEvent.get(function(error, logs){ ... });

...

// pararia de assistir e desinstalaria o filtro
myEvent.stopWatching();
```

***

#### Contract allEvents

```js
var events = myContractInstance.allEvents([additionalFilterObject]);

// assiste por mudancas
events.watch(function(error, event){
  if (!error)
    console.log(event);
});

// Ou passa um callback e comeca a assistir imediatamente
var events = myContractInstance.allEvents([additionalFilterObject,] function(error, log){
  if (!error)
    console.log(log);
});

```

Will call the callback for all events which are created by this contract.
Chamará o *callback* para todos eventos criados por este contrato.

##### Parâmetros

1. `Object` - Opcões de filtro adicionais, veja [filters](#web3ethfilter) parâmetro 1 para mais. Por padrão filterObject tem o campo 'address' configurado como o endereco do contrato. Este método configura o tópico para a assinatura do evento, e não suporta tópicos adicionais.
2. `Function` - (opcional) Se você passar um *callback* como último parâmetro, comecará a assistir imediatamente e não precisará chamar `myEvent.watch(function(){})`. Veja [esta nota](#using-callbacks) para detalhes.

##### Callback return


`Object` - Veja [Eventos de Contrato](#contract-events) para mais.

##### Exemplo

```js
var MyContract = web3.eth.contract(abi);
var myContractInstance = MyContract.at('0x78e97bcc5b5dd9ed228fed7a4887c0d7287344a9');

// watch for an event with {some: 'args'}
var events = myContractInstance.allEvents({fromBlock: 0, toBlock: 'latest'});
events.watch(function(error, result){
   ...
});

// pegaria todos os logs passados
events.get(function(error, logs){ ... });

...

// pararia e desintalaria o filtro
myEvent.stopWatching();
```

****

#### web3.eth.getCompilers

    web3.eth.getCompilers([callback])

Pega uma lista de compiladores disponíveis.

##### Parâmetros

1. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Array` - Uma array de *string*s com compiladores disponíveis.

##### Exemplo

```js
var number = web3.eth.getCompilers();
console.log(number); // ["lll", "solidity", "serpent"]
```

***

#### web3.eth.compile.solidity

    web3.eth.compile.solidity(sourceString [, callback])

Compilar o código-fonte solidity.

##### Parâmetros

1. `String` - O código-fonte solidity.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes..

##### Retorna

`Object` - Informacão do contrato e compilador.


##### Exemplo

```js
var source = "" + 
    "contract test {\n" +
    "   function multiply(uint a) retorna(uint d) {\n" +
    "       return a * 7;\n" +
    "   }\n" +
    "}\n";
var compiled = web3.eth.compile.solidity(source);
console.log(compiled); 
// {
  "test": {
    "code": "0x605280600c6000396000f3006000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114602e57005b60376004356041565b8060005260206000f35b6000600782029050604d565b91905056",
    "info": {
      "source": "contract test {\n\tfunction multiply(uint a) retorna(uint d) {\n\t\treturn a * 7;\n\t}\n}\n",
      "language": "Solidity",
      "languageVersion": "0",
      "compilerVersion": "0.8.2",
      "abiDefinition": [
        {
          "constant": false,
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
}
```

***

#### web3.eth.compile.lll

    web3. eth.compile.lll(sourceString [, callback])

Compila o código-fonte LLL.

##### Parâmetros

1. `String` - O código-fonte LLL.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - O código LLL compilado representado como uma *string* HEX.


##### Exemplo

```js
var source = "...";

var code = web3.eth.compile.lll(source);
console.log(code); // "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b6021600435602b565b8060005260206000f35b600081600702905091905056"
```

***

#### web3.eth.compile.serpent

    web3.eth.compile.serpent(sourceString [, callback])

Compila o código-fonte serpent.

##### Parâmetros

1. `String` - O código-fonte serpent.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`String` - O código serpent compilado representado como uma HEX *string*.


```js
var source = "...";

var code = web3.eth.compile.serpent(source);
console.log(code); // "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b6021600435602b565b8060005260206000f35b600081600702905091905056"
```

***

#### web3.eth.namereg

    web3.eth.namereg

Retorna objeto GlobalRegistrar.

##### Usage

Veja exemplo [namereg](https://github.com/ethereum/web3.js/blob/master/exemplo/namereg.html)

***

### web3.db

#### web3.db.putString

    web3.db.putString(db, key, value)

Este método deve ser chamado quando queremos armazenar uma *string* no banco de dados leveldb local.

##### Parâmetros

1. `String` - O banco de dados no qual será armazenado.
2. `String` - O nome do que será armazenado.
3. `String` - O valor *string* a ser armazenado.

##### Retorna

`Boolean` - `true` se bem-sucedido, caso contrário `false`.

##### Exemplo

 param é o nome do db, segundo é a chave, e terceiro é o valor *string*.
```js
web3.db.putString('testDB', 'key', 'myString') // true
```

***

#### web3.db.getString

    web3.db.getString(db, key)

Este método deve ser chamado quando queremos pegar uma *string* do banco de dados leveldb local.

##### Parâmetros

1. `String` - O nome *string* do banco de dados de onde recuperar.
2. `String` - O nome do que foi armazenado.

##### Retorna

`String` - O valor armazenado.

##### Exemplo
param é nome do db e segundo é a chave do valor *string*.
```js
var value = web3.db.getString('testDB', 'key');
console.log(value); // "myString"
```

***

#### web3.db.putHex

    web3.db.putHex(db, key, value)

Este método deve ser chamado quando queremos armazenar dados binários em HEX no banco de dados leveldb local.

##### Parâmetros

1. `String` - O banco de dados onde armazenar.
2. `String` - Nome do que será armazenado.
3. `String` - A *string* HEX a ser armazenada.

##### Retorna

`Boolean` - `true` se bem-sucedido, caso contrário `false`.

##### Exemplo
```js
web3.db.putHex('testDB', 'key', '0x4f554b443'); // true

```

***

#### web3.db.getHex

    web3.db.getHex(db, key)

Este método deve ser chamado quando queremos pegar um dado binário em HEX do banco de dados leveldb local.

##### Parâmetros

1. `String` - O banco de dados para armazenar.
2. `String` - O nome do que está armazenado.
3. 
##### Retorna

`String` - O valor HEX armazenado.


##### Exemplo
param é o nome do db e segundo é a chave do valor.
```js
var value = web3.db.getHex('testDB', 'key');
console.log(value); // "0x4f554b443"
```

***

### web3.shh

[Whisper  Overview](https://github.com/ethereum/wiki/wiki/Whisper-Overview)

##### Exemplo

```js
var shh = web3.shh;
```

***

#### web3.shh.post

   web3.shh.post(object [, callback])

Este método deve ser chamado quando queremos postar uma mensagem "susurro" (*whisper*) para a rede.

##### Parâmetros

1. `Object` - O objeto post:
  - `from`: `String`, 60 Bytes HEX - (opcional) A identidade do remetente.
  - `to`: `String`, 60 Bytes  HEX - (opcional) A identidade do destinatário. Quando o susurro for encriptado para que só o destinatário possa desencriptar a mensagem.
  - `topics`: `Array of Strings` - *Array* de `String` tópicos, para o destinatário identificar mensagens.
  - `payload`: `String|Number|Object` - O *payload* da mensagem. Será automaticamente convertido para uma *string* HEX antes.
  - `priority`: `Number` - O *integer* da prioridade de ... (?). The integer of the priority in a rang from ... (?).
  - `ttl`: `Number` - *integer* do tempo de vida em segundos.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Boolean` - retorna `true` se a mensagem foi enviada, caso contrário `false`.


##### Exemplo

```js
var identity = web3.shh.newIdentity();
var topic = 'exemplo';
var payload = 'hello whisper world!';

var message = {
  from: identity,
  topics: [topic],
  payload: payload,
  ttl: 100,
  workToProve: 100 // ou prioridade TODO
};

web3.shh.post(message);
```

***

#### web3.shh.newIdentity

    web3.shh.newIdentity([callback])

Deve ser chamado para criar uma nova identidade.

##### Parâmetros

1. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.


##### Retorna

`String` - Uma nova identidade em HEX *string*.


##### Exemplo

```js
var identity = web3.shh.newIdentity();
console.log(identity); // "0xc931d93e97ab07fe42d923478ba2465f283f440fd6cabea4dd7a2c807108f651b7135d1d6ca9007d5b68aa497e4619ac10aa3b27726e1863c1fd9b570d99bbaf"
```

***

#### web3.shh.hasIdentity

    web3.shh.hasIdentity(identity, [callback])

Deve ser chamado se queremos checar se um usuário tem uma certa identidade.

##### Parâmetros

1. `String` - A identidade a ser checada The identity to check.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Retorna

`Boolean` - retorna `true` se a identidade existe, caso contrário `false`.

##### Exemplo

```js
var identity = web3.shh.newIdentity();
var result = web3.shh.hasIdentity(identity);
console.log(result); // true

var result2 = web3.shh.hasIdentity(identity + "0");
console.log(result2); // false
```

***

#### web3.shh.newGroup

##### Exemplo
```js
// TODO: nao implementado ainda
```

***

#### web3.shh.addToGroup

##### Exemplo
```js
// TODO: nao implementado ainda
```

***

#### web3.shh.filter

```js
var filter = web3.shh.filter(options)

// watch for changes
filter.watch(function(error, result){
  if (!error)
    console.log(result);
});
```
Assiste por mensagens fechadas entrando.

##### Parâmetros

1. `Object` - As opções de filtro
  * `topics`: `Array de Strings` - Filtra mensagens por este(s) tópico(s). Você pode usar as seguintes combinações:
    - `['topic1', 'topic2'] == 'topic1' && 'topic2'`
    - `['topic1', ['topic2', 'topic3']] == 'topic1' && ('topic2' || 'topic3')`
    - `[null, 'topic1', 'topic2'] == ANYTHING && 'topic1' && 'topic2'` -> `null` funciona como *wildcard*
  * `to`: Filtra pela identidate do destinatário da mensagem. Se provido e o nó tem esta identidade, irá desencriptar mensagens encriptadas.
2. `Function` - (opcional) Se você passar um *callback* a requisição HTTP será feita de forma assíncrona. Veja [esta nota](#using-callbacks) para detalhes.

##### Callback return

`Object` - A mensagen entrando:

  - `from`: `String`, 60 Bytes - O remetente da mensagem, se um remetente foi especificado
  - `to`: `String`, 60 Bytes - O destinatário da mensagem, se um destinatário foi especificado
  - `expiry`: `Number` - *Integer* representando o tempo em segundos de quando esta mensagem deveria expirar (?).
  - `ttl`: `Number` - *Integer* representando o tempo em segundos que a mensagem deve flutuar no sistema (?).
  - `sent`: `Number` - *Integer* representrando o *timestamp* unix de quando a mensagem foi enviada.
  - `topics`: `Array of String` - *Array* de toópicos `String` que a mensagem contém.
  - `payload`: `String` - O *payload* da mensagem.
  - `workProved`: `Number` - *Integer* do trabalho que esta mensagem requeriu antes de ser enviada (?).

***

#### web3.eth.sendIBANTransaction

```js
var txHash = web3.eth.sendIBANTransaction('0x00c5496aee77c1ba1f0854206a26dda82a81d6d8', 'XE81ETHXREGGAVOFYORK', 0x100);
```

Sends IBAN transaction from user account to destination IBAN address.
Envia uma tranação IBAN da conta de usuário para o endereço IBAN destinatário.

##### Parâmetros

- `*string*` - endereço do qual queremos enviar a transação
- `*string*` - endereço IBAN do qual queremos enviar a transação
- `value` - valor que queremos enviar na transação IBAN

***

#### web3.eth.iban

```js
var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
```

***

#### web3.eth.iban.fromAddress

```js
var i = web3.eth.iban.fromAddress('0x00c5496aee77c1ba1f0854206a26dda82a81d6d8');
console.log(i.toString()); // 'XE7338O073KYGTWWZN0F2WZ0R8PX5ZPPZS
```

***

#### web3.eth.iban.fromBban

```js
var i = web3.eth.iban.fromBban('ETHXREGGAVOFYORK');
console.log(i.toString()); // "XE81ETHXREGGAVOFYORK"
```

***

#### web3.eth.iban.createIndirect

```js
var i = web3.eth.iban.createIndirect({
  institution: "XREG",
  identifier: "GAVOFYORK"
});
console.log(i.toString()); // "XE81ETHXREGGAVOFYORK"
```

***

#### web3.eth.iban.isValid

```js
var valid = web3.eth.iban.isValid("XE81ETHXREGGAVOFYORK");
console.log(valid); // true

var valid2 = web3.eth.iban.isValid("XE82ETHXREGGAVOFYORK");
console.log(valid2); // false, porque o *checksum* está incorreto

var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
var valid3 = i.isValid();
console.log(valid3); // true

```

***

#### web3.eth.iban.isDirect

```js
var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
var direct = i.isDirect();
console.log(direct); // false
```

***

#### web3.eth.iban.isIndirect

```js
var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
var indirect = i.isIndirect();
console.log(indirect); // true
```

***

#### web3.eth.iban.checksum

```js
var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
var checksum = i.checksum();
console.log(checksum); // "81"
```

***

#### web3.eth.iban.institution

```js
var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
var institution = i.institution();
console.log(institution); // 'XREG'
```

***

#### web3.eth.iban.*client*

```js
var i = new web3.eth.iban("XE81ETHXREGGAVOFYORK");
var *client* = i.*client*();
console.log(*client*); // 'GAVOFYORK'
```

***

#### web3.eth.iban.address

```js
var i = new web3.eth.iban('XE7338O073KYGTWWZN0F2WZ0R8PX5ZPPZS');
var address = i.address();
console.log(address); // '00c5496aee77c1ba1f0854206a26dda82a81d6d8'
```

***

#### web3.eth.iban.toString

```js
var i = new web3.eth.iban('XE7338O073KYGTWWZN0F2WZ0R8PX5ZPPZS');
console.log(i.toString()); // 'XE7338O073KYGTWWZN0F2WZ0R8PX5ZPPZS'
```