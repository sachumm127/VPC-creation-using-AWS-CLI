# Creation of instances using AWS-CLI

Today while drinking the tea, I had a thought to create three servers using AWS-CLI. The two servers will be placed on a public network and one in private network. We will host a WordPress site for testing too. Usually, we use terraform for coding. Here I am going using the AWS cli for the creation.

In this part we use the AWS Command Line Interface (AWS CLI) to create a custom AWS Virtual Private Cloud (VPC).
### What are the Benefits of AWS CLI?
 - Easy Installation
 - Time-saving
 - Supports all Amazon Web Services
 - Process Automation

## Process

The commands that we are going include are the 
- creation of a custom IPv4 VPC 
- Both public and  private subnets 
- NAT gateway
- Route table 
- keypair and three instances(one in private and two in public)


#### Install AWS-CLI on your local macine
You may use the official doc of aws to install the awscli on your local machine. I know its not a big task for you.
#### Configure aws-cli using the below command
```sh
# aws configure
AWS Access Key ID [None]: xxxxxxxxxxx
AWS Secret Access Key [None]: xxxxxxxxxxxxxxxxxxxxx
Default region name [None]: ap-south-1
Default output format [None]: json
```

### Create a VPC

Create a VPC with a 172.16.0.0/16 CIDR block using the below command, and it results the ID of the new VPC.

```sh
# aws ec2 create-vpc --cidr-block 172.16.0.0/16 --query Vpc.VpcId --output text
vpc-09a2cd5b88fff813c
```
we can add a suitable nametag for the VPC using the below command. 

```sh
# aws ec2 create-tags --resources vpc-09a2cd5b88fff813c --tags Key=Name,Value=My-VPC
````
###  Create subnets and add name tags to each subnet

  Create a subnet in your VPC with a 172.16.0.0/18 CIDR block.  
 ```sh
 # aws ec2 create-subnet --vpc-id vpc-09a2cd5b88fff813c --cidr-block 172.16.0.0/18 --availability-zone ap-south-1a
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "aps1-az1",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:ap-south-1:681099220869:subnet/subnet-0892ce7f63c928c17",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-09a2cd5b88fff813c",
        "State": "available",
        "AvailabilityZone": "ap-south-1a",
        "SubnetId": "subnet-0892ce7f63c928c17",
        "OwnerId": "681099220869",
        "CidrBlock": "172.16.0.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
``````
Create tags for subnet 1

````sh
# aws ec2 create-tags --resources subnet-0892ce7f63c928c17 --tags Key=Name,Value=Subnet1
````

  Create the second subnet in your VPC with a 172.16.64.0/18 CIDR block.
 ```sh
 #  aws ec2 create-subnet --vpc-id vpc-09a2cd5b88fff813c --cidr-block 172.16.64.0/18 --availability-zone ap-south-1b
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:ap-south-1:681099220869:subnet/subnet-04a744aaae545aed7",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-09a2cd5b88fff813c",
        "State": "available",
        "AvailabilityZone": "ap-south-1b",
        "SubnetId": "subnet-04a744aaae545aed7",
        "OwnerId": "681099220869",
        "CidrBlock": "172.16.64.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
 ````
 create a tag for the subnet
 ```sh
 # aws ec2 create-tags --resources subnet-04a744aaae545aed7 --tags Key=Name,Value=Subnet2
 ````
   Create the third subnet in your VPC with a 172.16.128.0/18 CIDR block.  We are using this as a private subnet. 
  ```sh
  # aws ec2 create-subnet --vpc-id vpc-09a2cd5b88fff813c --cidr-block 172.16.128.0/18 --availability-zone ap-south-1b
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:ap-south-1:681099220869:subnet/subnet-085e9ebabdf2ff1a4",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-09a2cd5b88fff813c",
        "State": "available",
        "AvailabilityZone": "ap-south-1b",
        "SubnetId": "subnet-085e9ebabdf2ff1a4",
        "OwnerId": "681099220869",
        "CidrBlock": "172.16.128.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
  ````
  
 create a tag for the private subnet
 ```sh
 # aws ec2 create-tags --resources subnet-085e9ebabdf2ff1a4 --tags Key=Name,Value=Subnet3
 ````
 Enable public IP for the subnet1 and subnet2. 

