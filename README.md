# VPC-and-subnet-creation-with-count-using-terraform

# Description
-------------------------------------------------- 

Created a VPC with 6 subnets, 2 route tables, and internet gateway. Also associated the subnets to the appropriate route tables.

## Prerequisites
-------------------------------------------------- 

Before we get started you are going to need so basics:

* [Basic knowledge of Terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
* [Terraform installed](https://www.terraform.io/downloads)
* [Valid AWS IAM user credentials with required access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)

## Installation

If you need to download terraform , then click here [Terraform](https://www.terraform.io/downloads)

Lets create a file for declaring the variables.This is used to declare the variables that pass values through the terrafrom.tfvars file.

## Create a varriable.tf file

~~~
variable "region" { 
}
variable "access_key" {
}
variable "secret_key" {
}
variable "vpc_cidr" {
}
variable "project" {
}
~~~

## Create a provider.tf file

~~~
provider "aws" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
~~~

## Create a terraform.tfvars file

~~~
region = "put-your-region-here"

access_key = "put-your-access_key-here"

secret_key = "put-your-secret_key-here"

vpc_cidr = "X.X.X.X/16"

project = "name-of-your-project"
~~~

## Create a datasource.tf file

~~~
data "aws_availability_zones" "subnet" {
  state = "available"
}
~~~

**Go to the directory that you wish to save your tfstate files.Then Initialize the working directory containing Terraform configuration files using below command.**

~~~
terraform init
~~~

**Lets start with main.tf file, the details are below**

> To create VPC

~~~sh
resource "aws_vpc" "main" {
   cidr_block = var.vpc_cidr
   instance_tenancy = "default"
    enable_dns_support = true
    enable_dns_hostnames = true
    tags = {
        Name = var.project
    }
}
~~~

> To create Internet gateway

~~~sh
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags = {
    Name = var.project
  }
}
~~~

> To create public subent

~~~sh
resource "aws_subnet" "public" {
  count = 3
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 3, count.index)
  availability_zone = data.aws_availability_zones.subnet.names[count.index]
  map_public_ip_on_launch = true
tags = {
    Name = "${var.project}-public${count.index+1}"
  }
}
~~~

> To create Route table for public subnet

~~~sh
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

   tags = {
    Name = "${var.project}-public"
  }
}
~~~

> To associate Route table for public subnet

~~~sh
resource "aws_route_table_association" "public" {
  count =3 
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
  depends_on = [
    aws_subnet.public ]
}
~~~

> To create private subnet

~~~sh
resource "aws_subnet" "private" {
  count = 3
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 3, "${count.index+3}")
  availability_zone = data.aws_availability_zones.subnet.names[count.index]
  map_public_ip_on_launch = false
tags = {
    Name = "${var.project}-private${count.index+1}"
  }
}
~~~

> To create Route table for private subnet

~~~sh
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
   tags = {
    Name = "${var.project}-private"
  }
}
~~~

> To associate Route table for private subnet

~~~sh
resource "aws_route_table_association" "private" {
  count =3 
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
  depends_on = [
    aws_subnet.private ]
}
~~~

Lets validate the terraform files using

```
terraform validate
```

Lets plan the architecture and verify once again

```
terraform plan
```

Lets apply the above architecture to the AWS.

```
terraform apply
```

Conclusion
This is a simple VPC creation with count using terraform. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>

