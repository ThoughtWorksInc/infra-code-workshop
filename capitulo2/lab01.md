## Lab01 - Customizando uma _docker image_

### Parte 1 - Criando uma imagem docker com os softwares instalados manualmente no lab anterior
* Inicie um repositório git chamado **infra-code-devopslabs<##>** onde ## é o número da sua conta.
```
mkdir infra-code-devopslabs##
cd infra-code-devopslabs##
git init
```

* Crie um `Dockerfile` com o conteúdo que reproduz os comandos executados no lab anterior
```
# Dockerfile
FROM python:2

RUN apt-get update && apt-get install -y groff
RUN pip install awscli
```

* Crie um `Makefile` com o comando de _build_ da imagem docker
```
# Makefile
docker.build:
  docker build -t buildpack-iac .
```
**ATENÇÃO:** Em arquivos _make_, a indentação da primeira linha de comando (receita/_recipe_) após o alvo (_target_) deve ser indentada com TAB

* Construa sua imagem
```
make
```
`make` por padrão roda o primeiro _target_ do arquivo `Makefile` no diretório corrente, caso nem um nem outro sejam indicados.

* Crie um _target_ que nos dê um _shell_ dentro container para que seja fácil trabalhar no mesmo contexto que nosso _build_ ocorre
```
# Makefile
...
docker.shell
  docker run --rm -it buildpack-iac /bin/bash
```

### Parte 2 - Manipulando as credenciais permanentes (LLC)
Vamos trabalhar carregando nossas credenciais permamentes nas variáveis de ambiente do container e antes de executar qualquer comando, vamos assumir a _role becomeAdmin_. Para evitar estragos caso nossas credenciais permamentes vazem, vamos primeiro reduzir o acesso de **PowerUserAccess** para **ReadOnlyAccess**

* Acesse a console AWS com seu(sua) usuário(a) criado(a) no [Lab01](lab01.md#parte-1---completar-items-do-iam-security-status-e-deixá-los-todos-verdinhos)

  https://devopslabs<# da sua conta>.signin.aws.amazon.com/console

* Faça o _Switch Role_ para _becomeAdmin@devopslabs<#>_ como no Lab01

* Clique

  IAM > Groups > dev > (aba) Permissions

* Remova a politíca **PowerUserAccess** clicando em **_Detach Policy_** e em seguida **_Detach_**

* Clique **_Attach Policy_**

* Selecione **_ReadOnlyAccess_** e clique **_Attach Policy_**

* Verifique o conteúdo da política clicando em **_Show Policy_** e feche a console

* Na raiz do repositório crie um arquivo `.env` e salve as credenciais de seu(sua) usuário(a) nele da seguinte forma
```
# .env
export BUILD_AWS_ACCESS_KEY_ID=<sua AccessKey>
export BUILD_AWS_SECRET_ACCESS_KEY=<sua SecretAccessKey>
```

* Crie o arquivo `.gitignore` e inclua o arquivo `.env`
```
# .gitignore
.env
```

* Carregue estas variáveis em sua sessão
```
source .env
```

* Modifique o arquivo _make_ para que estas credenciais sejam passadas para o container
```
# Makefile
...
docker.shell:
  docker run --rm \
    -e "AWS_ACCESS_KEY_ID=${BUILD_AWS_ACCESS_KEY_ID}" \
    -e "AWS_SECRET_ACCESS_KEY=${BUILD_AWS_SECRET_ACCESS_KEY}" \
    -it buildpack-iac /bin/bash
```

* Rode o container e verifique seu funcionamento
```
make docker.shell
root@6a4b5ff9e134:/# aws s3 ls
root@6a4b5ff9e134:/# aws iam list-users
```

* Notou que é possível listar usuários sem assumir a _role_ com acessos administrativos? A política **_ReadOnlyAccess_** permite inclusive a leitura de objetos do serviço IAM, mas nenhuma criação ou alteração de qualquer item em qualquer serviço será permitida, por exemplo:
```
aws s3api create-bucket --bucket meuTeste

An error occurred (AccessDenied) when calling the CreateBucket operation: Access Denied
```

* Há ainda um outro problema, o `make` por padrão ecoa cada comando executado no terminal, o que expõe nossas credenciais. Para mudar este comportamento, adicione um `@` ao início da receita
```
# Makefile
...
docker.shell:
  @docker run --rm \
...

```

* Execute o container novamente e verifique se o `make` não imprime mais o comando
```
make docker.shell
```

