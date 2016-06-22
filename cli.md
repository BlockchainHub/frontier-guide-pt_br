<!-- Source: https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options.md -->

# Opções da linha de comando

```
NOME:
   geth - a interface da linha de comando go-ethereum

USO:
   geth [opções] command [opções do comando] [argumentos...]
   
VERSÃO:
   1.4.0
   
COMANDOS:
   blocktest    carrega o arquivo teste de um bloco
   import       importa um arquivo da blockchain
   export       exporta a blockchain para um arquivo
   upgradedb    atualiza o banco de dados da *chainblock*
   removedb     Remove a blockchain e bancos de dade de estado
   dump         despeja um determinado bloco do armazenamento
   monitor      Monitorador do Geth: monitoramento e visualização das métricas do nó
   account      administração de contas
   wallet       carteira de pré-venda da ethereum
   makedag      gera dag do et*hash* (para teste)
   gpuinfo      informação da GPU
   gpubench     *benchmark* da GPU
   version      imprime os números de versões da ethereum
   init         *bootstrap* e inicialização de uma novo bloco gênesis (JSON)
   console      Console do Geth: ambiente interativo de JavaScript
   attach       Console do Geth: ambiente interativo de JavaScript (conecta ao nó)
   js           executa os arquivos JavaScript fornecidos na VM de JavaScript do Geth
   help, h      Mostra uma lista de comandos ou de ajuda para um comando
   
OPÇÕES DA ETHEREUM:
  --datadir "/home/youruser/.ethereum"  Diretório de dados para os bancos de dados e *keystore*
  --keystore                            Diretório para o *keystore* (default = dentro do datadir)
  --networkid "1"                       Identificador de rede (inteiro, 0=Olympic, 1=Frontier, 2=Morden)
  --olympic                             Rede Olympic: rede de teste pré-configurada e pré-lançada
  --testnet                             Rede Morden: rede de teste pré-configurada com nonces iniciais modificados (proteção contra repetição)
  --dev                                 Modo de desenvolvedor: rede privada pré-configurada com várias *flags* de *debug*
  --genesis                             Insere/sobrescreve o bloco gênesis (formato JSON)
  --identity                            Nome de nó personalizado
  --fast                                Permite sincronização rápida através de downloads de estado
  --lightkdf				Reduz o uso de RAM e CPU para *key-derivation* em troca de força de KDF
  --cache "0"                           Megabytes de memória alocados para cache interno (mín 16MB / forçado pelo banco de dados)
  --blockchainversion "3"               Versão da Blockchain (inteiro)
  
OPÇÕES DA CONTA:
  --unlock      Destranca uma conta (pode ser índice de criação) até o momento de sair do programa (pede a senha)
  --password    Arquivo de senha para usar com opções/subcomandos que precisem de senha
  
OPÇÕES DE API E CONSOLE:
  --rpc                                                                 Ativa o servidor HTTP-RPC
  --rpcaddr "localhost"                                                 Interface para escutar o servidor HTTP-RPC
  --rpcport "8545"                                                      Porta para escutar o servidor HTTP-RPC
  --rpcapi "eth,net,web3"                                               API's oferecidas na interface HTTP-RPC
  --rpccorsdomain                                                       Domínios de onde aceitar *requests* de origens diferentes (forçado pelo *browser*)
  --ws                                                                  Ativa o weboscket RPC server
  --wsaddr "localhost"                                                  Interface para escutar o servidor WS-RPC
  --wsport "8546"                                                       Porta para escutar o servidor WS-RPC
  --wsapi "eth,net,web3"                                                API's oferecidas na interface WS-RPC
  --wsorigins                                                           Origens de onde aceitar *requests* de websockets
  --ipcdisable                                                          Desativa o servidor IPC-RPC
  --ipcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3"      API's oferecidas na interface IPC-RPC
  --ipcpath "/home/youruser/.ethereum/geth.ipc"                         Nome do arquivo para  IPC socket/pipe
  --jspath "."                                                          Caminho *root* de JavaSript para `loadScript` e documento *root* para `admin.httpGet`
  --exec                                                                Executa uma declaração de JavaScript (apenas em conjunto com *console/attach*)
  --preload                                                             Lista separada por vírgulas de arquivos JavaScript para pré-carregar no console

OPÇÕES DE REDE:
  --bootnodes           URLs em formato enode separadas por espaço para o *bootstrap* do descobrimento P2P
  --port "30303"        Porta para escutar a rede
  --max*peer*s "25"       Número máximo de **peer*s* na rede (desativado pela rede se estiver definido como 0)
  --maxpend*peer*s "0"    Número máximo de tentativas de conexões pendentes (o *default* será usado se estiver definido como 0)
  --nat "any"           Mecanismo de mapeamento de porta NAT (any|none|upnp|pmp|extip:<IP>)
  --nodiscover          Desativa o mecanismo de descobrimento de **peer*s* (adição manual de **peer**)
  --nodekey             Arquivo com chave do P2P node key file
  --nodekeyhex          Chave em hexadecimal do nó P2P (para teste)
  
OPÇÕES DE MINERAÇÃO:
  --mine                        Ativa a mineração
  --minerthreads "8"            Número de segmentos de CPU para serem usados na mineração
  --minergpus                   Lista de GPUs para serem usadas na mineração (e.g. '0,1' usará as duas primeiras GPUs achadas)
  --autodag                     Ativa a pré-geração automática de DAG
  --etherbase "0"               Endereço público para a recompensa de mineração de blocos (*default* = primeira conta criada)
  --targetgaslimit "4712388"	O limite alvo de *gas* define o limite alvo artificial mínimo de *gas* para os blocos serem minerados
  --gasprice "20000000000"      Preço mínimo de *gas* a ser aceitado por minerar uma transação
  --extradata                   Dados extra do bloco definidos pelo minerador (*default* = versão do **client**)
  
OPÇÕES DE PREÇO DE GAS DO ORACLE:
  --gpomin "20000000000"        Preço de *gas* mínimo sugerido
  --gpomax "500000000000"       Preço de *gas* máximo sugerido
  --gpofull "80"                Limite de bloco cheio para o cálculo do preço de gas (%)
  --gpobasedown "10"            Base de preço de *gas* sugerida de razão redutora ~~Suggested gas price base step down ratio~~ (1/1000)
  --gpobaseup "100"             Base de preço de *gas* sugerida de razão crescente ~~Suggested gas price base step up ratio~~ (1/1000)
  --gpobasecf "110"             Fator de correção da base de preço de *gas* sugerida (%)
  
OPÇÕES DE MÁQUINA VIRTUAL:
  --jitvm               Ativa a  JIT VM
  --forcejit            Força a JIT VM a ter precedência
  --jitcache "64"       Quantidade de programas "cacheados" pela JIT VM
  
OPÇÕES DE LOG E DEBUG:
  --metrics			Ativa métricas de coleção e reporte
  --verbosity "3"               Verbosidade de log: 0-6 (0=silent, 1=error, 2=warn, 3=info, 4=core, 5=debug, 6=debug detail)
  --vmodule ""                  Verbosidade por módulo: lista separada por vírgulas de  `<module>=<level>`, onde `<module>` é um arquivo literal ou um padrão glog
  --backtrace ":0"              Pede um *stack trace* em uma declaração de log específica (e.g. "block.go:271")
  --pprof                       Ativa o servidor de diagnóstico no *localhost*
  --pprofport "6060"            Porta de escuta do servidor de diagnóstico
  --memprofilerate "524288"	Liga o diagnóstico de memória com a taxa determinada
  --blockprofilerate "0"	Liga o diagnóstico de bloco com a taxa determinada
  --cpuprofile 			Escreve o diagnóstico de CPU para o dado arquivo
  --trace 			Escreve o *trace* de execução do arquivo determinado
  
OPÇÕES EXPERIMENTAIS:
  --shh         Ativa *Whisper*
  --natspec     Ativa a confirmação de notificação NatSpec
  
OPÇÕES MISCELÂNEAS:
  --solc "solc" Comando de compilação de Solidity a ser usado
  --help, -h    mostra ajuda
```