````sh
# aws ec2 modify-subnet-attribute --subnet-id subnet-04a744aaae545aed7  --map-public-ip-on-launch
# aws ec2 modify-subnet-attribute --subnet-id subnet-0892ce7f63c928c17 --map-public-ip-on-launch
`````
We can see the complete information of the VPC and subnets using the below command.
````sh
# aws ec2 describe-subnets  --filters "Name=vpc-id,Values=vpc-09a2cd5b88fff813c"
{
    "Subnets": [
        {
            "MapPublicIpOnLaunch": false,
            "AvailabilityZoneId": "aps1-az3",
            "Tags": [
                {
                    "Value": "Subnet3",
                    "Key": "Name"
                }
            ],
            "AvailableIpAddressCount": 16379,
            "DefaultForAz": false,
            "SubnetArn": "arn:aws:ec2:ap-south-1:681099220869:subnet/subnet-085e9ebabdf2ff1a4",
            "Ipv6CidrBlockAssociationSet": [],
            "VpcId": "vpc-09a2cd5b88fff813c",
            "MapCustomerOwnedIpOnLaunch": false,
            "AvailabilityZone": "ap-south-1b",
            "SubnetId": "subnet-085e9ebabdf2ff1a4",
            "OwnerId": "681099220869",
            "CidrBlock": "172.16.128.0/18",
            "State": "available",
            "AssignIpv6AddressOnCreation": false
        },
        {
            "MapPublicIpOnLaunch": true,
            "AvailabilityZoneId": "aps1-az1",
            "Tags": [
                {
                    "Value": "Subnet1",
                    "Key": "Name"
                }
            ],
            "AvailableIpAddressCount": 16379,
            "DefaultForAz": false,
            "SubnetArn": "arn:aws:ec2:ap-south-1:681099220869:subnet/subnet-0892ce7f63c928c17",
            "Ipv6CidrBlockAssociationSet": [],
            "VpcId": "vpc-09a2cd5b88fff813c",
            "MapCustomerOwnedIpOnLaunch": false,
            "AvailabilityZone": "ap-south-1a",
            "SubnetId": "subnet-0892ce7f63c928c17",
            "OwnerId": "681099220869",
            "CidrBlock": "172.16.0.0/18",
            "State": "available",
            "AssignIpv6AddressOnCreation": false
        },
        {
            "MapPublicIpOnLaunch": true,
            "AvailabilityZoneId": "aps1-az3",
            "Tags": [
                {
                    "Value": "Subnet2",
                    "Key": "Name"
                }
            ],
            "AvailableIpAddressCount": 16379,
            "DefaultForAz": false,
            "SubnetArn": "arn:aws:ec2:ap-south-1:681099220869:subnet/subnet-04a744aaae545aed7",
            "Ipv6CidrBlockAssociationSet": [],
            "VpcId": "vpc-09a2cd5b88fff813c",
            "MapCustomerOwnedIpOnLaunch": false,
            "AvailabilityZone": "ap-south-1b",
            "SubnetId": "subnet-04a744aaae545aed7",
            "OwnerId": "681099220869",
            "CidrBlock": "172.16.64.0/18",
            "State": "available",
            "AssignIpv6AddressOnCreation": false
```````
### Create an Internet Gateway
We create an Internet Gateway using the following command, which returns the new Internet Gateway ID.

