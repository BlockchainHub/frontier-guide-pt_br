{% sections "democracy-dao", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contract-Tutorial.md" -->
<!--Versão Publicada-->

# DAO democrática



Até agora, você criou um *token* trocável e o distribuiu com sucesso entre aqueles que estavam dispostos a ajudar no levantamento de 100 ethers. Isso é tudo muito interessante mas para que servem estes *tokens* exatamente? Por que alguém iria querer um ter ou trocar por qualquer outra coisa de valor? Se você conseguir convencer que o seu novo *token* é o mais novo e melhor dinheiro talvez outros irão querer ele, mas até agora o seu *token* não oferece nenhum valor próprio. Nós vamos mudar isso criando sua primeira organização autônoma descentralizada (*decentralized autonomous organization*), ou DAO.

Pense na DAO como a constituição de um país, o ramo executivo de um governo ou talvez um gerente robótico de uma organização. A DAO recebe o dinheiro que a sua organização levanta, o mantém a salvo e o usa para financiar o que quer que seus membros queiram. O robô é incorruptível, nunca irá fraudar o banco, nunca criará planos secretos, nunca usará o dinheiro para outra coisa que não seja o que os seus membros votaram. A DAO nunca desaparecerá nem fugirá e não pode ser controlada por ninguém mais além de seus cidadãos.

O token que nós distribuímos usando o *crowdsale* é o único documento de cidadão necessário. Qualquer um que tiver um *token* é capaz de criar e votar em propostas. Similarmente a ser um *shareholder* (possuidor de ações) em uma empresa, o *token* pode ser trocado em mercado aberto e o voto é proporcional à quantidade de *tokens* que o votante possui.  

Pare um momento para imaginar as possibilidades revolucionárias que isso permitiria, e agora você mesmo pode fazer isso, em menos de 100 linhas de código:


### O código

    


    contract token { mapping (address => uint) public coinBalanceOf;   function token() { }   function sendCoin(address receiver, uint amount) returns(bool sufficient) {  } }


    contract Democracy {
    
        uint public minimumQuorum;
        uint public debatingPeriod;
        token public voterShare;
        address public founder;
        Proposal[] public proposals;
        uint public numProposals;
        
        event ProposalAdded(uint proposalID, address recipient, uint amount, bytes32 data, string description);
        event Voted(uint proposalID, int position, address voter);
        event ProposalTallied(uint proposalID, int result, uint quorum, bool active);

        struct Proposal {
            address recipient;
            uint amount;
            bytes32 data;
            string description;
            uint creationDate;
            bool active;
            Vote[] votes;
            mapping (address => bool) voted;
        }
        
        struct Vote {
            int position;
            address voter;
        }
        
        function Democracy(token _voterShareAddress, uint _minimumQuorum, uint _debatingPeriod) {
            founder = msg.sender;  
            voterShare = token(_voterShareAddress);
            minimumQuorum = _minimumQuorum || 10;
            debatingPeriod = _debatingPeriod * 1 minutes || 30 days;
        }
    
        
        function newProposal(address _recipient, uint _amount, bytes32 _data, string _description) returns (uint proposalID) {
            if (voterShare.coinBalanceOf(msg.sender)>0) {
                proposalID = proposals.length++;
                Proposal p = proposals[proposalID];
                p.recipient = _recipient;
                p.amount = _amount;
                p.data = _data;
                p.description = _description;
                p.creationDate = now;
                p.active = true;
                ProposalAdded(proposalID, _recipient, _amount, _data, _description);
                numProposals = proposalID+1;
            }
        }
        
        function vote(uint _proposalID, int _position) returns (uint voteID){
            if (voterShare.coinBalanceOf(msg.sender)>0 && (_position >= -1 || _position <= 1 )) {
                Proposal p = proposals[_proposalID];
                if (p.voted[msg.sender] == true) return;
                voteID = p.votes.length++;
                p.votes[voteID] = Vote({position: _position, voter: msg.sender});
                p.voted[msg.sender] = true;
                Voted(_proposalID,  _position, msg.sender);
            }
        }
        
        function executeProposal(uint _proposalID) returns (int result) {
            Proposal p = proposals[_proposalID];
            /* Checa se o período de debate terminou */
            if (now > (p.creationDate + debatingPeriod) && p.active){   
                uint quorum = 0;
                /* tally the votes */
                for (uint i = 0; i <  p.votes.length; ++i) {
                    Vote v = p.votes[i];
                    uint voteWeight = voterShare.coinBalanceOf(v.voter); 
                    quorum += voteWeight;
                    result += int(voteWeight) * v.position;
                }
                /* executa o resultado */
                if (quorum > minimumQuorum && result > 0 ) {
                    p.recipient.call.value(p.amount)(p.data);
                    p.active = false;
                } else if (quorum > minimumQuorum && result < 0) {
                    p.active = false;
                }
                ProposalTallied(_proposalID, result, quorum, p.active);
            }
        }
    }





Há muito acontecendo, mas é mais simples do que parece. As regras da sua organização são muito simples: qualquer um com pelo menos um *token* pode criar propostas para enviar fundos da conta do país. Depois de uma semana de debate e voto, se ela tiver recebido votos somando um total de 100 *tokens* ou mais e tiver mais aprovações do que rejeições, os fundos serão enviados. Se o quórum não foi atingido ou terminou em empate, então a votação continua até que seja resolvido. Caso contrário, a proposta é trancada e mantida para propósitos históricos.

Então vamos recapitular o que isso significa: nas últimas duas seções, você criou 10.000 *tokens*, enviou 1.000 deles para outra conta que você controla, 2.000 para uma amiga chamada Alice e distribuiu 5.000 deles por meio de *crowdsale*. Isso significa que você não possui mais a maioria (mais de 50%) dos votos na DAO e, se Alice e a comunidade se juntarem, eles podem vencer qualquer decisão de gasto dos 100 ethers levantados até agora. Isso é exatamente como uma democracia deveria funcionar. Se você não quer mais ser parte do seu país, a única coisa que você pode fazer é vender os seus *tokens* em uma casa de câmbio descentralizada e optar por sair, mas você não pode impedir os outros de fazerem isso.


### Configure a sua organização

Então abra o seu console e vamos nos preparar para finalmente botar o seu contrato online. Primeiro, vamos ver os parâmetros corretos, escolha-os com cuidado:

    var _voterShareAddress = token.address;
    var _minimumQuorum = 10; // Quantidade mínima de tokens de votantes necessária para a proposta passar
    var _debatingPeriod = 60; // período de debate, em minutos
    
Com esses parâmetros *default* qualquer um com *tokens* pode fazer uma proposta sobre como gastar o dinheiro da organização. A proposta tem 1 hora para ser debatida e irá passar se tiver pelo menos votos de pelo menos 0.1% do total de *tokens* e tiver mais suporte do que rejeição. Escolha esses parâmetros com cuidado, você não poderá mudá-los no futuro.
    
    var daoCompiled = eth.compile.solidity('contract token { mapping (address => uint) public coinBalanceOf; function token() { } function sendCoin(address receiver, uint amount) returns(bool sufficient) { } } contract Democracy { uint public minimumQuorum; uint public debatingPeriod; token public voterShare; address public founder; Proposal[] public proposals; uint public numProposals; event ProposalAdded(uint proposalID, address recipient, uint amount, bytes32 data, string description); event Voted(uint proposalID, int position, address voter); event ProposalTallied(uint proposalID, int result, uint quorum, bool active); struct Proposal { address recipient; uint amount; bytes32 data; string description; uint creationDate; bool active; Vote[] votes; mapping (address => bool) voted; } struct Vote { int position; address voter; } function Democracy(token _voterShareAddress, uint _minimumQuorum, uint _debatingPeriod) { founder = msg.sender; voterShare = token(_voterShareAddress); minimumQuorum = _minimumQuorum || 10; debatingPeriod = _debatingPeriod * 1 minutes || 30 days; } function newProposal(address _recipient, uint _amount, bytes32 _data, string _description) returns (uint proposalID) { if (voterShare.coinBalanceOf(msg.sender)>0) { proposalID = proposals.length++; Proposal p = proposals[proposalID]; p.recipient = _recipient; p.amount = _amount; p.data = _data; p.description = _description; p.creationDate = now; p.active = true; ProposalAdded(proposalID, _recipient, _amount, _data, _description); numProposals = proposalID+1; } else { return 0; } } function vote(uint _proposalID, int _position) returns (uint voteID){ if (voterShare.coinBalanceOf(msg.sender)>0 && (_position >= -1 || _position <= 1 )) { Proposal p = proposals[_proposalID]; if (p.voted[msg.sender] == true) return; voteID = p.votes.length++; Vote v = p.votes[voteID]; v.position = _position; v.voter = msg.sender; p.voted[msg.sender] = true; Voted(_proposalID, _position, msg.sender); } else { return 0; } } function executeProposal(uint _proposalID) returns (int result) { Proposal p = proposals[_proposalID]; /* Checa se o período de debate terminou */ if (now > (p.creationDate + debatingPeriod) && p.active){ uint quorum = 0; /* conta os votos */ for (uint i = 0; i < p.votes.length; ++i) { Vote v = p.votes[i]; uint voteWeight = voterShare.coinBalanceOf(v.voter); quorum += voteWeight; result += int(voteWeight) * v.position; } /* executa o resultado */ if (quorum > minimumQuorum && result > 0 ) { p.recipient.call.value(p.amount)(p.data); p.active = false; } else if (quorum > minimumQuorum && result < 0) { p.active = false; } } ProposalTallied(_proposalID, result, quorum, p.active); } }');

    var democracyContract = web3.eth.contract(daoCompiled.Democracy.info.abiDefinition);
    
    var democracy = democracyContract.new(
        _voterShareAddress, 
        _minimumQuorum, 
        _debatingPeriod, 
        {
          from:web3.eth.accounts[0], 
          data:daoCompiled.Democracy.code, 
          gas: 3000000
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

**Se você está usando o _online compiler_, copie o código de contrato para o [compilador de solidity online](https://chriseth.github.io/cpp-ethereum/), e depois pegue o conteúdo da caixa nomeada **Geth Deploy**. Como você já definiu os parâmetros, você não precisa mudar nada naquele texto, simplesmente cole o texto resultante na sua janela do geth.**

Espere um minuto até que os mineradores a peguem. Ela irá custar cerca de 850 mil gas. Uma vez que ela foi pega, é hora de instanciá-la e configurá-la, apontando-a para o endereço correto do contrato de *token* qur você criou anteriormente. 

Se tudo funcionou corretamente, você pode dar uma olhada na organização como um todo executando essa *string*:

    "This organization has " +  democracy.numProposals() + " proposals and uses the token at the address " + democracy.voterShare() ;

Se tudo está configurado então a sua DAO deve retornar uma contagem de proposta de 0 e um endereço marcado como o "founder" (fundador). Enquanto não houver propostas, o fundador da DAO pode mudar o endereço do *token* para qualquer coisa que quiser. 

### Registre o nome da sua organização

Vamos também registrar um nome para o seu contrato para que ele seja facilmente acessível (não se esqueça de checar a disponibilidade do nome com registrar.addr("nomeQueVoceQuer") antes de reservar!)

    var name = "MyPersonalDemocracy"
    registrar.reserve.sendTransaction(name, {from: eth.accounts[0]})
    var democracy = eth.contract(daoCompiled.Democracy.info.abiDefinition).at(democracy.address);
    democracy.setup.sendTransaction(registrar.addr("MyFirstCoin"),{from:eth.accounts[0]})

Espere até que as transações anteriores tenham sido pegas e então:

    registrar.setAddress.sendTransaction(name, democracy.address, true,{from: eth.accounts[0]});


### Os robôs vigilantes (*Watchbots*) da democracia


    var event = democracy.ProposalAdded({}, '', function(error, result){
      if (!error)
        console.log("New Proposal #"+ result.args.proposalID +"!\n Send " + web3.fromWei(result.args.amount, "ether") + " ether to " + result.args.recipient.substring(2,8) + "... for " + result.args.description  )
    });
    var eventVote = democracy.Voted({}, '', function(error, result){
      if (!error)
        var opinion = "";
        if (result.args.position > 0) { 
          opinion = "in favor" 
        } else if (result.args.position < 0) { 
          opinion = "against" 
        } else { 
          opinion = "abstaining" 
        }

        console.log("Vote on Proposal #"+ result.args.proposalID +"!\n " + result.args.voter + " is " + opinion )
    });
    var eventTally = democracy.ProposalTallied({}, '', function(error, result){
      if (!error)
        var totalCount = "";
        if (result.args.result > 1) { 
          totalCount = "passed" 
        } else if (result.args.result < 1) { 
          totalCount = "rejected" 
        } else { 
          totalCount = "a tie" 
        }
        console.log("Votes counted on Proposal #"+ result.args.proposalID +"!\n With a total of " + Math.abs(result.args.result) + " out of " + result.args.quorum + ", proposal is " + totalCount + ". Proposal is " + (result.args.active? " still on the floor" : "archived") )
    });


### Interagindo com a DAO

Depois que você estiver satisfeito com o que quer, é hora de pegar todo aquele ether que você conseguiu no *crowdfunding* e botar na sua organização:

    eth.sendTransaction({from: eth.accounts[1], to: democracy.address, value: web3.toWei(100, "ether")})

Isso deve levar apenas um minuto e o seu país está pronto para negócio! Agora, como primeira prioridade, seu organização precisa de uma boa logo mas, a não ser que você seja um designer, você não deve ter a menor ideia de como fazer isso. Para fins de exemplo, considere que você descobriu que o seu amigo Bob é um ótimo designer que está disposto a fazer a logo por apenas 10 ethers, então você quer propor que ele seja contratado. 

    recipient = registrar.addr("bob");
    amount =  web3.toWei(10, "ether");
    shortNote = "Logo Design";

    democracy.newProposal.sendTransaction( recipient, amount, '', shortNote,  {from: eth.accounts[0], gas:1000000})

Depois de um minuto, qualquer um pode checar o recipiente da proposta e o valor executando estes comandos:

    "This organization has " +  (Number(democracy.numProposals())+1) + " pending proposals";

### Fique de olho na organização

Diferente da maior parte dos governos, o governo do seu país é completamente transparente e facilmente programável. Como uma pequena demonstração, aqui está um fragmento de código que varre todas as atuais propostas e imprime o que elas são e para quem elas são:

       

    function checkAllProposals() {
        console.log("Country Balance: " + web3.fromWei( eth.getBalance(democracy.address), "ether") );
        for (i = 0; i< (Number(democracy.numProposals())); i++ ) { 
            var p = democracy.proposals(i); 
            var timeleft = Math.floor(((Math.floor(Date.now() / 1000)) - Number(p[4]) - Number(democracy.debatingPeriod()))/60);  
            console.log("Proposal #" + i + " Send " + web3.fromWei( p[1], "ether") + " ether to address " + p[0].substring(2,6) + " for "+ p[3] + ".\t Deadline:"+ Math.abs(Math.floor(timeleft)) + (timeleft>0?" minutes ago ":" minutes left ") + (p[5]? " Active":" Archived") ); 
        }
    }

    checkAllProposals();

Um cidadão preocupado poderia facilmente escrever um *bot* que periodicamente faz um *ping* na blockchain e então publica qualquer nova proposta que foi feita, garantindo total transparência.

Agora, é claro que você quer que outras pessoas sejam capazes de votar nas suas propostas. Você pode checar o tutorial de *crowdsale* para o melhor jeito de registrar seu *app* de contrato para que tudo que o usuário precise seja um nome mas, por enquanto, vamos usar a versão mais fácil. Qualquer um deve ser capaz de instanciar uma cópia local do seu país no próprio computador usando esse comando gigante:


    democracy = eth.contract( [{ constant: true, inputs: [{ name: '', type: 'uint256' } ], name: 'proposals', outputs: [{ name: 'recipient', type: 'address' }, { name: 'amount', type: 'uint256' }, { name: 'data', type: 'bytes32' }, { name: 'descriptionHash', type: 'bytes32' }, { name: 'creationDate', type: 'uint256' }, { name: 'numVotes', type: 'uint256' }, { name: 'quorum', type: 'uint256' }, { name: 'active', type: 'bool' } ], type: 'function' }, { constant: false, inputs: [{ name: '_proposalID', type: 'uint256' } ], name: 'executeProposal', outputs: [{ name: 'result', type: 'uint256' } ], type: 'function' }, { constant: true, inputs: [ ], name: 'debatingPeriod', outputs: [{ name: '', type: 'uint256' } ], type: 'function' }, { constant: true, inputs: [ ], name: 'numProposals', outputs: [{ name: '', type: 'uint256' } ], type: 'function' }, { constant: true, inputs: [ ], name: 'founder', outputs: [{ name: '', type: 'address' } ], type: 'function' }, { constant: false, inputs: [{ name: '_proposalID', type: 'uint256' }, { name: '_position', type: 'int256' } ], name: 'vote', outputs: [{ name: 'voteID', type: 'uint256' } ], type: 'function' }, { constant: false, inputs: [{ name: '_voterShareAddress', type: 'address' } ], name: 'setup', outputs: [ ], type: 'function' }, { constant: false, inputs: [{ name: '_recipient', type: 'address' }, { name: '_amount', type: 'uint256' }, { name: '_data', type: 'bytes32' }, { name: '_descriptionHash', type: 'bytes32' } ], name: 'newProposal', outputs: [{ name: 'proposalID', type: 'uint256' } ], type: 'function' }, { constant: true, inputs: [ ], name: 'minimumQuorum', outputs: [{ name: '', type: 'uint256' } ], type: 'function' }, { inputs: [ ], type: 'constructor' } ] ).at(registrar.addr('MyPersonalCountry'))

Então, qualquer um que tiver algum de seus *tokens* pode votar nas propostas fazendo isso:

    var proposalID = 0;
    var position = -1; // +1 para votar sim, -1  para votar não, 0 abstém mas conta como quórum
    democracy.vote.sendTransaction(proposalID, position, {from: eth.accounts[0], gas: 1000000});

    var proposalID = 1;
    var position = 1; // +1 para votar sim, -1  para votar não, 0 abstém mas conta como quórum
    democracy.vote.sendTransaction(proposalID, position, {from: eth.accounts[0], gas: 1000000});


A não ser que você tenha mudado os parâmetros básicos no código, qualquer proposta terá que ser debatida por pelo menos uma semana até que possa ser executada. Depois disso, qualquer um - até mesmo um não-cidadão - pode demandar que os votos sejam contados e que a proposta seja executada. Os votos são contados e pesados nesse momento e, se a proposta for aceita, então o ether é enviado imediatamente e a proposta é arquivada. Se a votação terminar em empate ou o quórum mínimo não tiver sido atingido, a votação é mantida em aberto até que o impasse seja resolvido. Se ela perder, então é arquivada e não pode ser votada novamente.

    var proposalID = 1;
    democracy.executeProposal.sendTransaction(proposalID, {from: eth.accounts[0], gas: 1000000});


Se a proposta passou, então você deve ser capaz de ver os ethers de Bob chegando em seu endereço:

    web3.fromWei(eth.getBalance(democracy.address), "ether") + " ether";
    web3.fromWei(eth.getBalance(registrar.addr("bob")), "ether") + " ether";


**Tente você mesmo:**  Este é um contrato democrático muito simples que poderia ser amplamente melhorado: atualmente, todas as propostas têm o mesmo tempo de debate e são vencidas por voto direto e maioria simples. Será que você pode mudar isso para que em algumas situações, dependendo da quantia proposta, o tempo de debate seja mais longo ou a maioria necessária seja maior? Pense também em algum jeito para que os cidadãos não precisem votar em todas as questões e possam delegar temporariamente seus votos para um representante. Você pode ter notado também que nós adicionamos uma uma pequena descrição para cada proposta. Isso poderia ser usado como um título para a proposta ou poderia ser um *hash* de um documento maior descrevendo-a em detalhes.

### Vamos explorar!

Você alcançou o final deste tutorial, mas é apenas o começo de uma grande aventura. Olhe para trás e veja o quanto você fez: você criou um robô vivo e falante, sua própria criptomoeda, levantou fundos através de um *crowdfunding* que não depende de confiança e o usou para dar o pontapé inicial na sua organização democrática pessoal.  

Por questão de simplicidade, nós usamos a organização democrática que você criou apenas para enviar ether, a moeda nativa da ethereum. Enquanto isso pode ser bom o suficiente para alguns, isso é apenas um arranhão na superfície do que pode ser feito. Na rede ethereum, contratos têm os mesmos direitos que um usuário normal, significando que sua organização poderia ter feito qualquer uma das transações que você executou vindo de suas próprias contas.

### O que poderia acontecer agora?

* O contrato *greeter* que você criou no início poderia ser melhorado para cobrar ether por seus serviços e poderia destinar esses fundos para a DAO.

* Os *tokens* que você ainda controla poderiam ser vendidos em uma casa de câmbio descentralizada ou trocados por bens e serviços para financiar ainda mais o desenvolvimento do primeiro contrato e o crescimento da organização.

* Sua DAO poderia ser dona do próprio nome do *registrar* de nome, e depois mudar seu local de redirecionamento para se atualizar se os donos de *tokens* aprovarem.

* A organização poderia guardar não apenas ethers, mas qualquer outro tipo de moeda criada na ethereum, incluindo bens cujos valores estejam ligados ao bitcoin ou ao dólar.

* A DAO poderia ser programada para permitir uma proposta com múltiplas transações, algumas agendadas para o futuro.  
Ela poderia também ser dona de *shares* de outras DAO's, significando que ela poderia votar em uma organização maior ou ser parte de uma federação de DAO's.

* O contrato de *token* poderia ser reprogramado para segurar ether ou segurar outros *tokens* e distribuí-los aos donos de *tokens*. Isso ligaria o valor do *token* ao valor de outros bens, então o pagamento de dividendos poderia ser feito simplesmente movendo fundos para o endereço do *token*

Tudo isso significa que essa pequena sociedade que você criou poderia crescer, pegar financiamento com terceiros, pagar salários recorrentes, ser dona de qualquer tipo de *crypto-asset* e até usar *crowdsales* para financiar suas atividades. Tudo com total transparência, completa responsabilidade e completa imunidade de interferência humana. Enquanto a rede vive, os contratos irão executar exatamente o código que eles foram criados para executar, sem qualquer exceção, para sempre. 

Então o que o seu contrato será? Será um país, uma empresa, um grupo sem fins lucrativos? O que o seu código fará? 

Isso é com você.