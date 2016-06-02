{% sections "creating-and-deploying-a-contract", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" -->

# Criando e implementando um contrato

Agora que você tem tanto uma conta destrancada quando alguns fundos, você pode criar um contrato na blockchain [enviando uma transação](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethsendtransaction) para o endereço vazio com o código EVM como dado. Simples, né?

```js
primaryAddress = eth.accounts[0]
MyContract = eth.contract(abi);
contact = MyContract.new(arg1, arg2, ...,{from: primaryAddress, data: evmCode})
```

Todos os dados binários são serializados em forma hexadecimal. Strings em hex tem um prefixo `0x`.

Note que `arg1, arg2, ...` são os argumentos para o construtor de contrato, caso ele aceite algum.

Note também que esse passo requer que você pague pela execução. Seu balança na conta (que você bota como remetende no campo `from`) será reduzido de acordo com as regras de gas da VM uma vez que a transação chegue a um bloco. Mais sobre isso depois. Depois de um certo tempo, sua transação deve aparecer incluída em um bloco confirmando que o estado que ela trouxe foi consenso. Seu contrato agora vive na blockchain.

O modo assíncrono de fazer o mesmo se parece com isso:

```js
MyContract.new([arg1, arg2, ...,]{from: primaryAccount, data: evmCode}, function(err, contract) {
  if (!err && contract.address)
    console.log(contract.address); 
});
```

