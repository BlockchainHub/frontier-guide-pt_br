<!--include "git+https://github.com/ethereum/wiki.wiki.git/Ethereum-Contract-ABI.md"-->

# Interop com ABI


# Funções

## Concepção básica

Nós assumimos que a *Application Binary Interface* (ABI) seja fortemente tipada, conhecida no momento de compilação e estática. Nenhum mecanismo de introspecção será fornecido. Nós garantimos que todos os contratos terão as definições de interface de qualquer contrato que eles chamem  e disponíveis no momento de compilação.

Essa especificação não é direcionada a contratos cujas interfaces são dinâmicas ou conhecidas apenas no momento da execução. Se esses casos se tornarem importantes eles podem ser adequadamente tratados como instalações construídas dentro do ecossistema Ethereum.

## Seletor de funções

Os primeiros quatro bytes de de dados da chamada para uma chamada de função especificam a função aser chamada. São os primeiros (esquerda, de alta-ordem em *big-endian*) 4 bytes do *hash* Keccak (SHA-3) da assinatura da função. A assinatura é definida como a expressão canônica do protótipo básico, isto é, o nome da função com a lista de tipos de parâmetros entre parênteses. Tipos de parâmetros são separados por uma única vírgula - nenhum espaço é usado.

## Codificação de argumento

Começando do quinto byte, os argumentos codififcados seguem. Essa codificação também é usada em outros locais, por exemplo, os valores de retorno e também os argumentos de eventos são codificados da mesma forma, sem os quatro bytes especificando a função.

### Tipos

Os seguintes tipos elementares existem:
- `uint<M>`: tipo inteiro não-assinados de `M` bits, `0 < M <= 256`, `M % 8 == 0`. e.g. `uint32`, `uint8`, `uint256`.
- `int<M>`: dois é tipo inteiro assinado complementar de `M` bits, `0 < M <= 256`, `M % 8 == 0`.
- `address`: equivalente a `uint160`, exceto pela interpretação assumida e tipagem de linguagem.
- `uint`, `int`: sinônimo de `uint256`, `int256` respectivamente (não deve ser usado para computar o seletor de função).
- `bool`: equivalente a `uint8` restringido aos valores 0 e 1
- `fixed<M>x<N>`: número de ponto fixo assinado de `M+N` bits, `0 < M + N <= 256`, `M % 8 == N % 8 == 0`. Corresponde ao valor binário equivalente de int256 dividido por `2^M`.
- `ufixed<M>x<N>`: variante não-assinada de `fixed<M>x<N>`.
- `fixed`, `ufixed`: sinônimos de `fixed128x128`, `ufixed128x128` respectivamente não deve ser usado para computar o seletor de função).
- `bytes<M>`: tipo binário de `M` bytes, `0 < M <= 32`.

O seguinte tipo de *array* (tamanho fixo) existe:
- `<type>[M]`: uma *array* de comprimento fixo do dado tipo de comprimento fixo.

Os seguintes tipos de tamanho não fixo existem: 
- `bytes`: sequência de bytes de tamanho dinâmico.
- `string`: *string unicode* de tamanho dinâmico assumida como codificada em UTF-8.
- `<type>[]`: uma *array* de comprimento variável do dado tipo de comprimento fixo.

### Especificaçao formal da codificação

Nós iremos especificar formalmente a codificação para que ela tenha as seguintes propriedades, que são especialmente úteis se alguns argumentos forem *arrays* aninhadas:

**Propriedades:**

1. O número de leituras necessárias para acessar um valor é no máximo a profundidade do valor
dentro da estrutura de *array* de argumentos, isto é, 4 leituras são necessárias para resgatar `a_i[k][l][r]`. Em uma versão anterior da ABI, o número de leituras escalava linearmente com o número de parâmetros dinâmicos no pior dos casos.

2. Os dados de uma variável ou elemento *array* não são intercalados com outros dados e são 
realocáveis, isto é, eles usam apenas "endereços" relativos.

Nós distinguimos tipos estáticos e dinâmicos. Tipos estáticos são codificados no local e tipos dinâmicos são codificados em locações alocadas separadamente após o bloco atual.

**Definição:** Os seguintes tipos são chamados "dinâmicos":
* `bytes`
* `string`
* `T[]` para qualquer`T`
* `T[k]` para qualquer `T` dinâmico e qualquer `k > 0`

Todos os outros tipos são chamados "estáticos".

**Definição:** `len(a)` é o número de bytes de uma *string* binária `a`.
O tipo de `len(a)` é assumido como `uint256`.

