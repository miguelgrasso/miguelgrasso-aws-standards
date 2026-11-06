# Estimación de Costos — Recursos AWS frecuentes en Terraform

Precios de referencia en **us-east-1**. Consultar el MCP `aws-pricing` para
valores actualizados antes de entregar el reporte.

## Recursos de alto impacto económico

### Cómputo

| Recurso Terraform | Tipo ejemplo | $/hora ref. | Notas |
|---|---|---|---|
| `aws_instance` | t3.medium | ~$0.042 | Considerar Savings Plans o Reserved si uso > 1 mes |
| `aws_instance` | m5.xlarge | ~$0.192 | Evaluar Fargate si es carga containerizada |
| `aws_eks_node_group` | t3.medium ×2 | ~$0.084 | Mínimo recomendado para HA |
| `aws_ecs_task_definition` (Fargate) | 0.25 vCPU / 0.5 GB | ~$0.011 | Sin gestión de nodos |

**Orden de preferencia para cómputo nuevo:**
1. Fargate / ECS — sin gestión de nodos
2. Lambda — cargas event-driven
3. EC2 — solo si se necesita control de kernel, GPU o licencias

### Red

| Recurso Terraform | $/hora ref. | Notas |
|---|---|---|
| `aws_nat_gateway` | ~$0.045 + $0.045/GB | **Alto impacto**: cobro por gateway + transferencia |
| `aws_lb` (ALB) | ~$0.008 + LCU | LCU variable según tráfico |
| `aws_lb` (NLB) | ~$0.006 + NLCU | Más barato para TCP puro |
| `aws_vpn_gateway` | ~$0.05 | Por hora de conexión activa |
| `aws_dx_connection` | Variable | Cargo fijo por puerto + transferencia |

**Alerta NAT Gateway**: uno por AZ en prod es correcto para HA, pero
uno por AZ en dev/test es sobredimensionado — evaluar NAT instance o
compartir un único NAT Gateway en entornos no productivos.

### Almacenamiento y bases de datos

| Recurso Terraform | Referencia | $/hora ref. | Notas |
|---|---|---|---|
| `aws_db_instance` | db.t3.medium RDS MySQL | ~$0.068 | Multi-AZ duplica el costo |
| `aws_db_instance` | db.r6g.large RDS Postgres | ~$0.240 | Evaluar Aurora Serverless v2 para cargas variables |
| `aws_elasticache_cluster` | cache.t3.micro Redis | ~$0.017 | |
| `aws_s3_bucket` | — | $0.023/GB-mes | Revisar lifecycle policies y storage classes |
| `aws_efs_file_system` | — | $0.30/GB-mes | Caro vs S3; justificar uso |
| `aws_dynamodb_table` | On-demand | $0.00025/RCU | Evaluar provisioned si el patrón es predecible |

### Observabilidad

| Recurso Terraform | Referencia | Costo ref. | Notas |
|---|---|---|---|
| `aws_cloudwatch_log_group` | — | $0.50/GB ingestado | Definir `retention_in_days` siempre |
| `aws_cloudwatch_metric_alarm` | — | $0.10/alarm-mes | |

## Checks de optimización de costos

- [ ] `aws_cloudwatch_log_group` tiene `retention_in_days` definido (no 0/indefinido)
- [ ] `aws_s3_bucket` tiene lifecycle rule para objetos no accedidos > 90 días
- [ ] NAT Gateways: ¿cuántos hay? ¿son todos necesarios en dev/test?
- [ ] Instancias EC2: ¿están correctamente dimensionadas o son el tamaño por defecto?
- [ ] RDS Multi-AZ: ¿está activado en dev? (no debería, duplica costo)
- [ ] `aws_db_instance`: `deletion_protection = true` en prod, `skip_final_snapshot = false`