```sh
# aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-0a0c4642fa1846676
````
Attach the internet gateway to our VPC to allow the instances(and NAT) to communicate with the outside world.
```sh
]# aws ec2 attach-internet-gateway --vpc-id vpc-09a2cd5b88fff813c --internet-gateway-id igw-0a0c4642fa1846676
```
### Create a NAT Gateway.
Create a NAT Gateway after allocating an elastic IP using the allocate-address command

````sh
# aws ec2 allocate-address
{
    "Domain": "vpc",
    "PublicIpv4Pool": "amazon",
    "PublicIp": "43.204.250.43",
    "AllocationId": "eipalloc-0f1cd1c15346746a4",
    "NetworkBorderGroup": "ap-south-1"
}
````

````sh
# aws ec2 create-nat-gateway --subnet-id subnet-04a744aaae545aed7 --allocation-id eipalloc-0f1cd1c15346746a4
{
    "NatGateway": {
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-0f1cd1c15346746a4"
            }
        ],
        "VpcId": "vpc-09a2cd5b88fff813c",
        "State": "pending",
        "NatGatewayId": "nat-0cde282bae7236377",
        "SubnetId": "subnet-04a744aaae545aed7",
        "CreateTime": "2022-12-15T18:17:17.000Z"
    },
    "ClientToken": "bcabf912-7e39-4b86-a6f3-525a4e0d5809"
  ````

## Update route tables
  A route table will be created by default.  We can find the default route table information of the VPC using the below command
 ```sh
 # aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-09a2cd5b88fff813c"
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-0f4a13d68f8cd6b14",
                    "Main": true,
                    "RouteTableId": "rtb-094d3668fa1e72789"
                }
            ],
            "RouteTableId": "rtb-094d3668fa1e72789",
            "VpcId": "vpc-09a2cd5b88fff813c",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                }
            ],
            "OwnerId": "681099220869"
        }
````
  Create a custom route table for private subnet using the below command and associate the private subnet with it.
 ```sh
 # aws ec2 create-route-table --vpc-id vpc-09a2cd5b88fff813c --query RouteTable.RouteTableId --output text
rtb-0495f6847c093b451
```
````sh
# aws ec2 associate-route-table  --subnet-id subnet-085e9ebabdf2ff1a4 --route-table-id rtb-0495f6847c093b451
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-06e0d47d11d22f657"
}

````
### Add route table entries.
Add route table entry for IGW in the default route table to allow the instances to communicate with the internet.
```sh
# aws ec2 create-route --route-table-id rtb-094d3668fa1e72789 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0a0c4642fa1846676
{
    "Return": true
}

```
Add route table entry in the custom route table to enable instances in the private subnet to communicate with the NAT gateway.

```sh
# aws ec2 create-route --route-table-id rtb-0495f6847c093b451 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-0cde282bae7236377
{
    "Return": true
}
```
 - To confirm that the route has been created and is active, we can describe the route table using the following describe-route-tables command.
```sh 
# aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-09a2cd5b88fff813c"
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "SubnetId": "subnet-085e9ebabdf2ff1a4",
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-06e0d47d11d22f657",
                    "Main": false,
                    "RouteTableId": "rtb-0495f6847c093b451"
                }
            ],
            "RouteTableId": "rtb-0495f6847c093b451",
            "VpcId": "vpc-09a2cd5b88fff813c",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "Origin": "CreateRoute",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "NatGatewayId": "nat-0cde282bae7236377",
                    "State": "active"
                }
            ],
            "OwnerId": "681099220869"
        },
        {
            "Associations": [
                {
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-0f4a13d68f8cd6b14",
                    "Main": true,
                    "RouteTableId": "rtb-094d3668fa1e72789"
                }
            ],
            "RouteTableId": "rtb-094d3668fa1e72789",
            "VpcId": "vpc-09a2cd5b88fff813c",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "GatewayId": "igw-0a0c4642fa1846676",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "State": "active",
                    "Origin": "CreateRoute"
                }
            ],
            "OwnerId": "681099220869"
````
### Create security groups
Security group for bastion server
```sh
]# aws ec2 create-security-group --group-name bastion --description "Allow SSH access" --vpc-id vpc-09a2cd5b88fff813c
{
    "GroupId": "sg-066026975bc969da1"
}

````
 
