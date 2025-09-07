# 08. Infraestructura como Código (IaC)

## 🏗️ ¿Qué es Infraestructura como Código?

La Infraestructura como Código (IaC) es la práctica de gestionar y aprovisionar recursos de infraestructura a través de archivos de código legible por máquinas, en lugar de procesos manuales.

### Beneficios Principales

- **Reproducibilidad**: Crear entornos idénticos consistentemente
- **Versionado**: Control de versiones para infraestructura
- **Automatización**: Reduce errores humanos y tiempo de deployment
- **Escalabilidad**: Facilita la gestión de infraestructura compleja
- **Documentación**: El código sirve como documentación viva

## 🛠️ Herramientas de IaC

### Terraform

Terraform es una herramienta de IaC agnóstica de proveedor que permite definir infraestructura usando HCL (HashiCorp Configuration Language).

#### Ejemplo: Infraestructura AWS con Terraform

```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.0"
}

# Configurar provider AWS
provider "aws" {
  region = var.aws_region
}

# Variables
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-igw"
    Environment = var.environment
  }
}

# Subnets públicas
resource "aws_subnet" "public" {
  count = 2

  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name        = "${var.environment}-public-subnet-${count.index + 1}"
    Environment = var.environment
    Type        = "public"
  }
}

# Data source para AZs disponibles
data "aws_availability_zones" "available" {
  state = "available"
}

# Security Group para aplicación web
resource "aws_security_group" "web" {
  name_prefix = "${var.environment}-web-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.environment}-web-sg"
    Environment = var.environment
  }
}

# Outputs
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}
```

## 🚀 Comandos Básicos de Terraform

```bash
# Inicializar proyecto Terraform
terraform init

# Validar configuración
terraform validate

# Planificar cambios
terraform plan

# Aplicar cambios
terraform apply

# Mostrar outputs
terraform output

# Destruir infraestructura
terraform destroy

# Formatear código
terraform fmt
```

## 🏗️ Buenas Prácticas de IaC

### 1. Estructura de Proyecto

```
infrastructure/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
└── scripts/
```

### 2. Uso de Módulos

```hcl
# modules/networking/main.tf
variable "environment" {
  description = "Environment name"
  type        = string
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}
```

## 🔒 Seguridad en IaC

### 1. Scanning de Seguridad

```bash
# Instalar tfsec
brew install tfsec

# Escanear configuración
tfsec .
```

### 2. Políticas de Seguridad

```hcl
# security.tf
resource "aws_s3_bucket_encryption_configuration" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

## ⚙️ Ejercicios Prácticos

1. **Infraestructura básica**: Crear una VPC con subnets públicas y privadas
2. **Aplicación web**: Desplegar una aplicación con Load Balancer
3. **Base de datos**: Agregar RDS con configuración segura
4. **Monitoreo**: Implementar CloudWatch y alertas

## 🎯 Buenas Prácticas

- ✅ **Versionado**: Mantener código de infraestructura en Git
- ✅ **Modularización**: Crear módulos reutilizables
- ✅ **State management**: Usar backend remoto para el estado
- ✅ **Testing**: Validar configuraciones antes de aplicar
- ✅ **Seguridad**: Escanear código por vulnerabilidades
- ✅ **Documentación**: Documentar módulos y decisiones

## 🔗 Recursos Adicionales

- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)

---

**Anterior**: [07. Contenedores](./07-contenedores.md) | **Siguiente**: [09. Cloud](./09-cloud.md) | **Inicio**: [README](../README.md)