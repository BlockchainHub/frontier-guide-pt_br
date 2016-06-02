{% sections "interfaces", "" %}
{% endsections %}
<!-- source: https://raw.githubusercontent.com/wiki/ethereum/go-ethereum/Geth.md -->

# Geth

`geth` é a interface da linha de comando para rodar um nó completo da Ethereum implementado em Go.
É a principal entrega do [lançamento Frontier](https://github.com/ethereum/go-ethereum/wiki/Frontier).

## Capabilidades

Ao instalar e rodar `geth`, você pode fazer parte da rede ethereum frontier e
* minerar ether real 
* transferir fundos entre endereços
* criar contratos e enviar transações
* explorar o histórico de blocos
* e muito muito mais

## Instalação

As plataformas suportadas são Linux, Mac Os e Windows.

Nós suportamos dois tipos de instalação: instalação binária ou por script para usuários.
Veja as [instruções de instalação](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum) para instalações binárias e por script.

Desenvolvedores e comunidade entusiasta são aconselhados a ler o [Guia do Desenvolvedor](https://github.com/ethereum/go-ethereum/wiki/Developers%27-Guide), que contém instruções bem explidadas para fazer uma *build* manual da fonte (em qualquer plataforma), assim como dicas detalhadas de teste, monitoramento, contrubuição, "debugagem" e submissão de *pull requests* no Github.

## Interfaces

* Console de Javascript: `geth`pode ser iniciado com um console interativo que provê um ambiente de execução de javascript expondo uma API javascript para interagir com o seu nó. [A API do Console de Javascript ](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) inclui a API Dapp de javascript `web3`, assim como uma API admin adicional. 
* Servidor JSON-RPC: `geth` pode ser iniciado com um servidor json-rpc que expõe a [API JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC)
* [Opções na linha de comando](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options): documenta parâmetros da linha de comando, bem como os subcomandos.

## Documentação de uso de caso básico

* [Administração de contas](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts)
* [Mineração](https://github.com/ethereum/go-ethereum/wiki/mining)
* [Contratos e Transações](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions)

**Nota** A compra e venda de ether através de exchanges não é discutida aqui.

## Licença

O Protocolo Central da Ethereum foi licenciado sob a [*GNU Lesser General Public License*](https://www.gnu.org/licenses/lgpl.html). Todo software de **client* frontend*(sob [cmd](https://github.com/ethereum/go-ethereum/tree/develop/cmd)) é licenciado sob a [GNU General Public License](https://www.gnu.org/copyleft/gpl.html).

## Solucionando Problemas

Se algo der errado, leia primeiro a *checklist* [Troubleshooting](https://github.com/ethereum/go-ethereum/wiki/Troubleshooting) assim como a sessão de [FAQ](https://github.com/ethereum/go-ethereum/wiki/Troubleshooting). Se você ainda não achou sua resposta, abra uma *issue* no Github ou contacte nossa central de ajuda em helpdesk@ethereum.org.

## Reportando 

Questões de segurança devem ser enviadas para security@ethereum.org ou compartilhadas através de mensagens privadas com os desenvolvedores de um dos canais (veja Comunidade e Suporte).

O reporte de bugs não-sensíveis é bem-vindo no Github. Por favor, sempre diga qual a versão (no master) ou o *commit* da sua *build* (se em desenvolvimento), dê o máximo de detalhes possível sobre a situação e a anomalia que ocorreu. Forneça logs ou *stacktrace*, se possível.

## Contribuidores

A Ethereum é um trabalho conjunto entre ETHDEV e a comunidade.

Name or blame = lista de contribuidores:
* [go-ethereum](https://github.com/ethereum/go-ethereum/graphs/contributors)
* [cpp-ethereum](https://github.com/ethereum/cpp-ethereum/graphs/contributors)
* [web3.js](https://github.com/ethereum/web3.js/graphs/contributors)
* [et*hash*](https://github.com/ethereum/et*hash*/graphs/contributors)
* [netstats](https://github.com/cubedro/eth-netstats/graphs/contributors), 
[netintelligence-api](https://github.com/cubedro/eth-net-intelligence-api/graphs/contributors)

## Comunidade e Suporte

### Ethereum nas redes sociais

- Site principal: https://www.ethereum.org
- ETHDEV: https://ethdev.com
- Fórum: https://forum.ethereum.org
- Github: https://github.com/ethereum
- Blog: https://blog.ethereum.org
- Wiki: http://wiki.ethereum.org
- Twitter: http://twitter.com/ethereumproject
- Reddit: http://reddit.com/r/ethereum
- Meetups: http://ethereum.meetup.com
- Facebook: https://www.facebook.com/ethereumproject
- Youtube: http://www.youtube.com/ethereumproject
- Google+: http://google.com/+EthereumOrgOfficial

### IRC 

Canais *Freenote* do IRC:
* `#ethereum`: para discussões gerais
* `#ethereum-dev`: para perguntas e discussões específicas sobre desenvolvimento
* `##ethereum`: para offtopic e brincadeiras
* `#ethereumjs`: para perguntas relacionadas a web3.js e node-ethereum
* `#ethereum-markets`: Compra e Venda (*trading*)
* `#ethereum-mining` Mineração
* `#dappdevs`: Canal de desenvolvedores de Dapp
* `#ethdev`: *buildserver* etc

[Logs do IRC por ZeroGox](https://zerogox.com/bot/log)

### Gitter 

* [go-ethereum Gitter](https://gitter.im/ethereum/go-ethereum)
* [cpp-ethereum Gitter](https://gitter.im/ethereum/cpp-ethereum)
* [web3.js Gitter](https://gitter.im/ethereum/web3.js)
* [Documentação da ethereum para o projeto Gitter](https://gitter.im/ethereum/frontier-guide)

### Fórum

- [Fórum](https://forum.ethereum.org/categories/go-implementation)

### O robô (bot) ZeroGox

[Bot ZeroGox](https://zerogox.com/bot)

### Lista de e-mail dos desenvolvedores de Dapp

https://dapplist.net/

### Central de Ajuda 

No gitter, irc, skype ou enviando e-mail para helpdesk@ethereum.org.