{% sections "account-types-and-transactions", "" %}
{% endsections %}

<--! Source: https://github.com/ethereum/go-ethereum.wiki.git/Contracts-and-Transactions.md" %} -->

# Tipos de contas e transações

Há dois tipos de contas no estado da Ethereum:
* Conta normal ou controlada externamente e
* contratos, i.e., fragmentos de código, pense uma classe ~~i.e., snippets of code, think a class.~~

Ambos tipos de contas possuem um balanço de ether.

Transações podem ser iniciadas de ambos tipos de contas, apesar de que contratos iniciam apenas transações em resposta a outras transações que eles receberam. Portanto, toda ação na blockchain da ethereum é movimentada por transações iniciadas por contas controladas externamente.

As transações mais simples são as de transferência de ether. Mas, antes de entrarmos nisso, você deveria ler sobre [contas](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts) e talvez sobre [mineração](https://github.com/ethereum/go-ethereum/wiki/Mining).
