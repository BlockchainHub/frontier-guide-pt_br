<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contract-Tutorial.md" -->
<!--Versão Publicada-->

## Seu primeiro cidadão: o *greeter*

Agora que você dominou os básicos da ethereum, vamos começar seu primeiro contrato sério. A frontier é um território vasto e aberto e, às vezes, você pode se sentir sozinho, então nossa primeira ordem de negócio será criar um pequeno companheiro automático para te cumprimentar sempre que você estiver se sentindo sozinho. Nós o chamaremos de "Greeter".

O *Greeter* é uma entidade digital inteligente que vive na blockchain e é capaz de ter conversar com qualquer um que interagir com ela, baseada em seus inputs. Ele pode não ser falador, mas é um ótimo ouvinte. Aqui está o código:

```js
contract mortal {
    /* Define variável owner do tipo endereço*/
    address owner;

    /* essa função é executada ao iniciar e define o dono do contrato */
    function mortal() { owner = msg.sender; }

    /* Função para recuperar os fundos no contrato */
    function kill() { if (msg.sender == owner) suicide(owner); }
}

contract greeter is mortal {
    /* define variável greeting do tipo string */
    string greeting;
    
    /* isto roda quando o contrato é executado */
    function greeter(string _greeting) public {
        greeting = _greeting;
    }

    /* função principal */
    function greet() constant returns (string) {
        return greeting;
    }
}
```

Você notará que há dois contratos diferentes neste código: _"mortal"_ e _"greeter"_. Isto é necessário porque Solidity (a linguagem de contrato de alto nível que nós estamos usando) tem **herança**, querendo dizer que um contrato pode herdar características de outro. Isso é muito útil para simplificar o código já que traços comuns de contratos não precisam ser reescritos todas as vezes e todos os contratos podem ser escritos em blocos menores e mais legíveis. Então, ao declarar que  _greeter is mortal_  você herdou todas as características do contrato "mortal" e manteve o código do *greeter* simples e fácil de ler.

A característica herdada _"mortal"_ significa simplesmente que o contrato *greeter* pode ser terminado por seu dono, para limpar a blockchain e recuperar os fundos trancados nela quando o contrato não for mais necessário. Contratos na ethereum são, por *default*, imortais e não possuem donos, significando que, uma vez implementados, o autor não possui mais nenhum privilégio especial. Considere isso antes de implementar.

### Instalando um compilador

Antes de você ser capaz de implementá-lo, no entanto, você precisa de suas coisas: o código do compilador e a *Application Binary Interface*, que é um tipo de template de referência que define como se deve interagir com o contrato.

O primeiro você consegue pegar usando um compilador. Você deve ter um compilador de solidity *built in* no seu console do geth. Para testá-lo, use este comando:

    eth.getCompilers()

Se você o tiver instalado, ele deve dar um *output* deste tipo:

    ['Solidity' ]

Se, caso contrário, o comando retornar um erro, então você precisa instalá-lo.

#### Usando um compilador online

