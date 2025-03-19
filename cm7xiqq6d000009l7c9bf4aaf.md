---
title: "Automate AWS Infrastructure Using Terraform"
seoTitle: "AWS Automation with Terraform"
seoDescription: "Learn how to deploy AWS infrastructure with Terraform, including VPC, subnet, security group, S3 Bucket, Load Balancer EC2 and Apache server setup"
datePublished: Thu Mar 06 2025 15:45:50 GMT+0000 (Coordinated Universal Time)
cuid: cm7xiqq6d000009l7c9bf4aaf
slug: automate-aws-infrastructure-using-terraform
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1741152613816/b954b2f3-5e09-463b-8e3c-c903b4aa40df.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1741268189131/ce581f48-e867-40cf-afdf-0eb38a3d7383.jpeg
tags: cloud, cloud-computing, devops, terraform, terraform-cloud

---

## Infrastructure Deployment Steps with Terraform

This blog contains the neccessary instructions and Terraform code to deploy an infrastructure stack on a cloud provider using Terraform.

These are the following steps will guide you through the process of creating a **VPC, Internet gateway, custom route table, subnet, security group, network interface, elastic IP, and an Ubuntu server with Apache2 installed and enabled.**

## Prerequisites

Before starting the deployment process, make sure you have the following prerequisites:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741268048102/4da0c49d-8e8a-4aaa-aa80-419937d73a6c.gif align="center")

* Terraform is installed on your local machine. You can download it from the official Terraform website. [Download Terraform](https://developer.hashicorp.com/terraform/install)
    
* An account with the chosen cloud provider (AWS here).
    
* Appropriate access and permissions to create the required resources.
    
* Basic knowledge of Terraform and the cloud provider's infrastructure and networking concepts.
    

***I highly recommend you go through the attached docs as they can be really helpful.***

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741268131015/fdc8ebb7-dedf-49c2-8fe5-915463c26f38.jpeg align="center")

***Note:*** *Terraform is declarative that is you describe the desired state of your infrastructure in a Terraform configuration file, specifying the resources, their properties, and the relationships between them. You define the desired end result rather than writing step-by-step instructions or imperative commands to achieve that result.*

## 🪜*Deployment Steps*

Follow the steps below to deploy the infrastructure stack using Terraform:

### 1\. Set Up Terraform and Configure AWS Provider