Nós definimos `enc`, a real codificação, como um mapeamento de valores dos tipos da ABI para *strings* binárias, de forma que `len(enc(X))` depende do valor de `X` se e somente se o tipo de  `X` for dinâmico.

**Definição:** Para qualquer valor ABI `X`, nós definimos recursivamente `enc(X)`, dependendo se o tipo de `X` é

- `T[k]` para qualquer `T` e `k`:

  `enc(X) = head(X[0]) ... head(X[k-1]) tail(X[0]) ... tail(X[k-1])`

  onde `head` e `tail` são definidos por `X[i]` sendo de um tipo estático como
    `head(X[i]) = enc(X[i])` e `tail(X[i]) = ""` (a *string* vazia)
  e como
    `head(X[i]) = enc(len(head(X[0]) ... head(X[k-1]) tail(X[0]) ... tail(X[i-1])))`
    `tail(X[i]) = enc(X[i])`
  caso contrário.

  Note que no caso dinâmico, `head(X[i])` é bem definido já que os comprimentos das partes *head* dependem apenas dos tipos e não dos valore. O seu valor é a distância do início de `tail(X[i])` relativo ao início de `enc(X)`.
  
- `T[]` onde `X` tem `k` elementos (`k` é assumido como tipo `uint256`):

  `enc(X) = enc(k) enc([X[1], ..., X[k]])`

  isto é, é codificado como se fosse uma *array* de tamanho estático `k`, prefixado com o número de elementos.

- `bytes`, de comprimento `k` (que é assumido como tipo `uint256`):

  `enc(X) = enc(k) pad_right(X)`, isto é, o número de bytes é codificado como
    `uint256` seguido pelo real valor de `X` como uma sequência de bytes, seguido pelo número mínimo de bytes zero, de forma que `len(enc(X))` é um múltiplo de 32.

- `string`:

  `enc(X) = enc(enc_utf8(X))`, isto é, `X` é codificado como utf-8 e esse valor é interpretado como tipo de `bytes` e codificado ainda mais. Note que o comprimento usado nessa codificação subsequente é o número de bytes da *string* codificada em utf-8, não o seu número de caracteres.

- `uint<M>`: `enc(X)` é a codificação *big-endian* de `X`, com base no lado da alta ordem (esquerda) com bytes zero, de forma que o comprimento é um múltiplo de 32 bytes.
- `address`: como no caso `uint160`
- `int<M>`: `enc(X)` é a codificação *big-endian* ~~is the big-endian two's complement encoding of `X`, padded on the higher-oder (left) side with~~ `0xff` para `X` negativo com zero bytes para `X` positivo, de forma que o comprimento é um múltiplo de 32 bytes.
- `bool`: como no caso `uint8`, onde `1` é usado para `true` e `0` para `false`
- `fixed<M>x<N>`: `enc(X)` é `enc(X * 2**N)` onde `X * 2**N` é interpretado como `int256`.
- `fixed`: como no caso `fixed128x128`
- `ufixed<M>x<N>`: `enc(X)` é `enc(X * 2**N)` onde `X * 2**N` é interpretado como `uint256`.
- `ufixed`: como no caso`ufixed128x128`
- `bytes<M>`: `enc(X)` é a sequência de bytes em `X` acomodado com bytes zero para um comprimento de 32.

Note que para qualquer `X`, `len(enc(X))` é um múltiplo de 32.

## Seletor de função e codificação de argumento

Uma chamada para a função `f` com parâmetros `a_1, ..., a_n` é codificada como

  `function_selector(f) enc([a_1, ..., a_n])`

e retorna os valores `v_1, ..., v_k` de `f` codificados como

  `enc([v_1, ..., v_k])`

onde os tipos de `[a_1, ..., a_n]` e `[v_1, ..., v_k]` são assumidos como
*arrays* de tamanho fixo de comprimento `n` e `k`, respectivamente. Note que, estritamente
`[a_1, ..., a_n]` pode ser uma "array" com elementos de diferentes tipo, mas
a codificação ainda é bem definida já que o tipo comum assumido `T` (acima) não é realmente usado.

## Exemplos

Dado o contrato:

```js
contract Foo {
  function bar(fixed[2] xy) {}
  function baz(uint32 x, bool y) returns (bool r) { r = x > 32 || y; }
  function sam(bytes name, bool z, uint[] data) {}
}
```

Portanto, para o nosso exemplo `Foo`, se quiséssemos chamar `baz`  com os parâmetros `69` e `true`, nós passaríamos 68 bytes no total, os quais podem ser quebrados em:

