******

<details>
<summary>Exercise 0: Prerequisites </summary>
 <br />

Having
* admin user with credentials
* installed aws CLI via `brew install awscli`
* configured default profile via `aws configure` with `Access Key`, `Secret access key`, `region` = `eu-central-1`, `output` = `json`

</details>

******

<details>
<summary>Exercise 1: Create IAM user </summary>
 <br />

#### Create user group `devops`
```shell
(base) ➜  ~ aws iam create-group --group-name devops
{
    "Group": {
        "Path": "/",
        "GroupName": "devops",
        "GroupId": "AGPAS4WET2TXRHXKOJF5X",
        "Arn": "arn:aws:iam::199054578927:group/devops",
        "CreateDate": "2023-04-16T15:27:25+00:00"
    }
}
```

#### Create user `ireshniov`
```shell
(base) ➜  ~ aws iam create-user --user-name ireshniov
{
    "User": {
        "Path": "/",
        "UserName": "ireshniov",
        "UserId": "AIDAS4WET2TX37NJOYUQZ",
        "Arn": "arn:aws:iam::199054578927:user/ireshniov",
        "CreateDate": "2023-04-16T15:43:29+00:00"
    }
}
```

#### Add user `ireshniov` to group `devops`
```shell
(base) ➜  ~ aws iam add-user-to-group --user-name ireshniov --group-name devops
```

#### Check that user `ireshniov` is in group `devops`
```shell
(base) ➜  ~ aws iam get-group --group-name devops
{
    "Users": [
        {
            "Path": "/",
            "UserName": "ireshniov",
            "UserId": "AIDAS4WET2TX37NJOYUQZ",
            "Arn": "arn:aws:iam::199054578927:user/ireshniov",
            "CreateDate": "2023-04-16T15:43:29+00:00"
        }
    ],
    "Group": {
        "Path": "/",
        "GroupName": "devops",
        "GroupId": "AGPAS4WET2TXRHXKOJF5X",
        "Arn": "arn:aws:iam::199054578927:group/devops",
        "CreateDate": "2023-04-16T15:27:25+00:00"
    }
}
```

#### Get arn of `AmazonEC2FullAccess` policy:
```shell
(base) ➜  ~ export EC2_FULL_ACCESS_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`AmazonEC2FullAccess`].{ARN:Arn}' --output text)
(base) ➜  ~ echo $EC2_FULL_ACCESS_ARN
arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

#### Attach policy (permission) `AmazonEC2FullAccess` to user group `devops`
```shell
(base) ➜  ~ aws iam attach-group-policy \
> --group-name devops \
> --policy-arn $EC2_FULL_ACCESS_ARN
```

#### Check that `AmazonEC2FullAccess` policy attached to `devops` user group
```shell
(base) ➜  ~ aws iam list-attached-group-policies --group-name devops
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonEC2FullAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
        }
    ]
}
```

#### Give the user `ireshniov` password
```shell
(base) ➜  ~ aws iam create-login-profile \
> --user-name ireshniov \
> --password W490902w \
> --password-reset-required
{
    "LoginProfile": {
        "UserName": "ireshniov",
        "CreateDate": "2023-04-16T19:32:56+00:00",
        "PasswordResetRequired": true
    }
}
```

#### Get user by name
```shell
(base) ➜  ~ aws iam get-user --user-name ireshniov --query 'User.{ARN:Arn}' --output text
arn:aws:iam::199054578927:user/ireshniov
199054578927 -> account ID
```

#### Create json file `changePasswordPolicy.json` with policy description:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:GetAccountPasswordPolicy",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:ChangePassword",
      "Resource": "arn:aws:iam::199054578927:user/${aws:username}"
    }
  ]
}
```

#### Create `changePassword` policy
```shell
(base) ➜  ~ aws iam create-policy \
> --policy-name changePassword \
> --policy-document file://changePasswordPolicy.json
{
  "Policy": {
    "PolicyName": "changePassword",
    "PolicyId": "ANPAS4WET2TXXEATI5Z4D",
    "Arn": "arn:aws:iam::199054578927:policy/changePassword",
    "Path": "/",
    "DefaultVersionId": "v1",
    "AttachmentCount": 0,
    "PermissionsBoundaryUsageCount": 0,
    "IsAttachable": true,
    "CreateDate": "2023-04-16T20:46:38+00:00",
    "UpdateDate": "2023-04-16T20:46:38+00:00"
  }
}
```

