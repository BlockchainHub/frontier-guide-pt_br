<!--include "git+https://github.com/ethereum/wiki.wiki.git/Ethereum-Development-Tutorial.md" %}
Was  https://github.com/ethereum/go-ethereum/wiki/Opcodes-cost-and-gas but currently missing
Versão publicada-->

### Máquina de estado

Computação na EVM é feita usando uma linguagem *bytecode* e *stack-based* que é como uma cruz entre o Script do Bitcoin, a tradicional Assembly e Lisp (sendo a relação com Lisp por conta da funcionalidade de envio de mensagem recursivamente). Um programa na EVM é uma sequência de *opcodes*, assim:

    PUSH1 0 CALLDATALOAD SLOAD NOT PUSH1 9 JUMPI STOP JUMPDEST PUSH1 32 CALLDATALOAD PUSH1 0 CALLDATALOAD SSTORE

O propósito deste contrato em particular é servir como um registro de nome; qualquer um pode envir uma mensagem contendo 64 bytes de dados, 32 para a chave e 32 para o valor. O contrato checa se a chave já foi registrada no armazenamento e, se ela não tiver sido, então contrato registra o valor naquela chave.

Durante a execução, uma *array* de bytes infinitamente expandível chamada "memory" (memória), o "program counter" (contador do programa) apontando para a atual instrução, e a *stack* (pilha) de valores de 32 bytes é mantida. No início da execução, a memória e a pilha estão vazias e o PC é zero. Agora, vamos supor que o contrato com esse código está sendo acessado pela primeira vez e uma mensagem é enviada com 123 wei (10<sup>18</sup> wei = 1 ether) e 64 bytes de dados onde os primeiros 32 bytes codificam o número 54 e os segundos 32 bytes codificam o número 2020202020.

Então, o estado no início é:

    PC: 0 STACK: [] MEM: [], STORAGE: {}

A instrução na posição 0 é PUSH1, que empurra um valor de um byte na pilha e pula dois passos no código. Assim, nós temos:

    PC: 2 STACK: [0] MEM: [], STORAGE: {}

A instrução na posição 2 é CALLDATALOAD, que retira um valor da pilha, carrega os 32 bytes de dados de mensagem começando por aquele índice, e o empurra para a pilha. Lembre que os primeiros 32 bytes aqui codificam 54.


    PC: 3 STACK: [54] MEM: [], STORAGE: {}

SLOAD retira um da pilha e empurra o valor no armazenamento do contrato naquele índice. Como o contrato está sendo usado pela primeira vez, ele não tem nada nele, então zero.

    PC: 4 STACK: [0] MEM: [], STORAGE: {}

NOT retira um valor e empurra 1 se o valor for zero, caso contrário, 0

    PC: 5 STACK: [1] MEM: [], STORAGE: {}

em seguida, nós fazemos PUSH1 9.

    PC: 7 STACK: [1, 9] MEM: [], STORAGE: {}

A instrução JUMPI retira 2 valores e pula para a instrução designada pelo primeiro apenas se o segundo não for zero. Aqui, o segundo não é zero, então nós pulamos. Se o valor armazenado no índice 54 não tivesse sido zero, então o segundo valor do topo na pilha teria sido 0 (por conta do NOT), então nós não teríamos pulado e teríamos avançado para a instrução STOP, que nos teria levado a parar a execução.

    PC: 9 STACK: [] MEM: [], STORAGE: {}

Aqui, nós fazemos PUSH1 32.

    PC: 11 STACK: [32] MEM: [], STORAGE: {}

Agora, nós fazemos CALLDATALOAD novamente, retirando 32 e empurrando os bytes nos dados de mensagem  começando do byte 32 até o byte 63.

    PC: 13 STACK: [2020202020] MEM: [], STORAGE: {}

Em seguido, nós fazemos PUSH1 0.

    PC: 14 STACK: [2020202020, 0] MEM: [], STORAGE: {}

Agora, nós carregamos os bytes 0-31 de dados da mensagem novamente (carregar dados de mensagem é tão barato quanto carregar memória, então nós não nos damos o trabalho de salvar na memória)

    PC: 16 STACK: [2020202020, 54] MEM: [], STORAGE: {}

Finalmente, nós fazemos SSTORE para salvar o valor 2020202020 no armazenamento no índice 54.

    PC: 17 STACK: [] MEM: [], STORAGE: {54: 2020202020}

No índice 17, não há instrução, então paramos. Se houvesse algo ainda na pilha ou memória, seria deletado, mas o armazenamento permanecerá e estará disponível na próxima vez em que alguém enviar uma mensagem. Assim, se o remetente desta mensagem enviar a mesma mensagem novamente (ou talvez outra pessoa tente re-registrar 54 para 3030303030), na próxima vez o `JUMPI` na posição 7 não iria ser processado e a execução pararia antecipadamente na posição 8.

Felizmente, você não precisa programar em assembly de baixo nível; uma linguagem de alto nível existe, feita especialmente para escrever contratos, conhecida como [Solidity](https://github.com/ethereum/wiki/wiki/Solidity), que faz ser muito mais fácil para você escrever contratos (há muitas outras também, incluindo [LLL](https://github.com/ethereum/cpp-ethereum/wiki/LLL-PoC-5), [Serpent](https://github.com/ethereum/wiki/wiki/Serpent) e [Mutan](https://github.com/ethereum/go-ethereum/wiki/Mutan-0.2), que você pode achar mais fácil de aprender ou usar dependendo da sua experiência). Qualquer código escrito nessas linguagens é compilado em EVM e, para criar os contratos, você envia a transação contendo o *bytecode* da EVM.

Há dois tipos de transação: uma transação de envio e uma transação de criação de contrato. Uma transação de envio é uma transação padrão, contendo um endereço de recebimento, uma quantia em ether, uma *array* de bytes de dados e alguns outros parâmetros, e uma assinatura da chave privada associada à conta do rementente. Uma transação de criação de contrato se parece com uma transação padrão, exceto que o endereço de recebimento fica em branco. Quando uma transação de criação de contrato chega à blockchain, a *array* de bytes de dados na transação é interpretada como código EVM e o valor retornado pela execução daquela EVM é dado como o código do novo contrato; consequentemente, você pode ter uma transação que faz certas coisas durante a inicialização. O endereço do novo contrato é calculado deterministicamente baseado no endereço e envio e o número de vezes que a conta de envio fez uma transação antes (esse valor, chamado de *nonce* da conta, também é mantido por razões de segurança não relacionadas) Então, o código completo que você precisa botar na blockchain para produzir o registro de nome acima é como segue:

    PUSH1 16 DUP PUSH1 12 PUSH1 0 CODECOPY PUSH1 0 RETURN STOP PUSH1 0 CALLDATALOAD SLOAD NOT PUSH1 9 JUMPI STOP PUSH1 32 CALLDATALOAD PUSH1 0 CALLDATALOAD SSTORE

Os *opcodes* chave são CODECOPY, copia os 16 bytes de código começando do byte 12 para a memória começando no índice 0 e RETURN, retorna os bytes 0-16 de memória, isto é, bytes de código 12-28 (sinta-se a vontade para "rodar" a execução manualmente no papel para verificar que essas partes do código e memória são realmente copiados e retornados). Bytes 12-28 de código são, é claro, o código de fato como vimos acima.

###*Opcodes* de máquinas virtuais

Uma lista completa dos *opcodes* na EVM pode ser encontrada no *yellow paper*. Note que linguagens de alto nível irão frequentemente ter suas próprias cápsulas para esses *opcodes*, às vezes com interfaces bem diferentes.
