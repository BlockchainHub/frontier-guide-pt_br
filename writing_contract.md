<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md"-->

# Escrevendo um contrato

Contratos vivem na blockchain em um formato binário específico da Ethereum (*Ethereum Virtual Machine* (=EVM) bytecode). Entretanto, contratos são tipicamente escritos em alguma linguagem de alto nível, como [solidity](https://github.com/ethereum/wiki/wiki/Solidity-Tutorial), e depois compilado em *byte code* para ser carregado na blockchain.

Note que outras linguagens também existem, notavelmente [serpent](https://github.com/ethereum/wiki/wiki/Serpent) e [LLL](https://github.com/ethereum/cpp-ethereum/wiki/LLL). Legacy Mutan (uma antiga linguagem similar a C) não é mais oficialmente mantida.

## Recursos de linguagem

### Solidity

#### Documentações e tutoriais 
* [Tutorial da Ethereum wiki](https://github.com/ethereum/wiki/wiki/Solidity-Tutorial)
* [Solidity FAQ - Fórum Ethereum](https://forum.ethereum.org/discussion/1460/solidity-faq)
* [A linguagem de programação Solidity · ethereum/wiki](https://github.com/ethereum/wiki/wiki/The-Solidity-Programming-Language)
* [Ethereum ÐΞVcon-0: Solidity, Visão e Roadmap - Vídeo do YouTube](https://www.youtube.com/watch?v=DIqGDNPO5YM)
* [Dapps para inciantes](https://dappsforbeginners.wordpress.com/)
* [Tutorial 1](https://forum.ethereum.org/discussion/1634/tutorial-1-your-first-contract/p1)
* [Tutorial 2](https://forum.ethereum.org/discussion/1635/tutorial-2-rainbow-coin)
* [Tutorial 3 (API JavaScript para Ethereum)](https://forum.ethereum.org/discussion/1636/tutorial-3-introduction-to-the-javascript-api) (Outdated)
* [Solidity tutorial 1 por Eris Industries](https://eng.erisindustries.com/tutorials/2015/03/11/solidity-1/)
* [Dapp tutoriais por Andreas Olofsson (Eris Industries)](https://www.youtube.com/playlist?list=PL_kFomDrqPoZBu5uxd8OBGColQPYbuz3i)
* [Recursos de Eris de Solidity](https://github.com/eris-ltd/solidity-resources)


#### Exemplos
* [uma listagem de dapp](https://github.com/ethereum/wiki/wiki/FAQ#where-can-i-find-example-%C3%90apps)
* [Contratos de Solidity na Ethereum - Ether.Fund](https://ether.fund/contracts/solidity)
* [Ethereum dapp bin](https://github.com/ethereum/dapp-bin/)
* [Biblioteca padrão de Solidity](https://github.com/ethereum/wiki/wiki/Solidity-standard-library)
* [Whisper chat Dapp](https://github.com/ethereum/meteor-dapp-whisper-chat-client/tree/master/dist/deploy) written in meteor
* [árvore de estatísticas de ordem](https://github.com/drcode/ethereum-order-statistic-tree) by Conrad Bars

#### Compiladores

* [Compilador de Solidity em tempo real](https://chriseth.github.io/browser-solidity/)

### Serpent 

* [fonte no github](https://github.com/ethereum/serpent)
* [especificações de linguagem serpent](https://github.com/ethereum/wiki/wiki/Serpent)

## Ambientes e *frameworks de desenvolvimento de Contrato/Dapp

* [Mix standalone IDE](https://github.com/ethereum/wiki/wiki/Mix:-The-DApp-IDE) por ETHDEV
* in-browser [Cosmo](http://meteor-dapp-cosmo.meteor.com) que conecta ao `geth` via RPC. Por Nick Dodson
* [embark framework](https://github.com/iurimatias/embark-framework/) por Iuri Mathias
* [truffle](https://github.com/ConsenSys/truffle) por Tim Coulter
