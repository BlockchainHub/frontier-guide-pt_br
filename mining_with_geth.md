{% sections "cpu-mining-with-geth", "" %}
{% endsections %}

# Mineração com CPU usando Geth

Na Frontier, a primeira versão da Ethereum, você precisará apenas de a) uma GPU e b) um **client** da Ethereum, o Geth. Mineração com CPU será possível mas ineficiente demais para dar retorno financeiro.

No momento, o Geth inclui apenas uma mineradora de CPU e o time está testando um ramo com uma mineradora de GPU, mas isso não fará parte da Frontier.

A implementação em C++ da Ethereum também oferece uma mineradora de GPU como parte do Eth (o seu CLI), do AlethZero (sua GUI) e do EthMiner (uma mineradora independente).


_**NOTA:** Garanta que a sua cópia da blockchain está completamente sincronizada com a chain principal antes de começar a minerar. Caso contrário, você não estará minerando na chain principal.

Quando você começa o seu nó ethereum com `geth`, ele não está minerando por *default*. Para iniciá-lo em modo de mineração, você deve usar a opção `--mine` [na linha de comando](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options). O parâmetro `-minerthreads` pode ser usado para definir o número de segmentos de mineração em paralelo (tendo como padrão o número total de núcleos do processador).

`geth --mine --minerthreads=4`

Você também pode iniciar e parar a mineração em CPU usando o [console](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#adminminerstart). `miner.start` recebe um parâmetro opcional para o número de segmentos da mineradora.

```
> miner.start(8)
true
> miner.stop()
true
```

Note que minerar por ether de verdade apenas faz sentido se você estiver sincronizado com a rede (já que você minera por cima do bloco de consenso). Por isso, o "downloader"/sincronizador eth da blockchain irá atrasar a mineração até que a sincronização esteja completa e, após isso, a mineração começará automaticamente a não ser que você cancele intencionalmente com `miner.stop()`.

Para receber ether, você precisa ter o seu endereço **etherbase** (ou**coinbase**) definido.
Esse etherbase tem como *default* a sua [conta primária](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts). Se você não tiver um endereço etherbase, então `geth --mine` não iniciará.

Você pode definir o seu etherbase por linha de comando:

```
geth --etherbase 1 --mine  2>> geth.log // 1 is index: second account by creation order OR
geth --etherbase '0xa4d8e9cae4d04b093aac82e6cd355b6b963fb7ff' --mine 2>> geth.log
```

Você pode resetar o seu etherbase no console também:
```
miner.setEtherbase(eth.accounts[2])
```

Note que seu etherbase não precisa ser um endereço de uma conta local, apenas uma conta existente.

Existe uma opção [para adicionar dados extra](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#adminminersetextra) (32 bytes apenas) em seus blocos minerados. Por convenção, isto é interpretado como uma **string** *unicode*, então pode definir seu rótulo personalizado.
```
miner.setExtra("ΞTHΞЯSPHΞЯΞ")
...
debug.printBlock(131805)
BLOCK(be465b020fdbedc4063756f0912b5a89bbb4735bd1d1df84363e05ade0195cb1): Size: 531.00 B TD: 643485290485 {
NoNonce: ee48752c3a0bfe3d85339451a5f3f411c21c8170353e450985e1faab0a9ac4cc
Header:
[
...
        Coinbase:           a4d8e9cae4d04b093aac82e6cd355b6b963fb7ff
        Number:             131805
        Extra:              ΞTHΞЯSPHΞЯΞ
...
}
```

Veja também [esta proposta](https://github.com/ethereum/wiki/wiki/Extra-Data).

Você pode checar o seu **hash*rate* com `[miner.*hash*rate](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#adminminer*hash*rate)`. O resultado é em H/s (operações de Hash por segundo). 

```
> miner.*hash*rate
712000
```

Depois de minerar alguns blocos com sucesso, você pode checar o balanço de sua conta etherbase. Assumindo que sua conta etherbase é local:

```
> eth.getBalance(eth.coinbase).toNumber();
'34698870000000' 
```

Para gastar os seus ganhos em [combustível para transacionar](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions), você precisará ter essa conta destrancada.

```
> personal.unlockAccount(eth.coinbase)
Password
true
```

Você pode checar quais blocos são minerados por um determinado minerador (endereço) com o seguinte fragmento de código no console:

```
function minedBlocks(lastn, addr) {
  addrs = [];
  if (!addr) {
    addr = eth.coinbase
  }
  limit = eth.blockNumber - lastn
  for (i = eth.blockNumber; i >= limit; i--) {
    if (eth.getBlock(i).miner == addr) {
      addrs.push(i)
    }
  }
  return addrs
}
// varre os últimos 1000 blocos e retorna os números de blocos dos blocos minerados por seu coinbase.
// (mais precisamente, blocos cuja recompensa de mineração foi enviada para seu coinbase).   
minedBlocks(1000, eth.coinbase);
//[352708, 352655, 352559]
```

Note que acontecerá com frequência de você encontrar um bloco e, ainda assim, ele nunca chegar à *chain* canônica. Isso significa que, quando você inclui localmente um bloco minerado, o estado atual mostrará a recompensa de mineração creditada em sua conta. Entretanto, depois de um tempo, a melhor *chain* é descoberta e ocorre a mudança para a *chain* na qual o seu bloco não está incluído e, consequentemente, a recompensa não é creditada. Por isso, é bem possível que um minerador monitorando o seu balanço do coinbase verá o saldo flutuando um pouco. 
 
Os logs mostram blocos minerados localmente como confirmados após 5 blocos. No momento, você pode achar mais fácil e rápido gerar a lista dos seus blocos minerados a partir desses logs.

O sucesso da mineração depende da dificuldade de bloco definida. A dificuldade de bloco ajusta dinamicamente cada bloco de forma a regular o poder de **hash** da rede para que ele produza um tempo de bloco de 12 segundos. Suas chances de encontrar um bloco, portanto, dependem da sua taxa de **hash** (**hash*rate*) em relação à dificuldade. O tempo esperado que você precisa aguardar para encontrar um bloco pode ser estimado com o seguinte código:

**INCORRETO...CHECANDO**
```
etm = eth.getBlock("latest").difficulty/miner.*hash*rate; // tempo estimado em segundos
Math.floor(etm / 3600.) + "h " + Math.floor((etm % 3600)/60) + "m " +  Math.floor(etm % 60) + "s";
// 1h 3m 30s
```

Dada uma dificuldade de 3 bilhões, espera-se que uma CPU típica com 800H/s encontre um bloco a cada...?