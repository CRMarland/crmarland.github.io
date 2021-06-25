---
layout: post
title:  "Getting Your Head Around AWS CloudFormation"
date:   2021-02-17
image:  images/170221-cover.PNG
---
# The Basics

CloudFormation is an AWS service that allows you to provision resources using code quickly and consistently. By building templates you can model your architecture once and deploy as many times as you wish. Better yet, while this might sound restrictive, you can add parameters to your scripts that allow you/clients/others in your organisation to vary the deployment of your templates according to values you specify.

The first concept to get one’s head around is stacks, these aren’t anything complicated, they’re just files of code held in a repository (CloudFormation). 

![]({{site.baseurl}}/images/170221-stacks.PNG)

When you come to create a stack you can do so in three ways:

1.	Use a ready-made template, this can either be a file saved in S3 (personally, I recommend doing it this way, just remember to use the object URL, the fact it says ‘S3 URL’ is misleading) or you can upload directly to CloudFormation.

2.	Use a sample template – AWS have a handful of pre-written templates that you can adapt and create in the designer.

3.	Create template in Designer – here you can simply write a script from scratch using the inbuilt designer which helps to give you a visual idea of your resources. 

![]({{site.baseurl}}/images/170221-create-stack.PNG)

Whichever you chose is up to you, I can see the benefits of both Template is ready and Create template in Designer, my suggestion is that you play around with them and see which one takes your fancy.

# Writing the Code