1. Make sure to install Terraform on your device then install the Terraform extension from VS Code.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741154600248/673fecd7-075b-41e9-b0bf-6b447b3f8281.png align="center")
    
    2. Create a folder and give a name of your choice.
        
    3. Create [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file. Terraform file is created using `.tf` extension.
        
    4. Configure **AWS provider** with Terraform.
        

To get your Access key and Secret key follow this article - [AWS Account and Access Keys](https://docs.aws.amazon.com/powershell/latest/userguide/creds-idc.html)

```bash
// Configure the AWS Provider

provider "aws" {
region      = ""  # Add Region
access_key  = ""  # Add access_key
secret_key  = ""  # Add secret_key
}
```

> ***It is not recommended to hard code the secrets there are other methods to securely store your credentials. But for now, keep it simple!***

Docs - [AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

### 2\. Create VPC

* Configure the desired VPC settings, such as CIDR block, name, and other parameters (if you want ).
    
    ```bash
    resource "aws_vpc" "myvpc" {
      cidr_block = "10.0.0.0/16"
    
      tags = {
        Name = "myvpc"
      }
    }
    ```
    
    * While creating a VPC we need to give the resource type and the resource name.
        
    * `cidr_block` is a required parameter for creating the VPC. This is the CIDR block of VPC.
        
    * `tags` are optional.
        

Docs - [Terraform AWS VPC](https://registry.terraform.io/providers/hashicorp/aws/3.3.0/docs/resources/vpc)

### 3\. Create an Internet Gateway

1. Within the same [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file, configure the internet gateway settings.
    
2. Associate the internet gateway with the previously created VPC.
    

```bash
// 2. Create Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.myvpc.id
}
```

Docs - [Terraform AWS Internet Gateway](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway)

4\. Create a Custom Route Table

1. In the [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file, configure the custom route table settings.
    
2. Associate the routing with the VPC.
    

```bash
// Create Custom Route Table

resource "aws_route_table" "RT" {
  vpc_id = aws_vpc.myvpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

// for ipv6
route {
ipv6_cidr_block = "::/0"
gateway_id = aws_internet_gateway.igw.id
}
}
```

* Here we set our default route to “0.0.0.0/0”
    
* `aws_internet_gateway` refers to the Terraform resource type that represents an internet gateway in AWS.
    
* `.igw` is a reference to a specific instance of the `aws_internet_gateway` resource. In this case, `igw` is the name given to the `aws_internet_gateway` resource instance that we previously gave in step 2.
    
* `.id` refers to the `id` attribute of the `aws_internet_gateway` resource instance. The `id` attribute uniquely identifies the internet gateway resource within AWS.
    
* In `ipv6_cidr_block` ,`::/0` is the IPv6 equivalent of `0.0.0.0/0` in IPv4 notation. It denotes that the route matches all IPv6 addresses, effectively making it a default route for IPv6 traffic.
    

Docs - [Terraform AWS Route Tables](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table)

> 🤷‍♂️Do you know what is AWS Route Table❓
> 
> A route table contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.

### 5\. Create a Subnet and associate it with the route table.

1. ### Create Subnet.
    
    ```bash
    // 4. Create a Subnet
    
    resource "aws_subnet" "sub2" {
      vpc_id                  = aws_vpc.myvpc.id
      cidr_block              = "10.0.1.0/24"
      availability_zone       = "us-east-1b"
      map_public_ip_on_launch = true
    
      tags = {
        Name = "sub2"
      }
    }
    ```
    
    Docs - [Terraform AWS Subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet)
    

**Associate the Subnet with the Route table.**

```bash
// 5. Associate subnet with the Route table

resource "aws_route_table_association" "rta1" {
  subnet_id      = aws_subnet.sub1.id
  route_table_id = aws_route_table.RT.id
}

resource "aws_route_table_association" "rta2" {
  subnet_id      = aws_subnet.sub2.id
  route_table_id = aws_route_table.RT.id
}
```

### 6\. Create a Security Group to Allow Port 22, 80, 443

1. Allow inbound traffic on ports 22 (SSH), 80(HTTP), and 443(HTTPS).
    

```bash
// 6. Create Security Group to allow port 22, 80, 443

resource "aws_security_group" "websg" {
  name   = "web"
  vpc_id = aws_vpc.myvpc.id

  ingress {
    description = "HTTP form VPC"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
ingress {
description = "HTPS"
from_port = 443
to_port   = 443
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
}

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "websg"
  }
}
```

* This declares an `aws_security_group` resource named "websg" and specifies a name and description for the security group. It also associates the security group with a VPC identified by `aws_vpc.myvpc.id`.
    
* This `ingress` block allows **inbound traffic** on port 443 (HTTPS) with the "**tcp**" protocol. The `cidr_blocks` parameter allows traffic from any source IP address (`0.0.0.0/0`), meaning it permits access from any location.
    
* Similarly next `ingress` block allows inbound traffic on port 80 (HTTP) with the “**tcp**” protocol and `cidr_blocks` parameter allows traffic from (`0.0.0.0/0`).
    
* The `egress` block allows all **outbound traffic** by specifying `from_port` and `to_port` as 0, `protocol` as "-1" (indicating all protocols), and permitting all destination IP addresses (`0.0.0.0/0` for IPv4 and `::/0` for IPv6).
    

Docs - [Terraform AWS Security Group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)

> 🤷‍♂️***Do you know what is the difference between ingress and load balancer***❓
> 
> ***Ingress refers to*** ***the process of incoming traffic or data entering a network or a specific network component, such as a server or a service. It generally refers to the path or entry point through which external traffic reaches the internal network.***
> 
> ***A load Balancer is a service that acts as a single entry point for incoming traffic and intelligently distributes it across the available resources, such as web servers or application instances, in a way that balances the workload.***

### 7\. Create a Network Interface in the subnet

* Associate the network interface with the previously created subnet that was created in step 4.
    
    ```bash
    // 7. Create a Network Interface in the Subnet
    
    resource "aws_network_interface" "web-server" {
      subnet_id       = aws_subnet.sub1.id
      private_ips     = ["10.0.1.50"]
      security_groups = [aws_security_group.websg.id]
    
    }
    ```
    
    * `subnet_id` specifies the ID of the subnet in which the network interface will be created. It references the `id` attribute of an existing `aws_subnet` resource named `subnet-1`.
        
    * `private_ips` is an attribute that allows you to assign one or more private IP addresses to the network interface.
        

Docs - [Terraform AWS Network Interface](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/network_interface)

### 8.Assign an Elastic IP to the Network Interface

1. Associate the Elastic IP with the network interface that was created in step 7.
    

```bash
// 8. Assign an elastic IP to the network interface.
resource "aws_eip" "one" {
  vpc = true
  network_interface = aws_network_interface.web-server.id
  associate_with_private_ip = "10.0.1.50"
  depends_on = [ aws_internet_gateway.igw ]
}
```

* `depends_on` allows you to create an explicit dependency between two resources.
    

Docs - [Terraform AWS EIP](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip)

### 9\. Create a Load Balancer (Application Load Balancer)

1. In the [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file, configure the load balancer settings.
    

```bash
resource "aws_lb" "myalb" {
  name               = "myalb"
  internal           = false
  load_balancer_type = "application"

  security_groups = [aws_security_group.websg.id]
  subnets         = [aws_subnet.sub1.id, aws_subnet.sub2.id]

  tags = {
    Name = "web"
  }
}
```

**Associate Load Balancer with Target group**

```bash
resource "aws_lb_target_group" "tg" {
  name     = "myTG"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.myvpc.id

  health_check {
    path = "/health"
    port = "traffic-port"
  }
}
```

Docs - [AWS Load Balancer Target Group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group)

**Assigning Load Balancer Target Group Attachment**

```bash
resource "aws_lb_target_group_attachment" "attach1" {
  target_group_arn = aws_lb_target_group.tg.arn
  target_id        = aws_instance.webserver1.id
  port             = 80
}

resource "aws_lb_target_group_attachment" "attach2" {
  target_group_arn = aws_lb_target_group.tg.arn
  target_id        = aws_instance.webserver2.id
  port             = 80
}
```

Docs - [AWS Load Balancer Target Group Attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group_attachment)

Now Assigning Load Balancer Listener

```bash
resource "aws_lb_listener" "listener" {
  load_balancer_arn = aws_lb.myalb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    target_group_arn = aws_lb_target_group.tg.arn
    type             = "forward"
  }
}
```

Docs - [AWS Load Balancer Listener](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener)

> `aws_lb_listener`is a process that checks for incoming connection requests to a load balancer and forwards them to the appropriate target group based on the protocol and port configured.
> 
> `aws_lb_listener`defines how the load balancer will handle incoming traffic and how it will route the requests to backend resources.

**Generating DNS Name for the Load Balancer**

* In the [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file, you can configure the dns setting
    

```bash
output "loadbalancerdns" {
  value = aws_lb.myalb.dns_name
}
```

> Displays the DNS name of the load balancer after creation. This DNS name can be used to route traffic to your application.

### **10\. Creating S3 Bucket**

* In the [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file, you can configure S3 Bucket Costom Setting
    

```bash
resource "aws_s3_bucket" "example" {
  bucket = "amit-terraform-2025-project"
}
```

Docs - [AWS S3 Bucket](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket)

### **11\. Create Ubuntu Server and Install/Enable Apache2**

1. Create the Ubuntu server and install Apache2.
    
2. In the [main.tf](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/main.tf) file, configure the instance settings.
    

```bash
resource "aws_instance" "webserver1" {
  ami                    = "ami-04b4f1a9cf54c11d0"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.websg.id]
  subnet_id              = aws_subnet.sub1.id
  user_data              = base64encode(file("userdata.sh"))
}

resource "aws_instance" "webserver2" {
  ami                    = "ami-04b4f1a9cf54c11d0"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.websg.id]
  subnet_id              = aws_subnet.sub2.id
  user_data              = base64encode(file("userdata1.sh"))
}
```

* Add your preferred `ami` (Amazon Machine Image) and add it's `instance type`, `aws_security_group`, `subnet_id`, and `userdata`.
    
* If you have trouble generating your key pair refer to these docs on AWS key-pair.
    

Docs - [AWS Key-Pair](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-keypair.html)

* In `userdata.sh` and `userdata1.sh` add the bash code to install apache2 on the Ubuntu server.
    

### 11\. Time to run the server🏃

Open up the terminal in VS Code or you can also use cmd to make sure that you are in your correct directory.

> ***Here are some basic CLI commands of Terraform: -***
> 
> Docs - [Terraform Basic CLI Commands](https://developer.hashicorp.com/terraform/cli/commands)

1. Run `terraform init`
    

You would see somethings similar to this -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741263964529/8d8ba1db-2833-4b21-8248-564797f7b73a.png align="center")

Running the previous command terraform will automatically create the required files.

2\. Now run `terraform plan`

* It is used to preview the changes that terraform will make to your infrastructure before applying them - it’s dry run.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264068027/0cebfe08-1407-4010-97e5-6922aa4e7701.png align="center")

As you can see there are no running instances in AWS EC2 instances.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264130038/d7380ff3-0b56-4aa2-b919-0de059737dff.png align="center")

As you can see there is not created any load balancer in AWS.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264324490/7094a4bc-86c1-48c9-bd3d-b6822bf4499b.png align="center")

3. Let us now run `terraform apply`
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264417627/87634eda-367f-4344-b399-61ce72aea81d.png align="center")

3. Type `yes`
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264469500/76f23250-83e4-4f1a-b0ce-2e0be69de663.png align="center")

> You can also use`terraform apply -auto-approve` to auto-approve all the actions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264539999/29c87276-2f32-43df-a551-357e917cd648.jpeg align="center")

🎉**WOOOOOOHOOOOO!!!** Your Ubuntu EC2 instance got successfully launched!!!!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264576244/276c7ed6-724d-4b9d-bcec-26b9b9bd459b.png align="center")

Hurray!!!

You can also verify this by using a Public IPv4 address, it must show a result similar to this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264653076/71a9c8e2-a963-4e5b-89e9-af380248a8f8.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741264677336/64b554c2-f745-47e6-9bce-24285df0ffae.png align="center")

You can see your very first web server written that is you successfully launched your instance and created a web server.

### 🏁**The End**

**Congratulations**! You have successfully deployed an infrastructure stack using Terraform. The stack includes a VPC, internet gateway, custom route table, subnet, security group, network interface, elastic IP, and an Ubuntu server with Apache2 installed and enabled. You can now access the Ubuntu server over the internet using the elastic IP and start hosting your websites or applications.

Please refer to the Terraform documentation and guides provided by your cloud provider for more detailed information on each step and additional customization options.

So, far you have seen the power of Iac using Terraform. I hope this blog might have helped you get to know about Terraform and a small demonstration of what it can do.

**Don’t forget to try it by yourself!**

### Architecture Diagram

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741261385139/249c135d-8c4d-4589-9761-8edbca08b2e8.png align="center")

Git Repo -

1. Star and fork the repository.
    
2. Clone the repository.
    
3. Read the steps from [Steps.md](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/blob/main/Steps.md)
    
4. Automate your AWS Infrastructure.
    

[https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/tree/main](https://github.com/amitkumar-Github8/Automating-AWS-Infrastructure-Using-Terraform/tree/main)