### Security group for the frontend web server
```sh
# aws ec2 create-security-group --group-name webserver --description "Allow SSH access from bastion and  http access from all" --vpc-id vpc-09a2cd5b88fff813c
{
    "GroupId": "sg-01741c25809330268"
}
```
### Security group for the backend database server
```sh
# aws ec2 create-security-group --group-name dbserver --description "Allow SSH access from bastion" --vpc-id vpc-09a2cd5b88fff813c
{
    "GroupId": "sg-08e69b4dc4b3e7bf4"
}
```
### Add rules to the security groups.
  Add a rule to allow SSH access to the bastion server.
 
```sh
# aws ec2 authorize-security-group-ingress --group-id sg-066026975bc969da1 --protocol tcp --port 22 --cidr 0.0.0.0/0
````

Add a rule to allow SSH access to the frontend server from the bastion server

```sh
# aws ec2 authorize-security-group-ingress --group-id sg-01741c25809330268 --protocol tcp --port 22 --source-group sg-066026975bc969da1
```
Add a rule to allow HTTP access to the frontend server from the outside world(All IPv4).

```sh
]# aws ec2 authorize-security-group-ingress --group-id sg-01741c25809330268 --protocol tcp --port 80 --cidr 0.0.0.0/0
`````
Add a rule to allow SSH access to the backend server from the bastion server.
```sh
# aws ec2 authorize-security-group-ingress --group-id sg-08e69b4dc4b3e7bf4 --protocol tcp --port 22 --source-group sg-066026975bc969da1
```
Add a rule to enable the frontend server to access mysql on the backend server.
```sh
# aws ec2 authorize-security-group-ingress --group-id sg-08e69b4dc4b3e7bf4 --protocol tcp --port 3306 --source-group sg-01741c25809330268
````
###  Create a key pair
Create a key pair and save it to a file.  Also, update the permission of the file.
```sh
# aws ec2 create-key-pair --key-name aws.pem --query 'KeyMaterial' --output text > aws.pem
]# chmod 400 aws.pem
````
Enabled public DNS hostname for the VPC and verified it using describe-vpc-attribute command. 

```sh
#aws ec2 modify-vpc-attribute --vpc-id vpc-09a2cd5b88fff813c --enable-dns-hostnames "{\"Value\":true}"
# aws ec2 describe-vpc-attribute --vpc-id vpc-09a2cd5b88fff813c --attribute enableDnsHostnames
{
    "VpcId": "vpc-09a2cd5b88fff813c",
    "EnableDnsHostnames": {
        "Value": true
`````
### Launch instances
Launch the bastion server with the required security group, keypair, subnet, and AMI (Here, I've used Amazon Linux AMI)
```sh
#aws ec2 run-instances --image-id ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-066026975bc969da1 --subnet-id subnet-04a744aaae545aed7
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2022-12-15T19:06:52.000Z",
            "PrivateIpAddress": "172.16.94.4",
            "ProductCodes": [],
            "VpcId": "vpc-09a2cd5b88fff813c",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-0e46b0193e412956d",
            "EnaSupport": true,
            "ImageId": "ami-074dc0a6f6c764218",
            "PrivateDnsName": "ip-172-16-94-4.ap-south-1.compute.internal",
            "KeyName": "aws.pem",
            "SecurityGroups": [
                {
                    "GroupName": "bastion",
                    "GroupId": "sg-066026975bc969da1"
                }
            ],
            "ClientToken": "ed2fe057-8c0c-4ba8-a0c7-f7d99ec90365",
            "SubnetId": "subnet-04a744aaae545aed7",
            "InstanceType": "t2.micro",
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "0a:f8:ea:7c:fa:d4",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-09a2cd5b88fff813c",
                    "Description": "",
                    "NetworkInterfaceId": "eni-0154d7cf6913cbf5f",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-16-94-4.ap-south-1.compute.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.16.94.4"
                        }
                    ],
                    "PrivateDnsName": "ip-172-16-94-4.ap-south-1.compute.internal",
                    "InterfaceType": "interface",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-092eac26a16078cf6",
                        "AttachTime": "2022-12-15T19:06:52.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "bastion",
                            "GroupId": "sg-066026975bc969da1"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "681099220869",
                    "SubnetId": "subnet-04a744aaae545aed7",
                    "PrivateIpAddress": "172.16.94.4"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "ap-south-1b"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "MetadataOptions": {
                "State": "pending",
                "HttpEndpoint": "enabled",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1
            },
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-0ef49c877c9729381",
    "Groups": [],
    "OwnerId": "681099220869"
}