Your script can either be in JSON or YAML. Personally, I find YAML easier to write and easier for other people to read and so that’s my preference, but I’ll present both in this blog.
<br>
##### YAML
```yml
AWSTemplateFormatVersion: "2010-09-09"
Description: A template for creating an EC2 t2 instance in CloudFormation
Parameters:
  KeyName:
    Description: Name of an EC2 KeyPair in the account for SSH access
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t1.micro
      - t2.nano
      - t2.micro
      - t2.small
      - t2.medium
      - t2.large
    ConstraintDescription: must be one of the t\d instances.
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: "9"
    MaxLength: "18"
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: restrict the CIDR range to a valid format via length and RegEx restrictions
  EnvType:
    Description: Environment type.
    Default: test
    Type: String
    AllowedValues:
      - prod
      - test
    ConstraintDescription: must specify prod or test.
Rules:
  InstanceRegion:
    Assertions:
      Assert:
        "Fn::Contains":
          - - "training"
          - Ref! KeyName
      AssertDescription: 'KeyPair name must include the word "training" '
Mappings:
  AWSInstanceType2Arch:
    t1.micro:
      Arch: PV64
    t2.nano:
      Arch: HVM64
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64
    t2.large:
      Arch: HVM64
  AWSInstanceType2NATArch:
    t1.micro:
      Arch: NATPV64
    t2.nano:
      Arch: NATHVM64
    t2.micro:
      Arch: NATHVM64
    t2.small:
      Arch: NATHVM64
    t2.medium:
      Arch: NATHVM64
    t2.large:
      Arch: NATHVM64
  AWSRegionArch2AMI:
    eu-west-1:
      PV64: ami-4cdd453f
      HVM64: ami-f9dd458a
    eu-west-2:
      PV64: NOT_SUPPORTED
      HVM64: ami-886369ec
    eu-central-1:
      PV64: ami-6527cf0a
      HVM64: ami-ea26ce85
Conditions:
  CreateProdResources: Fn::Equals
    - !Ref EnvType
    - prod
Resources:
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Condition: CreateProdResources
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: "22"
          ToPort: "22"
          SourceSecurityGroup: sg-12345678
          SourceSecurityGroupOwnerId: "123456789012"
        - IpProtocol: tcp
          FromPort: "80"
          ToPort: "80"
          CidrIp:
            Ref: SSHLocation
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType:
        Ref: InstanceType
      SecurityGroups:
        - Ref: InstanceSecurityGroup
      KeyName:
        Ref: KeyName
      ImageId:
        Fn::FindInMap:
          - AWSRegionArch2AMI
          - Ref: AWS::Region
          - Fn::FindInMap:
              - AWSInstanceType2Arch
              - Ref: InstanceType
              - Arch
Outputs:
  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value:
      Ref: EC2Instance
  AZ:
    Description: Availability Zone of the newly created EC2 instance
    Value:
      Fn::GetAtt:
        - EC2Instance
        - AvailabilityZone
  PublicDNS:
    Description: Public DNSName of the newly created EC2 instance
    Value:
      Fn::GetAtt:
        - EC2Instance
        - PublicDnsName
  PublicIP:
    Description: Public IP address of the newly created EC2 instance
    Value:
      Fn::GetAtt:
        - EC2Instance
        - PublicIp
```
<br>
##### JSON
```json
{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "A template for creating an EC2 t2 instance in CloudFormation",
    "Parameters": {
        "KeyName": {
            "Description": "Name of an EC2 KeyPair in the account for SSH access",
            "Type": "AWS::EC2::KeyPair::KeyName",
            "ConstraintDescription": "must be the name of an existing EC2 KeyPair."
        },
        "InstanceType": {
            "Description": "WebServer EC2 instance type",
            "Type": "String",
            "Default": "t2.micro",
            "AllowedValues": [
                "t1.micro",
                "t2.nano",
                "t2.micro",
                "t2.small",
                "t2.medium",
                "t2.large"
            ],
            "ConstraintDescription": "must be one of the t\\d instances."
        },
        "SSHLocation": {
            "Description": "The IP address range that can be used to SSH to the EC2 instances",
            "Type": "String",
            "MinLength": "9",
            "MaxLength": "18",
            "Default": "0.0.0.0/0",
            "AllowedPattern": "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})",
            "ConstraintDescription": "restrict the CIDR range to a valid format via length and RegEx restrictions"
        },
        "EnvType": {
            "Description": "Environment type.",
            "Default": "test",
            "Type": "String",
            "AllowedValues": [
                "prod",
                "test"
            ],
            "ConstraintDescription": "must specify prod or test."
        }
    },
    "Rules": {
        "InstanceRegion": {
            "Assertions": {
                "Assert": {
                    "Fn::Contains": [
                        [
                            "training"
                        ],
                        "Ref! KeyName"
                    ]
                },
                "AssertDescription": "KeyPair name must include the word \"training\" "
            }
        }
    },
    "Mappings": {
        "AWSInstanceType2Arch": {
            "t1.micro": {
                "Arch": "PV64"
            },
            "t2.nano": {
                "Arch": "HVM64"
            },
            "t2.micro": {
                "Arch": "HVM64"
            },
            "t2.small": {
                "Arch": "HVM64"
            },
            "t2.medium": {
                "Arch": "HVM64"
            },
            "t2.large": {
                "Arch": "HVM64"
            }
        },
        "AWSInstanceType2NATArch": {
            "t1.micro": {
                "Arch": "NATPV64"
            },
            "t2.nano": {
                "Arch": "NATHVM64"
            },
            "t2.micro": {
                "Arch": "NATHVM64"
            },
            "t2.small": {
                "Arch": "NATHVM64"
            },
            "t2.medium": {
                "Arch": "NATHVM64"
            },
            "t2.large": {
                "Arch": "NATHVM64"
            }
        },
        "AWSRegionArch2AMI": {
            "eu-west-1": {
                "PV64": "ami-4cdd453f",
                "HVM64": "ami-f9dd458a"
            },
            "eu-west-2": {
                "PV64": "NOT_SUPPORTED",
                "HVM64": "ami-886369ec"
            },
            "eu-central-1": {
                "PV64": "ami-6527cf0a",
                "HVM64": "ami-ea26ce85"
            }
        }
    },
    "Conditions": {
        "CreateProdResources": "Fn::Equals - !Ref EnvType - prod"
    },
    "Resources": {
        "InstanceSecurityGroup": {
            "Type": "AWS::EC2::SecurityGroup",
            "Condition": "CreateProdResources",
            "Properties": {
                "GroupDescription": "Enable SSH access via port 22",
                "SecurityGroupIngress": [
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "22",
                        "ToPort": "22",
                        "SourceSecurityGroup": "sg-12345678",
                        "SourceSecurityGroupOwnerId": "123456789012"
                    },
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "80",
                        "ToPort": "80",
                        "CidrIp": {
                            "Ref": "SSHLocation"
                        }
                    }
                ]
            }
        },
        "EC2Instance": {
            "Type": "AWS::EC2::Instance",
            "Properties": {
                "InstanceType": {
                    "Ref": "InstanceType"
                },
                "SecurityGroups": [
                    {
                        "Ref": "InstanceSecurityGroup"
                    }
                ],
                "KeyName": {
                    "Ref": "KeyName"
                },
                "ImageId": {
                    "Fn::FindInMap": [
                        "AWSRegionArch2AMI",
                        {
                            "Ref": "AWS::Region"
                        },
                        {
                            "Fn::FindInMap": [
                                "AWSInstanceType2Arch",
                                {
                                    "Ref": "InstanceType"
                                },
                                "Arch"
                            ]
                        }
                    ]
                }
            }
        }
    },
    "Outputs": {
        "InstanceId": {
            "Description": "InstanceId of the newly created EC2 instance",
            "Value": {
                "Ref": "EC2Instance"
            }
        },
        "AZ": {
            "Description": "Availability Zone of the newly created EC2 instance",
            "Value": {
                "Fn::GetAtt": [
                    "EC2Instance",
                    "AvailabilityZone"
                ]
            }
        },
        "PublicDNS": {
            "Description": "Public DNSName of the newly created EC2 instance",
            "Value": {
                "Fn::GetAtt": [
                    "EC2Instance",
                    "PublicDnsName"
                ]
            }
        },
        "PublicIP": {
            "Description": "Public IP address of the newly created EC2 instance",
            "Value": {
                "Fn::GetAtt": [
                    "EC2Instance",
                    "PublicIp"
                ]
            }
        }
    }
}
```
<br>
## Format Version
At the time of writing, the only valid template format version was “2010-09-09” and so there’s no need to worry too much about this section of the code. In theory, this part identifies the capabilities of the template, if another improved format version is released in the future then I imagine you may want to put that value in instead, but, as you can see, it’s been the same format version for over ten years.
<br>
##### YAML
```yml
AWSTemplateFormatVersion: "2010-09-09"
```
<br>
##### JSON
```json
"AWSTemplateFormatVersion": "2010-09-09"
```
<br>
## Description
Here you can write a little bit of documentation in the template itself (which is always a good idea) this is the place to do it. Normally a line or two about the purpose of the template should suffice.
<br>
##### YAML
```yml
Description: A template for creating an EC2 t2 instance in CloudFormation
```
<br>
##### JSON
```json
"Description": "A template for creating an EC2 t2 instance in CloudFormation"
```
<br>
## Parameters