- `0xcdcd77c0`: a ID do método. Isso é derivado como os primeiros 4 bytes do *hash* Keccak da forma ASCII da assinatura `baz(uint32,bool)`.
- `0x0000000000000000000000000000000000000000000000000000000000000045`: o primeiro parâmetro, um uint32 de valor `69` acomodado para 32 bytes
- `0x0000000000000000000000000000000000000000000000000000000000000001`: o segundo parâmetro - *boolean* `true`, acomodado para 32 bytes

No total:
```
0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001
```
Ele retorna um único `bool`. se, por exemplo, retornasse `false`, seus *outputs* seriam a *array* de byte único `0x0000000000000000000000000000000000000000000000000000000000000000`, um único *bool*.

Se nós quiséssemos chamar `bar` com o argumento `[2.125, 8.5]`, nós passaríamos 68 bytes no total, os quais podem ser quebrados em:
- `0x3e279860`: A ID do método. Isso é derivado da assinatura `bar(fixed128x128[2])`. Note que `fixed` é substituído com a sua representação canônica `fixed128x128`.
- `0x0000000000000000000000000000000220000000000000000000000000000000`: a primeira parte do primeiro parâmetro, um fixed128x128 de valor `2.125`.
- `0x0000000000000000000000000000000880000000000000000000000000000000`: a segunda parte do primeiro parâmetro, um fixed128x128 de valor `8.5`.

No total:
```
0x3e27986000000000000000000000000000000002200000000000000000000000000000000000000000000000000000000000000880000000000000000000000000000000
```

Se nós quiséssemos chamar `sam` com os argumentos `"dave"`, `true` e `[1,2,3]`, nós passaríamos 292 bytes no total, quebrados em:
- `0xa5643bf2`: A ID do método. Isso é derivado da assinatura `sam(bytes,bool,uint256[])`. Note que `uint` é substituído com a sua representação canônica `uint256`.
- `0x0000000000000000000000000000000000000000000000000000000000000060`: a localização da parte de dados do primeiro parâmetro (tipo dinâmico), medida em bytes do início do bloco de argumentos. Nesse caso, `0x60`.
- `0x0000000000000000000000000000000000000000000000000000000000000001`: o segundo parâmetro: *boolean true*.
- `0x00000000000000000000000000000000000000000000000000000000000000a0`:  a localização da parte de dados do terceiro parâmetro (tipo dinâmico), medido em bytes. Nesse caso, `0xa0`.
- `0x0000000000000000000000000000000000000000000000000000000000000004`: a parte de dados do primeiro argumento, ela começa com o comprimento da *array* de bytes em elementos, nesse caso, 4.
- `0x6461766500000000000000000000000000000000000000000000000000000000`: o conteúdo do primeiro argumento: a codificação em UTF-8 (igual a ASCII nesse caso) de `"dave"`, acomodada à direita para 32 bytes.
- `0x0000000000000000000000000000000000000000000000000000000000000003`: a parte de dados do terceiro argumento, ela começa com o comprimento da *array* de bytes em elementos, nesse caso, 3.
- `0x0000000000000000000000000000000000000000000000000000000000000001`: a primeira entrada do terceiro parâmetro.
- `0x0000000000000000000000000000000000000000000000000000000000000002`: a segunda entrada do terceiro parâmetro.
- `0x0000000000000000000000000000000000000000000000000000000000000003`: a terceira entrada do terceiro parâmetro.

No total:
```
0xa5643bf20000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000000464617665000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003
```


### Uso de tipos dinâmicos

Uma chamada de uma função com a assinatura `f(uint,uint32[],bytes10,bytes)` com valores `(0x123, [0x456, 0x789], "1234567890", "Hello, world!")` é codificada da seguinte forma:

Nós pegamos os primeiros quatro bytes de `sha3("f(uint256,uint32[],bytes10,bytes)")`, isto é, `0x8be65246`.
Depois, nós codificamos os cabeçalhos de todos os quatro argumentos. Para os tipos estáticos `uint256` e `bytes10`, estes são diretamente os valores que queremos passar, enquanto para os tipos dinâmicos `uint32[]` e `bytes`, nós usamos a distância em bytes ao início de suas áreas de dados, medidos do início da codificação do valor (isto é, não contando os primeiros quatro bytes contendo o *hash* da assinatura da função). Estes são:

 - `0x0000000000000000000000000000000000000000000000000000000000000123` (`0x123` Acomodados para 32 bytes)
 - `0x0000000000000000000000000000000000000000000000000000000000000080` (distância ao início da parte dos dados do segundo parâmetro, 4*32 bytes, exatamente o tamanho do cabeçalho)
 - `0x3132333435363738393000000000000000000000000000000000000000000000` (`"1234567890"` Acomodados para 32 bytes à direita)
 - `0x00000000000000000000000000000000000000000000000000000000000000e0` (distância ao início da parte dos dados do quarto parâmetro = distância ao início da parte dos dados do primeiro parâmetro dinâmico + tamanho da parte de dados do primeiro parâmetro dinâmico = 4*32 + 3*32 (veja abaixo))

