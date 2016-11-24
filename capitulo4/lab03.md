## Lab03 - Rodando um _playbook_

### Parte 1 - ansible.cfg
* Para evitar repetir em todos os comandos _ansible_ opções que repetem-se, crie o arquivo _ansible.cfg_ e coloque-as nele
```
# ansible.cfg
[defaults]
force_color = 1
host_key_checking = False
private_key_file=/iac/key.pem
remote_user = ec2-user
```

### Parte 2 - Configurando a instância com _ansible-playbook_
* Crie um _playbook_ básico que declara a instalação de Java na instância
```
# pb.yml
---
- hosts: all
  tasks:
  - name: instala java
    yum:
      name: java-1.8.0-openjdk
      state: installed
```

* Crie um arquivo de inventário contendo a instância criada no lab de _CloudFormation_
```
# inventory
<FQDN>
```

* Exemplo
```
# inventory
ec2-54-198-161-200.compute-1.amazonaws.com
```

* Não esqueça de adicionar estes arquivos no container
```
# Dockerfile
...
WORKDIR /iac
COPY entrypoint.sh entrypoint.sh
COPY cfn_example.json cfn_example.json
COPY ansible.cfg ansible.cfg
COPY inventory inventory
COPY pb.yml pb.yml
...
```

* Teste a execução em uma sessão de _shell_ no container
```
make docker.shell
ansible-playbook -i inventory pb.yml
```

* Conecte na instância e verifique a versão do Java
```
ssh -i key.pem ec2-user@<FQDN>
java -version
```

* Como ainda está apontando para o Java 1.7, vamos alterar a _playbook_ para que também remova a versão que não usaremos
```
# pb.yml
---
- hosts: all
  become: yes
  tasks:
  - name: instala java 1.8
    yum:
      name: java-1.8.0-openjdk
      state: installed

  - name: remove java 1.7
    yum:
      name: java-1.7.0-openjdk
      state: absent
```

* Repita o processo, execute o playbook
```
ansible-playbook -i inventory pb.yml
```

* Acesse a instância e check a versão
```
ssh -i key.pem ec2-user@<FQDN>
java -version
```

* Feito!

### Parte 3 - Modificando o inventário de instâncias dinamicamente
Não faz sentido manter uma lista estática de _hostnames_ e/ou endereços IPs de instâncias na nuvem, já que isto vai mudar toda hora. Por isso, o _Ansible_ provê um script de inventário dinâmico.

* Baixe o script de inventário dinâmico na raíz do repositório (execute fora do container)
```
curl \
-O https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py \
-O https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini
```

* Torne o script executável
```
chmod 755 ec2.py
```

* Copie ambos para o container
```
# Dockerfile
...
COPY inventory inventory
COPY pb.yml pb.yml
COPY ec2.py ec2.py
COPY ec2.ini ec2.ini
...
```

* Entre no container e execute o script para testá-lo e verificar o resultado
```
make docker.shell
./ec2.py
```

* Ops! Erro
```
Traceback (most recent call last):
  File "./ec2.py", line 128, in <module>
    import boto
ImportError: No module named boto
```

* Instale a dependência no container
```
# Dockerfile
...
RUN pip install awscli ansible credstash boto
...
```

* Tente executar o inventário dinâmico novamente
```
make docker.shell
./ec2.py
```

* Ah não! Outro erro...
```
ERROR: "Authentication error retrieving ec2 inventory.
 - AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment vars found but may not be correct
 - No Boto config found at any expected location '/etc/boto.cfg, ~/.boto, ~/.aws/credentials'", while: getting EC2
```

* Acontece que o _ansible_ e o _ec2.py_ dependem do **_boto_**, que é muito antigo. O SDK Python atual é o **_boto3_**. Mas o boto (boto2) ainda procura por variáveis de ambiente não mais usadas pela AWS.

* Adicione então as variáveis antigas procuradas pelo _boto_ no script _entrypoint_
```
# entrypoint.sh
export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION:-us-east-1}
export AWS_ACCESS_KEY_ID=${KST[0]}
export AWS_SECRET_ACCESS_KEY=${KST[1]}
export AWS_SESSION_TOKEN=${KST[2]}
# boto (not 3) used by ansible depends on old vars version
export AWS_ACCESS_KEY=${KST[0]}
export AWS_SECRET_KEY=${KST[1]}
export AWS_SECURITY_TOKEN=${KST[2]}
export AWS_DELEGATION_TOKEN=${KST[2]}
```

* Teste o _script_ de inventário dinâmico novamente
```
make docker.shell
./ec2.py
```

* Demora muito né? Isso porque a configuração padrão encontrada no arquivo **_ec2.ini_** faz o _ec2.py_ buscar por instâncias em **TODAS** as regiões da AWS.

* Para agilizar, procure pela opção abaixo e altere conforme indicado
```
# ec2.ini
...
# regions = all
regions = us-east-1 
...
```

* Repare nos grupos dinâmicos criados pelo script e o que eles contêm.