#### Attach created `changePassword` policy to the user group `devops`
```shell
(base) ➜  ~ export EC2_CHANGE_PASSWORD_POLICY_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`changePassword`].{ARN:Arn}' --output text)
(base) ➜  ~ echo $EC2_CHANGE_PASSWORD_POLICY_ARN
arn:aws:iam::199054578927:policy/changePassword

(base) ➜  ~ aws iam attach-group-policy \
> --group-name devops \
> --policy-arn $EC2_CHANGE_PASSWORD_POLICY_ARN

(base) ➜  ~ aws iam list-attached-group-policies --group-name devops
{
    "AttachedPolicies": [
        {
            "PolicyName": "changePassword",
            "PolicyArn": "arn:aws:iam::199054578927:policy/changePassword"
        },
        {
            "PolicyName": "AmazonEC2FullAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
        }
    ]
}
```

#### Give the user `ireshniov` access keys
```shell
(base) ➜  ~ aws iam create-access-key \
> --user-name ireshniov
{
    "AccessKey": {
        "UserName": "ireshniov",
        "AccessKeyId": "AKIAS4WET2TX2WKE3A4Q",
        "Status": "Active",
        "SecretAccessKey": "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",
        "CreateDate": "2023-04-16T21:14:32+00:00"
    }
}
```
</details>

******

<details>
<summary>Exercise 2: Configure AWS CLI </summary>
 <br />

```shell
(base) ➜  ~ aws configure --profile ireshniov
AWS Access Key ID [None]: AKIAS4WET2TX2WKE3A4Q
AWS Secret Access Key [None]: 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
Default region name [None]: eu-central-1
Default output format [None]: json

(base) ➜  ~ cat ~/.aws/config
[default]
region = eu-central-1
output = json
[profile ireshniov]
region = eu-central-1
output = json
(base) ➜  ~ cat ~/.aws/credentials
[default]
aws_access_key_id = AKIAS4WET2TX42LV3KX3
aws_secret_access_key = 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
[ireshniov]
aws_access_key_id = AKIAS4WET2TX2WKE3A4Q
aws_secret_access_key = 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
```

</details>

******

<details>
<summary>Exercise 3: Create VPC </summary>
 <br />

#### Create VPC with 1 Subnet
```shell
(base) ➜  ~ aws ec2 create-vpc --cidr-block 10.0.0.0/24 --query Vpc.VpcId --output text
vpc-0da22eb0a57546f52

(base) ➜  ~ aws ec2 describe-vpcs --vpc-id vpc-0da22eb0a57546f52
{
    "Vpcs": [
        {
            "CidrBlock": "10.0.0.0/24",
            "DhcpOptionsId": "dopt-05d6b0c1a7eb484c6",
            "State": "available",
            "VpcId": "vpc-0da22eb0a57546f52",
            "OwnerId": "199054578927",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-07d88f45f26262ca1",
                    "CidrBlock": "10.0.0.0/24",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false
        }
    ]
}
```

```shell
(base) ➜  ~ aws ec2 create-subnet --vpc-id vpc-0da22eb0a57546f52 --cidr-block 10.0.0.0/25 --query Subnet.SubnetId --output text
subnet-00eb9ddb466f29d6c

# Enable assigning public IPv4 addresses to instances in this subnet
aws ec2 modify-subnet-attribute --subnet-id subnet-00eb9ddb466f29d6c --map-public-ip-on-launch
````

#### Make our subnet public by attaching it internet gateway
```shell
(base) ➜  ~ aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-01feb4f3a828f3f76

(base) ➜  ~ aws ec2 attach-internet-gateway --vpc-id vpc-0da22eb0a57546f52 --internet-gateway-id igw-01feb4f3a828f3f76

(base) ➜  ~ aws ec2 describe-internet-gateways --internet-gateway-ids igw-01feb4f3a828f3f76
{
    "InternetGateways": [
        {
            "Attachments": [
                {
                    "State": "available",
                    "VpcId": "vpc-0da22eb0a57546f52"
                }
            ],
            "InternetGatewayId": "igw-01feb4f3a828f3f76",
            "OwnerId": "199054578927",
            "Tags": []
        }
    ]
}
```

#### Create route table with routes
```shell
(base) ➜  ~ aws ec2 create-route-table --vpc-id vpc-0da22eb0a57546f52 --query RouteTable.RouteTableId --output text
rtb-02e6aa5cd1ce2f1fb

