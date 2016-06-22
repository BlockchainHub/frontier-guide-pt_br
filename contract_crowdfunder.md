<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contract-Tutorial.md" -->
<!--Versão Publicada-->

# Funde sua ideia com *crowdfunding*

Às vezes, uma boa ideia precisa de muito financiamento e esforço coletivo. Você poderia perdir doações, mas doadores preferem doar para projetos que eles têm mais certeza de que irão ter tração e conseguirão financiamento apropriado. Este é um exemplo onde crowdfunding seria ideal: você define uma meta e um prazo para atingir a meta. Se você não alcançar a meta, as doações são retornadas, reduzindo assim o risco para os doadores. Como o código é aberto e auditável, não há necessidade de uma plataforma centralizada de confiança e, portanto, as únicas taxas que serão pagas serão apenas as taxas de gas.

Em um *crowdfunding* prêmios costumam ser dados. Isto requeriria que você tivesse as informações de contato de todos e mantivesse registro de quem é dono de que. Mas como você acabou de criar o seu próprio *token*, por que não usá-lo para acompanhar os prêmios? Isto permite a doadores serem imediatamente donos de algo depois que eles doaram. Eles podem guardar isso com segurança, mas também podem vender ou trocar se eles decidirem que não querem mais o prêmio. Se a sua ideia for algo físico, tudo o que você precisa fazer quando o projeto estiver completo é dar o produto a todos que enviarem de volta um *token*. Se o projeto é digital, o próprio *token* pode ser usado imediatamente para os usuários participarem ou terem entrada no seu projeto.

### O código

A forma que esse contrato de *crowdsale* em particular funciona é com você definindo um preço de troca para o seu *token* e então os doadores receberão uma quantidade proporcional de *tokens* em troca do ether deles. Você também escolherá uma meta de financiamento e um prazo: uma vez que o prazo se esgote você pode fazer um *ping* no contrato e, se a meta tiver sido atingida, ele irá enviar o ether levantado para você; caso contrário, ele retorna para os doadores. Doadores permanecem com seus *tokens* mesmo se o projeto não atingir a meta, como prova de que eles contribuíram.
    
    
    contract token { mapping (address => uint) public coinBalanceOf; function token() {}  function sendCoin(address receiver, uint amount) returns(bool sufficient) {  } }
    
    contract Crowdsale {
        
        address public beneficiary;
        uint public fundingGoal; uint public amountRaised; uint public deadline; uint public price;
        token public tokenReward;   
        Funder[] public funders;
        event FundTransfer(address backer, uint amount, bool isContribution);
        
        /* estrutura de dados para segurar informação sobre os contribuidores da campanha */
        struct Funder {
            address addr;
            uint amount;
        }
        
        /*  ao iniciar, definir o dono */
        function Crowdsale(address _beneficiary, uint _fundingGoal, uint _duration, uint _price, token _reward) {
            beneficiary = _beneficiary;
            fundingGoal = _fundingGoal;
            deadline = now + _duration * 1 minutes;
            price = _price;
            tokenReward = token(_reward);
        }   
        
        /* A função sem nome é a função default que é chamada sempre que alguém envia fundos a um contrato */
        function () {
            uint amount = msg.value;
            funders[funders.length++] = Funder({addr: msg.sender, amount: amount});
            amountRaised += amount;
            tokenReward.sendCoin(msg.sender, amount / price);
            FundTransfer(msg.sender, amount, true);
        }
            
        modifier afterDeadline() { if (now >= deadline) _ }

        /* checa se a meta ou o prazo foi alcançado e termina a campanha */
        function checkGoalReached() afterDeadline {
            if (amountRaised >= fundingGoal){
                beneficiary.send(amountRaised);
                FundTransfer(beneficiary, amountRaised, false);
            } else {
                FundTransfer(0, 11, false);
                for (uint i = 0; i < funders.length; ++i) {
                  funders[i].addr.send(funders[i].amount);  
                  FundTransfer(funders[i].addr, funders[i].amount, false);
                }               
            }
            suicide(beneficiary);
        }
    }

### Define os parâmetros

Antes de irmos mais longe, vamos começar definindo os parâmetros da *crowdsale*:

    var _beneficiary = eth.accounts[1];    // cria uma conta para isso
    var _fundingGoal = web3.toWei(100, "ether"); // levanta 100 ether
    var _duration = 30;     // número de minutos que a campanha irá durar
    var _price = web3.toWei(0.02, "ether"); // o preço dos tokens, em ether
    var _reward = token.address;   // o endereço de contrato do token