````
Created a nametag for the server
```sh
# aws ec2 create-tags --resources i-0e46b0193e412956d --tags Key=Name,Value=bastion
```
Launch the frontend webserver with the required security group, keypair, subnet, and AMI

```sh
# aws ec2 run-instances --image-id  ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-01741c25809330268 --subnet-id subnet-0892ce7f63c928c17

# aws ec2 run-instances --image-id  ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-01741c25809330268 --subnet-id subnet-0892ce7f63c928c17
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2022-12-15T19:17:02.000Z",
            "PrivateIpAddress": "172.16.39.41",
            "ProductCodes": [],
            "VpcId": "vpc-09a2cd5b88fff813c",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-04f8186272eff3024",
            "EnaSupport": true,
            "ImageId": "ami-074dc0a6f6c764218",
            "PrivateDnsName": "ip-172-16-39-41.ap-south-1.compute.internal",
            "KeyName": "aws.pem",
            "SecurityGroups": [
                {
                    "GroupName": "webserver",
                    "GroupId": "sg-01741c25809330268"
                }
            ],
            "ClientToken": "03f53ada-88d4-4a3a-b84b-f63443c795ea",
            "SubnetId": "subnet-0892ce7f63c928c17",
            "InstanceType": "t2.micro",
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "02:6f:8f:9e:76:c4",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-09a2cd5b88fff813c",
                    "Description": "",
                    "NetworkInterfaceId": "eni-08dfe392ed44fc6de",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-16-39-41.ap-south-1.compute.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.16.39.41"
                        }
                    ],
                    "PrivateDnsName": "ip-172-16-39-41.ap-south-1.compute.internal",
                    "InterfaceType": "interface",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-022a4e862fae854c2",
                        "AttachTime": "2022-12-15T19:17:02.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "webserver",
                            "GroupId": "sg-01741c25809330268"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "681099220869",
                    "SubnetId": "subnet-0892ce7f63c928c17",
                    "PrivateIpAddress": "172.16.39.41"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "ap-south-1a"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "MetadataOptions": {
                "State": "pending",
                "HttpEndpoint": "enabled",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1
            },
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-07e5d402bd5880abd",
    "Groups": [],
    "OwnerId": "681099220869"
}

# aws ec2 create-tags --resources i-04f8186272eff3024 --tags Key=Name,Value=frontend
````

Launch the backend database server with the required security group, keypair, subnet, and AMI.
```sh
# aws ec2 run-instances --image-id  ami-074dc0a6f6c764218 --count 1 --instance-type t2.micro --key-name aws.pem --security-group-ids sg-08e69b4dc4b3e7bf4 --subnet-id subnet-085e9ebabdf2ff1a4
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2022-12-15T19:21:57.000Z",
            "PrivateIpAddress": "172.16.142.111",
            "ProductCodes": [],
            "VpcId": "vpc-09a2cd5b88fff813c",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-06834f3b4faf1fe16",
            "EnaSupport": true,
            "ImageId": "ami-074dc0a6f6c764218",
            "PrivateDnsName": "ip-172-16-142-111.ap-south-1.compute.internal",
            "KeyName": "aws.pem",
            "SecurityGroups": [
                {
                    "GroupName": "dbserver",
                    "GroupId": "sg-08e69b4dc4b3e7bf4"
                }
            ],
            "ClientToken": "c1405dbf-0d92-4630-839b-81bcf60b88a0",
            "SubnetId": "subnet-085e9ebabdf2ff1a4",
            "InstanceType": "t2.micro",
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "0a:32:2a:48:76:9c",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-09a2cd5b88fff813c",
                    "Description": "",
                    "NetworkInterfaceId": "eni-0e257e20c1e956b28",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-16-142-111.ap-south-1.compute.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.16.142.111"
                        }
                    ],
                    "PrivateDnsName": "ip-172-16-142-111.ap-south-1.compute.internal",
                    "InterfaceType": "interface",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-086396bf25882245e",
                        "AttachTime": "2022-12-15T19:21:57.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "dbserver",
                            "GroupId": "sg-08e69b4dc4b3e7bf4"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "681099220869",
                    "SubnetId": "subnet-085e9ebabdf2ff1a4",
                    "PrivateIpAddress": "172.16.142.111"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "ap-south-1b"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "MetadataOptions": {
                "State": "pending",
                "HttpEndpoint": "enabled",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1
            },
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-032f9aba2c9a78477",
    "Groups": [],
    "OwnerId": "681099220869"

