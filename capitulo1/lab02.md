## Lab02 - Acesso e switch role através da CLI
**CLI:** _Command Line Interface_

### Acesso
A ferramenta de linha de comando da AWS e seus SDKs procuram por credenciais em diversos lugares. A precedência é:

* Command Line Options
* Environment Variables – AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, etc.
* The AWS credentials file – ~/.aws/credentials | C:\Users\USERNAME \.aws\credentials 
* The CLI configuration file – ~/.aws/config | C:\Users\USERNAME \.aws\config
* Instance profile credentials

Detalhes aqui:
http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html
ou Google > "aws credentials"

### LLC e SLC
Existem dois tipos de credenciais que podem ser usadas na sua automação: **LLC** e **SLC**

#### LLC - Long Lived Credentials (Credenciais de Vida Longa)
Duram pra sempre e dependem do time ter uma boa prática de rotacionamento de suas credenciais, ou seja, duram pra sempre 😋

Exemplo:
```
AWS_ACCESS_KEY_ID=AKIABCDEF23BXPETLZZA
AWS_SECRET_ACCESS_KEY=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
```

##### Parte 1 - Usando AWS com LLC
* Inicie um _shell_ no container
```
docker run -it python:2 /bin/bash
```

* Instale a `awscli`
```
pip install awscli
```

* Pegue as credenciais do seu usário, lá do arquivo baixado no Lab01, e as declare usando as variáveis de ambiente **LLC**
```
export AWS_ACCESS_KEY_ID=<sua AccessKey>
export AWS_SECRET_ACCESS_KEY=<sua SecretAccessKey>
```
**ou** via arquivo de configuração
```
aws configure
```

* Teste o acesso
```
aws s3 ls
aws ec2 describe-instances
aws ec2 describe-instances --region us-east-1
```

##### Parte 2 - Explorando a awscli
* A `awscli` tem diversos subcomandos
```
aws x
```

* Ajuda específica para cada subcomando
```
aws s3 help
```

* Instale a dependência que está faltando no container
```
apt-get install groff
```

* O container vem com o cache do repositórios de pacotes vazios. Popule o cache e aí sim execute a instalação do `groff`
```
apt-get update
apt-get install groff
```

* Tente usar a ajuda novamente
```
aws s3 help
aws s3 cp help
aws ec2 help
aws ec2 run-instances help
```

* Tente listar os usuários da sua conta AWS
```
aws iam x
aws iam list-users
```

* Erro
```
An error occurred (AccessDenied) when calling the ListUsers operation: 
User: arn:aws:iam::644540006937:user/rafa is not authorized to perform: 
iam:ListUsers on resource: arn:aws:iam::644540006937:user/
```

**Importante:** Não encerre o _shell_ no container. Vamos usar na próxima parte.

O erro acontece porque nosso usuário faz parte de um grupo (dev) que tem permissão **PowerUserAccess**, que não permite acesso a IAM, lembra?
Vamos então executar o _"switch role"_ através da linha de comando e aprender sobre as credenciais temporárias.

#### SLC - Short Lived Credentials (Credenciais de Vida Curta)
Também conhecidas como credenciais temporárias, elas duram até no máximo 60 minutos, exigindo que você as renove após a expiração. Há casos onde elas renovam-se automaticamente a cada 15 minutos, como quando usando em um _Instance Profile_ (assunto para mais tarde). A `awscli` e os SDKs precisam das duas chaves usadas com LLC e mais uma terceira:

```
AWS_ACCESS_KEY_ID=ASIAICJTTNHDQIRPP3CQ
AWS_SECRET_ACCESS_KEY=OMimmxumm47tnIlXrj1DJ6Su6jsFttldzDI9yzRX
AWS_SESSION_TOKEN=FQoDYXdzEH0aDFT+EJo8fs+LkikIMiLOAULzrmBTdk4VqOJ//k+npzQRo3lM+E1NUGlakiy1PVs+eyo+VtAKK3+vO7Z7P8IId51BltSowsXGfcN8M5cgIfSl2KsTNKJ5t2Qz9Nf/KfXSrxUVOACNuzdtw9DFITpITXxTGIOHNFOoG4MdpR0V+Hv+d1M9VDH3ngWC9Km6mk7/opJbLzdATgdzBbfN30HKVCsXgjP3rG4CVXHeYp56UDAvRNSO8qPY/MgEuLVRUfWxfNshpkr9GB4UDxZ5alKmZxExEEGP9WYpmqL8/wC4KJ7NmMEF
```

Não é possível começar do zero usando credenciais temporárias.
Por isso, a recomendação é dar acessos bem restritos a usuários(as) que autenticam-se com credenciais permanentes (ou através de federação) e criar **_policies_** (políticas) específicas para cada tipo de operação necessária no seu processo de desenvolvimento da infraestrutura.

Depois de criadas estas **_policies_**, delega-se acesso aos usuários(as) restritos com **_roles_**, como a **_becomeAdmin_** que criamos no Lab01.

Tudo bem, nossa _role becomeAdmin_ está longe de ser específica, mas serve para efeito didático 😁

Para fazer uso desta permissão na linha de comando, como fizemos através do _Switch Role_ na console Web, solicitamos ao serviço STS (_Security Token Service_) as credenciais temporárias com o nível de acesso maior.

##### Parte 1 - Switch Role na CLI chama-se "assume role"
* Verifique os argumentos que vamos ter que preencher para construir o comando
```
aws sts x
aws sts assume-role help
```

* Foque nesta seção (as com colchetes são opicionais)
```
SYNOPSIS
            assume-role
          --role-arn <value>
          --role-session-name <value>
          [--policy <value>]
          [--duration-seconds <value>]
          [--external-id <value>]
          [--serial-number <value>]
          [--token-code <value>]
          [--cli-input-json <value>]
          [--generate-cli-skeleton]
```

* Precisamos descobrir o ARN da _role becomeAdmin_, já o `--role-session-name` pode ser um texto qualquer

* Acesse a console AWS com seu(sua) usuário(a) criado(a) no [Lab01](lab01.md#parte-1---completar-items-do-iam-security-status-e-deixá-los-todos-verdinhos)
https://devopslabs<# da sua conta>.signin.aws.amazon.com/console

* Faça o _Switch Role_ para _becomeAdmin@devopslabs<#>_ como no Lab01

* Clique
IAM > Roles > becomeAdmin

* Copie o **_Role ARN_**
**_ARN_:** é [_Amazon Resource Names_](http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)

* Monte o comando e execute
```
aws sts assume-role --role-arn arn:aws:iam::644540006937:role/becomeAdmin --role-session-name "Lab02CLI"
```

* Copie e cole a cada valor retornado na variável de ambiente correspondente
```
export AWS_ACCESS_KEY_ID=<valor de Credentials.AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<valor de Credentials.SecretAccessKey>
export AWS_SESSION_TOKEN=<valor de Credentials.SessionToken>
```

* E, finalmente, repita o comando que exige esse acesso mais alto
```
aws iam list-users
```

* Funcionou! (Espero hehe)