(base) ➜  ~ aws ec2 create-route --route-table-id rtb-02e6aa5cd1ce2f1fb --destination-cidr-block 0.0.0.0/0 --gateway-id igw-01feb4f3a828f3f76
{
    "Return": true
}


(base) ➜  ~ aws ec2 describe-route-tables --route-table-id rtb-02e6aa5cd1ce2f1fb
{
    "RouteTables": [
        {
            "Associations": [],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-02e6aa5cd1ce2f1fb",
            "Routes": [
                {
                    "DestinationCidrBlock": "10.0.0.0/24",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "GatewayId": "igw-01feb4f3a828f3f76",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-0da22eb0a57546f52",
            "OwnerId": "199054578927"
        }
    ]
}

(base) ➜  ~ aws ec2 associate-route-table --route-table-id rtb-02e6aa5cd1ce2f1fb --subnet-id subnet-00eb9ddb466f29d6c
{
    "AssociationId": "rtbassoc-046e2a62a5c436443",
    "AssociationState": {
        "State": "associated"
    }
}
```

#### Create security group in the VPC to allow access on port 22
* For `jenkins-master`
```shell
(base) ➜  ~ aws ec2 create-security-group --group-name jenkins-master-sg --description "Security group for Jenkins master" --vpc-id vpc-0da22eb0a57546f52
{
    "GroupId": "sg-01cc28a249cb552bd"
}

(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-01cc28a249cb552bd --protocol tcp --port 22 --cidr 158.181.78.72/32
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0ffc42b2a9af3dcd0",
            "GroupId": "sg-01cc28a249cb552bd",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "158.181.78.72/32"
        }
    ]
}

(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-01cc28a249cb552bd --protocol tcp --port 8080 --cidr 158.181.78.72/32
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-08692595c207bb331",
            "GroupId": "sg-01cc28a249cb552bd",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "158.181.78.72/32"
        }
    ]
}

(base) ➜  ~ aws ec2 describe-security-groups --group-ids sg-01cc28a249cb552bd
{
    "SecurityGroups": [
        {
            "Description": "Security group for Jenkins master",
            "GroupName": "jenkins-master-sg",
            "IpPermissions": [
                {
                    "FromPort": 8080,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "158.181.78.72/32"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 8080,
                    "UserIdGroupPairs": []
                },
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "158.181.78.72/32"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": []
                }
            ],
            "OwnerId": "199054578927",
            "GroupId": "sg-01cc28a249cb552bd",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-0da22eb0a57546f52"
        }
    ]
}
```

* For `web-server`
```shell
(base) ➜  ~ aws ec2 create-security-group --group-name web-server-sg --description "Security group for web server" --vpc-id vpc-0da22eb0a57546f52
{
    "GroupId": "sg-0ff109743d18491fd"
}

(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-0ff109743d18491fd --protocol tcp --port 22 --cidr 158.181.78.72/32
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-039a1eeba644c0054",
            "GroupId": "sg-0ff109743d18491fd",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "158.181.78.72/32"
        }
    ]
}

