# Specify the provider (AWS in this case)
provider "aws" {
  region = "us-west-2"  # Replace with your desired AWS region
}

# Define a key pair (optional: required to SSH into the instance)
resource "aws_key_pair" "my_key" {
  key_name   = "my_key"
  public_key = file("~/.ssh/id_rsa.pub")  # Replace with your public key path
}

# Create a security group that allows SSH and HTTP access
resource "aws_security_group" "my_sg" {
  name_prefix = "terraform-example-"

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

# Provision an EC2 instance
resource "aws_instance" "my_instance" {
  ami           = "ami-0c55b159cbfafe1f0"  # Replace with your preferred AMI ID
  instance_type = "t2.micro"

  # Use the security group and key pair defined above
  security_groups = [aws_security_group.my_sg.name]
  key_name        = aws_key_pair.my_key.key_name

  # User data to configure the instance at launch (optional)
  user_data = <<-EOF
              #!/bin/bash
              sudo apt-get update
              sudo apt-get install -y apache2
              sudo systemctl start apache2
              EOF

  tags = {
    Name = "Terraform Example Instance"
  }
}

# Output the public IP of the instance
output "instance_ip" {
  value = aws_instance.my_instance.public_ip
}
