# Terraform code to provision new VPC on AWS with one public and one or more private subnets with NAT Gateway

## High Level Overview

<img src="diagrams/AWS-VPC-NatGW.png" />

## Prerequisites

- git
- terraform (>= 0.12)
- AWS subscription

## How to use

- Get the repo

```
git clone https://github.com/achuchulev/terraform-aws-vpc-natgw.git
cd terraform-aws-vpc-natgw
```

- Create `terraform.tfvars` file

#### Inputs

| Name  |	Description |	Type |  Default |	Required
| ----- | ----------- | ---- |  ------- | --------
| aws_access_key | AWS access key | string | - | yes
| aws_secret_key | AWS secret key | string | - | yes
| aws_region | AWS region | string | "us-east-1" | no
| vpc_cidr_block | VPC subnet CIDR block | string | "10.0.0.0/16" | no
| vpc_subnet_cidr_blocks | VPC subnet CIDR blocks | list | "10.0.0.0/24","10.0.1.0/24" | no
| vpc_tags | VPC subnet CIDR blocks | map  | "" | no
| ssh_port | VPC subnet CIDR blocks | map  | "22" | no
| ssh_cidr | VPC subnet CIDR blocks | map  | "0.0.0.0/0" | no
| icmp_cidr | VPC subnet CIDR blocks | map  | "0.0.0.0/0" | no

- Initialize terraform and plan/apply

```
terraform init
terraform plan
terraform apply
```

- `Terraform apply` will:
  - create new VPC in the specified AWS region
  - create two or more subnets for the VPC
  - allocate one Elastic IP needed for NAT GW
  - create second route table
  - create Internet GW and route for the Public Subnet
  - create NAT GW and route through it to Internet for the Private Subnet(s)
  - assosciate main VPC route table with Private subnet(s)
  - assosciate second VPC route table with Public subnet
  
  
#### Outputs

| Name  |	Description 
| ----- | ----------- 
| vpc_id | The ID of created VPC
| vpc_name | The name of the VPC
| requester_subnet_ids | List VPC Subnet Ids
| azs | List Availability Zones in which Subnet are created


## Consume as a module

```
#### variables.tf ####

variable "aws_access_key" {}
variable "aws_secret_key" {}
variable "aws_region" {}

variable "vpc_name" {
  description = "Set a VPC name"
  default     = ""
}

variable "vpc_cidr_block" {
  description = "Define requester VPC cidr blocks"
  default     = "10.200.0.0/16"
}

variable "vpc_subnet_cidr_blocks" {
  type        = list(string)
  description = "Define VPC subnet cidr blocks"
  default     = ["10.200.0.0/24", "10.200.1.0/24"]
}

#### main.tf ####

module "new_vpc" {
  source = "git@github.com:achuchulev/terraform-aws-vpc-natgw.git"

  aws_access_key = var.aws_access_key
  aws_secret_key = var.aws_secret_key
  aws_region     = var.aws_region

  vpc_cidr_block         = var.vpc_cidr_block
  vpc_subnet_cidr_blocks = var.vpc_subnet_cidr_blocks

  vpc_tags = {
    Name = var.vpc_name
    Side = var.region
  }
}

#### outputs.tf ####

output "vpc_id" {
  value = module.new_vpc.vpc_id
}

output "vpc_name" {
  value = module.new_vpc.vpc_name
}


output "subnet_ids" {
  value = module.new_vpc.subnet_ids
}

output "azs" {
  value = module.new_vpc.azs
}

#### versions.tf ####

terraform {
  required_version = ">= 0.12"
}

```