(base) ➜  ~ aws ec2 describe-security-groups --group-ids sg-0ff109743d18491fd
{
    "SecurityGroups": [
        {
            "Description": "Security group for web server",
            "GroupName": "web-server-sg",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "158.181.78.72/32"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": []
                }
            ],
            "OwnerId": "199054578927",
            "GroupId": "sg-0ff109743d18491fd",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-0da22eb0a57546f52"
        }
    ]
}
```
</details>

******

<details>
<summary>Exercise 4: Create EC2 Instance </summary>
 <br />

#### Create key pairs
* For `jenkins-master`
```shell
(base) ➜  ~ aws ec2 create-key-pair --key-name JenkinsMasterKeyPair --query "KeyMaterial" --output text > jenkins-master-key-pair.pem
(base) ➜  ~ chmod 400 jenkins-master-key-pair.pem
```
* For `web-server`
```shell
(base) ➜  ~ aws ec2 create-key-pair --key-name WebServerKeyPair --query "KeyMaterial" --output text > web-server-key-pair.pem
(base) ➜  ~ chmod 400 web-server-key-pair.pem
```

#### Create EC2 instances into our subnet
* For `jenkins-master`
```shell
(base) ➜  ~ aws ec2 run-instances --image-id ami-0fa03365cde71e0ab --count 1 --instance-type t2.micro --key-name JenkinsMasterKeyPair --security-group-ids sg-01cc28a249cb552bd --subnet-id subnet-00eb9ddb466f29d6c
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0fa03365cde71e0ab",
            "InstanceId": "i-05e0d4cae8e87e68f",
            "InstanceType": "t2.micro",
            "KeyName": "JenkinsMasterKeyPair",
            "LaunchTime": "2023-04-19T14:16:53+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "eu-central-1b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-79.eu-central-1.compute.internal",
            "PrivateIpAddress": "10.0.0.79",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-00eb9ddb466f29d6c",
            "VpcId": "vpc-0da22eb0a57546f52",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "b25e89b5-1fd2-487b-abd6-d3008553732a",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-04-19T14:16:53+00:00",
                        "AttachmentId": "eni-attach-09eeb4bc83ab0ac0b",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "jenkins-master-sg",
                            "GroupId": "sg-01cc28a249cb552bd"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:4e:9a:7b:6a:32",
                    "NetworkInterfaceId": "eni-00399de92edb6add8",
                    "OwnerId": "199054578927",
                    "PrivateIpAddress": "10.0.0.79",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateIpAddress": "10.0.0.79"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-00eb9ddb466f29d6c",
                    "VpcId": "vpc-0da22eb0a57546f52",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "jenkins-master-sg",
                    "GroupId": "sg-01cc28a249cb552bd"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "required",
                "HttpPutResponseHopLimit": 2,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "BootMode": "uefi-preferred",
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios"
        }
    ],
    "OwnerId": "199054578927",
    "ReservationId": "r-0c80e1dd50d6ac2ae"
}
```
* For `web-server`
```shell
(base) ➜  ~ aws ec2 run-instances --image-id ami-0fa03365cde71e0ab --count 1 --instance-type t2.micro --key-name WebServerKeyPair --security-group-ids sg-0ff109743d18491fd --subnet-id subnet-00eb9ddb466f29d6c
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0fa03365cde71e0ab",
            "InstanceId": "i-0a6912bca9fc9125b",
            "InstanceType": "t2.micro",
            "KeyName": "WebServerKeyPair",
            "LaunchTime": "2023-04-19T14:06:05+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "eu-central-1b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-9.eu-central-1.compute.internal",
            "PrivateIpAddress": "10.0.0.9",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-00eb9ddb466f29d6c",
            "VpcId": "vpc-0da22eb0a57546f52",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "9d92cdcc-fbb8-4943-a0b3-6cd56682f96e",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-04-19T14:06:05+00:00",
                        "AttachmentId": "eni-attach-0f05c99fb983b4f28",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "web-server-sg",
                            "GroupId": "sg-0ff109743d18491fd"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:ad:56:57:95:f2",
                    "NetworkInterfaceId": "eni-06ee3acd495d5c672",
                    "OwnerId": "199054578927",
                    "PrivateIpAddress": "10.0.0.9",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateIpAddress": "10.0.0.9"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-00eb9ddb466f29d6c",
                    "VpcId": "vpc-0da22eb0a57546f52",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "web-server-sg",
                    "GroupId": "sg-0ff109743d18491fd"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "required",
                "HttpPutResponseHopLimit": 2,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "BootMode": "uefi-preferred",
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios"
        }
    ],
    "OwnerId": "199054578927",
    "ReservationId": "r-0f8d8ea97c58537d3"
}
```
</details>

******

<details>
<summary>Exercise 5: SSH into the server and install Docker on it </summary>
 <br />

#### Ssh into web server
```shell
(base) ➜  ~ ssh -i "~/web-server-key-pair.pem" ec2-user@3.65.39.143
```

#### Install Docker and Docker compose on Amazon Linux2
```shell
[ec2-user@ip-10-0-0-9 ~]$ sudo yum update -y
[ec2-user@ip-10-0-0-9 ~]$ sudo yum -y install docker
[ec2-user@ip-10-0-0-9 ~]$ sudo systemctl enable docker
[ec2-user@ip-10-0-0-9 ~]$ sudo systemctl start docker
[ec2-user@ip-10-0-0-9 ~]$ sudo systemctl status docker

[ec2-user@ip-10-0-0-9 ~]$ sudo usermod -a -G docker ec2-user
[ec2-user@ip-10-0-0-9 ~]$ sudo chmod 666 /var/run/docker.sock
[ec2-user@ip-10-0-0-9 ~]$ docker version

[ec2-user@ip-10-0-0-9 ~]$ sudo curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

