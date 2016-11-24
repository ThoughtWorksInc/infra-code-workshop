## Lab02 - Tratando segredos com Credstash
Os segredos de infraestrutura, como chaves SSH, podem ser guardados no [credstash](https://github.com/fugue/credstash).

Este workshop demonstra rapidamente como configurá-lo e usá-lo. Detalhes sobre seu uso e toda mecânica que o torna uma ferramenta segura pode ser encontrada em

* https://github.com/fugue/credstash

* https://blog.fugue.co/2015-04-21-aws-kms-secrets.html

### Parte 1 - Instalando e configurando o CredStash
* O **credstash** é facilmente instalado através do _pip_
```
# Dockerfile
...
RUN pip install awscli ansible credstash
...
```

* Crie uma chave no AWS KMS para o _credstash_ em sua conta seguindo as instruções encontradas https://github.com/fugue/credstash#setting-up-kms

* Abre uma sessão _shell_ no container
```
make docker.shell
```

* Finalize a configuração do _credstash_
```
credstash setup
```

### Parte 2 - Gravando e recuperando dados no/do CredStash
O _credstash_ é uma _key/value store_

* Para salvar coisas nele, use
```
credstash put minha-chave meu_valor
```

* E para recuperar
```
credstash get minha-chave
```

Simples assim! 😉

### Parte 3 - Salvando a chave SSH no _credstash_
* Para guardar
```
credstash put ssh_private_key "$(cat <chave_ssh_privada.pem>)"
```

* Para consultar
```
credstash get ssh_private_key
```

### Parte 4 - Usando o credstash no container
* Remova a cópia da chave para dentro do container **apagando** a linha abaixo
```
# Dockerfile
...
COPY <chave_ssh_privada.pem> <chave_ssh_privada.pem>
...
```

* Altere o _entrypoint_ script para, em tempo de execução, criar a chave ssh no container
```
# entrypoint.sh
...
export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION:-us-east-1}
export AWS_ACCESS_KEY_ID=${KST[0]}
export AWS_SECRET_ACCESS_KEY=${KST[1]}
export AWS_SESSION_TOKEN=${KST[2]}

# retrive ssh private key from previously configured credstash
credstash get ssh_private_key > /iac/key.pem
chmod 400 /iac/key.pem || echo "ERROR loading private ssh key"

exec $@

```

* Para evitar erros, exclua os arquivos abaixo do radar do git
```
# .gitignore
.env
*.pem
```

