---
name: "revision-terraform"
description: "Revisa código Terraform contra estándares de seguridad AWS (IAM mínimo privilegio), estima costos en us-east-1, y valida tags obligatorios. Activa con: revisar terraform, checklist seguridad, iam review, tags aws, audit iac."
---

# Revisión de Terraform — Seguridad, Costo y Tags

## Cuándo usar esta skill

Úsala cuando el usuario pida revisar, auditar o validar archivos `.tf` contra estándares de seguridad, costo o tagging de AWS.

## Proceso de revisión

Sigue este orden para cada archivo `.tf` que se te proporcione:

### 1. Seguridad IAM

Revisar políticas IAM contra el checklist en [iam-checklist.md](iam-checklist.md).

Reglas de bloqueo inmediato — reportar como **CRÍTICO** si aparece cualquiera de estas:
- `"Action": "*"` o `"Action": ["*"]` sin comentario de justificación
- `"Resource": "*"` en policies que no sean S3 list-level o CloudWatch genérico
- Credenciales o access keys hardcodeadas en cualquier campo
- `assume_role_policy` que permita `"Principal": "*"` sin condición
- Roles con `AdministratorAccess` managed policy adjunta

Reportar como **ADVERTENCIA**:
- Uso de AWS managed policies donde podría usarse customer managed
- Roles compartidos entre múltiples servicios o tareas
- Permisos sin scope de Resource por ARN específico

### 2. Cifrado en reposo

Verificar que los siguientes recursos tengan cifrado activado:

| Recurso Terraform | Campo requerido |
|---|---|
| `aws_s3_bucket` | `aws_s3_bucket_server_side_encryption_configuration` |
| `aws_ebs_volume` / `aws_instance` | `encrypted = true` en el bloque `ebs_block_device` o root |
| `aws_rds_instance` | `storage_encrypted = true` |
| `aws_dynamodb_table` | `server_side_encryption { enabled = true }` |
| `aws_efs_file_system` | `encrypted = true` |
| `aws_sqs_queue` | `kms_master_key_id` definido |
| `aws_secretsmanager_secret` | `kms_key_id` definido (CMK para datos sensibles) |

Para entornos `prod` o `staging`, verificar que se use CMK (`aws_kms_key`) en lugar de la key AWS-managed.

### 3. Secretos en texto plano

Buscar en todos los campos de string:
- Patrones que parezcan passwords, tokens o API keys hardcodeados
- Variables sin `sensitive = true` que contengan datos sensibles
- Outputs que exporten valores sensibles sin `sensitive = true`

Si hay secretos, recomendar migración a `data "aws_secretsmanager_secret_version"` o `data "aws_ssm_parameter"`.

### 4. TLS y comunicaciones

- Load balancers: verificar que no usen `ssl_policy` con TLS < 1.2
- RDS: `ca_cert_identifier` definido y `publicly_accessible = false`
- Endpoints de API Gateway: `minimum_compression_size` y HTTPS enforced

### 5. Tags obligatorios

Revisar **todos** los recursos `aws_*` contra el checklist en [tags-policy.md](tags-policy.md).

Tags requeridos (deben existir en el bloque `tags = {}`):

```
Project     = "<nombre del proyecto>"
Owner       = "<usuario o equipo>"
Environment = "dev" | "staging" | "prod"
CostCenter  = "<centro de costo>"
```

Reportar cada recurso sin alguno de estos tags como **INCUMPLIMIENTO DE POLÍTICA**.

### 6. Estimación de costos

Para cada recurso con costo significativo, consultar el MCP `aws-pricing` y reportar:
- Costo estimado **por hora** en USD (región `us-east-1`)
- Costo mensual proyectado (× 730 horas)
- Alternativas más económicas si el recurso es sobredimensionado

Ver detalle de recursos a evaluar en [cost-checklist.md](cost-checklist.md).

## Formato del reporte

Entregar el resultado en secciones:

```
## Revisión Terraform — <nombre del archivo>

### 🔴 CRÍTICO (bloquea merge)
- [RECURSO] descripción del problema

### 🟡 ADVERTENCIA (debe resolverse antes de prod)
- [RECURSO] descripción del problema

### 💰 COSTOS ESTIMADOS
| Recurso | Tipo | $/hora | $/mes |
|---------|------|--------|-------|

### 🏷️ TAGS FALTANTES
| Recurso | Tags ausentes |
|---------|---------------|

### ✅ CONFORME
Lista de recursos que pasan todos los checks.
```

## Cierre de sesión

Al terminar la revisión, recordar al usuario ejecutar `terraform destroy` si se crearon recursos de prueba durante el análisis.