[ec2-user@ip-10-0-0-9 ~]$ sudo chmod +x /usr/local/bin/docker-compose
[ec2-user@ip-10-0-0-9 ~]$ docker-compose --version
Docker Compose version v2.17.2
```
</details>

******

<details>
<summary>Exercise 6: Add docker-compose for deployment </summary>
 <br />

```yaml
version: '3.8'
services:
  nodejs-app:
    image: ${IMAGE_NAME}
    ports:
     - "3000:3000"
```

</details>

******

<details>
<summary>Exercise 7: Add "deploy to EC2" step to your pipeline </summary>
 <br />

```shell
# Received disconnect from 18.195.51.39 port 22:2: Too many authentication failures
# Disconnected from 18.195.51.39 port 22
# https://rtfm.co.ua/en/ssh-the-too-many-authentication-failures-error-and-its-solution/
# https://linux.die.net/man/5/ssh_config
# Add `-o IdentitiesOnly=yes`
(base) ➜  ~ ssh -o IdentitiesOnly=yes -i "jenkins-master-key-pair.pem" ec2-user@18.195.51.39
```

#### Install Jenkins on `jenkins-master` ([See here](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/))
```shell
[ec2-user@ip-10-0-0-79 ~]$ sudo yum update –y
[ec2-user@ip-10-0-0-79 ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
[ec2-user@ip-10-0-0-79 ~]$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
[ec2-user@ip-10-0-0-79 ~]$ sudo yum upgrade
[ec2-user@ip-10-0-0-79 ~]$ sudo dnf install java-11-amazon-corretto -y
[ec2-user@ip-10-0-0-79 ~]$ sudo yum install jenkins -y
[ec2-user@ip-10-0-0-79 ~]$ sudo systemctl enable jenkins
[ec2-user@ip-10-0-0-79 ~]$ sudo systemctl start jenkins

[ec2-user@ip-10-0-0-79 ~]$ netstat -tlpn | grep :8080
[ec2-user@ip-10-0-0-79 ~]$ ps aux | grep jenkins

[ec2-user@ip-10-0-0-79 ~]$ sudo yum install git
[ec2-user@ip-10-0-0-79 ~]$ git version
git version 2.39.2
```

#### Initial configure Jenkins
```shell
[ec2-user@ip-10-0-0-79 ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
1bb70e1f4b584748b501a6cd5dcac042
```

#### Create and configure `jenkins-slave-1` EC2 instance ([See here](https://www.youtube.com/watch?v=ymNHxUZ2EOw))
* Create security group in the VPC to allow access on port 22 for `jenkins-slave`
```shell
(base) ➜  ~ aws ec2 create-security-group --group-name jenkins-slave-sg --description "Security group for Jenkins slave" --vpc-id vpc-0da22eb0a57546f52
{
    "GroupId": "sg-0d9307c3d060c4752"
}

# allow SSH connection only from Jenkins-master
(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-0d9307c3d060c4752 --protocol tcp --port 22 --source-group sg-01cc28a249cb552bd
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0ce189668e4280734",
            "GroupId": "sg-0d9307c3d060c4752",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "ReferencedGroupInfo": {
                "GroupId": "sg-01cc28a249cb552bd",
                "UserId": "199054578927"
            }
        }
    ]
}