Note que o *default* para `datadir` é específico da plataforma.Veja [backup & recuperação](https://github.com/ethereum/go-ethereum/wiki/Backup-&-restore) para mais informações.

## Exemplos

### Contas
Veja [administração de contas](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts)

Importa a carteira de pré-venda de ether para o seu nó (pede a sua senha):

    geth wallet import /path/to/my/etherwallet.json

Importa uma chave privada EC para uma conta ethereum (pede a sua senha):

    geth account import /path/to/key.prv

### Ambiente de execução JavaScript do Geth

Veja [Console javascript do geht](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console)

Bring up the geth javascript console:

    geth --verbosity 5 --jspath /mydapp/js console 2>> /path/to/logfile

Executa o javascript `test.js` usando uma API js e loga mensagens de debug em `/path/to/logfile`:

    geth --verbosity 6 js test.js  2>> /path/to/logfile

### Importa/exporta chains e despeja blocos

Importa a blockchain do arquivo:

    geth import blockchain.bin

### Atualiza o banco de dados da chainblock

Quando o algoritmo de consenso é mudado, blocos devem ser re-importados com o novo algoritmo. Geth irá informar o usuário com instruções sobre quando e como fazer isso quando for necessário.

    geth upgradedb

### Mineração e comunicação de rede

Inicia dois nós de mineração usando diretórios de dados diferentes escutando nas portas 30303 e 30304, respectivamente:

    geth --mine --minerthreads 4 --datadir /usr/local/share/ethereum/30303 --port 30303
    geth --mine --minerthreads 4 --datadir /usr/local/share/ethereum/30304 --port 30304
    
Inicia um **client** rpc na porta 8000:

    geth --rpc --rpcport 8000 --rpccorsdomain '"*"'

Inicia o **client** sem rede:

    geth --max*peer*s 0 --nodiscover --networdid 3301 js justwannarunthis.js

#### Resetando a blockchain

No datadir, deleta o diretório da blockchain. Para um exemplo:

    rm -rf /usr/local/share/ethereum/30303/blockchain

### Usagem de amostra em ambiente de teste

As linhas abaixo são destinadas apenas a rede de teste e ambientes seguros para uso com script não-interativo.

```
geth --datadir /tmp/eth/42 --password <(echo -n notsosecret) account new 2>> /tmp/eth/42.log
geth --datadir /tmp/eth/42 --port 30342  js <(echo 'console.log(admin.nodeInfo().NodeUrl)') > enode 2>> /tmp/eth/42.log
geth --datadir /tmp/eth/42 --port 30342 --password <(echo -n notsosecret) --unlock primary --minerthreads 4 --mine 2>> /tmp/eth/42.log
```

### Attach
Anexa um console à uma instância de geth em execução. Por *default*, isso acontece sobre o IPC no *endpoint default* do IPC mas, quando necessário, um *endpoint* customizado pode ser especificado:

```
geth attach                   # connect over IPC on default endpoint
geth attach ipc:/some/path    # connect over IPC on custom endpoint
geth attach http://host:8545  # connect over HTTP
geth attach ws://host:8546    # connect over websocket
```

### Init
Com o comando init é possível criar uma *chain* com um bloco gênesis e configuração de *chain* customizados (atualmente, apenas o bloco *homestead transition* pode ser configurado). Ele respeita o argumento `--datadir` e aceita um arquivo JSON descrevendo a configuração da *chain*.

```
geth --datadir <some/location/where/to/create/chain> init genesis.json
```

Examplo de arquivo genesis.json:
```
{
        "config": {
                "homesteadBlock": "10"
        },
        "nonce": "0",
        "difficulty": "0x20000",
        "mix*hash*": "0x00000000000000000000000000000000000000647572616c65787365646c6578",
        "coinbase": "0x0000000000000000000000000000000000000000",
        "timestamp": "0x00",
        "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "extraData": "0x",
        "gasLimit": "0x2FEFD8",
        "alloc": {}
}
```

Para usar a *chain* criada, inicie o geth com:
```
geth --datadir <some/location/where/to/create/chain>
```
