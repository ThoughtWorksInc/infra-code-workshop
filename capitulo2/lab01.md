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