(base) ➜  ~ aws ec2 describe-security-groups --group-ids sg-0d9307c3d060c4752
{
    "SecurityGroups": [
        {
            "Description": "Security group for Jenkins slave",
            "GroupName": "jenkins-slave-sg",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 22,
                    "UserIdGroupPairs": [
                        {
                            "GroupId": "sg-01cc28a249cb552bd",
                            "UserId": "199054578927"
                        }
                    ]
                }
            ],
            "OwnerId": "199054578927",
            "GroupId": "sg-0d9307c3d060c4752",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-0da22eb0a57546f52"
        }
    ]
}
```
* Create key pairs for `jenkins-slave`
```shell
(base) ➜  ~ aws ec2 create-key-pair --key-name JenkinsSlave1KeyPair --query "KeyMaterial" --output text > jenkins-slave-1-key-pair.pem
(base) ➜  ~ chmod 400 jenkins-slave-1-key-pair.pem
```
* Create EC2 instance into our subnet for `jenkins-slave-1`
```shell
(base) ➜  ~ aws ec2 run-instances --image-id ami-0fa03365cde71e0ab --count 1 --instance-type t2.micro --key-name JenkinsSlave1KeyPair --security-group-ids sg-0d9307c3d060c4752 --subnet-id subnet-00eb9ddb466f29d6c
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0fa03365cde71e0ab",
            "InstanceId": "i-03b3a2693c2a56a89",
            "InstanceType": "t2.micro",
            "KeyName": "JenkinsSlave1KeyPair",
            "LaunchTime": "2023-04-19T14:12:26+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "eu-central-1b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-6.eu-central-1.compute.internal",
            "PrivateIpAddress": "10.0.0.6",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-00eb9ddb466f29d6c",
            "VpcId": "vpc-0da22eb0a57546f52",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "9d627a17-cf62-49c8-9f5a-256fb359607d",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-04-19T14:12:26+00:00",
                        "AttachmentId": "eni-attach-00c1ffd09f0c33f3a",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "jenkins-slave-sg",
                            "GroupId": "sg-0d9307c3d060c4752"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:53:ee:9d:e9:3a",
                    "NetworkInterfaceId": "eni-0867013465d3e8cc4",
                    "OwnerId": "199054578927",
                    "PrivateIpAddress": "10.0.0.6",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateIpAddress": "10.0.0.6"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-00eb9ddb466f29d6c",
                    "VpcId": "vpc-0da22eb0a57546f52",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "jenkins-slave-sg",
                    "GroupId": "sg-0d9307c3d060c4752"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "required",
                "HttpPutResponseHopLimit": 2,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "BootMode": "uefi-preferred",
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios"
        }
    ],
    "OwnerId": "199054578927",
    "ReservationId": "r-0c95878186db0abae"
}
```
* Login to `jenkins-master`
```shell
(base) ➜  ~ ssh -o IdentitiesOnly=yes -i "jenkins-master-key-pair.pem" ec2-user@18.195.51.39
```
* Copy `jenkins-slave-1-key-pair.pem` from local machine
```shell
(base) ➜  ~ cat jenkins-slave-1-key-pair.pem
```
* and create `jenkins-slave-1-key-pair.pem` on `jenkins-master`
```shell
[ec2-user@ip-10-0-0-79 ~]$ vi jenkins-slave-1-key-pair.pem
[ec2-user@ip-10-0-0-79 ~]$ chmod 400 jenkins-slave-1-key-pair.pem
```
* Login to `jenkins-slave-1` from `jenkins-master` via SSH
```shell
# used Private IPv4 address
[ec2-user@ip-10-0-0-79 ~]$ ssh -i "jenkins-slave-1-key-pair.pem" ec2-user@10.0.0.6
```
* Install java, docker, git, nodejs, npm on `jenkins-slave-1`
```shell
[ec2-user@ip-10-0-0-6 ~]$ sudo yum update –y
[ec2-user@ip-10-0-0-6 ~]$ sudo dnf install java-11-amazon-corretto -y

[ec2-user@ip-10-0-0-9 ~]$ sudo yum -y install docker
[ec2-user@ip-10-0-0-9 ~]$ sudo systemctl enable docker
[ec2-user@ip-10-0-0-9 ~]$ sudo systemctl start docker
[ec2-user@ip-10-0-0-9 ~]$ sudo systemctl status docker

[ec2-user@ip-10-0-0-9 ~]$ sudo usermod -a -G docker ec2-user
[ec2-user@ip-10-0-0-9 ~]$ sudo chmod 666 /var/run/docker.sock
[ec2-user@ip-10-0-0-9 ~]$ docker version

[ec2-user@ip-10-0-0-6 ~]$ sudo yum install git

[ec2-user@ip-10-0-0-6 ~]$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
[ec2-user@ip-10-0-0-6 ~]$ nvm install 18.14.2
[ec2-user@ip-10-0-0-6 ~]$ node --version
v18.14.2
```

#### Set up an agent (`jenkins-slave-1`)
```shell
Login as admin on jenkins-admin-panel
Set free temporary space threshold down to 0.4GB (because of warning)
Dashboard -> Nodes -> Create Node:
  "Node name": jenkins-slave-1
  "Remote root directory": /home/ec2-user
  "Usage": Use this node as much as possible
  "Launch method": Launch via SSH:
    Host: (Private IPv4 Address of jenkins-slave-1 EC2 instance) 10.0.0.6
    Add credentials
      Global
      Kind: SSH Username with private key
      Scope Global
      ID: jenkins-slave-1
      Description: jenkins-slave-1
      Username: ec2-user
      Private key: Enter directly (from local machine: "cat jenkins-slave-1-key-pair.pem")
      Host key verification strategy: Manually trusted key verification strategy
  "Availability": Keep this agent as much as possible