Depois disso, a parte de dados do primeiro argumento dinâmico, `[0x456, 0x789]` segue:

 - `0x0000000000000000000000000000000000000000000000000000000000000002` (número de elementos da *array*, 2)
 - `0x0000000000000000000000000000000000000000000000000000000000000456` (primeiro elemento)
 - `0x0000000000000000000000000000000000000000000000000000000000000789` (segundo elemento)

Finalmente, nós codificamos a parte de dados do segundo argumento dinâmico, `"Hello, world!"`:

 - `0x000000000000000000000000000000000000000000000000000000000000000d` (número de elementos (bytes nesse caso): 13)
 - `0x48656c6c6f2c20776f726c642100000000000000000000000000000000000000` (`"Hello, world!"` Acomodados para 32 bytes à direita)

Com tudo junto, a codificação é (nova linha depois do seletor de função e 32 bytes cada para clareza):

```
0x8be65246
0000000000000000000000000000000000000000000000000000000000000123
0000000000000000000000000000000000000000000000000000000000000080
3132333435363738393000000000000000000000000000000000000000000000
00000000000000000000000000000000000000000000000000000000000000e0
0000000000000000000000000000000000000000000000000000000000000002
0000000000000000000000000000000000000000000000000000000000000456
0000000000000000000000000000000000000000000000000000000000000789
000000000000000000000000000000000000000000000000000000000000000d
48656c6c6f2c20776f726c642100000000000000000000000000000000000000
```


# Eventos

Eventos são uma abstração do protocolo de logging/observação de evento da Ethereum. Entradas de Log provêm o endereço do contrato, uma série de até quatro tópicos e alguns dados binários de comprimento arbitrário. Eventos alavancam a existente função ABI para poder interpretar isso (junto com uma especificação de interface) como uma estrutura própriamente tipada. 

Dado um nome de evento e uma série de parâmetros de evento, nós os dividimos em 2 sub-séries: aqueles que são indexados e aqueles que não são. Aqueles que são indexados, que podem ser até 3, são usados junto com o *hash* Keccak da assinatura do evento para formar os tópicos da entrada do log. Aqueles que não são indexados formam a *array* de bytes do evento.

Na realidade, uma entrada de log usando essa ABI é descrita como:

- `address`: o endereço do contrato (intrinsecamente provida pela Ethereum);
- `topics[0]`: `keccak(EVENT_NAME+"("+EVENT_ARGS.map(canonical_type_of).join(",")+")")` (`canonical_type_of` é uma função que simplesmente retorna o tipo canônico de um dado argumento, por exemplo, para `uint indexed foo`, ele retornaria `uint256`). Se o evento for declarado como `anonymous` o `topics[0]` não é gerado;
- `topics[n]`: `EVENT_INDEXED_ARGS[n - 1]` (`EVENT_INDEXED_ARGS` é a série de `EVENT_ARGS` que são indexados);
- `data`: `abi_serialise(EVENT_NON_INDEXED_ARGS)` (`EVENT_NON_INDEXED_ARGS` é a série de `EVENT_ARGS` que não são indexados, `abi_serialise` é a função de serialização ABI usada para retornar uma série de valores tipados de uma função, como descrito acima).

# JSON

O formato JSON para uma interface de contrato é dado por uma *array* de descrições de função e/ou evento. A descrição de uma funçao é um objeto JSON com os campos:

- `type`: `"function"` ou `"constructor"` (pode ser omitido, tendo como *default* function);
- `name`: o nome da funçao (apenas presente para tipos de funções);
- `inputs`: uma *array* de objetos, cada uma contendo:
  * `name`: o nome do parâmetro;
  * `type`: o tipo canônico do parâmetro.
- `outputs`: uma *array* de objetos similar a `inputs`, pode ser omitido.

Uma descrição de evento é um objeto JSON com campos razoavelmente similares:

- `type`: sempre `"event"`
- `name`: o nome do evento;
- `inputs`: uma *array* de objetos, cada uma contendo:
  * `name`: o nome do parâmetro;
  * `type`: o tipo canônico do parâmetro.
  * `indexed`: `true` se o campo é parte dos tópicos do log, `false` se é um dos segmentos de dados do log.
- `anonymous`: `true` se o evento foi declarado como `anonymous`.

Por exemplo, 

