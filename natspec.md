{% sections "natspec", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" -->

# NatSpec 

Essa seção irá elaborar ainda mais o que você pode fazer com a contrução de contratos e transações em um protocolo NatSpec. Solidity implementa comentários inteligentes em estilo doxigen, que pode ser usado depois para gerar vários meta documentos da fachada do código. Um uso de caso desse tipo é gerar mensagens customizadas para confirmação de transação que *clients* podem mostrar ao usuário. 

Então agora nós extendemos o contrato `multiply7` com um comentário inteligente especificando uma mensagem de confirmação customizada (notificação).

```js
contract test {
   /// @notice Will multiply `a` by 7.
   function multiply(uint a) returns(uint d) {
       return a * 7;
   }
}
```

Esse comentário tem expressões entre *backticks* que serão avaliados no momento em que a mensagem de confirmação de transação for apresentada ao usuário. As variáveis que se referem aos parâmetros das chamadas de métodos são, então, instanciados de acordo com os dados de transação enviados pelo usuário (ou a dapp do usuário). O suporte da NatSpec para notificações de confirmação está totalmente implementado no `geth`. NatSpec depende de ambos a definição abi assim como o componente userDoc para gerar confirmações apropriadas. Portanto, para acessar isso, o contrato precisa ter registrado suas informações de contrato, como descrito acima.

Vamos ver um exemplo completo. Como um desenvolvedor de *smart contract* muito consciente, você primeiro cria o seu contrato e o implementa de acordo com os passos recomendados acima:

```js
source = "contract test {
   /// @notice Will multiply `a` by 7.
   function multiply(uint a) returns(uint d) {
       return a * 7;
   }
}"
contract = eth.compile.solidity(source).test
MyContract = eth.contract(contract.info.abiDefinition)
contenthash = admin.saveInfo(contract.info, "~/dapps/shared/contracts/test/info.json")
MyContract.new({from: primary, data: contract.code}, function(error, contract){
  if(!error && contract.address) {
    admin.register(primary, contract.address, contenthash)
    // put it up on your favourite oldworld site:
    admin.registerUrl(contentHash, "http://dapphub.com/test/info.json")
  }
});
```

Note que se usarmos um sistema de armazenamento  direcionado ao conteúdo como *swarm* o segundo passo é desnecessário, já que o *contenthash* é (deterministicamente traduzido para) o próprio endereço único do conteúdo.

Para o objetivo de um simples exemplo, use apenas o esquema de url de arquivo (não exatamente a nuvem, mas irá mostrar como funciona) sem precisar implementar. 

```js
admin.registerUrl(contentHash, "file:///home/nirname/dapps/shared/contracts/test/info.json")
```

Agora você acabou como um desenvolvedor, então troque de papéis e finja que você é um usuário enviando uma transação para o infame contrato `multiply7`.

Vocẽ precisa iniciar o *client* com a flag `--natspec` para ativar confirmações inteligentes e a busca de *contractInfo*. Você também pode definir isso no console com `admin.startNatSpec()` e `admin.stopNatSpec()`.

```
geth --natspec --unlock primary console 2>> /tmp/eth.log
```

Agora digite no console:

```js
// obtém a definição abi para o seu contrato
var info = admin.getContractInfo(address)
var abiDef = info.abiDefinition
// instancia um contrato para transações
var Multiply7 = eth.contract(abiDef);
var myMultiply7 = Multiply7.at(address);
```

E agora tente de fato enviar uma transação:

```js
> myMultiply7.multiply.sendTransaction(6, {from: eth.accounts[0]})
NatSpec: Will multiply 6 by 7. 
Confirm? [y/n] y
>
```

Quando essa transação for incluída em um bloco, em algum lugar no computador de um minerador sortudo, 6 será multiplicado por 7, com o resultado ignorado. Missão cumprida.

Se a transação não for minerada, nós a podemos ver com:

```js
eth.pendingTransactions
```

Isto acumula todas as transações enviadas, até as que foram rejeitadas e não estão incluídas no atual bloco minerado (estado *trans*). Essas últimas podem ser mostradas por:

```js
eth.getBlock("pending", true).transactions()
```

Se você identificar o índice da sua transação rejeitada, você pode reenviá-la com o limite de gas e o preço de gas modificados (ambos parâmetros opcionais):

```js
tx = eth.pendingTransactions[1]
eth.resend(tx, newGasPrice, newGasLimit)
```
