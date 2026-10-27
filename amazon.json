packer {
  required_plugins {
    amazon = {
      version = ">= 1.2.0"
      source  = "github.com/hashicorp/amazon"
    }
  }
}

variable "aws_region" {
  type    = string
  default = "us-east-1"
}

source "amazon-ebs" "php83-app" {
  region        = var.aws_region
  instance_type = "t3.micro"
  ssh_username  = "ec2-user"
  ami_name      = "app-php83-apache-{{timestamp}}"

  source_ami_filter {
    filters = {
      name                = "al2023-ami-*-x86_64"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["amazon"]
  }
}

build {
  sources = ["source.amazon-ebs.php83-app"]

  # 1. Instalar Apache, PHP 8.3 e o PHP-FPM (essencial para o Apache processar PHP)
  provisioner "shell" {
    inline = [
      "sudo dnf update -y",
      "sudo dnf install -y httpd php8.3 php8.3-cli php8.3-fpm php8.3-mbstring php8.3-xml php8.3-mysqlnd php8.3-gd php8.3-zip",
      "sudo systemctl enable httpd php-fpm"
    ]
  }

  # 2. Copiar o código da aplicação
  provisioner "file" {
    source      = "app/"
    destination = "/tmp/app"
  }

  # 3. Mover a aplicação para o webroot padrão do Apache e ajustar permissões
  provisioner "shell" {
    inline = [
      "sudo rm -rf /var/www/html/*",
      "sudo cp -r /tmp/app/* /var/www/html/",
      "sudo chown -R apache:apache /var/www/html",
      "sudo chmod -R 755 /var/www/html"
    ]
  }
}