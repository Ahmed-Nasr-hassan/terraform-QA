# Terraform-QA

## lab1

![Alt text](./lab1.png?raw=true "Title")

### providers file

```bash
provider "aws" {
  shared_credentials_files = ["~/.aws/credentials"]
  shared_config_files = ["~/.aws/config"]
}
```

### vpc-components

```bash
resource "aws_vpc" "nasr-vpc" {
    cidr_block = "10.0.0.0/16"
    tags = {
        "name" : "terraform-vpc"
    }
    
}

resource "aws_subnet" "nasr-subnet" {
    vpc_id = aws_vpc.nasr-vpc.id
    cidr_block = "10.0.0.0/24"
}

resource "aws_internet_gateway" "nasr-gateway" {
    vpc_id = aws_vpc.nasr-vpc.id
}

resource "aws_route_table" "nasr-route-table" {
    vpc_id = aws_vpc.nasr-vpc.id
    
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.nasr-gateway.id
  }

  route {
    ipv6_cidr_block        = "::/0"
    gateway_id = aws_internet_gateway.nasr-gateway.id
  }

}

resource "aws_route_table_association" "rt-association" {
  subnet_id      = aws_subnet.nasr-subnet.id
  route_table_id = aws_route_table.nasr-route-table.id
}
```

### ec2 config

```bash
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical

}

resource "aws_network_interface" "ec2-network" {
  subnet_id   = aws_subnet.nasr-subnet.id
  private_ips = ["10.0.0.99"]
  tags = {
    Name = "primary_network_interface"
  }
}

resource "aws_security_group" "nasrsg" {
  description = "Allow SSH and HTTP inbound traffic"
  vpc_id      = aws_vpc.nasr-vpc.id

  ingress {
    description      = "SSH from Anywhere"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  ingress {
    description      = "HTTP from Anywhere"
    from_port        = 80
    to_port          = 80
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_ssh_http"
  }
}

resource "aws_instance" "nasr-ec2" {
  ami = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id = aws_subnet.nasr-subnet.id
  associate_public_ip_address = true
  vpc_security_group_ids = [aws_security_group.nasrsg.id]
  user_data = <<EOF
#!/bin/bash
sudo apt update -y
sudo apt install -y apache2
EOF
}
```

## Photos

![Alt text](./ec2-details.png?raw=true "Title")

---

![Alt text](./apache-page.png?raw=true "Title")

---