Em beneficiário (*beneficiary*), coloque o novo endereço que irá receber os fundos levantados. A meta de financiamento (*fundingGoal*) é a quantidade de ether a ser levantada. Prazo (*deadline*) é medido em *blocktimes* que representam em média 12 segundos, então o *default* é cerca de 4 semanas. O preço (*price*) é complicado: mas mude apenas o número 2 pela quantidade de *tokens* que o contribuidor receberá para cada ether doado. Finalmente, a recompensa (*reward*) deveria ser o endereço do contrato do *token* que você criou na última seção. 

Nesse exemplo, você está vendendo na *crowdsale* metade de todos os *tokens* que existem em troca de 100 ether. Decida esses parâmetros com muito cuidado, já que eles irão ter um papel importante na próxima parte do guia.

### Implementação

Você conhece o esquema: se você está usando o compilador solC, [remova as quebras de linha](http://www.textfixer.com/tools/remove-line-breaks.php) e copie os seguintes comandos no terminal:


    var crowdsaleCompiled = eth.compile.solidity(' contract token { mapping (address => uint) public coinBalanceOf; function token() {} function sendCoin(address receiver, uint amount) returns(bool sufficient) { } } contract Crowdsale { address public beneficiary; uint public fundingGoal; uint public amountRaised; uint public deadline; uint public price; token public tokenReward; Funder[] public funders; event FundTransfer(address backer, uint amount, bool isContribution); /* estrutura de dados para guardar informação sobre os contribuidores da campanha */ struct Funder { address addr; uint amount; } /* ao iniciar, definir o dono */ function Crowdsale(address _beneficiary, uint _fundingGoal, uint _duration, uint _price, token _reward) { beneficiary = _beneficiary; fundingGoal = _fundingGoal; deadline = now + _duration * 1 minutes; price = _price; tokenReward = token(_reward); } /* A função sem nome é a sunção default que é chamada sempre que alguém envia fundos a um contrato */ function () { Funder f = funders[++funders.length]; f.addr = msg.sender; f.amount = msg.value; amountRaised += f.amount; tokenReward.sendCoin(msg.sender, f.amount/price); FundTransfer(f.addr, f.amount, true); } modifier afterDeadline() { if (now >= deadline) _ } /* checa se a meta ou o prazo foi atingido e termina a campanha */ function checkGoalReached() afterDeadline { if (amountRaised >= fundingGoal){ beneficiary.send(amountRaised); FundTransfer(beneficiary, amountRaised, false); } else { FundTransfer(0, 11, false); for (uint i = 0; i < funders.length; ++i) { funders[i].addr.send(funders[i].amount); FundTransfer(funders[i].addr, funders[i].amount, false); } } suicide(beneficiary); } }');

    var crowdsaleContract = web3.eth.contract(crowdsaleCompiled.Crowdsale.info.abiDefinition);
    var crowdsale = crowdsaleContract.new(
      _beneficiary, 
      _fundingGoal, 
      _duration, 
      _price, 
      _reward,
      {
        from:web3.eth.accounts[0], 
        data:crowdsaleCompiled.Crowdsale.code, 
        gas: 1000000
      }, function(e, contract){
        if(!e) {

          if(!contract.address) {
            console.log("Contract transaction send: TransactionHash: " + contract.transactionHash + " waiting to be mined...");

          } else {
            console.log("Contract mined! Address: " + contract.address);
            console.log(contract);
          }

        }    })

**Se você está usando o _online compiler_, copie o código de contrato para o [compilador de solidity online](https://chriseth.github.io/cpp-ethereum/), e depois pegue o conteúdo da caixa nomeada **Geth Deploy**. Como você já definiu os parâmetros, você não precisa mudar nada naquele texto, simplesmente cole o texto resultante na sua janela do geth.**

Aguarde até 30 segundos e você verá uma mensagem assim:

    Contract mined! address: 0xdaa24d02bad7e9d6a80106db164bad9399a0423e 

Se você recebeu esse alerta, então seu código deve estar online. Você sempre pode checar fazendo isso:

    eth.getCode(crowdsale.address)

Agora, funde o seu contrato recém-criado com os *tokens* necessários para que ele possa distribuir automaticamente a recompensa aos contribuidores!

    token.sendCoin.sendTransaction(crowdsale.address, 5000,{from: eth.accounts[0]})

Depois da transação ser pega, você pode checar a quantidade de *tokens* que o endereço de *crowdsale* tem e todas as outras variáveis dessa forma:

    "Current crowdsale must raise " + web3.fromWei(crowdsale.fundingGoal.call(), "ether") + " ether in order to send it to " + crowdsale.beneficiary.call() + "."



### Ative alguns observadores

Você quer ser alertado sempre que o seu *crowdsale* receber novos fundos, então cole este código:

    var event = crowdsale.FundTransfer({}, '', function(error, result){
      if (!error)
        
        if (result.args.isContribution) {
            console.log("\n New backer! Received " + web3.fromWei(result.args.amount, "ether") + " ether from " + result.args.backer  )

            console.log( "\n The current funding at " +( 100 *  crowdsale.amountRaised.call() / crowdsale.fundingGoal.call()) + "% of its goals. Funders have contributed a total of " + web3.fromWei(crowdsale.amountRaised.call(), "ether") + " ether.");
                  
            var timeleft = Math.floor(Date.now() / 1000)-crowdsale.deadline();
            if (timeleft>3600) {  console.log("Deadline has passed, " + Math.floor(timeleft/3600) + " hours ago")
            } else if (timeleft>0) {  console.log("Deadline has passed, " + Math.floor(timeleft/60) + " minutes ago")
            } else if (timeleft>-3600) {  console.log(Math.floor(-1*timeleft/60) + " minutes until deadline")
            } else {  console.log(Math.floor(-1*timeleft/3600) + " hours until deadline")
            }

        } else {
            console.log("Funds transferred from crowdsale account: " + web3.fromWei(result.args.amount, "ether") + " ether to " + result.args.backer  )
        }

    });

      


### Registre o contrato

Você está pronto. Qualquer um já pode contribuir simplesmente enviando ether para o endereço de *crowdsale*, mas para simplificar ainda mais, vamos registrar um nome para a sua venda. Primeiro, escolha um nome para o seu *crowdsale*:

    var name = "mycrowdsale"

Cheque se esse nome está disponível e registre:

    registrar.addr(name) 
    registrar.reserve.sendTransaction(name, {from: eth.accounts[0]});
 
Aguarde a transação anterior ser pega e então:

    registrar.setAddress.sendTransaction(name, crowdsale.address, true,{from: eth.accounts[0]});


### Contribua para o *crowdsale*

Contribuir para o* crowdsale* é muito simples, não requer nem que você instancie o contrato. Isso é porque o *crowdsale* responde a simples depósitos em ether, então qualquer um que enviar ether para o *crowdsale* irá automaticamente receber uma recompensa.
Qualquer um pode contribuir simplesmente executando este comando: 

    var amount = web3.toWei(5, "ether") // decide how much to contribute

    eth.sendTransaction({from: eth.accounts[0], to: crowdsale.address, value: amount, gas: 1000000})


Alternativamente, se você quer que outra pessoa envie, ela pode até mesmo usar o *registrar* de nome para contribuir:

    eth.sendTransaction({from: eth.accounts[0], to: registrar.addr("mycrowdsale"), value: amount, gas: 500000})


Agora espere um minuto para os blocos alcançarem e você pode checar se o contrato recebeu o ether fazendo qualquer um desses comandos: 

    web3.fromWei(crowdsale.amountRaised.call(), "ether") + " ether"
    token.coinBalanceOf.call(eth.accounts[0]) + " tokens"
    token.coinBalanceOf.call(crowdsale.address) + " tokens"


### Recuperar fundos

Uma vez terminado o prazo, alguém precisa acordar o contrato para ter os fundos enviados ao beneficiário ou de volta aos doadores (caso tenha falhado no alcance da meta). Isso acontece porque não há algo como um *loop* ativo ou um temporizador na ethereum, então quaisquer transações futuras precisam ser *pinged* por alguém. ~~This happens because there is no such thing as an active loop or timer on ethereum so any future transactions must be pinged by someone~~.

    crowdsale.checkGoalReached.sendTransaction({from:eth.accounts[0], gas: 2000000})

Você pode checar suas contas com essas linhas de código:

    web3.fromWei(eth.getBalance(eth.accounts[0]), "ether") + " ether"
    web3.fromWei(eth.getBalance(eth.accounts[1]), "ether") + " ether"
    token.coinBalanceOf.call(eth.accounts[0]) + " tokens"
    token.coinBalanceOf.call(eth.accounts[1]) + " tokens"

A instância do *crowdsale* é feita para se auto destruir uma vez que acabou seu trabalho, então se o prazo acabou e todos receberam seus prêmios o contrato deixa de existir, como você pode ver rodando isso:

    eth.getCode(crowdsale.address)

Então você levantou 100 ethers e distribuiu com sucesso sua moeda original entre os doadores do *crowdsale*. O que você pode fazer a seguir com isso?

