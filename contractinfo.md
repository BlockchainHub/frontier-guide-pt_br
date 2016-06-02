<!-- "git+https://github.com/ethereum/wiki.wiki.git/Contract-Metadata-Docs-(NatSpec,-ABI).md" -->

Este é o principal ponto de entrada para NatSpec e detalha de forma geral um _standard_ (padrão) seguro e eficiente para distribuição de metadados da ethereum.

Por metadados nós queremos dizer toda a informação relacionada a um contrato que é vista como relevante para ele e ligada de forma imutável a uma versão específica de um contrato na blockchain da ethereum.
Isso inclui:

* Código fonte do contrato
* [Definição da ABI](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) 
* [Documentação de usuário NatSpec](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format#user-documentation)
* [Documentação de desenvolvedor NatSpec](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format#developer-documentation)

Estes recursos têm sua _standard specification_ (especificação padrão) em formato json, idealmente feito para ser produzido por infraestruturas de IDE ou diretamente por compiladores.

Por exemplo, o compilador de [solidity](https://github.com/ethereum/wiki/wiki/Solidity-Tutorial) oferece um estilo `doxygen` de especificar natspec com comentários inteligentes em linha. Ao compilar, ele cria ambos a documentação de usuário NatSpec e a definição ABI. Mas note que não há nada inerentemente específico ao solidity nesses dados e outras linguagens de contrato são encorajadas a implementar seu suporte de NatSpec/ABI com IDEs potencialmente o extendendo.

Como DAPPs e IDEs irãi tipicamente querer interagir com esses recurso, padronizar a sua implementação e distribuição é importante para uma experiência suave na ethereum.

Um exemplo especiamente importante disso é o **esquema de notificação de confirmação de transação NatSpec**, o qual nós usaremos para ilustrar o ponto. Entretanto, a estratégia descrita aqui se extende arbitrariamente para metadados imutáveis fixados em um contrato ethereum.

# Notificação de confirmação de transação

A documentação de usuário NatSpec permite que criadores de contratos possam anexar notificações de confirmação customizadas para cada método.
Uma característica poderosa do NatSpec é prover *templating* que permite que parte das notificações de usuário possam ser instanciadas dependendo dos parâmetros da transação real enviada ao contrato.

Implementações confiadas de *client* ethereum são precisas para chamar de seus *backends* instanciando a notificação de transação natspec dos reais dados de transação e apresentando-os para o usuário como confirmação. 

Isto serve como uma primeira linha de defesa contra transações ilegítimas enviadas por DAPPs maliciosas em nome do usuário.

Certamente, isso só é possível se 
- o nó pode confirar na autencidade das fontes de metadados
- nós têm acesso seguro e confiável às fontes de metadados

Vamos ver como isso é alcançado.

## Autenticação de metadados

**Proposta**
Os *metadocs* são assumidos como uma única  estrutura JSON chamada arquivo `cmd`, que significa _contract metadata doc_.

O conteúdo do arquivo `cmd` é transformado em *hash* e o *hash* do conteúdo é registrado em um registro de nome via um contrato na blockchain da ethereum sob o *hash* do código, veja https://github.com/ethereum/dapp-bin/blob/master/NatSpecReg/contract.sol _Registering_ inesse contexto significa simplesmente que um par de valores-chaves é gravado em um armazenamento de contrato imutável como resultado de uma transação enviada ao contrato de registro.

Isso provê uma autenticação pública e imutável para os metadados do contrato, já que:
- a autencididade do link entre contrato e metadados é segurada por um consenso da ethereum
- a autencidade de metadados reais é segurada por ter sido feito *hash* do conteúdo
- a ligação é inviolável 


Ambientes de IDE de DAPP deveriam suportar a funcionalidade, para quando você criar um contrato, todo os seus metadados padrão são 
- agrupados em um único arquivo json `cmd`
- o código fonte compilado é *hasheado* com keccak -> `Sha3(<evmcode>)`
- `cmd` json é *hasheado* com keccak -> `Sha3(<cmdjson>)`
- os metadados são registrados com o contrato enviando uma transação para um registro de nome confiado. A transação grava simplesmente `Sha3(<evmcode>) -> Sha3(<cmdjson>)` como um par de valores-chave em armazenamento do contrato.

Para evitar agentes maliciosos sequestrando metadados ao sobrescrever a entrada de registro de nome, nós propomos os seguintes processos de negócio:

- o link deve ser registrado antes de um contrato ser implementado e ser imutável dali em diante.
- antes do contrato correspondente ser implementado, nós checamos se o agrupamento de metadados corretos está no registro e foi confirmado com um nível satisfatório de certeza (X blocos).

Como se parece em `Alethzero` está ilustrado [aqui](
https://github.com/ethereum/wiki/wiki/NatSpec-Example)

## Acesso a metadados

Para prover um serviço robusto (resitente a falhas, 100% de *uptime*), serviços de armazenamento de arquivos descentralizados podem e serão usados. No caso de sistemas direcionados a conteúdo como Swarm, para acessar e pegar o recurso você precisará apenas do *hash* de conteúdo `cmd` (usando `BZZHASH(<cmdjson>`).

Até que esse ponto seja alcançado, nós iremos voltar a usar distribuição HTTP. Para permitir isso, nós incluiremos um de muitos contratos URLHint, que dão dicas de URLs que permitem o download de *hashes* de conteúdo específicos. Encontre o contrato em *dapp-bin*.

O conteúdo baixado deve ser tratado de muitas formas (e *hasheado*) para se descobrir o que é o conteúdo. Possíveis formas incluem codificação em base 64, codificação em hex e raw, e qualquer corte de conteúdo necessário (ex: uma página HTML deve ter tudo até as tags *body* removido).

Será responsabilidade de quem fez upload da dapp/do conteúdo manter as entradas de URLHint atualizadas.

O endereço do contrato URLHint será especificado em uma base ad-hoc e usuários serão capazes de adicionar mais nos seus browsers. 

Essa funcionalidade de fazer upload e registrar a localização deveria ser suportada por infraestruturas de desenvolvedor e IDE. Desenvolvedores (ou IDE) são encorajados a registrar a localização cmd _antes_ do registro do *hash* de conteúdo (e, portanto, antes da real criação do contrato) para evitar sequestro de metadados. Note que se o registro do *hash* de conteúdo estiver correto, sequestrar a localização não permite conteúdo malicioso (já que o *hash* de conteúdo autentica o `cmd`), no entanto, isso ainda pode impedir que os seus metadados estejam acessíveis. Portanto 
- o armazenamento do contrato de *url-hint* deve ser, por definição, imutável
- *hash* do conteúdo para* url-hint* deve ser registrado antes da criação do contrato

(certamente, com antigas urls web, nós estamos expostos a falha/hacking de servidor, etc)

Leia mais [aqui](https://github.com/ethereum/wiki/wiki/NatSpec-Determination)

## Contratos de registro de nome

Interoperabilidade requer que *clients* saibam onde procurar para resolver os metadados de um contrato, isto é, qual registro de nome usar. Atualmente, nós resolvemos isso assim:
- criando os dois contratos de registro após o bloco gênesis
- e inserindo diretamente (*hardwiring*) seus endereços no código do *client* (ex: no módulo de notificação de confirmação de transação do natspec no lado do *client* e o módulo de *deployment* no lado do dev/IDE).


