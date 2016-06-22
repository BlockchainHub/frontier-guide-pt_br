# Administrando contas

**Aviso**
Não esqueça a sua senha. 

Se você perder a senha usada para encriptar a sua conta, você não será capaz de acessar esta conta.
Repetindo: NÃO é possível acessar a sua conta sem a senha e não há uma opção de _esqueci minha senha_ para fazer a recuperação. Não a esqueça.


**Nota**: A convenção de nomenclatura do arquivo chave mudou em `0.9.36`. Este documento tem o objetivo de refletir informações corretas das contas, da forma como elas são usadas no lançamento da frontier.

A CLI `geth` da Ethereum provê a administração de contas por meio do subcomando `account`:

```
account command [arguments...]
```

Administrar contas permite que você crie novas contas, liste todas as contas existentes, importe uma chave privada para uma nova conta, migre para os mais novos formatos de chave e mude sua senha.

Ela suporta um modo interativo, quando sua senha é pedida em um *prompt*, assim como um modo não-interativo, no qual as senhas são fornecidas através de um determinado arquivo de senha. O modo não-interativo é feito apenas para o uso com *script* em redes de teste ou ambientes conhecidamente seguros.

Garanta que você não irá esquecer a senha que você escolheu ao criar uma nova conta (com `new`, `update` ou `import`). Sem ela, você não será capaz de destrancar a sua conta.

Note que NÃO há suporte para a exportação de sua chave em formato não encriptado.

As chaves são guardadas sob `<DATADIR>/keystore`. Lembre-se de fazer back-up de suas chaves regularmente! Veja [DATADIR backup & restore](https://github.com/ethereum/go-ethereum/wiki/Backup-&-restore) para maiores informações. O mais novo formato dos arquivos de chave é: `UTC--<created_at UTC ISO8601>-<address hex>`. A ordem das contas quando listadas é lexicográfica mas, como consequência do formato do *timestamp*, ela é na verdade a ordem de criação.

É seguro transferir o diretório completo ou as chaves individuais contidas nele entre nós ethereum. Note que, caso você esteja adicionando chaves de outro nó ao seu nó, a ordem das contas pode mudar. Então tenha certeza de que você não está confiando nem mudou o índice em seus *scripts* ou fragmentos de código.

E, finalmente, **NÃO ESQUEÇA A SUA SENHA**.

```
SUBCOMANDOS:

        list    imprime os endereços da conta
        new     cria uma nova conta
        update  atualiza uma conta existente
        import  importa uma chave privada para uma nova conta
        

```

Você pode obter informações sobre outros subcomandos usando `geth account help <subcommand>`.

Contas também podem ser administradas através do [Console de JavaScript](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console).

## Exemplos
### Uso interativo

#### Criando uma conta

```
$ geth account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat Passphrase:
Address: {168bc315a2ee09042d83d7c5811b533620531f67}
```

#### Listando contas

```
$ geth account list
Account #0: {a94f5374fce5edbc8e2a8697c15331677e6ebf0b}
Account #1: {c385233b188811c9f355d4caec14df86d6248235}
Account #2: {7f444580bfef4b9bc7e14eb7fb2a029336b07c9d}
```

#### Importando uma chave privada

```
$ geth --datadir /someOtherEthDataDir  account import ./key.prv
The new account will be encrypted with a passphrase.
Please enter a passphrase now.
Passphrase:
Repeat Passphrase:
Address: {7f444580bfef4b9bc7e14eb7fb2a029336b07c9d}
```

#### Atualizando uma conta

```
$ geth account update a94f5374fce5edbc8e2a8697c15331677e6ebf0b
Unlocking account a94f5374fce5edbc8e2a8697c15331677e6ebf0b | Attempt 1/3
Passphrase:
0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b
Account 'a94f5374fce5edbc8e2a8697c15331677e6ebf0b' unlocked.
Please give a new password. Do not forget this password.
Passphrase:
Repeat Passphrase:
0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b
```

### Uso não-interativo
Você fornece um arquivo de senha em texto simples como argumento para a *flag* `--password`. O conteúdo do arquivo consiste dos caracteres simples da senha, seguidos por uma única nova linha.

**Nota**:
Fornecer a senha diretamente como parte da linha de comando não é recomendado, mas você sempre pode usar truques de terminal para contornar essa restrição.

```
$ geth --password /path/to/password account new

$ geth --password /path/to/password account update b0047c606f3af7392e073ed13253f8f4710b08b6

$ geth account list

$ geth --datadir /someOtherEthDataDir --password /path/to/anotherpassword account import ./key.prv
```
