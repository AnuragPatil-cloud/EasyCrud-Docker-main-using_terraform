# 🚀 EasyCRUD – Full Stack Deployment using Terraform (AWS + Docker)

EasyCRUD is a **Full Stack CRUD (Create, Read, Update, Delete) Application** deployed using **Terraform Infrastructure as Code (IaC)** on AWS.

The infrastructure automatically provisions:

* **EC2 Instance** – Application Server
* **Amazon RDS (MariaDB)** – Database
* **Security Groups** – Network access control
* **Docker Containers** – Backend & Frontend services

The application stack includes:

* **Frontend:** React (Vite) + Apache Web Server
* **Backend:** Spring Boot (Java)
* **Database:** MariaDB (Amazon RDS)
* **Containerization:** Docker
* **Infrastructure Provisioning:** Terraform

---

# 🛠️ Tech Stack

* **Cloud:** AWS (EC2, RDS)
* **Infrastructure as Code:** Terraform
* **Backend:** Spring Boot
* **Frontend:** React (Vite)
* **Database:** MariaDB
* **Containerization:** Docker
* **Web Server:** Apache

---

# 📋 Prerequisites

Before starting, ensure you have:

* AWS Account
* AWS CLI Installed
* Terraform Installed
* Git Installed
* SSH Key Pair
* IAM user with EC2 and RDS permissions

Verify installations:

```bash
aws --version
terraform -version
git --version
```

---

# 📁 Project Structure

```
easycrud-terraform/
│
├── provider.tf
├── main.tf
├── ec2.tf
├── rds.tf
├── security-group.tf
├── variables.tf
├── outputs.tf
│
└── user-data.sh
```

---

# 🔹 Step 1: Configure AWS CLI

Configure AWS credentials.

```bash
aws configure
```

Provide:

```
AWS Access Key
AWS Secret Key
Region: ap-south-1
Output format: json
```

---

# 🔹 Step 2: Create Terraform Provider

Create file:

```
provider.tf
```

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

---

# 🔹 Step 3: Create Security Group

Create file:

```
security-group.tf
```

```hcl
resource "aws_security_group" "easycrud_sg" {

  name = "easycrud-sg"

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Frontend"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Backend"
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Database"
    from_port   = 3306
    to_port     = 3306
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
```

---

# 🔹 Step 4: Create RDS Database

Create file:

```
rds.tf
```

```hcl
resource "aws_db_instance" "easycrud_db" {

  identifier = "student-db"

  engine            = "mariadb"
  instance_class    = "db.t3.micro"
  allocated_storage = 20

  db_name  = "student_db"
  username = "admin"
  password = "Admin12345"

  publicly_accessible = true

  skip_final_snapshot = true

}
```

---

# 🔹 Step 5: Create EC2 Instance

Create file:

```
ec2.tf
```

```hcl
resource "aws_instance" "easycrud_ec2" {

  ami           = "ami-0f5ee92e2d63afc18"
  instance_type = "t2.micro"

  key_name = "my-key"

  vpc_security_group_ids = [
    aws_security_group.easycrud_sg.id
  ]

  user_data = file("user-data.sh")

  tags = {
    Name = "easycrud-server"
  }

}
```

---

# 🔹 Step 6: Create EC2 Startup Script

Create file:

```
user-data.sh
```

```bash
#!/bin/bash

apt update -y
apt install docker.io git -y

systemctl start docker

git clone https://github.com/AnuragPatil-cloud/EasyCRUD-Updated.git

cd EasyCRUD-Updated/backend

docker build -t backend:v1 .

docker run -d -p 8080:8080 backend:v1

cd ../frontend

docker build -t frontend:v1 .

docker run -d -p 80:80 frontend:v1
```

---

# 🔹 Step 7: Initialize Terraform

Run:

```bash
terraform init
```

Terraform will download required AWS provider plugins.

---

# 🔹 Step 8: Validate Configuration

```bash
terraform validate
```

---

# 🔹 Step 9: Preview Infrastructure

```bash
terraform plan
```

Terraform will show resources it will create.

Example:

```
+ EC2 Instance
+ RDS Database
+ Security Group
```

---

# 🔹 Step 10: Deploy Infrastructure

```bash
terraform apply
```

Confirm by typing:

```
yes
```

Terraform will create:

* EC2 Instance
* RDS Database
* Security Groups

---

# 🔹 Step 11: Access Application

After deployment, get the **EC2 Public IP**.

Open browser:

Frontend

```
http://EC2_PUBLIC_IP
```

Backend

```
http://EC2_PUBLIC_IP:8080
```

---

# 🔹 Step 12: Destroy Infrastructure

To remove all AWS resources:

```bash
terraform destroy
```

Confirm:

```
yes
```

---

# 🎯 DevOps Concepts Used

* Infrastructure as Code (Terraform)
* AWS EC2 provisioning
* AWS RDS database deployment
* Docker containerization
* Full Stack application deployment
* Cloud security groups

---

# 👨‍💻 Author

**Anurag Patil**
DevOps Engineer

GitHub
https://github.com/AnuragPatil-cloud

---

⭐ If you like this project, don't forget to **star the repository**.