```
* Check `jenkins-slave-1` node logs and ensure that agent connected successfully

* Configure `web-server` security group:
   add inbound rule: tcp 22 source: internal IP of jenkins-slave (because we ssh `web-server` from `jenkins-slave` in deploy)
```shell
# allow SSH connection only from Jenkins-slave
(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-0ff109743d18491fd --protocol tcp --port 22 --source-group sg-0d9307c3d060c4752
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-07f746eb21269f1e7",
            "GroupId": "sg-0ff109743d18491fd",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "ReferencedGroupInfo": {
                "GroupId": "sg-0d9307c3d060c4752",
                "UserId": "199054578927"
            }
        }
    ]
}
```

#### Configure Jenkins (credentials, plugins, Envs, Shared library)
```text
   Credentials:
      ec2-web-server
      gitlab-credentials
      docker-credentials
   Plugins:
      SSH Agent
   Environment variables: (Manage Jenkins -> Configure System -> Global properties option)
      // private address
      EC2_INSTANCE = 'ec2-user@10.0.0.9'
      GIT_REPO = 'gitlab.com/ireshniov/bootcamp-node-project.git'
      DOCKER_REPO = 'ireshniov1/demo-app'
   Shared library (Manage Jenkins -> Configure System -> Global Pipeline Libraries ):
      library identifier: 'bootcamp-jenkins-shared-library@main', retriever: modernSCM([
         $class: 'GitSCMSource',
         remote: 'https://gitlab.com/ireshniov/bootcamp-jenkins-shared-library.git',
         credentialsId: 'gitlab-credentials'
      ])
   Git plugin (Manage Jenkins -> Configure System -> Git plugin)
      user.name -> jenkins
      user.email -> example@gmail.com
```

#### Add ec2-web-server credentials
```text
   Global
      Kind: SSH Username with private key
      Scope Global
      ID: ec2-web-server
      Description: ec2-web-server
      Username: ec2-user
      Private key: Enter directly (from local machine: "cat ~/web-server-key-pair.pem")
```

#### Improve Jenkinsfile (add deploy logic)

* Jenkinsfile
```text
#!/usr/bin/env groovy

@Library('bootcamp-jenkins-shared-library')_

def gv

