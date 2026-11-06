---
name: "diagrama-arquitectura"
description: "Genera un diagrama de arquitectura AWS en formato draw.io a partir de archivos Terraform. Activa con: diagrama, arquitectura aws, drawio, generar diagrama, visualizar infraestructura."
---

# Diagrama de Arquitectura AWS desde Terraform

## Cuándo usar esta skill

Úsala cuando el usuario quiera visualizar la infraestructura definida en archivos `.tf`
como un diagrama de arquitectura AWS en formato draw.io (`.drawio`).

## Proceso

### Paso 1 — Leer y parsear los archivos Terraform

Leer todos los archivos `.tf` del directorio indicado. Identificar y clasificar los
recursos en estas categorías:

**Red:**
- `aws_vpc` → VPC
- `aws_subnet` (public/private según `map_public_ip_on_launch` o nombre)
- `aws_internet_gateway` → IGW
- `aws_nat_gateway` → NAT GW
- `aws_security_group` → SG (agrupar visualmente alrededor de los recursos que protege)
- `aws_route_table` → no representar como shape, sino como flechas de routing

**Cómputo:**
- `aws_instance` → EC2
- `aws_ecs_cluster` + `aws_ecs_service` → ECS cluster / Fargate task
- `aws_eks_cluster` → EKS cluster
- `aws_lambda_function` → Lambda

**Almacenamiento y bases de datos:**
- `aws_s3_bucket` → S3
- `aws_db_instance` → RDS (indicar motor: mysql, postgres, etc.)
- `aws_dynamodb_table` → DynamoDB
- `aws_elasticache_cluster` → ElastiCache

**Red de entrega y entrada:**
- `aws_lb` (ALB/NLB) → Load Balancer
- `aws_cloudfront_distribution` → CloudFront
- `aws_api_gateway_rest_api` / `aws_apigatewayv2_api` → API Gateway

**Identidad y seguridad:**
- `aws_iam_role` → no representar como shape a menos que sea IRSA o cross-account
- `aws_kms_key` → KMS (solo si está referenciado en cifrado de datos)
- `aws_secretsmanager_secret` → Secrets Manager

### Paso 2 — Inferir relaciones

Construir las conexiones entre recursos a partir de las referencias en el código:

```
# Ejemplos de relaciones a trazar:
aws_nat_gateway.*.subnet_id          → conecta NAT con la subnet
aws_instance.*.vpc_security_group_ids → conecta EC2 con SG
aws_db_instance.*.db_subnet_group_name → conecta RDS con subnets
aws_lb_target_group attachment        → conecta ALB con instancias/ECS
aws_ecs_service.*.network_configuration.subnets → Fargate en subnet
```

### Paso 3 — Organizar el layout

Usar este orden de izquierda a derecha / arriba a abajo:

```
[Internet] → [IGW] → [ALB/NLB] → [Subnets Públicas]
                                          ↓
                               [NAT GW] → [Subnets Privadas]
                                                  ↓
                                    [ECS/EKS/EC2] [RDS] [ElastiCache]
```

Agrupar recursos dentro de rectángulos que representen:
- La VPC (contenedor externo)
- Cada AZ (si hay subnets en múltiples AZs)
- Subnets públicas vs privadas (color diferente: azul claro = público, verde claro = privado)

### Paso 4 — Generar el diagrama con el MCP drawio

Usar las herramientas del MCP `drawio` para:

1. Obtener los shapes AWS disponibles con `search_shapes` (buscar por nombre de servicio)
2. Construir el XML del diagrama con `open_drawio_mermaid` o directamente con XML
3. Abrir el resultado con `open_drawio_xml` para que el usuario lo vea

Consultar los shapes oficiales en [aws-shapes.md](aws-shapes.md).

### Paso 5 — Guardar el archivo

Guardar el diagrama como `arquitectura-<nombre-proyecto>.drawio` en el directorio
del proyecto Terraform (mismo nivel que los `.tf`).

Indicar al usuario:
```
Diagrama guardado en: arquitectura-<nombre>.drawio
Ábrelo en draw.io (https://app.diagrams.net) o con la extensión de VS Code.
```

## Notas de calidad

- Representar solo los recursos que realmente existen en el Terraform, no inferir
  recursos implícitos (ej. no añadir un WAF si no está en el código)
- Si hay más de 30 recursos, preguntar al usuario si quiere un diagrama completo
  o uno de alto nivel (solo VPC, subnets y servicios principales)
- Indicar explícitamente si hay recursos sin relaciones definidas en el código
  (pueden ser recursos standalone o módulos externos)
