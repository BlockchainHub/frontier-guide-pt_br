{% sections "register-a-name-for-your-coin", "" %}
{% endsections %}

<!-- "git+https://github.com/ethereum/go-ethereum.wiki.git/Contract-Tutorial.md" -->
<!--Versão Publicada-->

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
