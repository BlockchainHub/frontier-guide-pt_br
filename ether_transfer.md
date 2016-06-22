<!--Source: https://raw.githubusercontent.com/wiki/ethereum/go-ethereum/Contracts-and-Transactions.md-->


# Transferência de Ether

Assumindo que a conta que você está usando tem fundos suficientes, enviar ether não poderia ser mais fácil. O que também é uma razão para você ter cuidado com isso! Você foi avisado.

```js
eth.sendTransaction({from: '0x036a03fc47084741f83938296a1c8ef67f6e34fa', to: '0xa8ade7feab1ece71446bed25fa0cf6745c19c3d5', value: web3.toWei(1, "ether")})
```

Note a unidade de conversão no campo `value`. Valores de transação são expressados em weis, a unidade de valor mais granular. Se você quiser usar alguma outra unidade (como `ether` no exemplo acima), use a função `web3.toWei` para conversão.

Também, esteja ciente de que a quantidade debitada da conta fonte será levemente superior ao que é creditado na conta de destino, que é o valor que foi especificado. A diferença é uma pequena taxa de transação, discutida em mais detalhes mais tarde.

Contratos podem receber transferências assim como contas controladas externamente, mas eles também podem receber transações mais complicadas que realmente rodam (parte de) seus códigos e atualizam seu estado. Para entender essas transações, um entedimento rudimentar de contratos é requerido.

