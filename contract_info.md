{% sections "contract-info-metadata", "" %}
{% endsections %}

<!-- include "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" -->

# Informações de contrato (metadados)

Na sessão anterior, nós explicamos como você cria um contrato na blockchain. Agora, nós lidamos com o restante do output do compilador, os **metadados de contrato** ou informações de contrato. A ideia é que

* informações de contrato são carregadas em algum alugar identificável por uma `url` que é publicamente acessível
* qualquer um pode descobrir qual é a `url` apenas sabendo o endereço do contrato

Esses requerimentos são alcançados de forma muito simples usando um registro de 2 passos na blockchain. O primeiro passo registra o código do contrato (*hash*) com um *hash* de conteúdo em um contrato chamado `HashReg`. O segundo passo registra uma url com o *hash* de conteúdo no contrato `UrlHint`.
Estes [contratos simples de registro](https://github.com/ethereum/go-ethereum/blob/develop/common/registrar/contracts.go) serão parte da proposição frontier.

Usando esse esquema, é suficiente saber o endereço de um contrato para procurar a url e pegar o pacote de metadados do contrato. Continue lendo para saber porque isso é bom.

Então se você for um criador de contrato consciente, os passos são os seguinte:

1. Implementa o próprio contrato na blockchain
2. Pega o arquivo json de informação do contrato
3. Implementa o arquivo json de informação de contrato em qualquer url de sua escolha
4. Registra o *codehash* -> *hash* de conteúdo -> url

A API JS torna esse processo muito fácil provendo ajudantes. Chama [`admin.register`]() para extrair informação do contrato, escreve sua serialização em json no dado arquivo, calcula o *hash* de conteúdo do arquivo e finalmente registra esse *hash* de conteúdo no *hash* de código do contrato.
Uma vez que você já implementou esse arquivo em qualquer url, você pode usar  [`admin.registerUrl`]() para registrar a url com o seu *hash* de conteúdo na blockchain também. (Note que, caso um modelo dirigido de conteúdo fixo seja usado como armazenamento de documento, a dica de url não será mais necessária.)

```js
source = "contract test { function multiply(uint a) returns(uint d) { return a * 7; } }"
// compila com solc
contract = eth.compile.solidity(source).test
// cria objeto de contrato
var MyContract = eth.contract(contract.info.abiDefinition)
// extrai informação de contrato, salva a serialização em json no dado arquivo
contenthash = admin.saveInfo(contract.info, "~/dapps/shared/contracts/test/info.json")
// envia o contrato para a blockchain
MyContract.new({from: primaryAccount, data: contract.code}, function(error, contract){
  if(!error && contract.address) {
    // calcula o hash do conteúdo e o registra com o hash do código em `HashReg`
    // ele usa endereço para enviar a transação
    // retorna o hash do conteúdo que nós usamos para registrar a url
    admin.register(primaryAccount, contract.address, contenthash)
    // Aqui você implementa  ~/dapps/shared/contracts/test/info.json para uma url
    admin.registerUrl(primaryAccount, hash, url)
  }
});
```