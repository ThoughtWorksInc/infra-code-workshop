## Lab02 - Manipulando credenciais na automação

### Parte 1 - Manipulando as credenciais permanentes (LLC)
Vamos trabalhar carregando nossas credenciais permamentes nas variáveis de ambiente do container e antes de executar qualquer comando, vamos assumir a _role becomeAdmin_. Para evitar estragos caso nossas credenciais permamentes vazem, vamos primeiro reduzir o acesso de **PowerUserAccess** para **ReadOnlyAccess**

* Acesse a console AWS com seu(sua) usuário(a) criado(a) no [Lab01](lab01.md#parte-1---completar-items-do-iam-security-status-e-deixá-los-todos-verdinhos)

  https://devopslabs<# da sua conta>.signin.aws.amazon.com/console

* Faça o _Switch Role_ para _becomeAdmin@devopslabs<#>_ como no Lab01

* Clique

  IAM > Groups > dev > (aba) Permissions

* Remova a politíca **PowerUserAccess** clicando em **_Detach Policy_** e em seguida **_Detach_**

* Clique **_Attach Policy_**

* Selecione **_ReadOnlyAccess_** e clique **_Attach Policy_**

* Verifique o conteúdo da política clicando em **_Show Policy_**

* Esta permissão restringe até a ação **_sts:AssumeRole_**, necessária para o Switch Role que acabamos de fazer

* Para continuar permitindo a troca de **_role_** clique

  Policies > Create Policy > Policy Generator

  * **_Effect_:** Allow
  * **_AWS Service_:** AWS Security Token Service
  * **_Actions_:** AssumeRole
  * **_Amazon Resource Name (ARN)_:** arn:aws:iam::111111111111:role/becomeAdmin
  Você pode encontrar sua _role ARN_ no capitulo1-lab02

* Clique **_Add Statement_** e **_Next Step_**

* Altere **_Policy Name_** para _allowBecomeAdmin_

* De volta à lista de _Policies_, clique sobre a política criada

* Na aba **_Attached Entities_**, clique **_Attach_**

* Selecione o grupo _dev_ e clique em **_Attach Policy_**

* Pronto. Tudo certo, feche a console

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

### Parte 2 - Manipulando as credenciais temporárias (SLC)
No [capítulo anterior](../capitulo1/lab02.md#slc---short-lived-credentials-credenciais-de-vida-curta) usamos o comando `aws sts assume-role` para pegar credenciais temporárias.
Mas o retorno foi um JSON e tivemos e copiar que colar o resultado nas variáveis de ambientes para só então fazer uso do acesso com permissões elevadas.
Neste lab vamos apresentar uma das formas de automatizar isso.

* Crie o arquivo `entrypoint.sh` na raiz do repositório com o seguinte código
```
#!/bin/bash -e
# with pieces borrowed from Erik Doernenburg:
# https://my.thoughtworks.com/groups/techops-community/blog/2015/11/17/aws-account-access-via-temporary-api-tokens#comment-38435
ROLE=becomeAdmin
ACCOUNT=644540006937
DURATION=3600

# KST=access*K*ey, *S*ecretkey, session*T*oken
KST=(`aws sts assume-role --role-arn "arn:aws:iam::$ACCOUNT:role/$ROLE" \
--role-session-name buildpack-iac \
--duration-seconds $DURATION \
--query '[Credentials.AccessKeyId,Credentials.SecretAccessKey,Credentials.SessionToken]' \
--output text`)

export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION:-us-east-1}
export AWS_ACCESS_KEY_ID=${KST[0]}
export AWS_SECRET_ACCESS_KEY=${KST[1]}
export AWS_SESSION_TOKEN=${KST[2]}

exec $@
```

* Adicione o arquivo à imagem docker e use-o como comando de entrada
```
# Dockerfile
...
WORKDIR /iac
COPY entrypoint.sh entrypoint.sh

ENTRYPOINT ["./entrypoint.sh"]
```

* Abra um _shell_ no container e verifique se o script funciona
```
make docker.shell
```

* O script não existe no container, certo?

* Temos que rodar o _build_ da imagem novamente

* Altere o arquivo _make_ para adicionar a etapa **docker.build** como dependência da **docker.shell**
```
# Makefile
...
docker.shell: docker.build
  @docker run --rm \
...
```

* Tente usar o _script_ no container novamente, realizando uma operação que cria recurso na AWS
```
make docker.shell
root@46598b963f3f:/iac# aws s3api create-bucket --bucket meuTeste
{
    "Location": "/meuTeste"
}
```

É isso 😉, agora o container tem permissões suficientes para executar as próximas tarefas que vamos automatizar.