# aws ec2 create-tags --resources i-06834f3b4faf1fe16 --tags Key=Name,Value=backend
````
ssh to bastion server
use describe command with instance ID to get the public IP
`````sh
]# ssh -i aws.pem ec2-user@43.204.234.43
The authenticity of host '43.204.234.43 (43.204.234.43)' can't be established.
ECDSA key fingerprint is SHA256:SrlqiHSd48rSM5B1Er+eqnEWWP39SPaCSdqbsGRBhhw.
ECDSA key fingerprint is MD5:bd:8b:85:66:dd:fe:46:56:36:e3:f9:21:34:7d:41:9b.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '43.204.234.43' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.

$ sudo hostnamectl set-hostname bastion.ap-south-1.compute.internal

`````
ssh from bastion to frontend server using private IP from describe command
Copy the keypair to bastion server and change the ownership
```sh
$ sudo ssh -i aws.pem ec2-user@172.16.39.41
The authenticity of host '172.16.39.41 (172.16.39.41)' can't be established.
ECDSA key fingerprint is SHA256:xuYPSEpthtSNo1qphPobC7X4oJPxdJr0kOkVDMmzR5Q.
ECDSA key fingerprint is MD5:76:dc:79:a8:d0:64:af:45:8b:c0:6c:72:01:19:de:d7.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.39.41' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/

$ sudo hostnamectl set-hostname frontend.ap-south-1.compute.internal
````
Since we are able to access the servers from the bastion server today’s experiment seems to be success, however we are just trying to host a WordPress site on the setup.

Install, start, and enable MariaDB on this backend server.

```sh
sudo yum install mariadb-server -y
sudo systemctl restart mariadb.service
sudo systemctl enable mariadb.service
````
Set database root password.
```sh
# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n]
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
Create a database and user, and grant the privileges.
````sh
 mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
 mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.68-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> create database wpdb;
Query OK, 1 row affected (0.00 sec)                                                 
MariaDB [(none)]> create user 'wpdbuser'@'%' identified by '***********';
Query OK, 0 row affected (0.00 sec)                                                 
MariaDB [(none)]> grant all privileges on wpdb.* to 'wpdbuser'@'%';
Query OK, 0 row affected (0.00 sec)                                                 
MariaDB [(none)]> flush privileges;                                                  
Query OK, 0 row affected (0.00 sec)       

````

SSH to the frontend server using its private IP and update the hostname.

```sh
 sudo ssh -i aws.pem ec2-user@172.16.39.41
Last login: Thu Dec 15 19:51:35 2022 from ip-172-16-94-4.ap-south-1.compute.internal

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
[ec2-user@frontend ~]$
```

- Install Apache and PHP.
- Download Wordpress, extract it, and move the contents to the document root.
- Correct the permissions for the files
- Update the database credentials(database name, username, password, and private IP of the backend database server) in the wp-config file. 

```sh
    #  sudo amazon-linux-extras install php7.4
    #  sudo systemctl restart httpd.service
    #  sudo systemctl enable httpd.service
    #  wget https://wordpress.org/latest.zip
    #  unzip latest.zip
    #  sudo mv wordpress/* /var/www/html/
    #  sudo chown -R apache:apache /var/www/html/*
    #  sudo mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
    #  sudo vi /var/www/html/wp-config.php
 ````


Now access the WordPress installation page using the public IP or public DNS name, or domain name complete the WordPress installation and design the website.