Se você não tem solC instalado, nós temos um [compilador de solidity online](https://chriseth.github.io/cpp-ethereum/) disponível. Mas esteja ciente de que **se o compilador estiver comprometido, seu contrato não estará salvo**. Por essa razão, se você quiser usar o compilador online, nós o encorajamos a [hospedar o seu próprio](https://github.com/chriseth/browser-solidity).

#### Instalar SolC no Ubuntu

Pressione control+c para sair do console (ou digite _exit_) e retorne para a linha de comando. Abra o terminal e execute estes comandos:

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc
    which solc

Anote o caminho dado pela última linha, você precisará dele logo.

#### Instalar SolC no Mac OSX

Você precisa de [brew](http://brew.sh) para instalar no seu mac

    brew install cpp-ethereum
    brew linkapps cpp-ethereum
    which solc

Anote o caminho dado pela última linha, você precisará dele logo.

#### Instalar SolC no Windows

Você precisa de [chocolatey](http://chocolatey.org) para instalar solc.

    cinst -pre solC-stable

Windows é mais complicado que isso, você precisará esperar um pouco mais.

Se você tem o compilador de solidity SolC instalado, você precisa agora reformatar removendo espaços para ele caber em uma variável *string* [(há algumas ferramentas online que fazem isso)](http://www.textfixer.com/tools/remove-line-breaks.php):

#### Compilar da fonte

    git clone https://github.com/ethereum/cpp-ethereum.git
    mkdir cpp-ethereum/build
    cd cpp-ethereum/build
    cmake -DJSONRPC=OFF -DMINER=OFF -DETHKEY=OFF -DSERPENT=OFF -DGUI=OFF -DTESTS=OFF -DJSCONSOLE=OFF ..
    make -j4 
    make install
    which solc

#### Linkando seu compilador no Geth

Agora [retorne ao console](../geth) e digite este comando para instalar solC, substituindo _path/to/solc_ pelo caminho que você pegou no último comando que você fez:

    admin.setSolc("path/to/solc")

Agora digite novamente:

    eth.getCompilers()

Se você agora tem solC instalado, então parabéns, você pode continuar lendo. Se não, então vá para os nossos [fórums](http://forum.ethereum.org) ou [subreddit](http://www.reddit.com/r/ethereum) e nos repreenda por falhar em tornar esse processo mais fácil.


### Compilando seu contrato 


Se você tem o compilador instalado, você precisa agora reformatar seu contrato removendo separações de linha para que ele caiba em uma variável *string* [(há algumas ferramentas online que fazem isso)](http://www.textfixer.com/tools/remove-line-breaks.php):

    var greeterSource = 'contract mortal { address owner; function mortal() { owner = msg.sender; } function kill() { if (msg.sender == owner) suicide(owner); } } contract greeter is mortal { string greeting; function greeter(string _greeting) public { greeting = _greeting; } function greet() constant returns (string) { return greeting; } }'

    var greeterCompiled = web3.eth.compile.solidity(greeterSource)

Você já compilou seu código. Agora você precisa prepará-lo para implementação, isso inclui definir algumas variáveis, como qual é o seu cumprimento. Edite a linha abaixo para algo mais interessante do que "Hello World" e execute estes comandos:

    var _greeting = "Hello World!"
    var greeterContract = web3.eth.contract(greeterCompiled.greeter.info.abiDefinition);

    var greeter = greeterContract.new(_greeting,{from:web3.eth.accounts[0], data: greeterCompiled.greeter.code, gas: 1000000}, function(e, contract){
      if(!e) {

        if(!contract.address) {
          console.log("Contract transaction send: TransactionHash: " + contract.transactionHash + " waiting to be mined...");

        } else {
          console.log("Contract mined! Address: " + contract.address);
          console.log(contract);
        }

      }
    })

#### Usando o compilador online

Se você não tem solC instalado, você pode simplesmente usar o compilador online. Copie o código fonte acima para o [compilador solidity online](https://chriseth.github.io/cpp-ethereum/) e depois o seu código deve aparecer no painel esquerdo. Copie o código na caixa nomeada **Geth deploy** para um arquivo de texto. Agora mude a primeira linha do seu cumprimento:

    var _greeting = "Hello World!"

Agora você pode colar o texto resultante na sua janela do geth. Aguarde até 30 segundos e você verá uma mensagem assim:

    Contract mined! address: 0xdaa24d02bad7e9d6a80106db164bad9399a0423e 

Você provavelmente será solicitado a senha que você pegou no início, pois você precisa pagar pelos custos de gas da implementação do contrato. Este contrato é estimado custar 172 mil gas para implementação (de acordo com o [compilador solidity online](https://chriseth.github.io/cpp-ethereum/)), na data de escrita, gas na rede teste está avaliado entre 1 e 10 microethers por unidade de gas (apelidado "szabo" = 1 seguido de 12 zeros em wei). Para saber o preço mais recente em ether você pode ver os [preços mais recentes de gas na página de estatísticas da rede](https://stats.ethdev.com) e multiplicar ambos os termos. 

**Note que o custo não é pago aos [desenvolvedores ethereum](../foundation), e sim aos _Mineradores_, pessoas que estão rodando computadores que mantêm a rede funcionando. O preço de gas é definido por mercado das atuais oferta e demanda de computação. Se os preços de gas aumentarem muito, você pode ser um minerador e reduzir o preço que você está pedindo.**

Depois de menos de um minuto, você deveria ter um log com o endereço do contrato, isso significa que você implementou o contrato com sucesso. Você pode verificar o código implementado (compilado) usando este comando:

    eth.getCode(greeter.address)

Se ele retornar qualquer coisa além de "0x" então parabéns! Seu pequeno *Greeter* está vivo! Se o contrato for criado novamente (fazendo outra eth.sendTransaction), ele será publicado em um novo endereço. 


### Rodar o *Greeter*

Para chamar o seu robô, digite o seguinte comando no seu terminal:

    greeter.greet();

Já que essa chamada não muda nada na blockchain, ela retorna instantaneamente e sem custo de gas. Você deve vê-la retornando o seu cumprimento:

    'Hello World!'


### Fazendo outras pessoas interagirem com o seu código

Para que outras pessoas rodem o seu contrato elas precisam de duas coisas: o endereço onde o contrato está localizado e a ABI (*Application Binary Interface*) que é um tipo de manual de usuário, descrevendo o nome de suas funções e como chamá-las. Para ter os dois, rode estes comandos:

    greeterCompiled.greeter.info.abiDefinition;
    greeter.address;

Então você pode instanciar um objeto javascript que pode ser usado para chamar o contrato em qualquer máquina conectada à rede. Substitua 'ABI' e 'address' para criar um objeto de contrato em javascript:

    var greeter = eth.contract(ABI).at(Address);

Este exemplo em particular, pode ser instanciado por qualquer um apenas chamando:

    var greeter2 = eth.contract([{constant:false,inputs:[],name:'kill',outputs:[],type:'function'},{constant:true,inputs:[],name:'greet',outputs:[{name:'',type:'string'}],type:'function'},{inputs:[{name:'_greeting',type:'string'}],type:'constructor'}]).at('greeterAddress');

Substitua _greeterAddress_ pelo endereço do seu contrato.


**Dice: se o compilador de solidity não está corretamente instalado na sua máquina, você pode pegar a ABI do compilador online. Para isso, use o código abaixo com cuidado substituindo _greeterCompiled.greeter.info.abiDefinition_  pela abi do seu compilador.**


### Limpando a sua bagunça> 

Você deve estar muito animado de ter seu primeiro contrato em funcionamento, mas essa animação acaba, às vezes, quando os donos continuam escrevendo outros contratos, levando à situação desagradável de se ver contratos abandonados na blockchain. No futuro, um aluguel da blockchain pode ser implementado para aumentar a escalabilidade da blockchain mas, por enquanto, seja um bom cidadão e humanamente desligue seus *bots* abandonados. 

Diferente da última vez, nós não faremos uma chamada pois queremos mudar algo na blockchain. Isso requer que uma transação seja enviada à rede e que uma taxa seja paga pelas as mudanças feitas. O suicídio é subsidiado pela rede então custará bem menos do que uma transação convencional.

    greeter.kill.sendTransaction({from:eth.accounts[0]})

Você pode verificar que o trabalho está feito simplementes vendo se isto retorna 0:

    eth.getCode(greeter.contractAddress)

Note que todo contrato tem que implementar sua própria cláusula de morte. Nesse caso em particular, apenas a conta que o criou pode matá-lo.

Se você não adicionar uma cláusula de morte, ele poderia potencialmente viver para sempre (ou pelo menos até os contratos da frontier serem todos apagados) independentemente de você ou qualquer fronteira terrestre. Então, antes de colocá-lo ao viv,o cheque o que suas leis locais falam sobre o assunto, incluindo qualquer possível limitação na exportação de tecnologia, restrições de expressão e talvez qualquer legislação sobre direitos civis de seres sentientes. Trate seus *bots* humanamente.
