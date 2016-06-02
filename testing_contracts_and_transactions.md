{% sections "testing-contracts-and-transactions", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" -->


# Testando contratos e transações

Com frequência, você precisa apelar para uma estratégia de baixo nível para testar e debugar contratos e transações.
Essa seção introduz algumas ferramentas e práticas de debug que você pode usar.
Para testar contratos e transações sem consequências reais, é melhor você testar em uma blockchain privada. Isso pode ser alcançado configurando uma id de rede alternativa (escolha um inteiro único) e/ou desativar *peers*. É recomendado que, para teste, você use um diratório alternativo de dados e portas para que você nunca acidentalmente se choque com o seu nó que roda ao vivo (assumindo que ele rode usando *defaults*.
Iniciando o seu `geth` em modo de VM debug com *profiling* e o nível mais alto verbosidade de *logging* é recomendado:

```js
geth --datadir ~/dapps/testing/00/ --port 30310 --rpcport 8110 --networkid 4567890 --nodiscover --maxpeers 0 --vmdebug --verbosity 6 --pprof --pprofport 6110 console 2>> ~/dapp/testint/00/00.log
```

Antes que você possa submeter qualqer transação, você precisa minerar algum ether na sua blockchain privada e, para isso, você precisa de uma conta. Veja as seções sobre [Mineração](https://github.com/ethereum/go-ethereum/wiki/Mining) e [Contas](https://github.com/ethereum/go-ethereum/wiki/Managing-Your-Accounts)

```js
// cria conta. Irá pedir uma senha
personal.newAccount("mypassword");
// nomeie a sua conta primária, irá usar com frequência
primary = eth.accounts[0];
// checa o seu balanço (denominado em ether)
balance = web3.fromWei(eth.getBalance(primary), "ether");
```

```js
// assume uma conta primária destrancada existente
primary = eth.accounts[0];

// minera 10 blocos para gerar ether 

// iniciando mineradora
miner.start(8);
// dorme por 10 blocos
admin.sleepBlocks(10);
// então pára de minerar (apenas para não gerar calor em vão)
miner.stop()  ;
balance = web3.fromWei(eth.getBalance(primary), "ether");
```

Depois de criar transações, você pode processá-las a força com as seguintes linhas:

```
miner.start(1);
admin.sleepBlocks(1);
miner.stop()  ;
```

Você pode checar suas transações pendentes com: 

```js
// mostra pool de transações
txpool.status
// número de transações pendentes
eth.getBlockTransactionCount("pending");
// imprime todas as transações pendentes
eth.getBlock("pending", true).transactions
```

Se você submeter uma transações de criação de contrato, você pode checar se o código desejado foi inserido na atual blockchain:

```js
txhash = eth.sendTansaction({from:primary, data: code})
//... mineração
contractaddress = eth.getTransactionReceipt(txhash);
eth.getCode(contractaddress)
```
