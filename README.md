#### psautoawscloudform
#####using parameters:
Ex:
```
{
  "AWSTemplateFormatVersion":"2010-09-09",
  "Description":"Template",
  "Parameters":{
    "VPC":"{
      "Description":"VPC",
      "Type":"AWS::EC2::VPC::Id"
    },
    "Subnet":{
      "Description":"Subnet",
      "Type":"AWS::EC2::Subnet::Id"
    },
    "KeyPair":{
      "Description":"KeyPair",
      "Type":"AWS::EC2::KeyPair::KeyName"
    },
    "InstanceType":{
      "Description":"InstanceType",
      "Type":"String",
      "AllowedValues":["t2.nano","t2.micro"]
    }
  },
  "Mappings":{
    "RegionMap":{
      "us-east-1":{"AMI":"ami-123453"},
      "us-east-2":{"AMI":"ami-234353"}
    }
  },
  "Resources":{
    "EC2Instance":{
      "Type":"AWS::EC2::Instance",
      "Properties":{
        "IamgeId":{"Fn::FindInMap:["RegionMap",{"Ref":"AWS::Region"},"AMI"]},
        "InstanceType":{"Ref":"InstanceType"},
        "NetworkInterfaces":[{
          "AssociatePublicIpAddress":"true",
          "DeviceIndex":"0",
          "GroupSet":[{"Ref":"SecurityGroup"}],
          "SubnetId":{"Ref":"Subnet"},
          "UserData":...
        }],
        "Tags":[{
          "key":"Name",
          "Value":"ssh-host"
        }],
        "KeyName":{"Ref":"KeyPair"}
      }
    },
    "ElasticIP":{
      "Type":"AWS::EC2::EIP",
      "Properties":{
        "InstanceId":{"Ref":"EC2Instance"},
        "Domain":"vpc"
      }
    },
    "SecurityGroup":{
      "Type":"AWS::EC2::SecurityGroup",
      "Properties":{
        "GroupDescription":"ssh",
        "VpcId":{"Ref":"VPC"},
        "SecurityGroupIngress":[{
          "CidrIp":"0.0.0.0/0",
          "FromPort":22,
          "IpProtocal":"tcp",
          "ToPort":22
        }]
      }
    }
  },
  "Outputs":{
    "SSHBastionHostPublicIp":{
      "Description":"dest",
      "Value":{"Ref":"ElasticIP"}
    },
    "SSHBastionHostPrivateIp":{
    }
  }
```
using command:
```
aws cloudformation create-stack --stack-name name --template-body file://temp.json --parameters ParameterKey=VPC,ParameterValue=vpc-12345 \
ParameterKey=Subnet,ParameterValue=subnet-12345 \
ParameterKey=KeyPair,ParameterValue=mykey \
ParameterKey=InstanceType,ParameterValue=t2.micro \
```
The "Outputs" will show using
```
aws cloudformation --describe-stacks
```
#####Building more powerful.
######mappings
######Pseudo param
AWS::AccountId  
AWS::NotificationARNs  
AWS::NoValue   //removes attr  
AWS::Region  
AWS::StackId  
AWS::StackName
######user data
some usage:
```
{"Fn::Join":[";",["a","b","c"]]}
{"Fn::Base64":"string"}
```
######useit
```
"UserData":{"Fn":"Base64":{"Fn::Join":["\n",[
  "#!/bin/bash -ex",
  "yum install -y httpd",
  "cd /var/www/html",
  "echo '<html>test</html>' > index.html",
  "service httpd start"
]]}}
```
