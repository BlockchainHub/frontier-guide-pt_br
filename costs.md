{% sections "gas-and-transaction-costs", "" %}
{% endsections %}

<!--include "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md"-->

# Custos de gas e transação

Como você pagou por tudo isso, então? Por baixo dos panos, a transção especificou um limite de gas e um preço de gas, ambos os quais poderiam ter sido especificados diretamente no objeto de transação.

O limite de gas es†á lá para protegê-lo de códigos com bug rodando até que seus fundos se esgotem. O produto de `gasPrice` e `gas` representa a quantidade máxima de wei que você está disposto a pagar para executar a transação. O que você especificar como `gasPrice` será usado por mineradores para ranquear transações para inclusão na blockchain. É o preço em wei de uma unidade de gas, na qual operações de VM são precificadas.

A despesa de gas incorrida por rodar o seu contrato será comprada com o ether que você tem na sua conta a um preço que você especificou na transação com `gasPrice`. Se você não tem o ether para cobrir todos os requerimentos de gas para terminar de rodar o seu código, o processo aborta e todas as mudanças de estado intermediárias fazem *roll back* para a situação pré-transação. O gas usado até o momento em que a execução parou foi de fato usado, então o balanço de ether da sua conta diminuirá. Esses parâmetros podem ser ajustados nos campos do objeto de transação `gas` e `gasPrice`. O campo `value` é usado da mesma forma como em transações de transferência de ether entre contas normais. Em outras palavras, a transferência de fundos está disponível entre quaisquer duas contas, tanto normal (i.e. externamente controlada) ou contrato. Se seu contrato ficar sem fundos, você deve ver um erro de fundos insuficientes.

Para teste e brincar com contratos você pode usar a rede de teste ou [configurar um nó privado (ou cluster)](https://github.com/ethereum/go-ethereum/wiki/Setting-up-private-network-or-local-cluster) potencialmente isolado de todos os outros nós. Se depois você minerar, você pode garantir que sua transação será incluída no próximo bloco. Você pode ver transações pendentes com:

```js
eth.getBlock("pending", true).transactions
```

Você pode recuperar blocos por número (altura) ou pelo seu *hash*:

```js
genesis = eth.getBlock(0)
eth.getBlock(genesis.hash).hash == genesis.hash
true
```

Use `eth.blockNumber` para pegar a atual altura da blockchain e o "mais recente" parâmetro mágico para acessar o atual cabeçalho (bloco mais novo).

```js
currentHeight = eth.blockNumber()
eth.getBlock("latest").hash == eth.getBlock(eth.blockNumber).hash
true
```
