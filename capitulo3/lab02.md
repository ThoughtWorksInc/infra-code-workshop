## Lab02 - Passando parâmetros para o CloudFormation

### Parte 1 - Alterando o template
* Crie um novo arquivo na raiz do repositório chamado `cfn_example_with_param.json` contendo
```
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "single instance example stack.",
  "Parameters": {
    "KeyPairName": {
      "Description": "Chave SSH previamente criada com a qual a instância será configura",
      "Type": "AWS::EC2::KeyPair::KeyName"
    },
    "Vpc": {
      "Description": "VPC onde a instância será colocada",
      "Type": "AWS::EC2::VPC::Id"
    },
    "Subnet": {
      "Description": "Subnet onde a instância será colocada",
      "Type": "AWS::EC2::Subnet::Id"
    }
  },
  "Resources": {
    "Instance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "KeyName": {
          "Ref": "KeyPairName"
        },
        "ImageId": "ami-b73b63a0",
        "InstanceType": "t2.micro",
        "SubnetId": {
          "Ref": "Subnet"
        },
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
        "VpcId": {
          "Ref": "Vpc"
        },
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

* Abra o painel do serviço _CloudFormation_ na console web
https://console.aws.amazon.com/cloudformation

* Clique 

  Create Stack > Choose a template > Upload a template to Amazon S3 > Choose File

* Escolha o arquivo criado acima e clique **_Next_**

* Compare a lista de parâmetros com as seção _Parameters_ adicionada ao _template CloudFormation_

* Preencha os campos de acordo com seu ambiente e clique **_Next_**

* Na tela _Options_ deixe tudo como está e clique **_Next_** e em seguida **_Create_**

* Acompanhe o progresso da criação da _stack_

