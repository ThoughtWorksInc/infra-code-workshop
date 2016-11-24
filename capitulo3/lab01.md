## Lab01 - Criando uma instância com CloudFormation

### Parte 1 - Criando a _KeyPair_
Ao criar uma instância, você deve indiciar qual **_KeyPair_** deve estar configurada nela para permitir acesso via SSH (no caso de instâncias Linux).

O _CloudFormation_ não possui recurso para automatizar a criação de _KeyPairs_ e você precisará indicar uma no _template_ do _CloudFormation_. Por isso crie uma manualmente acessando o serviço **EC2** na console web e clicando em:

* (Nav bar na esquerda) Key Pairs > Create Key Pair

* Use o nome da sua conta como nome para a _KeyPair_ ex: "devopslabs01"

* Um arquivo será baixado pelo seu navegador

* Anote onde ele foi baixado e altere as permissões deste arquivo para
```
chmod 600 /path/para/arquivo/<chave_ssh_privada.pem>
```

**NOTA:** Caso necessite automatizar a criação de _KeyPairs_, existe a opção de criar a chave localmente e importá-la em sua conta Amazon [via API](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)

### Parte 2 - Primeiro template CloudFormation
* Crie um arquivo na raiz do repositório chamado **cfn_example.json** com o seguinte conteúdo
```
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "single instance example stack.",
  "Resources": {
    "Instance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "KeyName": "<nome da sua key pair>",
        "ImageId": "ami-b73b63a0",
        "InstanceType": "t2.micro",
        "SubnetId": "<sua Subnet ID>",
        "SecurityGroupIds": [
          {
            "Ref": "ExampleSshSg"
          }
        ],
        "Tags": [
          {
            "Key": "Name",
            "Value": "example"
          }
        ]
      }
    },
    "ExampleSshSg": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription": "SSH connections",
        "VpcId": "<sua VPC ID>",
        "SecurityGroupIngress": [
          {
            "CidrIp": "0.0.0.0/0",
            "FromPort": 22,
            "ToPort": 22,
            "IpProtocol": "tcp"
          }
        ]
      }
    }
  },
  "Outputs": {
    "InstanceId": {
      "Description": "Instance ID",
      "Value": {
        "Ref": "Instance"
      }
    },
    "AvailabilityZone": {
      "Description": "Datacenter Location",
      "Value": {
        "Fn::GetAtt": [
          "Instance",
          "AvailabilityZone"
        ]
      }
    },
    "PublicDnsName": {
      "Description": "Internet resolvable name",
      "Value": {
        "Fn::GetAtt": [
          "Instance",
          "PublicDnsName"
        ]
      }
    },
    "PublicIp": {
      "Description": "Internet reachable address",
      "Value": {
        "Fn::GetAtt": [
          "Instance",
          "PublicIp"
        ]
      }
    }
  }
}
```

* Altere a imagem docker incluindo este arquivo
```
# Dockerfile
...
WORKDIR /iac
COPY entrypoint.sh entrypoint.sh
COPY cfn_example.json cfn_example.json
...
```

* Abra um _shell_ no container
```
make docker.shell
```

* E execute a criação da instância com o comando
```
aws cloudformation create-stack --stack-name example-stack --template-body file://.//cfn_example.json
```

* Abra o painel do serviço _CloudFormation_ na console web para acompanhar a criação da _stack_
https://console.aws.amazon.com/cloudformation

* Verifique na aba **_Output_** o IP e FQDN públicos da instância e acesse com o comando
```
ssh -i /path/para/arquivo/<chave_ssh_privada.pem> ec2-user@<FQDN ou IP público>
```
