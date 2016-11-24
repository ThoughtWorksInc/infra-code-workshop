## Lab01 - Preparando o container para rodar Ansible

### Parte 1 - Instalando e testando o Ansible
* O Ansible é facilmente instalado através do _pip_
```
# Dockerfile
...
RUN pip install awscli ansible
...
```

* Copie a chave SSH para a raiz do repositório

* E instrua o docker a copiá-la também para dentro do container
```
# Dockerfile
...
COPY cfn_example.json cfn_example.json
COPY <chave_ssh_privada.pem> <chave_ssh_privada.pem>
...
```

* Abra um _shell_ no container
```
make docker.shell
```

* Usando o FQDN da instância criada no lab anterior, teste a conectividade com o comando abaixo, prestando atenção na **vírgula** necessária na opção **-i**
```
ansible -i <FQDN>, \
        -m setup <FQDN> \
        --user ec2-user \
        --private-key <chave_ssh_privada.pem>
```

Porém, existe um problema grave nesta configuração.

Ao copiar a chave SSH privada para o container, facilitamos o acesso a ela para qualquer outra pessoa com acesso ao cache dos docker hosts onde este container for rodado, como servidores de CI por exemplo.

A chave SSH é um segredo e deve ser tratada como tal, evitando ser persistida.

No próximo lab, apresentamos uma sugestão de como tratar segredos quando automatizando infra na AWS.

