<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md"-->

# Compilando um contrato

Contratos vivem na blockchain em um formato binário específico da Ethereum (*Ethereum Virtual Machine* (=EVM) bytecode). Entretanto, contratos são tipicamente escritos em alguma linguagem de alto nível, como [solidity](https://github.com/ethereum/wiki/wiki/Solidity-Tutorial), e depois compilado em *byte code* para ser carregado na blockchain.

Para o lançamento frontier, `geth` suporta compilação de solidity através de uma chamada do sistema para `solc`, [o compilador solidity](https://github.com/ethereum/cpp-ethereum/tree/develop/solc) na linha de comando, por Christian R. and Lefteris K. Você pode testar o [Compilador solidity em tempo real](https://chriseth.github.io/cpp-ethereum/) (por Christian R) ou [Cosmo](http://meteor-dapp-cosmo.meteor.com) ou [Mix](https://github.com/ethereum/wiki/wiki/Mix:-The-DApp-IDE). 

Se você iniciar o seu nó `geth`, você pode checar se o compilador de solidity está disponível. Isto é o que acontece, se não estiver:

```js
> eth.compile.solidity("")
eth_compileSolidity method not available: solc (solidity compiler) not found
    at InvalidResponse (<anonymous>:-57465:-25)
    at send (<anonymous>:-115373:-25)
    at solidity (<anonymous>:-104109:-25)
    at <anonymous>:1:1
```

Depois de você ter descoberto como instalar `solc`, garanta que ele está no *path*. Se [`eth.getCompilers()`](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgetcompilers) ainda não o encontra (retorna uma *array* vazia), você pode definir um *path* customizado para o executável do `solc` na linha de comando usando a *flag* `solc`.

```
geth --datadir ~/frontier/00 --solc /usr/local/bin/solc --natspec
```

Você também pode definir essa opção durante o tempo de execução pelo console:

```js
> admin.setSolc("/usr/local/bin/solc")
solc v0.9.32
Solidity Compiler: /usr/local/bin/solc
Christian <c@ethdev.com> and Lefteris <lefteris@ethdev.com> (c) 2014-2015
true
```

Vamos tomar essa simples fonte de contrato:

```js
> source = "contract test { function multiply(uint a) returns(uint d) { return a * 7; } }"
```

Esse contrato oferece um método unário: chamado com um inteiro positivo `a`, ele retorna `a * 7`.
~~This contract offers a unary method: called with a positive integer `a`, it returns `a * 7`.~~ 

Você está pronto para compilar código solidity no console JS `geth` usando [`eth.compile.solidity`](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethcompilesolidity):

```js
> contract = eth.compile.solidity(source).test
{
  code: '605280600c6000396000f3006000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114602e57005b60376004356041565b8060005260206000f35b6000600782029050604d565b91905056',
  info: {
    language: 'Solidity',
    languageVersion: '0',
    compilerVersion: '0.9.13',
    abiDefinition: [{
      constant: false,
      inputs: [{
        name: 'a',
        type: 'uint256'
      } ],
      name: 'multiply',
      outputs: [{
        name: 'd',
        type: 'uint256'
      } ],
      type: 'function'
    } ],
    userDoc: {
      methods: {
      }
    },
    developerDoc: {
      methods: {
      }
    },
    source: 'contract test { function multiply(uint a) returns(uint d) { return a * 7; } }'
  }
}
```

O compilador também está disponível via [RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) e, portanto, via [web3.js](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethcompilesolidity) para qualquer Ðapp *in-browser* conectando ao `geth` via RPC/IPC.

O exemplo seguinte mostra como você interage com `geth` via JSON-RPC para usar o compilador.

```
./geth --datadir ~/eth/ --loglevel 6 --logtostderr=true --rpc --rpcport 8100 --rpccorsdomain '*' --mine console  2>> ~/eth/eth.log
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_compileSolidity","params":["contract test { function multiply(uint a) returns(uint d) { return a * 7; } }"],"id":1}' http://127.0.0.1:8100
```

O output do compilador para uma fonte dará a você objetos de contrato, cada um representando um único contrato. O real valor de retorno de `eth.compile.solidity` é um mapa de pares com nome de contrato -- objeto de contrato. Como o nosso nome de contrato é `test`, `eth.compile.solidity(source).test` dará a você o objeto de contrato para o contrato "test" contendo os seguintes campos:

* `code`: o código de EVM compilado
* `info`: o restante das meta-informações que saem no output do compilador
  * `source`: o código fonte
  * `language`: linguagem do contrato (Solidity, Serpent, LLL)
  * `languageVersion`: versão da linguagem do contrato
  * `compilerVersion`: versão do compilador
  * `abiDefinition`: [*Application Binary Interface Definition*](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI)
  * `userDoc`: [Documentação para usuário do NatSpec](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification- Format)
  * `developerDoc`: [Documentação para desenvolvedor do NatSpec](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format)

A estruturação imediata do output do compilador (para `code` e `info`) reflete os dois **caminhos de implementação** muito diferentes.
O código EVM compilado é enviado para a blockchain com uma transação de criação de contrato enquanto o restante (informação) irá idealmente viver na nuvem descentralizada como metadados publicamente verificáveis complementando o código na blockchain.

Se a sua fonte contém múltiplos contratos, o output irá conter uma entrada para cada contrato, o objeto de informação de contrato correspondente pode ser recuperado com o nome do contrato como nome do atributo.
Você pode tentar isso inspecionando o mais recente código GlobalRegistrar:

```js
contracts = eth.compile.solidity(globalRegistrarSrc)
```
