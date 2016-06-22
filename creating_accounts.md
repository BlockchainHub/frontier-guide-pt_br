# Criando Contas

## Criando uma Nova Conta

```
geth account new
```

Cria uma nova conta e imprime o endereço.

No console, use:
```
> personal.newAccount("senha")
```

A conta é salva em formato encriptado. Você **precisa** lembrar essa senha para destrancar sua conta no futuro.

Para uso não-interativo, a senha pode ser especificada com a flag `--password`:
```
geth --password <passwordfile> account new
```

Note, isso foi feito para ser usado apenas para testes, é uma má ideia salvar a sua senha em um arquivo ou expô-la de qualquer outra forma.

## Criando uma conta pela importação de uma chave privada

```
    geth account import <keyfile>
```

Importa uma chave privada não encriptada de `<keyfile>` e cria uma nova conta e imprime o endereço.

Assume-se que o arquivo *keyfile* contém uma chave privada não encriptada como *bytes* simples representando a curva eliptica em hex.

A conta é salva em formato encriptado, você é pedido para entrar uma senha através de um prompt.

Você deve lembrar essa senha para destrancar a sua conta no futuro.

Para uso não-interativo, a senha pode ser especificada com a flag `--password`:
```
geth --password <passwordfile> account import <keyfile>
```

**Nota**:
Como você pode copiar diretamente suas contas encriptadas para uma outra instância da ethereum, esse mecanismo de `import`/`export` não é necessáro para transferir uma conta entre nós.


**Aviso:** 
Quando você copia chaves para o *keystore* de um nó existente, a ordem das contas que você está acostumado pode mudar. Por isso, tenha certeza de não confiar na ordem da conta ou de checar e atualizar os índices usados nos seus scripts.


**Aviso:**
Se você usar a flag `--password` com um arquivo de senha, é melhor ter certeza de que o arquivo não pode ser listado nem listado para ninguém além de você. Você pode fazer isso assim:

```
touch /path/to/password 
chmod 700 /path/to/password
cat > /path/to/password
>I type my pass here^D
```

## Atualizando uma conta existente

Você pode atualizar uma conta existente na linha de comando com o subcomando `update`, usando o endereço da conta ou o índex como parâmetro.

```
geth account update b0047c606f3af7392e073ed13253f8f4710b08b6
geth account update 2
```

A conta é salva na nova versão em formato encriptado, uma senha é solicitada à você para destrancar a conta e outra para salvar o arquivo atualizado.

Este mesmo comando pode, então, ser usado para migrar uma conta de uma formato obsoleto para o novo formato ou para mudar a senha de uma conta. 

Para uso não-interativo, a senha pode ser especificada com a flag `--password`:
```
geth --password <passwordfile> account new
```
Como apenas uma senha pode ser fornecida, somente a atualização de formato pode ser feita. A mudança de senha só pode ser feita interativamente.

**Nota**: 
A atualização de conta tem o efeito colateral de mudar a ordem das suas contas.

Após uma atualização com sucesso, todos os formatos/versões anteriores da mesma chave serão removidos!
