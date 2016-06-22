#Introdução

A palavra mineração é originada do contexto da analogia entre o ouro e as criptomoedas. Ouro ou metais preciosos são escassos, assim como *tokens* digitais, e a única forma de aumentar o volume é através da mineração. Isto é apropriado na medida que, também na rede Ethereum, a única forma de emissão pós-lançamento é via mineração. No entanto, ao contrário do exemplo dos metais preciosos, a mineração é também a forma de manter a segurança da rede com a criação, verificação, publicação e propagação dos blocos na blockchain.

* Minerar Ether = Dar segurança à rede = Verificar a computação

##O que é a mineração, afinal?

A Ethereum Frontier, como todas as tecnologias blockchain, usa um modelo de segurança baseado em incentivos. O consenso é baseado na escolha do bloco com a maior dificuldade total. Mineradores produzem blocos que são checados pelos outros para garantir a validade. Além de outros critérios de formação, um bloco só é considerado válido se contiver um **proof of work** (PoW) de uma determinada **dificuldade**. Note que no Ethereum 1.1, isso provavelmente será substituído por um modelo de **proof of stake**.


O algoritmo de proof of work usado é chamado [Et*hash*](https://github.com/ethereum/wiki/wiki/Et*hash*) (uma versão modificada do [Dagger-Hashimoto](https://github.com/ethereum/wiki/wiki/Dagger-Hashimoto)) e envolve achar um *input* de *nonce* para o algoritmo que dê um resultado abaixo de um determinado limite, dependendo da dificuldade. O ponto principal dos algoritmos de PoW é que não há estratégia melhor para achar o *nonce* do que enumerar todas as possibilidades, enquanto a verificação da solução é trivial e barata. Se os outputs têm uma distribuição uniforme, então podemos garantir que, na média, o tempo necessário para encontrar um nonce depende do limite relacionado à dificuldade. Isso torna possível o controle do tempo de se encontrar um novo bloco apenas pelo ajuste da dificuldade.

A dificuldade se ajusta dinamicamente para que, na média, um bloco seja produzido pela rede inteira a cada 12 segundos (i.e., 12s de tempo de bloco). Esse ritmo pontua basicamente a sincronização do estado do sistema e garante que manter um fork (para permitir duplo gasto) ou reescrever a história da rede seja impossível a não ser que o atacante possua mais da metade do poder de mineração da rede (o chamado "ataque de 51%").

Qualquer nó participante da rede pode ser um minerador e o retorno esperado da mineração será diretamente proporcional ao poder de mineração (relativo) ou ***hash*rate**, i.e., o número de *nonces* testados por segundo normalizados pelo *hash*rate total da rede.

O PoW Et*hash* é intensivo em memória, tornando-o basicamente resistente a ASIC. Isso significa basicamente que calcular o PoW requer a escolha de subsets de um recurso fixo dependente do nonce e do cabeçalho do bloco. Este recurso (um dado de alguns gigabytes) é chamado de **DAG**. O DAG muda totalmente a cada 30000 blocos (uma janela de 100 horas, chamada **epoch**) e leva um tempo para ser gerado. Como o DAG depende apenas da altura do bloco, ele pode ser pré-gerado mas, se não for, o **client** precisa esperar o final deste processo para produzir um bloco. Até que os **client*s* realmente façam um pré-cache dos DAGs antes do tempo, a rede poderá experienciar um atraso massivo nos blocos em cada transição de epoch. Note que o DAG não precisa ser gerado para verificar o Pow, permitindo essencialmente que a verificação seja feita com CPU baixa e memória pequena.

Como um caso especial, quando você inicia seu nó do zero, a mineração começará apenas uma vez que o DAG for construído para a epoch atual.

##Recompensas de mineração

Note que a mineraração "real" de Ether começará com o lançamento Frontier. Na testnet Olympics, o [pré-lançamento da Frontier](http://ethereum.gitbooks.io/frontier-guide/), o ether minerado não tem valor (mas veja as [recompensas da Olympic](https://blog.ethereum.org/2015/05/09/olympic-frontier-pre-release/)).

O minerador de PoW que for bem-sucedido em encontrar o bloco ganhador receberá:

 * Uma **recompensa estática por bloco** para o bloco "ganhador", consistindo de exatamente 5.0 Ether
 * Todo o combustível usado no bloco, isto é, todo o combustível consumido pela execução de todas as transações no bloco submetido pelo minerador vencedor é compensado por aqueles que enviam as transações. O custo de combustível incorrido é creditado na conta do minerador como parte do protocolo de consenso. Ao longo do tempo, espera-se que esse incentivo seja superior à recompensa estática por bloco.
 * Uma recompensa extra por incluir *Uncles* como parte do bloco, na forma de um extra de 1/32 por *Uncle* incluído.

Uncles são blocos antigos - isto é, com pai que é antecessor (máximo de 6 blocos atrás) do bloco sendo incluído. *Uncles* válidos são recompensados em ordem para neutralizar o efeito do atraso da rede na dispersão de recompensas de mineração, aumentando, assim, a segurança. *Uncles* incluídos em um bloco formado pelo minerador bem-sucedido de PoW recebem 7/8 da recompensa estática por bloco, isto é, 4.375 ether. Um máximo de 2 *Uncles* é permitido por bloco.

##DAG da Et*hash*

Et*hash* usa um **DAG** (*directed acyclic graph*) para o algoritmo de *proof of work*, isto é gerado para cada **epoch**, i.e a cada 30000 blocos (100 horas). O DAG leva um tempo longo para ser gerado. Se o **client** o gera apenas sob demanda, você pode experienciar uma longa espera a cada transição de epoch antes que o primeiro bloco da nova epoch possa ser encontrado. No entanto, o DAG depende apenas no número do bloco, então ele PODE e DEVE ser calculado antecipadamente para evitar a espera durante a transição da epoch. `geth` implementa a geração automática de DAG e mantém 2 DAGs por vez para uma transição suave de epoch. A geração automática de DAG pode ser ativada e desativada quando a mineração é controlada pelo console. Ela também é ativada por *default* se o `geth`for iniciado com a opção `--mine`. Note que **client*s* compartilham um recurso de DAG, então se você estiver rodando múltiplas instâncias de qualquer **client**, garanta que a geração automática de DAG está ativada em no máximo um **client**.

Para gerar o DAG para uma epoch arbitrária:

```geth makedag <block number> <outputdir>```

Por exemplo, `geth makedag 360000 ~/.et*hash*`. Note que o et*hash* usa `~/.et*hash*` (Mac/Linux) ou `~/AppData/Et*hash*` (Windows) para o DAG para que ele possa ser compartilhado entre **client*s*. 
