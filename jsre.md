<--! Source: https://raw.githubusercontent.com/wiki/ethereum/go-ethereum/JavaScript-Console.md-->
# JavaScript Runtime Environment

Ethereum implementa um **ambiente javascript em tempo de execução** (*javascript runtime environment*/JSRE) que pode ser usado tanto em modo interativo (console) quanto não-interativo (script).

O console JavaScript do Ethereum expõe a [API web3 JavaScript Dapp](https://github.com/ethereum/wiki/wiki/JavaScript-API) inteira e a [API admin](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#javascript-console-api)

## Uso interativo: O Console JSRE REPL
*\*REPL: read–eval–print loop*

A `CLI ethereum` executável `geth` tem um console JavaScript (um ambiente REPL expondo a JSRE), que pode ser iniciado com o subcomando `console` ou `attach`. O subcomando `console` inicia o nó geth e então abre o console. O subcomando `attach` não iniciará o nó geth e tentará abrir o console numa instância geth já rodando.

    $ geth console
    $ geth attach

O nó `attach` aceita um *endpoint* em caso do nó geth estar rodando com um *endpoint* ipc não padrão ou você pode preferir conectar através da interface rpc.

    $ geth attach ipc:/algum/caminho/arbitrário
    $ geth attach http://191.168.1.1:8545
    $ geth attach ws://191.168.1.1:8546

Note que, por padrão, o nó geth nnao inicia os serviços http e websocket e nem toda funcionalidade é provida através destas interfaces devido a razões de segurança. Estes padrões podem ser sobrepostos quando os arumentos `--rpcapi` e `--wsapi` são chamado ao inciar o nó geth, ou com [admin.startRPC](admin_startRPC) e [admin.startWS](admin_startWS).

Se você precisa informações de log, inicie com:

    $ geth --verbosity 5 console 2>> /tmp/eth.log

Ou coloque os seus logs no mudo, para não poluir a sua visão no console:

    $ geth console 2>> /dev/null

ou

    $ geth --verbosity 0 console

Geth tem suporte para carregar arquivos JavaScript customizados no console através do argumento `--preload`. Isso pode ser usado para carregar funções muito usadas, configurar objetos de contrato web3, ou ...

```
geth --preload "/minha/pasta/javascript/utils.js,/minha/pasta/javascript/contracts.js" console
```


## Uso não-interativo: Modo de script JSRE

Também é possível executar arquivos no interpretador JavaScript. Os subcomandos `console` e `attach` aceita o argumento `--exec` que é uma declaração javascript.

    $ geth --exec "eth.blockNumber" attach

Isso "printa" o número do bloco atual da instância geth em execução.

Ou execute um script local com declarações mais complexas num nó remoto via http:

    $ geth --exec 'loadScript("/tmp/checkbalances.js")' attach http://123.123.123.123:8545
    $ geth --jspath "/tmp" --exec 'loadScript("checkbalances.js")' attach http://123.123.123.123:8545

Veja um exemplo de script [aqui](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions#exemplo-script)

Use o `--jspath <caminho/para/minha/raiz/js>` para configurar um libdir para seus scripts js. Parâmetros passados ao `loadScript()` sem um caminho absoluto serão entendidos como caminhos relativos ao diretório atual.

Você pode sair do console de forma limpa digitando `exit` ou simplesmente `CTRL-C`.

## Aviso

O JSRE go-ethereum usa a [Otto JS VM](https://github.com/robertkrimen/otto) que tem algumas limitações:

* "use strict" irá apenas analisar sem fazer nada.
* O motor de expressão regular (re2/regexp) não é completamente compatível com a especificação ECMA5.

Note que a outra limitação conhecida da Otto (falta de *timers*) é cuidada.
O JSRE Ethereum implementa ambos `setTimeout` e `setInterval`. Em adição a isso, o console disponibiliza `admin.sleep(seconds)` também como um método "blocktime sleep" `admin.sleepBlocks(number)`.

Visto que `web3.js` usa a *library* [`bignumber.js`](https://github.com/MikeMcl/bignumber.js) (MIT Expat License), ela é automaticamente carregada.

## Timers

Em adição à funcionalidade completa do JS (conforme ECMA5), o JSRE ethereum é aumentado com vários timers. Ele implementa `setInteval`, `clearInterval`, `setTimeout`, `clearTimeout` que você pode estar acostumado a usar nas janelas de browser. Ele também provê implementação para `admin.sleep(seconds)` e um timer baseado em blocos, `admin.sleepBlocks(n)` que "dorme" até que o número de novos blocos adicionados seja igual ou maior a `n`, pense como em "esperar por n confirmações".

# APIs de Gerenciamento

Além da interface da [API DApp](https://github.com/ethereum/wiki/wiki/JSON-RPC) oficial o nó go ethereum tem suport para APIs de gerenciamento adicionais. Estas APIs são ofereidas usando [JSON-RPC](http://www.jsonrpc.org/specification) e seguem a mesma convenção que a usada na API DApp. O pacote go ethereum vem com um *client* console que tem suporte para todas as APIs adicionais.

# Como fazer

É possível especificar um conjunto de APIs que são oferecidas através de uma interface com o argumento de linha de comando `--${interface}api` para o daemon do go ethereum. Onde `${interface}` pode ser `rpc` para a interface `http` ou `ipc` para o socket unix no unix ou named pipe no Windows.

Por exemplo, `geth --ipcapi "admin,eth,miner" --rpcapi "eth,web3"` irá
* habilitar o admin, DApp oficial e API de mineração através da interface IPC
* habilitar a API eth e web3 através da interface RPC

Por favor, note que oferecer uma API através da interface `prc` irá dar acesso à API a todos que podem acessar esta interface (e.g. DApp's). Portanto, tenha cuidado com que APIs você habilita. Por padrão, geth habilita todas APIs através da interface `ipc` e apenas as APIs eth, net e web3 através das interfaces `rpc` e `ws`.

Para determinar quais APIs uma interface provê, a transação `modules` pode ser usada, e.g. através de uma interface `ipc` num sistema unix:

```
echo '{"jsonrpc":"2.0","method":"rpc_modules","params":[],"id":1}' | socat -,ignoreeof  UNIX-CONNECT:$HOME/.ethereum/geth.ipc
```
Dará todosos módulos habilitados incluindo o número de versão:
```
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "admin": "1.0",
    "debug": "1.0",
    "eth": "1.0",
    "miner": "1.0",
    "net": "1.0",
    "personal": "1.0",
    "rpc": "1.0",
    "txpool": "1.0",
    "web3": "1.0"
  }
}

```

## Integração

Estas APIs adicionais seguem as mesmas convenções que a API DApp oficial. Web3 pode ser [estendida](https://github.com/ethereum/web3.js/pull/229) e usada para consumir estas APIs adicionais.

As diferentes funções são divididas em múltiplas APIs menores logicamente agrupadas. Exemplos são dados para o [console Javascript](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) mas podem ser facilmente convertidos para uma requisição rpc.

**2 exemplos:**

* Console: `miner.start()`

* IPC: `echo '{"jsonrpc":"2.0","method":"miner_start","params":[],"id":1}' | socat -,ignoreeof  UNIX-CONNECT:$HOME/.ethereum/geth.ipc`

* RPC: `curl -X POST --data '{"jsonrpc":"2.0","method":"miner_start","params":[],"id":74}' localhost:8545`

Com o número de THREADS como um argumento:

* Console: `miner.start(4)`

* IPC: `echo '{"jsonrpc":"2.0","method":"miner_start","params":["0x04"],"id":1}' | socat -,ignoreeof  UNIX-CONNECT:$HOME/.ethereum/geth.ipc`

* RPC: `curl -X POST --data '{"jsonrpc":"2.0","method":"miner_start","params":["0x04"],"id":74}' localhost:8545`

## Referência da API de Gerenciamento

* [eth](#eth)
  * [sign](#ethsign)
  * [pendingTransactions](#ethpendingtransactions)
  * [resend](#ethresend)
* [admin](#admin)
  * [addPeer](#adminadd*peer*)
  * [*peer*s](#admin*peer*s)  
  * [nodeInfo](#adminnodeinfo)
  * [datadir](#admindatadir)
  * [importChain](#adminimportchain)
  * [exportChain](#adminexportchain)
  * [chainSyncStatus](#adminchainsyncstatus)
  * [startRPC](#adminstartrpc)
  * [stopRPC](#adminstoprpc)
  * [startWS](#adminstartws)
  * [stopWS](#adminstopws)
  * [verbosity](#adminverbosity)
  * [setSolc](#adminsetsolc)
  * [sleepBlocks](#adminsleepblocks)
  * [startNatSpec](#adminstartnatspec)
  * [stopNatSpec](#adminstopnatspec)
  * [getContractInfo](#admingetcontractinfo)
  * [register](#adminregister)
  * [registerUrl](#adminregisterurl)
* [miner](#miner)
  * [start](#minerstart)
  * [stop](#minerstop)
  * [startAutoDAG](#minerstartautodag)
  * [stopAutoDAG](#minerstopautodag)
  * [makeDAG](#minermakedag)
  * [setExtra](#minersetextra) 
  * [setGasPrice] (#minersetgasprice)
  * [setEtherbase] (#minersetetherbase)
* [personal](#personal)
  * [newAccount](#personalnewaccount)
  * [listAccounts](#personallistaccounts)
  * [deleteAccount](#personaldeleteaccount)
  * [unlockAccount](#personalunlockaccount)
* [txpool](#txpool)
  * [status](#txpoolstatus)
* [debug](#debug)
  * [setHead](#debugsethead)
  * [seedHash](#debugseed*hash*)
  * [processBlock](#debugprocessblock)
  * [getBlockRlp](#debuggetblockrlp)
  * [printBlock](#debugprintblock)
  * [dumpBlock](#debugdumpblock)
  * [metrics](#debugmetrics)
* [loadScript](#loadscript)
* [sleep](#sleep)
* [setInterval](#setInterval)
* [clearInterval](#clearInterval)
* [setTimeout](#setTimeout)
* [clearTimeout](#clearTimeout)
* [web3](#web3)
* [net](#net)
* [eth](#eth)
* [shh](#shh)
* [inspect](#inspect)

***

#### Personal
A api `personal` expôe métodos para personal gerenciar, controlar ou monitorar seu nó. Isto permite um acesso limitado ao filesystem.

***

#### personal.listAccounts
    personal.listAccounts

Lista todas as contas

#### Retorna
Coleção com contas

#### Exemplo
` personal.listAccounts`

***

#### personal.newAccount

    personal.newAccount(passwd)

Cria uma nova conta protegida por senha `passwd`

#### Retorna
`*string*` endereço da nova conta

#### Exemplo
` personal.newAccount("mypasswd")`

` personal.newAccount() # irá pedir uma senha`

***


#### personal.unlockAccount
    personal.unlockAccount(addr, passwd, duration)

Destranca a conta com o endereço `addr` passado, senha e uma duração `duration` (em segundos) opcional. Se a senha `passwd` não for dada você será pedido por ela.

#### Return
`boolean` indicação se a conta foi destrancada com sucesso

#### Exemplo
` personal.unlockAccount(eth.coinbase, "mypasswd", 300)`

***

### TxPool

#### txpool.status
    txpool.status

Número de transações pendentes/em fila

#### Retorna
`pending` todas as transações processáveis

`queued` todas as transações não processáveis

#### Exemplo
` txpool.status`

***


#### admin
O `admin` expõe métodos para gerenciar, controlar ou monitorar o nó. Permite um acesso limitado ao filesystem.

***


#### admin.nodeInfo

    admin.nodeInfo

##### Retorna

Informação sobre o nó.

##### Exemplo

```
> admin.nodeInfo
{
   Name: 'Ethereum(G)/v0.9.36/darwin/go1.4.1',
   NodeUrl: 'enode://c32e13952965e5f7ebc85b02a2eb54b09d55f553161c6729695ea34482af933d0a4b035efb5600fc5c3ea9306724a8cbd83845bb8caaabe0b599fc444e36db7e@89.42.0.12:30303',
   NodeID: '0xc32e13952965e5f7ebc85b02a2eb54b09d55f553161c6729695ea34482af933d0a4b035efb5600fc5c3ea9306724a8cbd83845bb8caaabe0b599fc444e36db7e',
   IP: '89.42.0.12',
   DiscPort: 30303,
   TCPPort: 30303,
   Td: '0',
   ListenAddr: '[::]:30303'
}
```

Para se conectar a um nó, use a nodeUrl [enode-format](https://github.com/ethereum/wiki/wiki/enode-url-format) como um argumento para [addPeer](#adminadd*peer*) com o parâmetro de CLI `bootnodes`.

***

#### admin.addPeer

    admin.addPeer(nodeURL)

Passa uma `nodeURL` para se conectar a um *peer* na rede. A `nodeURL` precisa estar em [formato enode URL](https://github.com/ethereum/wiki/wiki/enode-url-format). geth manterá a conexão até desligar e tentará reconectar se a conexão for interrompida de forma intermitente.

Você pode achar sua própria URL de nó usando [nodeInfo](#adminnodeinfo) ou buscando nos logs quando o nó é iniciado e.g.:

```
[P2P Discovery] Listening, enode://6f8a80d14311c39f35f516fa664deaaaa13e85b2f7493f37f6144d86991ec012937307647bd3b9a82abe2974e1407241d54947bbb39763a4cac9f77166ad92a0@54.169.166.226:30303
```

##### Retorna

`true` em caso de sucesso.

##### Exemplo

```javascript
> admin.addPeer('enode://6f8a80d14311c39f35f516fa664deaaaa13e85b2f7493f37f6144d86991ec012937307647bd3b9a82abe2974e1407241d54947bbb39763a4cac9f77166ad92a0@54.169.166.226:30303')
```

***

#### admin.*peer*s

    admin.*peer*s

##### Retorna

Uma array de objetos com informação sobre os *peer*s conectados.

##### Exemplo

```
> admin.*peer*s
[ { ID: '0x6cdd090303f394a1cac34ecc9f7cda18127eafa2a3a06de39f6d920b0e583e062a7362097c7c65ee490a758b442acd5c80c6fce4b148c6a391e946b45131365b', Name: 'Ethereum(G)/v0.9.0/linux/go1.4.1', Caps: 'eth/56, shh/2', RemoteAddress: '54.169.166.226:30303', LocalAddress: '10.1.4.216:58888' } { ID: '0x4f06e802d994aaea9b9623308729cf7e4da61090ffb3615bc7124c5abbf46694c4334e304be4314392fafcee46779e506c6e00f2d31371498db35d28adf85f35', Name: 'Mist/v0.9.0/linux/go1.4.2', Caps: 'eth/58, shh/2', RemoteAddress: '37.142.103.9:30303', LocalAddress: '10.1.4.216:62393' } ]
```
***

#### admin.importChain

    admin.importChain(file)

Importa a blockchain de um formato binário *marshalled*.
**Note** que a blockchain é resetada (até o genesis) antes que os blocos importados sejam inseridos na *chain*.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
admin.importChain('caminho/do/arquivo')
// true
```

***

#### admin.exportChain

    admin.exportChain(file)

Exporta a blockchain para o arquivo no formato binário dado.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
admin.exportChain('caminho/do/arquivo')
```

***

#### admin.startRPC

     admin.startRPC(host, portNumber, corsheader, modules)

Inciar o servidor HTTP para o [JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC).

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
admin.startRPC("127.0.0.1", 8545, "*", "web3,net,eth")
// true
```

***

#### admin.stopRPC

    admin.stopRPC() 

Para o servidor HTTP para o [JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC).

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
admin.stopRPC()
// true
```

***

#### admin.startWS

     admin.startWS(host, portNumber, allowedOrigins, modules)

Inciar o servidor websocket para o [JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC). 

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
admin.startWS("127.0.0.1", 8546, "*", "web3,net,eth")
// true
```

***

#### admin.stopWS

    admin.stopWS() 

Para o servidor websocket para o [JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC).

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
admin.stopWS()
// true
```

***

#### admin.sleepBlocks 

    admin.sleepBlocks(n)

"Dorme" por `n` blocos.

***

#### admin.datadir

    admin.datadir

O diretório que este nó armazena seus dados.

##### Retorna

O diretório em caso de sucesso.

##### Exemplo

```javascript
admin.datadir
'/Users/username/Library/Ethereum'
```

***

#### admin.setSolc

    admin.setSolc(path2solc)

Configura o compilador solidity

##### Retorna

Uma *string* descrevendo a versão do compilador quando o caminho for válido, caso contrário um erro.

##### Exemplo

```javascript
admin.setSolc('/algum/caminho/solc')
'solc v0.9.29
Solidity Compiler: /algum/caminho/solc
'
```

***

#### admin.startNatSpec

     admin.startNatSpec()

Ativa NatSpec: quando enviando uma trsanção para um contrato,
*Lookup* de registro e busca de URL é usado para recuperar informação do contrato autêntico. Permite notificar a um usuário com mensagens de confirmação específicas de contrato autêntico.

***

#### admin.stopNatSpec

     admin.stopNatSpec()

desativa NatSpec: quadno enviando uma transação, o usuário será notificado com uma mensagem de confirmação genéria, nenhuma informação de contrato é buscada.

***

#### admin.getContractInfo

     admin.getContractInfo(address)

Isto irá recuperar a [informação de contrato em json](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions#contract-info-metadata) para um contrato no endereço passado.

##### Retorna

Retorna o objeto de informação de contrato

##### Exemplos

```js
> info = admin.getContractInfo(contractaddress)
> source = info.source
> abi = info.abiDefinition
```

***
#### admin.saveInfo

    admin.saveInfo(contract.info, filename);

Escreverá a [informação de contrato em json](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions#contract-info-metadata) no arquivo alvo, calculará o *hash* do conteúdo. Este *hash* do conteúdo poderá, então, ser usado para associar uma url pública com onde a informação do contrato é publicamente disponvível e vewrificável. Se você registrar um code*hash* (*hash* do código do contrato em contractaddress).

##### Retorna

`content*hash*` em caso de sucesso, caso contrário `undefined`.

##### Exemplos

```js
source = "contract test {\n" +
"   /// @notice will multiply `a` by 7.\n" +
"   function multiply(uint a) retorna(uint d) {\n" +
"      return a * 7;\n" +
"   }\n" +
"} ";
contract = eth.compile.solidity(source).test;
tx*hash* = eth.sendTransaction({from: primary, data: contract.code });
// after it is uncluded
contractaddress = eth.getTransactionReceipt(tx*hash*);
filename = "/tmp/info.json";
content*hash* = admin.saveInfo(contract.info, filename);
```

***
#### admin.register

    admin.register(address, contractaddress, content*hash*);

Registrará o *hash* do conteúdo no code*hash* (*hash* do código do contrato em contractaddress). O a transação de registro é enviada do endereço no primeiro parâmetro. A transação precisa ser processada e confirmada na *chain* canônica para que o registro tenha efeito.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplos

```js
source = "contract test {\n" +
"   /// @notice will multiply `a` by 7.\n" +
"   function multiply(uint a) retorna(uint d) {\n" +
"      return a * 7;\n" +
"   }\n" +
"} ";
contract = eth.compile.solidity(source).test;
tx*hash* = eth.sendTransaction({from: primary, data: contract.code });
// após ser incluido
contractaddress = eth.getTransactionReceipt(tx*hash*);
filename = "/tmp/info.json";
content*hash* = admin.saveInfo(contract.info, filename);
admin.register(primary, contractaddress, content*hash*);
```

***

#### admin.registerUrl

    admin.registerUrl(address, code*hash*, content*hash*);

files. Address in the first parâmetro will be used to send the transaction.
Isto irá registar uma constante *hash* para o code*hash* do contrato. Será usado para localizar o [contract info json](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions#contract-info-metadata)

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplos

```js
source = "contract test {\n" +
"   /// @notice irá multiplicar `a` by 7.\n" +
"   function multiply(uint a) retorna(uint d) {\n" +
"      return a * 7;\n" +
"   }\n" +
"} ";
contract = eth.compile.solidity(source).test;
tx*hash* = eth.sendTransaction({from: primary, data: contract.code });
// após ser incluido
contractaddress = eth.getTransactionReceipt(tx*hash*);
filename = "/tmp/info.json";
content*hash* = admin.saveInfo(contract.info, filename);
admin.register(primary, contractaddress, content*hash*);
admin.registerUrl(primary, content*hash*, "file://"+filename);
```

***

### Miner



#### miner.start

    miner.start(threadCount)

Inicia [mineração](see https://github.com/ethereum/go-ethereum/wiki/Mining) com o dado `threadNumber` das threads paralelas. Este é um argumento opcional.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
miner.start()
// true
```

***

#### miner.stop

    miner.stop(threadCount)

Para os mineradores `threadCount`. Este é um argumento opcional.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo

```javascript
miner.stop()
// true
```

***


#### miner.startAutoDAG

    miner.startAutoDAG()

Inicia a pré-geração automática do [et*hash* DAG](https://github.com/ethereum/wiki/wiki/Et*hash*-DAG). Este processo certifica que o DAG para a época subsequente está disponível permitindo minerar logo após a nova época inciar. Se for usado pela maior parte dos nós da rede, então é esperado que os tempos de blocos sejam normais na época de transição. Auto DAG é ligado automaticamente quando a mineração é iniciada e desligado quando o minerador para. 

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

***

#### miner.stopAutoDAG

    miner.stopAutoDAG()

Para a pré-geração automática do [et*hash* DAG](https://github.com/ethereum/wiki/wiki/Et*hash*-DAG). Auto DAG é desligado automaticamente quando a mineração para.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

***

#### miner.makeDAG

    miner.makeDAG(blockNumber, dir)

Gera o DAG para época `blockNumber/epochLength`. dir especifica o diretório alvo. Se `dir` for uma *string* vazia, então et*hash* usará os diretórios padrões `~/.et*hash*` no Linux e MacOS e `~\AppData\Et*hash*` no Windows. O nome do arquivo DAG é `full-<revision-number>R-<seed*hash*>`.

##### Retorna

`true` em caso de sucesso, caso contrário `false`.

***

#### miner.setExtra

    miner.setExtra("extra data")

**Configura** os dados extra para o bloco quando achado um bloco. Limitado a 32 *bytes*.

***

#### miner.setGasPrice

    miner.setGasPrice(gasPrice)

**Configura** o preço do gas para o minerador.

***

#### miner.setEtherbase

    miner.setEtherbase(account)

**Configura** a base ehter, o endereço que receberá as recompensas de mineração.

***





### Debug

#### debug.setHead

    debug.setHead(blockNumber)

**Configura** a cabeça atual da bockchain para ser o blcoo referenciado por _blockNumber_.
Veja [web3.eth.getBlock](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetblock) para mais detalhes sobre campos de bloco e busca por número ou *hash*.

##### Retorna 

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo 

    debug.setHead(eth.blockNumber-1000)

***

#### debug.seedHash

    debug.seedHash(blockNumber)

Retorna o *hash* para a época em que o dado bloco está.

##### Retorna 

*hash* em formato hex

##### Exemplo 

    > debug.seedHash(eth.blockNumber)
    '0xf2e59013a0a379837166b59f871b20a8a0d101d1c355ea85d35329360e69c000'

***

#### debug.processBlock

    debug.processBlock(blockNumber)

Processa o dado bloco referenciado por _blockNumber_ com a VM em modo *debug*.
Veja [web3.eth.getBlock](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetblock) para mais detalhes nos campos do bloco e busca por número ou *hash*.
Em combinação com `setHead`, este pode ser usado para repetir o processamento de um bloco para *debugar* a execução da VM.

##### Retorna 

`true` em caso de sucesso, caso contrário `false`.

##### Exemplo 

    debug.processBlock(140101)

***


#### debug.getBlockRlp

    debug.getBlockRlp(blockNumber)

Retorna a representação hexadecimal da codificação RLP do bloco.
Veja [web3.eth.getBlock](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetblock) para detalhes sobre campos do bloco e busca por número ou *hash*.

##### Retorna 

A representação hex da codificação RLP do bloco.

##### Exemplo

```
> debug.getBlockRlp(131805)    'f90210f9020ba0ea4dcb53fe575e23742aa30266722a15429b7ba3d33ba8c87012881d7a77e81ea01dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d4934794a4d8e9cae4d04b093aac82e6cd355b6b963fb7ffa01f892bfd6f8fb2ec69f30c8799e371c24ebc5a9d55558640de1fb7ca8787d26da056e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421a056e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421b901000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000083bb9266830202dd832fefd880845534406d91ce9e5448ce9ed0af535048ce9ed0afce9ea04cf6d2c4022dfab72af44e9a58d7ac9f7238ffce31d4da72ed6ec9eda60e1850883f9e9ce6a261381cc0c0'
```

***

#### debug.printBlock

    debug.printBlock(blockNumber)

"Printa" informação sobre o bloco como o tamanho, dificuldade total, assim como os campos do cabeçalho devidamente formatados.

See [web3.eth.getBlock](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetblock) for more details on block fields and lookup by number or *hash*.

##### Retorna

Representação do bloco em uma *string* formatada

##### Exemplo

```
> debug.printBlock(131805)
BLOCK(be465b020fdbedc4063756f0912b5a89bbb4735bd1d1df84363e05ade0195cb1): Size: 531.00 B TD: 643485290485 {
NoNonce: ee48752c3a0bfe3d85339451a5f3f411c21c8170353e450985e1faab0a9ac4cc
Header:
[

        ParentHash:         ea4dcb53fe575e23742aa30266722a15429b7ba3d33ba8c87012881d7a77e81e
        UncleHash:          1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347
        Coinbase:           a4d8e9cae4d04b093aac82e6cd355b6b963fb7ff
        Root:               1f892bfd6f8fb2ec69f30c8799e371c24ebc5a9d55558640de1fb7ca8787d26d
        TxSha               56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421
        ReceiptSha:         56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421
        Bloom:              00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
        Difficulty:         12292710
        Number:             131805
        GasLimit:           3141592
        GasUsed:            0
        Time:               1429487725
        Extra:              ΞTHΞЯSPHΞЯΞ
        MixDigest:          4cf6d2c4022dfab72af44e9a58d7ac9f7238ffce31d4da72ed6ec9eda60e1850
        Nonce:              3f9e9ce6a261381c
]
Transactions:
[]
Uncles:
[]
}


```


***


#### debug.dumpBlock

    debug.dumpBlock(blockNumber)

##### Retorna

O *dump* cru de um bloco referenciado pelo número do bloco ou *hash* do bloco ou indefinido se o bloco não for encontrado. 
Veja [web3.eth.getBlock](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetblock) para mais detalhes nos campos de bloco e busca por número ou *hash*.

##### Exemplo


```js
> debug.dumpBlock(eth.blockNumber)
```


***

#### debug.metrics

    debug.metrics(raw)

##### Retorna

Coleção de métricas, veja [esta](https://github.com/ethereum/go-ethereum/wiki/Metrics-and-Monitoring) página wiki para mais informação.

##### Exemplo


```js
> metrics(true)
```


***


#### loadScript

     loadScript('/path/to/myfile.js');

Carrega um arquivo JavaSript e o executa. Caminhos relativos são interpretados como relativo ao `jspath` que é especificado como uma flag na linha de comando, veja [Opções de Linha de Comando](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options).

#### sleep 

    sleep(s)

Sleeps for s seconds. 

#### setInterval

    setInterval(s, func() {})

#### clearInterval
#### setTimeout
#### clearTimeout


***

#### web3
O `web3` expõe todos métodos da [API JavaScript](https://github.com/ethereum/wiki/wiki/JavaScript-API).

***

#### net
O `net` é um atalho para [web3.net](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3net).

***

#### eth
O `eth` é um atalho para [web3.eth](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3eth). Em adição às interfaces `web3` e `eth` expostas por [web3.js](https://github.com/ethereum/web3.js) algumas chamadas adicionais são expostas.

***

#### eth.sign

    eth.sign(signer, data)

#### eth.pendingTransactions

    eth.pendingTransactions

Retorna pending transactions that belong to one of the users `eth.accounts`.
Retorna transações pendentes que pertencem a um dos usuário `eth.accounts`.

***

#### eth.resend

    eth.resend(tx, <optional gas price>, <optional gas limit>)

Re-envia a transação passada retorna por `pendingTransactions()` e permite que você sobrescreva o preço do gas e o limite do gas para a transação

##### Exemplo

```javascript
eth.sendTransaction({from: eth.accounts[0], to: "...", gasPrice: "1000"})
var tx = eth.pendingTransactions[0]
eth.resend(tx, web3.toWei(10, "szabo"))
```

***


#### shh
O `ssh` é um atalho para [web3.shh](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3shh).

***


#### inspect
O método `inspect` *printa* de forma visualmente boa o valor (suporta cores)

***