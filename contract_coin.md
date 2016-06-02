{% sections "the-coin", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contract-Tutorial.md" -->
<!--Versão Publicada-->

## A moeda 

O que é uma moeda? Moedas são muito mais interessantes e úteis do que parecem, elas são essencialmente apenas um *token* trocável, mas podem se tornar muito mais, dependendo de como você as use. Seu valor depende do que você faz com ela: um *token* pode ser usado para controlar acesso (**um bilhete de entrada**), pode ser usado para direitos de voto em uma organização (**uma share**), pode ser um representante de um bem segurado por um terceiro (**um certificado de posse**) ou até pode ser simplesmente usado como um meio de troca de valor dentro de uma comunidade (**um dinheiro**)

Você poderia fazer todas essas coisas criando um servidor centralizado, mas usar um *token* de contratos ethereum vem com algumas funcionalidades grátis: em primeiro lugar, é um serviço descentralizado e *tokens* ainda podem ser trocados mesmo se o serviço original parar de funcionar por algum motivo. O código pode garantir que nenhum *token* será criado a mais do que o estipulado no código original. Finalmente, por ter cada usuário guardando o seu próprio *token*, isso elimina o cenário onde uma única invasão em um servidor pode resultar na perda de fundos de milhares de clientes.

Você poderia criar o seu próprio *token* em uma blockchain diferente, mas criar na ethereum é mais fácil - então você pode focar sua energia na inovação que fará a sua moeda se destacar - e é mais seguro, já que a sua segurança é provida por todos os mineradores que estão suportando a rede ethereum. Finalmente, ao criar o seu *token* na ethereum, sua moeda irá ser compatível com qualquer outro contrato rodando na ethereum.

### O código

Este é o código para o contrato que estamos construindo:
 
    contract token { 
        mapping (address => uint) public coinBalanceOf;
        event CoinTransfer(address sender, address receiver, uint amount);
      
      /* Inicializa o contrato com uma oferta inicial de tokens para o criador do contrato */
      function token(uint supply) {
            coinBalanceOf[msg.sender] = supply;
        }
      
      /* Função de troca muito simples */
        function sendCoin(address receiver, uint amount) returns(bool sufficient) {
            if (coinBalanceOf[msg.sender] < amount) return false;
            coinBalanceOf[msg.sender] -= amount;
            coinBalanceOf[receiver] += amount;
            CoinTransfer(msg.sender, receiver, amount);
            return true;
        }
    }

Se você já programou alguma vez, você não terá dificuldade em entender o que ele faz: é um contrato que gera 10 mil *tokens* para o criador do contrato e depois permite que qualquer um com saldo suficiente possa enviar para outros. Esses *tokens* são a menor unidade de troca e não podem ser subdivididos, mas para o usuário final eles poderiam ser apresentados como 100 unidades que podem ser divididas em 100 subunidades, então possuir um único *token* representaria ter 0.01% do total. Se a sua aplicação precisa de uma divisibilidade maior, então aumente a quantidade inicial emitida.

Neste exemplo, nós declaramos a variável "coinBalanceOf" como publica, isso irá criar automaticamente uma função que checa qualquer balanço de conta.

### Compilar e implementar

**Então vamos rodar isso!**

    var tokenSource = ' contract token { mapping (address => uint) public coinBalanceOf; event CoinTransfer(address sender, address receiver, uint amount); /* Inicializa contrato com oferta inical de tokens para o criador do contrato */ function token(uint supply) { coinBalanceOf[msg.sender] = supply; } /* Função de troca muito simples */ function sendCoin(address receiver, uint amount) returns(bool sufficient) { if (coinBalanceOf[msg.sender] < amount) return false; coinBalanceOf[msg.sender] -= amount; coinBalanceOf[receiver] += amount; CoinTransfer(msg.sender, receiver, amount); return true; } }'

    var tokenCompiled = eth.compile.solidity(tokenSource)

Agora vamos configurar o contrato, assim como fizemos na seção anterior. Mude a "initial Supply" (oferta inicial) para a quantidade de *tokens* não-divisíveis que você quer criar. Se você quer ter unidades divisíveis, você deveria fazê-lo no *frontend* do usuário mas mantê-los representados na unidade de conta mínima.

    var supply = 10000;
    var tokenContract = web3.eth.contract(tokenCompiled.token.info.abiDefinition);
    var token = tokenContract.new(
      supply,
      {
        from:web3.eth.accounts[0], 
        data:tokenCompiled.token.code, 
        gas: 1000000
      }, function(e, contract){
        if(!e) {

          if(!contract.address) {
            console.log("Contract transaction send: TransactionHash: " + contract.transactionHash + " waiting to be mined...");

          } else {
            console.log("Contract mined! Address: " + contract.address);
            console.log(contract);
          }

        }
    })

#### Compilador Online

**Se você não tem solC instalado, você pode simplesmente usar o compilador online.** Copie o código do contrato para o [compilador de solidity online](https://chriseth.github.io/cpp-ethereum/), se não houver erros no contrato você deve ver uma caixa de texto nomeada **Geth Deploy**. Copie o conteúdo para um arquivo de texto para que você possa mudar a primeira linha para definir a oferta inicial, assim:  

    var supply = 10000;

Agora você pode copiar o texto resultante na sua janela do geth. Aguarde até 300 segundos e você verá uma mensagem assim:

    Contract mined! address: 0xdaa24d02bad7e9d6a80106db164bad9399a0423e 


### Checar balanço assistindo transferências de moedas

Se tudo funcionou corretamente, você deve ser capaz de checar seu próprio balanço com:

    token.coinBalanceOf(eth.accounts[0]) + " tokens"

Ele deve ter todos os 10 000 *tokens* que foram criados quando o contrato foi publicado. Como não há nenhuma outra forma definida para novas moedas serem emitidas, essas são as únicas que vão existir.
Voce pode configurar um **Watcher** (observador) para reagir sempre que alguém envia uma moeda usando o seu contrato. Aqui está a forma de fazer isso:

    var event = token.CoinTransfer({}, '', function(error, result){
      if (!error)
        console.log("Coin transfer: " + result.args.amount + " tokens were sent. Balances now are as following: \n Sender:\t" + result.args.sender + " \t" + token.coinBalanceOf.call(result.args.sender) + " tokens \n Receiver:\t" + result.args.receiver + " \t" + token.coinBalanceOf.call(result.args.receiver) + " tokens" )
    });

### Enviando moedas

Agora, é claro que esses *tokens* não serão muito úteis se você guardar todos, então para enviar a alguém, use esse comando:

    token.sendCoin.sendTransaction(eth.accounts[1], 1000, {from: eth.accounts[0]})

Se um amigo registrou um nome no *registrar* você pode enviar sem saber o endereço dele, assim:

    token.sendCoin.sendTransaction(registrar.addr("Alice"), 2000, {from: eth.accounts[0]})


Note que nossa primeira função **coinBalanceOf** foi chamada diretamente na instância do contrato e retornou um valor. Isso foi possível já que essa era uma simples operação de leitura que não causa nenhuma mudança de estado e que executa localmente e síncronamente. Nossa segunda função **sendCoin** precisa de uma chamada **.sendTransaction()**. Como essa função é feita para mudar o estado (operação de escrita), ela é enviada como uma transação para a rede para ser pega pelos mineradores e incluída na blockchain canônica. Como resultado, o estado de consenso de todos os nós participantes irá refletir adequadamente a mudança de estado resultante da execução da transação. O endereço do remetente precisa ser enviado como parte da transação para financiar o combustível necessário para rodar a transação. Agora, espere um minuto e cheque o balanço de ambas contas: 

    token.coinBalanceOf.call(eth.accounts[0])/100 + "% of all tokens"
    token.coinBalanceOf.call(eth.accounts[1])/100 + "% of all tokens"
    token.coinBalanceOf.call(registrar.addr("Alice"))/100 + "% of all tokens"


### Sugestões de melhora

No momento, essa criptomoeda é bem limitada pois só existirão para sempre 10.000 moedas e todas são controladas pelo criador da moeda, mas você pode mudar isso. Você pode, por exemplo, recompensar os mineradores da ethereum criando uma transação que irá recompensar quem tiver encontrado o atual bloco:

    mapping (uint => address) miningReward;
    function claimMiningReward() {
      if (miningReward[block.number] == 0) {
        coinBalanceOf[block.coinbase] += 1;
        miningReward[block.number] = block.coinbase;
      }
    }

Você poderia modificar isso para qualquer outra coisa: talvez recompensar alguém que encontre uma solução para um novo quebra-cabeça, ganhe um jogo de xadrez, instale um painel solar - desde de que isso possa ser traduzido para um contrato. Ou talvez você queira criar um banco central para o seu país pessoal, para que você possa acompanhar as horas trabalhadas, favores devidos ou controle de propriedade. Nesse caso, você pode querer adicionar uma função para permitir que o banco possa remotamente congelar fundos e destruir tokens, se necessário.


### Registre um nome para a sua moeda

Os comandos mencionados funcionam apenas porque você tem um objeto *token* de javascript instanciado na sua máquina local. Se você enviar *tokens* a alguém, eles não serão capazes de movê-los novamente porque eles não possuem o mesmo objeto e não saberão onde procurar pelo seu contrato ou como chamar suas funções. De fato, se você reinicar seu console, esses objetos serão deletados e os contratos que você esteve trabalhando estarão perdidos para sempre. Então como instanciar o contrato em uma nova máquina?

Há duas formas. Vamos começar com a mais rápida, provendo seus amigos com uma referência à ABI do seu contrato:

    token = eth.contract([{constant:false,inputs:[{name:'receiver',type:'address'},{name:'amount',type:'uint256'}],name:'sendCoin',outputs:[{name:'sufficient',type:'bool'}],type:'function'},{constant:true,inputs:[{name:'',type:'address'}],name:'coinBalanceOf',outputs:[{name:'',type:'uint256'}],type:'function'},{inputs:[{name:'supply',type:'uint256'}],type:'constructor'},{anonymous:false,inputs:[{indexed:false,name:'sender',type:'address'},{indexed:false,name:'receiver',type:'address'},{indexed:false,name:'amount',type:'uint256'}],name:'CoinTransfer',type:'event'}]).at('0x4a4ce7844735c4b6fc66392b200ab6fe007cfca8')

Apenas substitua o endereço no final com o seu próprio endereço de *tokens*, então qualquer um que usar esse fragmento de código irá imediatamente ser capapz de usar o seu contrato. É claro que isso irá funcionar apenas para este contrato específico então vamos analizar passo a passoe ver como melhorar esse código para que você possa usá-lo em qualquer lugar.

Todas as contas são referenciadas na rede por seus endereços públicos. Mas endereços são longos, difíceis de escrever, difíceis de memorizar e imutáveis. O último é particularmente importante se você quer ser capaz de gerar novas contas no seu nome ou atualizar esse código do seu contrato. Para resolver isso, existe um contrato *registrar default* para nomeação que é usado para associar os longos endereços a nomes curtos e amigáveis.

Nomes tem que usar apenas caracteres alfanuméricos e não podem conter espaços em branco. Em versões futuras, o *registrar* de nome provavelmente implementará um processo de lances para evitar que fiquem guardando nomes mas, por agora, ele funciona na base de quem chegou primeiro pegou: se ninguém pegou o nome, você pode tê-lo.

Primeiro, se você registrar um nome, então você não precisará do endereço escrito no código (*hardcoded*) no final. Escolha um bom nome de moeda e tente reservá-lo para si mesmo. Primeiro, escolha o seu nome:

    var tokenName = "MyFirstCoin"

Então, cheque a disponibilidade do seu nome:

    registrar.addr(tokenName)

Se aquela função retornar "0x00..", você pode clamá-lo para si:

    registrar.reserve.sendTransaction(tokenName, {from: eth.accounts[0]});


Espere a transação anterior ser pega. Espere até 30 segundos e então tente:

    registrar.owner(myName)

Se ele retornar o seu endereço, significa que você possui o nome e pode definir o seu nome escolhido para qualquer endereço que você queira:

    registrar.setAddress.sendTransaction(tokenName, token.address, true,{from: eth.accounts[0]});

_Vocẽ pode subtituir **token.address** por **eth.accounts[0]** se você quiser usá-lo como apelido pessoal._

Espere um pouco para essa transação ser pega e para testá-la:

    registrar.addr("MyFirstCoin")

Você pode enviar uma transação para qualquer um ou qualquer contrato por nome ao invés da conta simplesmente digitando 

    eth.sendTransaction({from: eth.accounts[0], to: registrar.addr("MyFirstCoin"), value: web3.toWei(1, "ether")})

**Dica: não misture registrar.addr com registrar.owner. O primeiro é para qual endereço aquele nome está apontado: qualquer um pode apontar um nome para qualquer outro lugar, assim como alguém pode encaminhar um link para google.com, mas apenas o dono do nome pode mudar e atualizar o link.Você pode definir ambos para o mesmo endereço.**

Isto deveria retornar agora o seu endereço de *token*, significando que agora o código anterior para instanciar poderia usar um nome ao invés de um endereço.

    token = eth.contract([{constant:false,inputs:[{name:'receiver',type:'address'},{name:'amount',type:'uint256'}],name:'sendCoin',outputs:[{name:'sufficient',type:'bool'}],type:'function'},{constant:true,inputs:[{name:'',type:'address'}],name:'coinBalanceOf',outputs:[{name:'',type:'uint256'}],type:'function'},{inputs:[{name:'supply',type:'uint256'}],type:'constructor'},{anonymous:false,inputs:[{indexed:false,name:'sender',type:'address'},{indexed:false,name:'receiver',type:'address'},{indexed:false,name:'amount',type:'uint256'}],name:'CoinTransfer',type:'event'}]).at(registrar.addr("MyFirstCoin"))

Isso também significa que o dono da moeda pode atualizar a moeda apontando o *registrar* para o novo contrato. Isso iria, é claro, requerer que os possuidores de moeda confiassem no dono definido em registrar.owner("MyFirstCoin").

É claro que isso é um pedaço de código um tanto desagradável apenas para permitir que outros interajam com o contrato. Há algumas vias sendo investigadas para fazer *upload* da ABI do contrato para a rede, para que tudo que os usuários precisem seja o nome do contrato. Você pode [ler sobre essas abordagens](https://github.com/ethereum/go-ethereum/wiki/Contracts-and-Transactions#natsp) mas elas são bem experimentais e certamente irão mudar no futuro.


### Saiba mais 

* [Meta coin standard](https://github.com/ethereum/wiki/wiki/Standardized_Contract_APIs) é uma padronização proposta de nomes de funções para contratos de *token* e moedas, para permitir que eles sejam automaticamente adicionados a outro contrato ethereum que utilize troca, como câmbios e garantias.

* [Proofing formal](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format#documentation-output) é um jeito no qual o desenvolvedor do contrato será capaz de afirmar algumas qualidades invariáveis do contrato, como o *cap* total da moeda. *Ainda não implementado*.