```js
contract Test {
function Test(){ b = 0x12345678901234567890123456789012; }
event Event(uint indexed a, bytes32 b)
event Event2(uint indexed a, bytes32 b)
function foo(uint a) { Event(a, b); }
bytes32 b;
}
```

resultaria no JSON:

```js
[{
"type":"event",
"inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
"name":"Event"
}, {
"type":"event",
"inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
"name":"Event2"
}, {
"type":"event",
"inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
"name":"Event2"
}, {
"type":"function",
"inputs": [{"name":"a","type":"uint256"}],
"name":"foo",
"outputs": []
}]
```

# Exemplo de uso de JavaScript

```js
var Test = eth.contract(
[{
"type":"event",
"inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
"name":"Event"
}, {
"type":"event",
"inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
"name":"Event2"
}, {
"type":"function",
"inputs": [{"name":"a","type":"uint256"}],
"name":"foo",
"outputs": []
}]);
var theTest = new Test(addrTest);

// exemplos de uso:
// toda entrada de log ("event") vinda de theTest (isto é, Event & Event2):
var f0 = eth.filter(theTest);
// apenas entradas de log ("events") do tipo "Event" vindas de theTest:
var f1 = eth.filter(theTest.Event);
// também escrito como
var f1 = theTest.Event();
// apenas entradas de log ("events") do tipo "Event" e "Event2" vindas de theTest:
var f2 = eth.filter([theTest.Event, theTest.Event2]);
// apenas entradas de log ("events") do tipo "Event" vindas de theTest com parâmetro indexado 'a' igual a 69:
var f3 = eth.filter(theTest.Event, {'a': 69});
// também escrito como
var f3 = theTest.Event({'a': 69});
// apenas entradas de log ("events") do tipo "Event" vindas de theTest com parâmetro indexado 'a' igual a 69 ou 42:
var f4 = eth.filter(theTest.Event, {'a': [69, 42]});
// também escrito como
var f4 = theTest.Event({'a': [69, 42]});

// opções também podem ser fornecidas como um segundo parâmetro com `earliest`, `latest`, `offset` e `max`, como definido para `eth.filter`.
var options = { 'max': 100 };
var f4 = theTest.Event({'a': [69, 42]}, options);

var trigger;
f4.watch(trigger);

// chama Foo para criar um evento:
theTest.foo(69);

// chamaria o trigger como:
//trigger(theTest.Event, {'a': 69, 'b': '0x12345678901234567890123456789012'}, n);
// onde n é o número do bloco onde o evento é acionado.
```

Implementação:

```js
// ex. f4 seria similar a:
web3.eth.filter({'max': 100, 'address': theTest.address, 'topics': [ [69, 42] ]});
// exceto que os dados resultantes precisariam ser convertidos do formato básico de entrada de log como:
{
  'address': theTest.address,
  'topics': [web3.sha3("Event(uint256,bytes32)"), 0x00...0045 /* 69 in hex format */],
  'data': '0x12345678901234567890123456789012',
  'number': n
}
// em dados bons para o acionamento (trigger), especificamente os três campos:
  Test.Event // derivável do primeiro tópico
  {'a': 69, 'b': '0x12345678901234567890123456789012'} // derivável do bool 'indexed' na interface, o último 'topics' e o 'data'
  n // do 'number'
```

Resultado do evento:
```js
[ {
  'event': Test.Event,
  'args': {'a': 69, 'b': '0x12345678901234567890123456789012'},
  'number': n
  },
  { ...
  } ...
]
```

### Acabado! [develop branch]
**NOTA: Isso é ANTIGO - IGNORE a não ser que esteja lendo por proprósitos históricos**

- *Internal LogFilter*, mecanismo de correspondência de entradas de log e eth_installFilter necessários para suportar a correspondência de múltiplos valores (OU semântica) *por* índice de tópico (no presente, irá apenas corresponder tópicos E semântica e *set-inclusion*, não *per-index*).

Isto é, atualmente, você só pode pedir por cada um de um dado número de valores de tópicos para serem correspondidos através de cada tópico:

- `topics: [69, 42, "Gav"]` seria correspondido com logs de 3 tópicos `[42, 69, "Gav"]`, `["Gav", 69, 42]` mas **não** com logs de tópicos `[42, 70, "Gav"]`.

Nós precisamos ser capazes de prover um de um número de valores de tópicos e cada uma dessas opções para cada índice de tópico:

- `topics: [[69, 42], [] /* qualquer coisa */, "Gav"]` seria correspondido com logs de 3 tópicos `[42, 69, "Gav"]`, `[42, 70, "Gav"]` mas **não** com `["Gav", 69, 42]`.