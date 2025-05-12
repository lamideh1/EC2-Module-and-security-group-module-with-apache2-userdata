# EC2-Module-and-security-group-module-with-apache2-userdata
Got it! Here's a **clear, step-by-step breakdown** of what you're meant to do in the EC2 + Apache2 Terraform project:

---

## **STEP-BY-STEP TASKS**

### **STEP 1: Create Your Project Folder**

* Open terminal.

```bash
mkdir terraform-ec2-apache
cd terraform-ec2-apache
```

---

### **STEP 2: Create Module Folders**

```bash
mkdir -p modules/ec2
mkdir -p modules/security_group
```

---

### **STEP 3: Write Apache2 UserData Script**

* Create a file named `apache_userdata.sh` in the root folder:

```bash
nano apache_userdata.sh
```

* Paste:

```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" | sudo tee /var/www/html/index.html
```

* Make it executable:

```bash
chmod +x apache_userdata.sh
```

---

### **STEP 4: Create EC2 Module**

* File: `modules/ec2/main.tf`

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  key_name      = var.key_name
  user_data     = var.user_data
  vpc_security_group_ids = [var.security_group_id]

  tags = {
    Name = "Apache-EC2"
  }
}
```

* Add `variables.tf` in same folder with:

```hcl
variable "ami" {}
variable "instance_type" {}
variable "key_name" {}
variable "security_group_id" {}
variable "user_data" {}
```

* Add `outputs.tf` in same folder:

```hcl
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}
```

---

### **STEP 5: Create Security Group Module**

* File: `modules/security_group/main.tf`

```hcl
resource "aws_security_group" "web_sg" {
  name        = "apache-sg"
  description = "Allow HTTP and SSH"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
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
```

* Add `variables.tf` in same folder:

```hcl
variable "vpc_id" {}
```

* Add `outputs.tf` in same folder:

```hcl
output "security_group_id" {
  value = aws_security_group.web_sg.id
}
```

---

### **STEP 6: Write the Main Configuration**

* File: `main.tf` in root folder:

```hcl
provider "aws" {
  region = "us-east-1"
}

data "aws_vpc" "default" {
  default = true
}

module "security_group" {
  source  = "./modules/security_group"
  vpc_id  = data.aws_vpc.default.id
}

module "ec2_instance" {
  source            = "./modules/ec2"
  ami               = "ami-0c02fb55956c7d316" # Amazon Linux 2 AMI
  instance_type     = "t2.micro"
  key_name          = "your-key-name" # Change this to your EC2 key pair
  security_group_id = module.security_group.security_group_id
  user_data         = file("apache_userdata.sh")
}
```

* File: `outputs.tf` in root folder:

```hcl
output "public_ip" {
  value = module.ec2_instance.instance_public_ip
}
```

---

### **STEP 7: Initialize Terraform**

```bash
terraform init
```

---

### **STEP 8: Apply the Configuration**

```bash
terraform apply
```

* Type `yes` when prompted.

---

### **STEP 9: Verify Deployment**

* Copy the outputted public IP.
* Visit `http://<public_ip>` in a browser.
* You should see: **Hello from <hostname>**

---

### **STEP 10: Document Observations**

Include:

* Issues you encountered (e.g., permission errors, module errors)
* What you learned (e.g., using UserData, referencing module outputs)
* Confirmation of Apache2 web server working

---

Would you like a downloadable project template or a GitHub-ready folder structure as well?
