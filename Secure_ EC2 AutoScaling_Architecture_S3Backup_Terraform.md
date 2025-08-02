# Secure EC2 Auto Scaling Architecture with S3 Backup (Terraform)


## Introduction
This project provisions an auto scaling EC2 setup with a public subnet and secure S3 backup.

**Objective**
Provision a fault-tolerant architecture on AWS using free-tier resources

**Architecture**
- VPC with one public subnet
- Internet Gateway and routing
- EC2 with Auto Scaling Group (ASG)
- Security Group for SSH & HTTP
- S3 bucket with versioning and encryption
- Bash automation via `user-data.sh`

**Tech Stack**
- AWS (Free Tier)
- Terraform
- Bash
- Nginx (Web Server)

**Region**
us-east-1

**OS** 
Ubuntu (Free Tier)

**Tools**
Terraform
Bash scripting

**Features**
VPC with public/private subnets
Auto Scaling Group (ASG) with EC2 instances
NAT Gateway (free workaround used)
Bash automation for instance setup (e.g. Nginx)
S3 backup bucket with versioning and encryption

📁 Repository Structure

the name of your directory/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── user-data.sh
├── README.md


---

### 1. Create directory and cd to that directory
a. mkdir name of directory
b. cd the name of your directory

---

### 2. Create  **configuration files**

a. Create **provider.tf** file and the following below: 

```
provider "aws" {
  region = var.aws_region
}
```

<img width="671" height="203" alt="Image" src="https://github.com/user-attachments/assets/c4527615-e869-48f0-bd48-b29220632580" />


b. Create **main.tf** file and the following below: 

```
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone = "us-east-1a"

  tags = {
    Name = "public-subnet"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow HTTP & SSH"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_launch_template" "web_template" {
  name_prefix   = "web-launch-template"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  key_name      = var.key_name

  user_data = base64encode(file("user-data.sh"))

  network_interfaces {
    security_groups = [aws_security_group.web_sg.id]
    associate_public_ip_address = true
  }
}

resource "aws_autoscaling_group" "web_asg" {
  desired_capacity     = 1
  max_size             = 2
  min_size             = 1
  vpc_zone_identifier  = [aws_subnet.public.id]
  launch_template {
    id      = aws_launch_template.web_template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "web-server"
    propagate_at_launch = true
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

resource "aws_s3_bucket" "backup" {
  bucket = var.bucket_name
  tags = {
    Name = "BackupBucket"
  }
}

resource "aws_s3_bucket_versioning" "backup" {
  bucket = aws_s3_bucket.backup.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "backup" {
  bucket = aws_s3_bucket.backup.bucket

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

<img width="677" height="678" alt="Image" src="https://github.com/user-attachments/assets/ff9bccde-fe4e-4254-be18-c044b694f4f8" />


c. Create and configure variables.tf file
```
variable "aws_region" {
  default = "us-east-1"
}

variable "key_name" {
  description = "Your existing AWS key pair name"
  type        = string
}

variable "instance_type" {
  default = "t2.micro"
}

variable "bucket_name" {
  description = "S3 bucket name"
  type        = string
}
```

<img width="671" height="448" alt="Image" src="https://github.com/user-attachments/assets/2d775455-5d9c-4972-8675-fd88a2783f98" />


d. Create and Configure Outputs.tf file
```
output "bucket_name" {
  value = aws_s3_bucket.backup.bucket
}

output "asg_name" {
  value = aws_autoscaling_group.web_asg.name
}
```

<img width="667" height="295" alt="Image" src="https://github.com/user-attachments/assets/cdc380a5-085f-4394-8b81-4af05cc60d55" />


e. Create and Configure user-data.sh file
```
#!/bin/bash
apt update -y
apt install nginx -y
systemctl start nginx
systemctl enable nginx
echo "<h1>Welcome to My Auto Scaling App</h1>" > /var/www/html/index.html
```

<img width="675" height="189" alt="Image" src="https://github.com/user-attachments/assets/3b8615b6-4762-4964-9099-4b551fd5e81a" />


---

### 2. Initialize Terraform
Run

```
terraform init
```

<img width="635" height="719" alt="Image" src="https://github.com/user-attachments/assets/4cbb4fc3-f0f6-4464-bf80-5cca07d44490" />


---

### 3. Validate & Plan

Run

```
terraform validate
```

<img width="683" height="62" alt="Image" src="https://github.com/user-attachments/assets/bc36a295-cc42-4c01-ac67-72872cdbac67" />


Run

```
terraform plan
```

<img width="686" height="707" alt="Image" src="https://github.com/user-attachments/assets/cbbefa1c-ff5e-43be-8732-64c9dcf2149a" />


---

### 4. Deploy Resources


```
terraform apply
```
Type and enter **yes** when prompted.

<img width="690" height="729" alt="Image" src="https://github.com/user-attachments/assets/ef62c30e-666f-4e1f-8192-107a4d326524" />


---

### 5. Visit the Public IP
Terraform will print something like:

Outputs:
public_ip = "IP address"

Copy the IP address. Open your browser, paste the IP address and press the Enter key:

You’ll see: 🚀 Welcome to My Autoscaling App

<img width="500" height="371" alt="Image" src="https://github.com/user-attachments/assets/9061f11d-0fde-43b3-9d7f-5bdfe3152523" />


---

###. 6. Confirm resources provisioned on terraform

Run

```
terraform state list
```
and
```
terraform show
```

<img width="679" height="357" alt="Image" src="https://github.com/user-attachments/assets/d133054e-fa2e-484c-8e1a-5bed6791dcb8" />


---

### 7. Cleanup (Avoid Charges)
a. To destroy all provisioned infrastructure:


Run
```
terraform destroy
```

<img width="686" height="234" alt="Image" src="https://github.com/user-attachments/assets/be69403f-09c9-454e-b793-dbf89e134d8c" />

<img width="684" height="597" alt="Image" src="https://github.com/user-attachments/assets/9ede782d-3b3d-4624-b6e9-3a0f7a4ca959" />


b. Run the following to confirm
bash
```
terraform state list
```
and
```
terraform show
```

<img width="680" height="287" alt="Image" src="https://github.com/user-attachments/assets/62c3ece5-0ba0-48b3-84ae-56967973a498" />


---

###. 6. Confirm resources provisioned and destroyed from the Console

![Image](https://github.com/user-attachments/assets/eebc33d5-0f51-4da0-bacd-503c9af5eaa7)

<img width="1026" height="185" alt="Image" src="https://github.com/user-attachments/assets/d00dea42-0bd9-4545-bbb6-7046580a2631" />

![Image](https://github.com/user-attachments/assets/2b8e8826-700d-4835-a7c8-ae1c1ed47c78)

![Image](https://github.com/user-attachments/assets/078a5587-57d0-40e1-afd7-55c2ff3d45ba)

![Image](https://github.com/user-attachments/assets/a1a6e1e7-e24f-4c22-a568-601f339ad008)

![Image](https://github.com/user-attachments/assets/bcbabd91-389b-49ed-ba24-1fc88f554a09)