Here is where you can add some customisability into your templates, this is where you can determine what values users will be allowed to select. In terms of formatting, you will specify the name of the parameter and then after the colon you specify the description, type, default value, allowed values, an allowed pattern (this must be a regular expression) and a constraint description. 
<br>
##### YAML
```yml
Parameters:
  KeyName:
    Description: Name of an EC2 KeyPair in the account for SSH access
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t1.micro
      - t2.nano
      - t2.micro
      - t2.small
      - t2.medium
      - t2.large
    ConstraintDescription: must be one of the t\d instances.
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: "9"
    MaxLength: "18"
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: restrict the CIDR range to a valid format via length and RegEx restrictions
  EnvType:
    Description: Environment type.
    Default: test
    Type: String
    AllowedValues:
      - prod
      - test
    ConstraintDescription: must specify prod or test.
```
<br>
##### JSON
```json
"Parameters": {
    "KeyName": {
      "Description": "Name of an EC2 KeyPair in the account for SSH access",
      "Type": "AWS::EC2::KeyPair::KeyName",
      "ConstraintDescription": "must be the name of an existing EC2 KeyPair."
    },
    "InstanceType": {
      "Description": "WebServer EC2 instance type",
      "Type": "String",
      "Default": "t2.micro",
      "AllowedValues": [
        "t1.micro",
        "t2.nano",
        "t2.micro",
        "t2.small",
        "t2.medium",
        "t2.large"
      ],
      "ConstraintDescription": "must be one of the t\\d instances."
    },
    "SSHLocation": {
      "Description": "The IP address range that can be used to SSH to the EC2 instances",
      "Type": "String",
      "MinLength": "9",
      "MaxLength": "18",
      "Default": "0.0.0.0/0",
      "AllowedPattern": "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})",
      "ConstraintDescription": "restrict the CIDR range to a valid format via length and RegEx restrictions"
    },
    "EnvType": {
      "Description": "Environment type.",
      "Default": "test",
      "Type": "String",
      "AllowedValues": ["prod", "test"],
      "ConstraintDescription": "must specify prod or test."
    }
  }
```
<br>
If done right, when deploying a stack, you should get a view like this:

