# Política de Tags — Recursos AWS

## Tags obligatorios

Todo recurso `aws_*` en Terraform debe incluir los cuatro tags siguientes:

| Tag | Valores válidos | Ejemplo |
|---|---|---|
| `Project` | string libre, kebab-case | `eks-workshop` |
| `Owner` | usuario o equipo propietario | `miguelgrasso` / `platform-eng` |
| `Environment` | `dev` · `staging` · `prod` | `dev` |
| `CostCenter` | string libre, kebab-case | `platform-eng` |

## Cómo aplicarlos en Terraform

### Opción A — tags en cada recurso (explícito)

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Project     = var.project
    Owner       = var.owner
    Environment = var.environment
    CostCenter  = var.cost_center
  }
}
```

### Opción B — default_tags en el provider (recomendado)

```hcl
provider "aws" {
  region = "us-east-1"

  default_tags {
    tags = {
      Project     = var.project
      Owner       = var.owner
      Environment = var.environment
      CostCenter  = var.cost_center
    }
  }
}
```

Con `default_tags`, los tags se aplican a todos los recursos del provider
automáticamente. Los tags definidos a nivel de recurso los sobreescriben si hay
conflicto. Esta opción reduce el riesgo de olvidar tags en recursos nuevos.

### Variables recomendadas

```hcl
variable "project" {
  description = "Nombre del proyecto (tag obligatorio)"
  type        = string
}

variable "owner" {
  description = "Equipo o usuario responsable (tag obligatorio)"
  type        = string
}

variable "environment" {
  description = "Entorno de despliegue"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment debe ser dev, staging o prod."
  }
}

variable "cost_center" {
  description = "Centro de costo para billing (tag obligatorio)"
  type        = string
}
```

## Recursos que NO admiten tags en AWS

Algunos recursos no soportan tags a nivel de API (AWS los ignora o los rechaza).
En esos casos, documentar en un comentario `# tags: no-soportado-por-aws`:

- `aws_iam_policy_attachment`
- `aws_route`
- `aws_vpc_dhcp_options_association`

## Validación automática

Para forzar los tags en CI, usar [terraform-compliance](https://terraform-compliance.com/)
o AWS Config rule `required-tags`.
