# Mineração com GPU

***

## Hardware

O algoritmo é instenivo em memória e, para encaixar o DAG na memória, ele precisa de 1 a 2 GB de RAM em cada GPU. Se você receber `Error GPU mining. GPU memory fragmentation?`, você não tem memória suficiente.

A mineradora de GPU é implementada em openCL, então GPUs AMD serão "mais rápidas" dos que as GPUs NVIDIA da mesma categoria.

ASICs e FPGAs são relativamente ineficientes e, por isso, desencorajados. 

Para conseguir openCL para o seu *chipset* e sua plataforma, tente:
* [AMD SDK openCL](http://developer.amd.com/tools-and-sdks/opencl-zone/amd-accelerated-parallel-processing-app-sdk)
* [NVIDIA CUDA openCL](https://developer.nvidia.com/cuda-downloads)

## No Ubuntu
### AMD

* http://developer.amd.com/tools-and-sdks/opencl-zone/amd-accelerated-parallel-processing
* http://developer.amd.com/tools-and-sdks/graphics-development/display-library-adl-sdk/

download: `ADL_SDK8.zip ` and `AMD-APP-SDK-v2.9-1.599.381-GA-linux64.sh`

```
./AMD-APP-SDK-v2.9-1.599.381-GA-linux64.sh
ln -s /opt/AMDAPPSDK-2.9-1 /opt/AMDAPP
ln -s /opt/AMDAPP/include/CL /usr/include
ln -s /opt/AMDAPP/lib/x86_64/* /usr/lib/
ldconfig
reboot
```

```
apt-get install fglrx-updates
// wget, tar, opencl
sudo aticonfig --adapter=all --initial
sudo aticonfig --list-adapters
* 0. 01:00.0 AMD Radeon R9 200 Series

* - Default adapter
```

### Nvidia
As seguintes instruções são, em grande parte, relevantes para qualquer sistema com Ubuntu 14.04 e uma GPU Nvidia.
[Configurando uma instância EC2 para mineração](https://forum.ethereum.org/discussion/comment/8889/#Comment_8889)

## No MacOSx

```
wget http://developer.download.nvidia.com/compute/cuda/7_0/Prod/local_installers/cuda_7.0.29_mac.pkg
sudo installer -pkg ~/Desktop/cuda_7.0.29_mac.pkg -target /
brew update
brew tap ethereum/ethereum
brew reinstall cpp-ethereum --with-gpu-mining --devel --headless --build-from-source
```

Você checa seu status de resfriamento:

    aticonfig --adapter=0 --od-gettemperature

## Software de Mineração

O lançamento oficial Frontier do `geth` suporta apenas uma mineradora de CPU nativamente. Nós estamos trabalhando em uma [mineradora de GPU](https://github.com/ethereum/go-ethereum/tree/gpuminer), mas ela poderá não estar disponível no lançamento Frontier.`geth`, no entanto, pode ser usado em conjunto com `ethminer`, usando uma mineradora independente como "mão-de-obra" e o `geth`para agendamento de comunicações via [JSON-RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC). 

A [implementação em C++](https://github.com/ethereum/cpp-ethereum/) (não lançada oficialmente), no entanto, tem uma mineradora de GPU. Ela pode ser usada do `eth`, `AlethZero` (GUI) e `ethMiner` (a mineradora independente). 

[Você pode instalar isto](https://github.com/ethereum/cpp-ethereum/wiki/Installing-*client*s) via ppa no linux, brew tap no MacOS ou usando a fonte. 

No MacOS:
```
brew install cpp-ethereum --with-gpu-mining --devel --build-from-source
```

No Linux:
```
apt-get install cpp-ethereum 
```

No Windows: 
https://github.com/ethereum/cpp-ethereum/wiki/Building-on-Windows

## Mineração de GPU com ethminer 
Para minerar com `eth`:

```
eth -m on -G -a <coinbase> -i -v 8 //
```

Para instalar `ethminer` da fonte:

```
cd cpp-ethereum
cmake -DETHASHCL=1 -DGUI=0
make -j4
make install
```

Para configurar a mineração com GPU você precisa de uma conta coinbase. Ela pode ser uma conta criada localmente ou remotamente.


### Usando ethminer com geth

```
geth account new
geth --rpc --rpccorsdomain localhost 2>> geth.log &
ethminer -G  // -G for GPU, -M for benchmark
tail -f geth.log
```

O `ethminer` se comunica com `geth` na porta 8545 (a porta RPC padrão no `geth`). Você pode alterar isso dando a [opção `--rpcport`](https://github.com/ethereum/go-ethereum/Command-Line-Options) ao `geth`.
O Ethminer encontrará o geth em qualquer porta. Note que você precisa configurar o cabeçalho do CORS com `--rpccorsdomain localhost`. Você também pode definir a porta no `ethminer` com `-F http://127.0.0.1:3301`. Definir a porta é necessário se você quiser várias instâncias de mineração no mesmo computador, apesar de isso não ter muito propósito. Se você está testando em um *cluster* privado, nós recomendamos que você use mineração com CPU.

Note também que você NÃO precisa dar ao `geth` a opção `--mine` ou iniciar a mineradora no console, a não ser que você queira fazer mineração de CPU POR CIMA da mineração de GPU. 


Se o *default* para `ethminer` não funcionar, tente especificar o aparelho de OpenCL com: `--opencl-device X` onde X é 0, 1, 2, etc.
Quando rodar `ethminer` com `-M` (referência), você deve ver algo desse tipo:

    Benchmarking on platform: { "platform": "NVIDIA CUDA", "device": "GeForce GTX 750 Ti", "version": "OpenCL 1.1 CUDA" }


    Benchmarking on platform: { "platform": "Apple", "device": "Intel(R) Xeon(R) CPU E5-1620 v2 @ 3.70GHz", "version": "OpenCL 1.2 " }

Para depurar o `geth`:

```
geth  --rpccorsdomain "localhost" --verbosity 6 2>> geth.log
```

Para depurar a mineradora: 

```
make -DCMAKE_BUILD_TYPE=Debug -DETHASHCL=1 -DGUI=0
gdb --args ethminer -G -M
```

**Nota** 
A informação de **hash*rate* não está disponível no `geth` quando minerando com GPU. Cheque seu *hash*rate com `ethminer`, `miner.*hash*rate` sempre reportará 0. 


### ethminer e eth

`ethminer` pode ser usada em conjunção com `eth` via rpc

```
eth -i -v 8 -j // -j for rpc
ethminer -G -M // -G for GPU, -M for benchmark
tail -f geth.log
```

Ou você pode usar `eth`para mineração de GPU por conta própria:

```
eth -m on -G -a <coinbase> -i -v 8 //
```

# Outros recursos:

* [ether-proxy, uma interface web para *rigs* de mineração](https://github.com/sammy007/ether-proxy)
  (suporta proxy de mineração individual e em *pool* com interface web e monitoramento de disponibilidade de aparelhos)
* [fórum do ethereum com FAQ de mineração e atualizações](https://forum.ethereum.org/discussion/197/mining-faq-live-updates)
* [vídeo de mineração de 
* yates randall](https://www.youtube.com/watch?v=CnKnclkkbKg)
* https://blog.ethereum.org/2014/07/05/stake/
* https://blog.ethereum.org/2014/10/03/slasher-ghost-developments-proof-stake/
* https://blog.ethereum.org/2014/06/19/mining/
* https://github.com/ethereum/wiki/wiki/Et*hash*
* [resultados de referência para mineração de GPU](https://forum.ethereum.org/discussion/2134/gpu-mining-is-out-come-and-let-us-know-of-your-bench-scores)
* [momento histórico](https://twitter.com/gavofyork/status/586623875577937922)
* [estatísticas de mineração em tempo real](https://etherapps.info/stats/mining)
* [monitorador netstat da rede ethereum](stats.ethdev.com)
