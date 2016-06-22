# Enviando ether

O jeito básico de enviar uma transação simples de ether com o console é o seguinte:
```js
> eth.sendTransaction({from:sender, to:receiver, value: amount})
```


Usando o JavaScript *built-in*, você pode facilmente definir variáveis para guardar esses valores. Por exemplo:

```js
> var sender = eth.accounts[0];
> var receiver = eth.accounts[1];
> var amount = web3.toWei(0.01, "ether")
```

Alternativamente, você pode compor a transação em uma única linha com:

```js
> eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1], value: web3.toWei(0.05, "ether")})
Please unlock account d1ade25ccd3d550a7eb532ac759cac7be09c2719.
Passphrase: 
Account is now unlocked for this session.
'0xeeb66b211e7d9be55232ed70c2ebb1bcc5d5fd9ed01d876fac5cff45b5bf8bf4'
```

A transação resultante é:
`0xeeb66b211e7d9be55232ed70c2ebb1bcc5d5fd9ed01d876fac5cff45b5bf8bf4`

Se a senha estivesse errada, você receberia, em vez disso, um erro:

```js
error: could not unlock sender account
```
