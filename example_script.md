# Script de exemplo

O script de exemplo abaixo demonstra a maior parte das características discutidas nesse tutorial. Você pode rodá-lo com o [JSRE](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) como `geth js script.js 2>>geth.log` . Se você quer rodar esse teste em uma *chain* privada, entâo comece o `geth` com:

```
geth --maxpeers 0 --networkid 123456 --nodiscover --unlock primary js script.js 2>> geth.log
```

Note que `networkid` pode ser qualquer inteiro não-negativo arbitrário, 0 é sempre a rede ao vivo.

```
personal.newAccount("")

primary = eth.accounts[0];
balance = web3.fromWei(eth.getBalance(primary), "ether");
personal.unlockAccount(primary, "00");
// miner.setEtherbase(primary)

miner.start(8); admin.sleepBlocks(10); miner.stop()  ;

// 0xc6d9d2cd449a754c494264e1809c50e34d64562b
primary = eth.accounts[0];
balance = web3.fromWei(eth.getBalance(primary), "ether");

globalRegistrarTxHash = admin.setGlobalRegistrar("0x0");
//'0x0'
globalRegistrarTxHash = admin.setGlobalRegistrar("", primary);
//'0xa69690d2b1a1dcda78bc7645732bb6eefcd6b188eaa37abc47a0ab0bd87a02e8'
miner.start(1); admin.sleepBlocks(1); miner.stop();
//true
globalRegistrarAddr = eth.getTransactionReceipt(globalRegistrarTxHash).contractAddress;
//'0x3d255836f5f8c9976ec861b1065f953b96908b07'
eth.getCode(globalRegistrarAddr);
//...
admin.setGlobalRegistrar(globalRegistrarAddr);
registrar = GlobalRegistrar.at(globalRegistrarAddr);

hashRegTxHash = admin.setHashReg("0x0");
hashRegTxHash = admin.setHashReg("", primary);
txpool.status
miner.start(1); admin.sleepBlocks(1); miner.stop();
hashRegAddr = eth.getTransactionReceipt(hashRegTxHash).contractAddress;
eth.getCode(hashRegAddr);

registrar.reserve.sendTransaction("HashReg", {from:primary});
registrar.setAddress.sendTransaction("HashReg",hashRegAddr,true, {from:primary});
miner.start(1); admin.sleepBlocks(1); miner.stop();
registrar.owner("HashReg");
registrar.addr("HashReg");

urlHintTxHash = admin.setUrlHint("", primary);
miner.start(1); admin.sleepBlocks(1); miner.stop();
urlHintAddr = eth.getTransactionReceipt(urlHintTxHash).contractAddress;
eth.getCode(urlHintAddr);

registrar.reserve.sendTransaction("UrlHint", {from:primary});
registrar.setAddress.sendTransaction("UrlHint",urlHintAddr,true, {from:primary});
miner.start(1); admin.sleepBlocks(1); miner.stop();
registrar.owner("UrlHint");
registrar.addr("UrlHint");

globalRegistrarAddr = "0xfd719187089030b33a1463609b7dfea0e5de25f0"
admin.setGlobalRegistrar(globalRegistrarAddr);
registrar = GlobalRegistrar.at(globalRegistrarAddr);
admin.setHashReg("");
admin.setUrlHint("");

///// ///////////////////////////////

admin.stopNatSpec();
primary = eth.accounts[0];
personal.unlockAccount(primary, "00")

globalRegistrarAddr = "0xfd719187089030b33a1463609b7dfea0e5de25f0";
admin.setGlobalRegistrar(globalRegistrarAddr);
registrar = GlobalRegistrar.at(globalRegistrarAddr);
admin.setHashReg("0x0");
admin.setHashReg("");
admin.setUrlHint("0x0");
admin.setUrlHint("");


registrar.owner("HashReg");
registrar.owner("UrlHint");
registrar.addr("HashReg")
registrar.addr("UrlHint");


/////////////////////////////////////
eth.getBlockTransactionCount("pending");
miner.start(1); admin.sleepBlocks(1); miner.stop();

source = "contract test {\n" +
"   /// @notice will multiply `a` by 7.\n" +
"   function multiply(uint a) returns(uint d) {\n" +
"      return a * 7;\n" +
"   }\n" +
"} ";
contract = eth.compile.solidity(source).test;
txhash = eth.sendTransaction({from: primary, data: contract.code});

eth.getBlock("pending", true).transactions;

miner.start(1); admin.sleepBlocks(1); miner.stop();
contractaddress = eth.getTransactionReceipt(txhash).contractAddress;
eth.getCode(contractaddress);

multiply7 = eth.contract(contract.info.abiDefinition).at(contractaddress);
fortytwo = multiply7.multiply.call(6);

/////////////////////////////////

// registra um nome para o contrato
registrar.reserve.sendTransaction(primary,  {from: primary});
registrar.setAddress.sendTransaction("multiply7", contractaddress, true, {from: primary});
////////////////////////

admin.stopNatSpec();
filename = "/info.json";
contenthash = admin.saveInfo(contract.info, "/tmp" + filename);
admin.register(primary, contractaddress, contenthash);
eth.getBlock("pending", true).transactions;
miner.start(1); admin.sleepBlocks(1); miner.stop();

admin.registerUrl(primary, contenthash, "file://" + filename);
eth.getBlock("pending", true).transactions;
miner.start(1); admin.sleepBlocks(1); miner.stop();

////////////////////

// resgata o endereço do contrato usando a entrada global registrar com 'multply7'
contractaddress = registrar.addr("multiply7);
// resgata a informação usando a url 
info = admin.getContractInfo(contractaddress);
multiply7 = eth.contract(info.abiDefinition).at(contractaddress);
// tenta Natspec
admin.startNatSpec();
fortytwo = multiply7.multiply.sendTransaction(6, { from: primary });
```