pipeline {
    agent {
        label 'jenkins-slave-1'
    }

    stages {
        stage('Init') {
            steps {
                script {
                    gv = load "cicd/cicd-script.groovy"
                }
            }
        }

        stage('Increment version') {
            steps {
                script {
                    def version = incrementVersion()
                    env.IMAGE_NAME = "${env.DOCKER_REPO}:${version}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Run tests') {
            steps {
               script {
                    runTests()
               }
            }
        }

        stage('Build and push docker image') {
            steps {
                script {
                    dockerBuildImage "${env.IMAGE_NAME}"
                    dockerLogin()
                    dockerPush "${env.IMAGE_NAME}"
                }
            }
        }

        stage('Deploy app') {
            steps {
                script {
                    gv.deployApp "${env.EC2_INSTANCE}", "${env.IMAGE_NAME}"
                }
            }
        }

        stage('Commit version update') {
            steps {
                script {
                    commitVersionUpdate "${env.GIT_REPO}", "${env.GIT_BRANCH}"
                }
            }
        }
    }
}
```
* cicd/cicd-script.groovy
```text
def deployApp(String ec2Instance, String imageName) {
    echo "Deploying docker image to the EC2..."
    sshagent(credentials: ['ec2-web-server']) {
        def command = "bash ./server-commands.sh ${imageName}"

        // private address of docker-server
        sh "scp ./cicd/server-commands.sh ${ec2Instance}:/home/ec2-user"
        sh "scp ./docker-compose.yaml ${ec2Instance}:/home/ec2-user"
        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${command}"
    }
}

return this
```
* server-commands.sh
```shell
#!/usr/bin/env bash

export IMAGE_NAME=$1

docker-compose -f docker-compose.yaml up -d
echo "Success"
```

#### Create and configure multibranch pipeline

#### Build
</details>

******

<details>
<summary>Exercise 8: Configure access from browser (EC2 Security Group) </summary>
 <br />

```shell
(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-0ff109743d18491fd --protocol tcp --port 3000 --cidr 0.0
.0.0/0

{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0f3cfe65a87e1f79c",
            "GroupId": "sg-0ff109743d18491fd",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 3000,
            "ToPort": 3000,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```
* Access web-server at http://3.65.39.143:3000 via browser.

</details>

******

<details>
<summary>Exercise 9: Configure automatic triggering of multi-branch pipeline </summary>
 <br />

#### Increment version and Build and push docker image and Deploy app and Commit version update only the `main` branch
```text
#!/usr/bin/env groovy

@Library('bootcamp-jenkins-shared-library')_

def gv

pipeline {
    agent {
        label 'jenkins-slave-1'
    }

    stages {
        stage('Init') {
            steps {
                script {
                    gv = load "cicd/cicd-script.groovy"
                }
            }
        }

        stage('Increment version') {
            when {
                expression {
                    return env.GIT_BRANCH == "main"
                }
            }

            steps {
                script {
                    def version = incrementVersion()
                    env.IMAGE_NAME = "${env.DOCKER_REPO}:${version}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Run tests') {
            steps {
               script {
                    runTests()
               }
            }
        }

        stage('Build and push docker image') {
            when {
                expression {
                    return env.GIT_BRANCH == "main"
                }
            }

            steps {
                script {
                    dockerBuildImage "${env.IMAGE_NAME}"
                    dockerLogin()
                    dockerPush "${env.IMAGE_NAME}"
                }
            }
        }

        stage('Deploy app') {
            when {
                expression {
                    return env.GIT_BRANCH == "main"
                }
            }

            steps {
                script {
                    gv.deployApp "${env.EC2_INSTANCE}", "${env.IMAGE_NAME}"
                }
            }
        }

        stage('Commit version update') {
            when {
                expression {
                    return env.GIT_BRANCH == "main"
                }
            }

            steps {
                script {
                    commitVersionUpdate "${env.GIT_REPO}", "${env.GIT_BRANCH}"
                }
            }
        }
    }
}
```
* Configure automatic triggering
#### Triggers for freestyle or pipeline jobs
```text
   Manage Plugins:
      GitLab (plugin to build triggers)
   Configure Jenkins -> Configure System -> Gitlab
      Gitlab connection: gitlab-connection
      Gitlab host URL: https://gitlab.com/
      Credentials:
         Api token to access gitlab:
            Kind: GitLab API Token (Gitlab -> Preferencies -> Access tokens)
               Token name: jenkins
               Scopes:
                  api
         API token: ********
         ID: gitlab-token
   Check configuration in pipeline job (or freestyle job). Added GitLab Connection. In Build Triggers option to build when a change is pushed to GitLab
   Gitlab -> specific repository -> Settings -> Integrations -> Jenkins
      Trigger:
         Push
         Merge request
      Jenkins URL: http://18.195.51.39:8080/ (localhost not works)
      Project name: (pipeline job name)
      Username: (Jenkins admin user)
      Password: (Jenkins admin password)
```

#### Webhooks for multibranch pipelines
* Add security group for jenkins-master to allow 8080 port for gitlab webhooks at IP ranges 34.74.90.64/28 and 34.74.226.0/24. [From here](https://microfluidics.utoronto.ca/gitlab/help/user/gitlab_com/index.md#ip-range)
```shell
(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-0d9307c3d060c4752 --protocol tcp --port 8080 --cidr 34.74.90.64/28
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-040102dacd14fce42",
            "GroupId": "sg-0d9307c3d060c4752",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "34.74.90.64/28"
        }
    ]
}

(base) ➜  ~ aws ec2 authorize-security-group-ingress --group-id sg-0d9307c3d060c4752 --protocol tcp --port 8080 --cidr 34.74.226.0/24
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00b011ca49750c7b6",
            "GroupId": "sg-0d9307c3d060c4752",
            "GroupOwnerId": "199054578927",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 8080,
            "ToPort": 8080,
            "CidrIpv4": "34.74.226.0/24"
        }
    ]
}
```
```text
   Manage Plugins:
      Multibranch Scan Webhook Trigger
   bootcamp-node-project-pipeline (Multibranch Pipeline) -> Configure -> Scan Multibranch Pipeline Triggers -> Scan by webhook
      Trigger token: `gitlab-token` (any readable string, just a name)
         How to use token?
            Jenkins expects notifications from this url: `JENKINS_URL/multibranch-webhook-trigger/invoke?token=[Trigger token]` If token matches -> multibranch scan will be triggered
            Gitlab -> bootcamp-node-project (specific repository) -> Settings -> Webhooks
               URL: http://18.195.51.39:8080/multibranch-webhook-trigger/invoke?token=gitlab-token
               Trigger:
                  Push events
```

</details>

******
