# 🚀 EasyCRUD – Full Stack Application Deployment using Terraform

EasyCRUD is a **Full Stack CRUD (Create, Read, Update, Delete) application** deployed on **AWS using Terraform Infrastructure as Code (IaC)**.

The infrastructure is automatically created using **Terraform**, while the application runs inside **Docker containers**.

---

# 🛠️ Tech Stack

* **Cloud:** AWS (EC2, RDS)
* **Infrastructure as Code:** Terraform
* **Backend:** Spring Boot (Java)
* **Frontend:** React (Vite)
* **Database:** MariaDB (Amazon RDS)
* **Containerization:** Docker
* **Web Server:** Apache

---

# ⚙️ Prerequisites

Before starting, ensure you have:

* AWS Account
* AWS CLI installed
* Terraform installed
* Git installed
* SSH key pair
* Required ports open in Security Groups

Ports required:

* `22` SSH
* `80` Frontend
* `8080` Backend
* `3306` RDS

---

# 📁 Project Structure

```
easycrud-terraform/
│
├── provider.tf
├── ec2.tf
├── rds.tf
├── security-group.tf
├── user-data.sh
│
└── README.md
```

---

# 🔹 PHASE 1: Infrastructure Provisioning using Terraform

---

# Step 1: Configure AWS CLI

```
aws configure
```

Enter:

```
AWS Access Key
AWS Secret Key
Region: ap-south-1
Output format: json
```

---

# Step 2: Create Terraform Provider

Create file:

```
provider.tf
```

```
provider "aws" {
  region = "ap-south-1"
}
```

---

# Step 3: Create Security Group

Create file:

```
security-group.tf
```

```
resource "aws_security_group" "easycrud_sg" {

  name = "easycrud-sg"

  ingress {
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 80
    to_port = 80
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 8080
    to_port = 8080
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 3306
    to_port = 3306
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

}
```

---

# Step 4: Create RDS Database

Create file:

```
rds.tf
```

```
resource "aws_db_instance" "easycrud_db" {

  identifier = "student-db"

  engine = "mariadb"
  instance_class = "db.t3.micro"

  allocated_storage = 20

  db_name = "student_db"

  username = "admin"
  password = "Admin12345"

  publicly_accessible = true

  skip_final_snapshot = true

}
```

---

# Step 5: Create EC2 Instance

Create file:

```
ec2.tf
```

```
resource "aws_instance" "easycrud_ec2" {

  ami = "ami-0f5ee92e2d63afc18"

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

# Step 6: Create EC2 Startup Script

Create file:

```
user-data.sh
```

```
#!/bin/bash

apt update -y
apt install docker.io git mysql-client -y

systemctl start docker

git clone https://github.com/AnuragPatil-cloud/EasyCRUD-Updated.git
```

---

# Step 7: Initialize Terraform

```
terraform init
```

---

# Step 8: Validate Terraform

```
terraform validate
```

---

# Step 9: Preview Infrastructure

```
terraform plan
```

---

# Step 10: Deploy Infrastructure

```
terraform apply
```

Type:

```
yes
```

Terraform will create:

* EC2 Instance
* RDS Database
* Security Groups

---

# 🔹 PHASE 2: Database Setup

---

# Step 11: Connect to EC2

```
ssh -i <your-key.pem> ubuntu@<EC2_PUBLIC_IP>
```

---

# Step 12: Connect to RDS

```
mysql -h <RDS-ENDPOINT> -u admin -p
```

---

# Step 13: Create Database

```
SHOW DATABASES;
CREATE DATABASE student_db;
USE student_db;
```

---

# Step 14: Create Students Table

```
CREATE TABLE students (
  id BIGINT NOT NULL AUTO_INCREMENT,
  name VARCHAR(255),
  email VARCHAR(255),
  course VARCHAR(255),
  student_class VARCHAR(255),
  percentage DOUBLE,
  branch VARCHAR(255),
  mobile_number VARCHAR(255),
  PRIMARY KEY (id)
);
```

---

# 🔹 PHASE 3: Backend Deployment

---

# Step 15: Install Docker

```
sudo apt install docker.io -y
sudo systemctl start docker
```

---

# Step 16: Clone Project

```
git clone https://github.com/AnuragPatil-cloud/EasyCRUD-Updated.git
```

```
cd EasyCRUD-Updated/backend
```

---

# Step 17: Configure Backend Application

```
cp src/main/resources/application.properties .
```

```
nano application.properties
```

Update configuration:

```
server.port=8080
spring.datasource.url=jdbc:mariadb://<RDS-ENDPOINT>:3306/student_db
spring.datasource.username=<username>
spring.datasource.password=<password>
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

# Step 18: Create Backend Dockerfile

```
nano Dockerfile
```

```
FROM maven:3.8.5-openjdk-17
COPY . /opt/
WORKDIR /opt
RUN rm -rf src/main/resources/application.properties
RUN cp -r application.properties src/main/resources/
RUN mvn clean package
WORKDIR target/
CMD ["java","-jar","student-registration-backend-0.0.1-SNAPSHOT.jar"]
```

---

# Step 19: Build Backend Image

```
docker build -t backend:v1 .
```

---

# Step 20: Run Backend Container

```
docker run -d -p 8080:8080 backend:v1
```

Check:

```
docker ps
```

Backend URL:

```
http://EC2_PUBLIC_IP:8080
```

---

# 🔹 PHASE 4: Frontend Deployment

---

# Step 21: Navigate to Frontend

```
cd ../frontend
```

```
ls -a
```

---

# Step 22: Configure Environment File

```
nano .env
```

```
VITE_API_URL=http://EC2_PUBLIC_IP:8080/api
```

---

# Step 23: Create Frontend Dockerfile

```
nano Dockerfile
```

```
FROM node:25-alpine
COPY . /opt/
WORKDIR /opt
RUN npm install
RUN npm run build
RUN apk update && apk add apache2
RUN cp -rf dist/* /var/www/localhost/htdocs/
EXPOSE 80
CMD ["httpd","-D","FOREGROUND"]
```

---

# Step 24: Build Frontend Image

```
docker build -t frontend:v1 .
```

---

# Step 25: Run Frontend Container

```
docker run -d -p 80:80 frontend:v1
```

Check:

```
docker ps
```

---

# Step 26: Access Application

Frontend:

```
http://EC2_PUBLIC_IP
```

Backend:

```
http://EC2_PUBLIC_IP:8080
```

🎉 **EasyCRUD Application is successfully deployed using Terraform + Docker!**

---

# 🔹 Destroy Infrastructure

To remove all resources:

```
terraform destroy
```

---

# 👨‍💻 Author

Anurag Patil
DevOps Engineer

GitHub:
https://github.com/AnuragPatil-cloud
