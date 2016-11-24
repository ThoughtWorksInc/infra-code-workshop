# Workshop de Infraestrutura como Código
Infrastructure as Code (IaC) Workshop

## Pré-requisitos
Para garantir um denominador comum no ambiente de todos(as) os(as) participantes do workshop, vamos rodar todos os comandos, _builds_, etc de dentro de um container. Por isso, o seguinte software é esperado em seu computador:

* Docker Version: 1.12.3 - [download](https://www.docker.com/products/overview#/install_the_platform)
* Docker Image: [python:2](https://hub.docker.com/_/python/)

## Ferramentas usadas
A quantidade de ferramentas disponíveis para automação de _builds_ e infraestrutura é grande. Além disso, muitas delas tem também grande adoção na indústria de TI, ou seja, as escolhas de ferramentas deste workshop é muito mais baseada na preferência de quem escreveu isso aqui do que qualquer outra coisa. Trabalharemos com:

  * Docker como denominador comum pro ambiente de _build_
  * GNU Make para _build_
  * CloudFormation para criação de recursos na AWS
  * Ansible e seu jeito de organização do código, configuração das instâncias e _deploy_ da webapp

## Conteúdo

### Capítulo 1
  * [Primeiro acesso à AWS](capitulo1/)
    * [Lab01 - _IAM - Identity and Access Management_](capitulo1/lab01.md)
    * [Lab02 - Acesso e switch role através da CLI](capitulo1/lab02.md)

### Capítulo 2
  * [Iniciando automação](capitulo2/)
    * [Lab01 - Customizando uma _docker image_](capitulo2/lab01.md)
    * [Lab02 - Manipulando credenciais na automação](capitulo2/lab02.md)

### Capítulo 3
  * [Introdução ao AWS CloudFormation](capitulo3/)
    * [Lab01 - Criando uma instância com CloudFormation](capitulo3/lab01.md)
    * [Lab02 - Passando parâmetros para o CloudFormation](capitulo3/lab02.md)

### Capítulo 4
  * [Introdução ao Ansible](capitulo4/)
    * [Lab01 - Criando uma instância com CloudFormation](capitulo4/lab01.md)
    * [Lab02 - Tratando segredos com Credstash](capitulo4/lab02.md)
    * [Lab03 - abc](capitulo4/lab03.md)

