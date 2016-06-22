# Listando contas e checando balanços

### Listando suas contas atuais

Da linha de comando, chame a CLI com:
```
$ geth account list
Account #0: {d1ade25ccd3d550a7eb532ac759cac7be09c2719}
Account #1: {da65665fc30803cb1fb7e6d86691e20b1826dee0}
Account #2: {e470b1a7d2c9c5c6f03bbaa8fa20db6d404a0c32}
Account #3: {f4dd5c3794f1fd0cdc0327a83aa472609c806e99}
```

Para listar suas contas em ordem de criação.

**Nota**:
Esta ordem pode mudar se você copiar `keyfiles` de outros nós. Então, tenha certeza de não confiar nos índices ou de checar e atualizar os índices da sua conta em seus scripts, se você copiar chaves.

Quando estiver usando o console:
```
> eth.accounts
['0x407d73d8a49eeb85d32cf465507dd71d507100c1']
```

Ou via RPC:
```
# Requisição
$ curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1} http://127.0.0.1:8545'

# Resultadoado
{
  "id":1,
  "jsonrpc": "2.0",
  "result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]
}
```

Se você quiser usar uma conta não interativamente, você precisará destrancá-la. Isto pode ser feito na linha de comando com a opção `--unlock`, que aceita como argumento uma lista de contas separadas por um espaço (em hex ou índice) para que você possa destrancar as contas programaticamente por uma sessão.
Isto é útil caso você queira usar sua conta de Dapps via RPC. `--unlock` irá destrancar a primeira conta. Isto é útil quando sua conta foi criada programaticamente, você não precisa saber a conta exata para destrancá-la. 

Destrancando uma conta:
```
geth --password <(echo this is not secret!) account new 
geth --password <(echo this is not secret!) --unlock primary --rpccorsdomain localhost --verbosity 6 2>> geth.log 
```

Ao invés do endereço da conta, você pode usar índices inteiros que se referem à posição do endereço na listagem da conta (e corresponde à ordem de criação).

A linha de comando permite que você destranque múltiplas contas. Nesse caso, o argumento para destrancar é uma lista de endereços de contas ou índices delimitados por um espaço em branco.

```
geth --unlock "0x407d73d8a49eeb85d32cf465507dd71d507100c1 0 5 e470b1a7d2c9c5c6f03bbaa8fa20db6d404a0c32"
```

Se essa construção for usada não interativamente, seu arquivo de senha precisará conter as respectivas senhas para as contas em questão, uma por linha.

No console, você também pode destrancar contas (uma por vez).

```
personal.unlockAccount(address, "password")
```

Note que nós NÃO recomendamos usar o argumento de senha aqui, já que o histórico do console é logado, então você pode comprometer a sua conta. Você foi avisado.


### Checando o balanço de contas

Para checar o balanço da sua conta *etherbase*:
```
> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
6.5
```

Imprime todos os balanços com uma função de JavaScript:
```
function checkAllBalances() { 
var i =0; 
eth.accounts.forEach( function(e){
 	console.log("  eth.accounts["+i+"]: " +  e + " \tbalance: " + web3.fromWei(eth.getBalance(e), "ether") + " ether"); 
i++; 
})
}; 
```

Ela pode, então, ser executada assim:
```
> checkAllBalances();
  eth.accounts[0]: 0xd1ade25ccd3d550a7eb532ac759cac7be09c2719 	balance: 63.11848 ether
  eth.accounts[1]: 0xda65665fc30803cb1fb7e6d86691e20b1826dee0 	balance: 0 ether
  eth.accounts[2]: 0xe470b1a7d2c9c5c6f03bbaa8fa20db6d404a0c32 	balance: 1 ether
  eth.accounts[3]: 0xf4dd5c3794f1fd0cdc0327a83aa472609c806e99 	balance: 6 ether
```

Como essa função irá desaparecer após reiniciar o `geth`, pode ser útil guardar as funções comumente usadas para serem lembradas depois. A função [loadScript](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#loadscript) torna isso muito simples.

Primeiro, salve a definição da função `checkAllBalances()` em um arquivo em seu computador. Por exemplo, `/Users/username/gethload.js`. Então, carregue o arquivo do console interativo:

```
> loadScript("/Users/username/gethload.js")
true
```

O arquivo irá modificar o seu ambiente JavaScript como se você tivesse digitado os comandos manualmente. Sinta-se livre para experimentar!
