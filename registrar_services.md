<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" -->

# Serviços *Registrar* 

A *chain* frontier vem com alguns serviços bem básicos de *baselayer*, especialmente o *registrar*. O *registrar* é composto de 3 componentes.  

* *GlobalRegistrar* para associar nomes (*strings*) a contas (endereços). 
* *HashReg* para associar *hashes* a *hashes* (mapear qualquer objeto a um *hash* de 'conteúdo'.
* *UrlHint* para associar *hashes* de conteúdo a uma dica para a localização do conteúdo. Isso é necessário apenas se o armazenamento do conteúdo não está endereçado ao conteúdo, caso contrário o *hash* de conteúdo já é o endereço do conteúdo. Se for usado, conteúdo buscado pela url deveria ter o *hash* equivalente ao *hash* do conteúdo. Para verificar a autenticidade do conteúdo, você pode checar se isso pode ser verificado.

## Criar e implementar *GlobalRegistrar*, *HashReg* e *UrlHint*


Se os os contratos *registrar* não estão inseridas no código (*hardcoded*) da blockchain (eles não são no momento de escrita), os *registrars* precisam ser implementados ao menos uma vez em cada *chain*.
~~If the registrar contracts are not hardcoded in the blockchain (they are not at the time of writing), the registrars need to be deployed at least once on every chain.~~

Se você está na *live chain* principal, o endereço do principal *global registrar* está no código (*hardcoded*) dos *clients* mais recentes e, portanto, **você não precisa fazer nada**. Se você quer alterar isso ou você está em uma *chain* privada você precisa implementar esses contratos ao menos uma vez:

```js
primary = eth.accounts[0];

globalRegistrarAddr = admin.setGlobalRegistrar("", primary);
hashRegAddr = admin.setHashReg("", primary);
urlHintAddr = admin.setUrlHint("", primary);
```

Você precisa minerar ou esperar até as transações terem sido todas pegas.
Inicialize o *registrar* no novo endereço e cheque se os outros nomes de *registrars* apontam para os endereços corretos:

```js 
registrar = GlobalRegistrar.at(globalRegistrarAddr);
primary == registrar.owner("HashReg");
primary == registrar.owner("UrlHint");
hashRegAddr == registrar.addr("HashReg");
urlHintAddr registrar.addr("UrlHint");
```

e os seguintes retornam código correto:

```js
eth.getCode(registrar.address);
eth.getCode(registrar.addr("HashReg"));
eth.getCode(registrar.addr("UrlHint"));
```

Da segunda vez em diante, na mesma *chain* assim como em outros nós, você simplesmente alimenta com os endereços GlobalRegistrars, o resto é lidado através dele.

```js
primary = eth.accounts[0];
globalRegistrarAddr = "0x225178b4829bbe7c9f8a6d2e3d9d87b66ed57d4f"

// define o endereço global registrar
admin.setGlobalRegistrar(globalRegistrarAddr)
// define o endereço HashReg via globalRegistrar
hashRegAddr = admin.setHashReg()
// define o endereço UrlHint via globalRegistrar
urlHintAddr = admin.setUrlHint()

// (re)define a variável registrar para uma instância de contrato GlobalRegistrar 
registrar = GlobalRegistrar.at(globalRegistrarAddr);
```

Se isso for bem sucedido, você deve ser capaz de checar com os seguintes comandos se o *registrar* retorna endereços:

```js 
registrar.owner("HashReg");
registrar.owner("UrlHint");
registrar.addr("HashReg");
registrar.addr("UrlHint");
```

e os seguintes retornam código correto:

```js
eth.getCode(registrar.address);
eth.getCode(registrar.addr("HashReg"));
eth.getCode(registrar.addr("UrlHint"));
```

## Usando os serviços *registrar*

Pode fornecer interfaces úteis entre contratos e dapps. 

### *Global registrar*

Para reservar um nome, registre um endereço de conta com ele, você precisa do seguinte: ~~To reserve a name register an account address with it, you need the following:~~

```
registrar.reserve.sendTransaction(name, {from:primary})
registrar.setAddress.sendTransaction (name, address, true, {from: primary})
```

Você precisa esperar todas as transações serem pegas (ou minerá-las a força se você está em uma *chain* privada). Para checar, você pergunta ao *registrar*:

```js
registrar.owner(name)
registrar.addr(name)
```

### *HashReg* e *UrlHint* 

*HashReg* e *UrlHint* podem ser usados com as seguintes abis:

```js
hashRegAbi = '[{"constant":false,"inputs":[],"name":"setowner","outputs":[],"type":"function"},{"constant":false,"inputs":[{"name":"_key","type":"uint256"},{"name":"_content","type":"uint256"}],"name":"register","outputs":[],"type":"function"}]'
urlHintAbi = '[{"constant":false,"inputs":[{"name":"_hash","type":"uint256"},{"name":"idx","type":"uint8"},{"name":"_url","type":"uint256"}],"name":"register","outputs":[],"type":"function"}]'
```

definindo as instâncias do contrato: 

```js
hashReg = eth.contract(hashRegAbi).at(registrar.addr("HashReg")));
urlHint = eth.contract(UrlHintAbi).at(registrar.addr("UrlHint")));
```

Associa um *hash* de conteúdo a um *hash* de chave:

```js
hashReg.register.sendTransaction(keyhash, contenthash, {from:primary})
```

Associa uma url a um *hash* de conteúdo:

```js
urlHint.register.sendTransaction(contenthash, url, {from:primary})
```

Para checar a resolução: 

```js 
contenthash = hashReg._hash(keyhash);
url = urlHint._url(contenthash);