![]({{site.baseurl}}/images/170221-parameters.PNG)
 
In the resources section, you will need to reference your parameters with either:
<br>
##### YAML
```yml
Ref: YourParameterName
```
<br>
##### JSON
```
“Ref”: “YourParameterName”
```
<br>
## Rules

Now you’ve written your parameters, you may want to validate them according to some criteria, this is where you can do that validation. Rules can contain a rule condition which is where you specify under which conditions certain rules apply (this can allow you to create forbidden combinations “if parameter 1 = x, then rule = y”, and must contain an assertion, which is where you specify the rule itself.
In the following I’ve specified that the KeyName must include ‘training’.
<br>
##### YAML
```yml
Rules:
  InstanceRegion:
    Assertions:
      Assert:
        "Fn::Contains":
          - - "training"
          - Ref! KeyName
      AssertDescription: 'KeyPair name must include the word "training" '
```
<br>
##### JSON
```json
"Rules": {
    "InstanceRegion": {
      "Assertions": {
        "Assert": {
          "Fn::Contains": [["training"], "Ref! KeyName"]
        },
        "AssertDescription": "KeyPair name must include the word \"training\" "
      }
    }
  }
```
<br>
## Mappings

Mappings matches a key to a value or a set of values. You can then use the Fn::FindInMap function to retrieve values in a map. Think of this as a section that allows us CASE functionality. In the following, we specify the various components of the AMI according to instance type and AWS region.
<br>
##### YAML
```yml
Mappings:
  AWSInstanceType2Arch:
    t1.micro:
      Arch: PV64
    t2.nano:
      Arch: HVM64
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64
    t2.large:
      Arch: HVM64
  AWSInstanceType2NATArch:
    t1.micro:
      Arch: NATPV64
    t2.nano:
      Arch: NATHVM64
    t2.micro:
      Arch: NATHVM64
    t2.small:
      Arch: NATHVM64
    t2.medium:
      Arch: NATHVM64
    t2.large:
      Arch: NATHVM64
  AWSRegionArch2AMI:
    eu-west-1:
      PV64: ami-4cdd453f
      HVM64: ami-f9dd458a
    eu-west-2:
      PV64: NOT_SUPPORTED
      HVM64: ami-886369ec
    eu-central-1:
      PV64: ami-6527cf0a
      HVM64: ami-ea26ce85
```
<br>
##### JSON
```json
  "Mappings": {
    "AWSInstanceType2Arch": {
      "t1.micro": {
        "Arch": "PV64"
      },
      "t2.nano": {
        "Arch": "HVM64"
      },
      "t2.micro": {
        "Arch": "HVM64"
      },
      "t2.small": {
        "Arch": "HVM64"
      },
      "t2.medium": {
        "Arch": "HVM64"
      },
      "t2.large": {
        "Arch": "HVM64"
      }
    },
    "AWSInstanceType2NATArch": {
      "t1.micro": {
        "Arch": "NATPV64"
      },
      "t2.nano": {
        "Arch": "NATHVM64"
      },
      "t2.micro": {
        "Arch": "NATHVM64"
      },
      "t2.small": {
        "Arch": "NATHVM64"
      },
      "t2.medium": {
        "Arch": "NATHVM64"
      },
      "t2.large": {
        "Arch": "NATHVM64"
      }
    },
    "AWSRegionArch2AMI": {
      "eu-west-1": {
        "PV64": "ami-4cdd453f",
        "HVM64": "ami-f9dd458a"
      },
      "eu-west-2": {
        "PV64": "NOT_SUPPORTED",
        "HVM64": "ami-886369ec"
      },
      "eu-central-1": {
        "PV64": "ami-6527cf0a",
        "HVM64": "ami-ea26ce85"
      }
    }
  }
```
<br>
## Conditions
In the conditions section you can specify the conditions under which certain resources are created. Perhaps in a test environment you want a smaller database and in a prod environment you can a bigger database – this is where you would create this kind of logic. You create the condition in this section, and then reference it by adding a Condition value in your Resources section.
<br>
##### YAML
```yml
Conditions:
  CreateProdResources: Fn::Equals
    - !Ref EnvType
    - prod
```
<br>
##### JSON
```json
 "Conditions": {
    "CreateProdResources": "Fn::Equals - !Ref EnvType - prod"
  },
```
<br>
## Resources

This is where the real action happens. This is where the actual resources you want to be provisioned are defined. For a lot of this, you’ll want to be making reference back to your parameters, mappings and conditions with either a Ref statement or a Fn::FindInMap statement for mappings.
##### YAML
```yml
Resources:
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Condition: CreateProdResources
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: "22"
          ToPort: "22"
          SourceSecurityGroup: sg-12345678
          SourceSecurityGroupOwnerId: "123456789012"
        - IpProtocol: tcp
          FromPort: "80"
          ToPort: "80"
          CidrIp:
            Ref: SSHLocation
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType:
        Ref: InstanceType
      SecurityGroups:
        - Ref: InstanceSecurityGroup
      KeyName:
        Ref: KeyName
      ImageId:
        Fn::FindInMap:
          - AWSRegionArch2AMI
          - Ref: AWS::Region
          - Fn::FindInMap:
              - AWSInstanceType2Arch
              - Ref: InstanceType
              - Arch
```
<br>
##### JSON
```json
  "Resources": {
    "InstanceSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Condition": "CreateProdResources",
      "Properties": {
        "GroupDescription": "Enable SSH access via port 22",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "tcp",
            "FromPort": "22",
            "ToPort": "22",
            "SourceSecurityGroup": "sg-12345678",
            "SourceSecurityGroupOwnerId": "123456789012"
          },
          {
            "IpProtocol": "tcp",
            "FromPort": "80",
            "ToPort": "80",
            "CidrIp": {
              "Ref": "SSHLocation"
            }
          }
        ]
      }
    },
    "EC2Instance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "InstanceType": {
          "Ref": "InstanceType"
        },
        "SecurityGroups": [
          {
            "Ref": "InstanceSecurityGroup"
          }
        ],
        "KeyName": {
          "Ref": "KeyName"
        },
        "ImageId": {
          "Fn::FindInMap": [
            "AWSRegionArch2AMI",
            {
              "Ref": "AWS::Region"
            },
            {
              "Fn::FindInMap": [
                "AWSInstanceType2Arch",
                {
                  "Ref": "InstanceType"
                },
                "Arch"
              ]
            }
          ]
        }
      }
    }
  }
```
<br>

## Outputs

Outputs allows you to export out some metadata about your stacks and the resources you provision. These can then be referenced in other stacks with the Fn::ImportValue function or simply shown on the CloudFormation console under stack properties.
<br>
##### YAML
```yml
Outputs:
  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value:
      Ref: EC2Instance
  AZ:
    Description: Availability Zone of the newly created EC2 instance
    Value:
      Fn::GetAtt:
        - EC2Instance
        - AvailabilityZone
  PublicDNS:
    Description: Public DNSName of the newly created EC2 instance
    Value:
      Fn::GetAtt:
        - EC2Instance
        - PublicDnsName
  PublicIP:
    Description: Public IP address of the newly created EC2 instance
    Value:
      Fn::GetAtt:
        - EC2Instance
        - PublicIp
```
<br>
##### JSON
```json
"Outputs": {
    "InstanceId": {
      "Description": "InstanceId of the newly created EC2 instance",
      "Value": {
        "Ref": "EC2Instance"
      }
    },
    "AZ": {
      "Description": "Availability Zone of the newly created EC2 instance",
      "Value": {
        "Fn::GetAtt": ["EC2Instance", "AvailabilityZone"]
      }
    },
    "PublicDNS": {
      "Description": "Public DNSName of the newly created EC2 instance",
      "Value": {
        "Fn::GetAtt": ["EC2Instance", "PublicDnsName"]
      }
    },
    "PublicIP": {
      "Description": "Public IP address of the newly created EC2 instance",
      "Value": {
        "Fn::GetAtt": ["EC2Instance", "PublicIp"]
      }
    }
  }
```
<br>

# Finishing Off

Hopefully, this will have provided some help getting your head around the basics of CloudFront templates. It took me a couple of days to feel as if I really understood what was going on. AWS documentation can feel a little impenetrable sometimes and takes a little bit of getting used to.

What I realised is key is understanding that the various sections of the template anatomy are interdependent, with the resources section lying right in the middle drawing everything together. Parameters, mappings, rules and conditions add an incredible amount of flexibility to your templates once you know how to both write and reference them.

Once you understand all of this, it’s all about digging deep into that documentation and hoping to see the sun afterwards.

![]({{site.baseurl}}/images/170221-diagram.PNG)