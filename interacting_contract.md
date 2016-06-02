{% sections "interacting-with-contracts", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" -->

# Interagindo com contratos

[`eth.contract`](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethcontract) pode ser usado para definir uma _class_ de contrato, que estará de acordo com a interface do contrato como descrito em sua [definição ABI](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI).

```js
var Multiply7 = eth.contract(contract.info.abiDefinition);
var myMultiply7 = Multiply7.at(address);
```

Agora todas as chamadas da função especificadas na abi estão disponíveis na instância do contrato. Você pode apenas chamar esses métodos na instância do contrato e encadeá-la a `sendTransaction(3, {from: address})` ou `call(3)`. A diferença entre os dois é que `call` faz um "*dry run*" localmente, no seu computador, enquanto `sendTransaction` iria na verdade submeter sua transação para inclusão na blockchain e os resultados de sua execução eventualmente farão parte do consenso global. Em outras palavras, use `call` se você está interessado apenas no valor de retorno e use `sendTransaction` se você se importa apenas com os "efeitos colaterais" no estado do contrato.

No exemplo acima, não há efeitos colaterais, portanto `sendTransaction` apenas queima gas e aumenta a entropia do universo. Toda funcionalidade "útil" é exposta por `call`:

```js
myMultiply7.multiply.call(6)
42
```

Agora suponha que este contrato não é seu, e você gostaria de ver a documentação ou de olhar o código fonte.
Isto é possível deixando disponível o pacote de informações do contrato e registrando-o na blockchain.
A API `admin` fornece métodos convenientes para alcançar esse pacote de qualquer contrato que tenha escolhido se registrar.
Para ver como isso funciona, leia sobre [Metadados de contrato](https://github.com/ethereum/wiki/wiki/Contract-metadata) ou leia a seção de implementação das informações de contrato deste documento.

```js
// pega as informações de contrato para endereço de contrato para fazer verificação manual
var info = admin.getContractInfo(address) // procura, alcança, decodifica
var source = info.source;
var abiDef = info.abiDefinition