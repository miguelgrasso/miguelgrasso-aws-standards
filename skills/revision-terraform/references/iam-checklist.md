# IAM — Checklist de Mínimo Privilegio

## Regla fundamental

Nunca usar `*` en `Action` ni en `Resource` sin justificación explícita en un comentario
inline del código Terraform. Si existe el wildcard, debe haber un comentario que explique
por qué es necesario y cuál es el mitigante.

## Checklist por tipo de recurso

### `aws_iam_policy` / inline policies

- [ ] Cada `Action` lista solo los permisos realmente necesarios (no grupos como `s3:*`)
- [ ] Cada `Resource` apunta a ARNs específicos, no `*`
- [ ] Si se necesita `Resource: "*"` (ej. `ec2:Describe*`), está documentado con comentario
- [ ] La policy tiene una descripción (`description`) que explica su propósito
- [ ] No contiene `"Effect": "Allow"` con `"Principal": "*"` (resource-based policies)

### `aws_iam_role`

- [ ] El `assume_role_policy` restringe `Principal` al servicio o cuenta específica
- [ ] No usa `"Principal": "*"` sin condición (`Condition` con `aws:SourceArn` u otro)
- [ ] El rol tiene un único propósito (un servicio o tarea, no varios)
- [ ] `max_session_duration` ajustado al mínimo necesario (default 3600s)

### `aws_iam_role_policy_attachment`

- [ ] Preferir customer managed policies sobre AWS managed (excepto en dev)
- [ ] No adjuntar `arn:aws:iam::aws:policy/AdministratorAccess` salvo casos excepcionales documentados
- [ ] No adjuntar `arn:aws:iam::aws:policy/PowerUserAccess` en prod

### `aws_iam_user` / `aws_iam_access_key`

- [ ] **No crear IAM users con access keys en Terraform para aplicaciones**
- [ ] Usar roles + instance profiles o IRSA (IAM Roles for Service Accounts) en EKS
- [ ] Si existe `aws_iam_access_key`, es para un caso de automatización documentado

### `aws_iam_instance_profile`

- [ ] El rol asociado tiene solo los permisos que la instancia/tarea necesita
- [ ] No reutilizar el mismo instance profile para instancias con distintos roles funcionales

## Patrones a buscar en el código

```hcl
# ❌ MAL — wildcard sin justificación
actions   = ["*"]
resources = ["*"]

# ❌ MAL — acción amplia innecesaria
actions = ["s3:*"]

# ✅ BIEN — permisos específicos
actions = [
  "s3:GetObject",
  "s3:PutObject",
]
resources = ["arn:aws:s3:::mi-bucket/*"]

# ✅ BIEN — wildcard con justificación
# tfsec:ignore:aws-iam-no-policy-wildcards
# Justificación: ec2:Describe* no acepta resource-level permissions (AWS limitation)
actions   = ["ec2:Describe*"]
resources = ["*"]
```

## Reducción continua de permisos

Revisar y reducir permisos después de cada sprint. Herramientas de referencia:
- AWS IAM Access Analyzer: identifica permisos no usados
- AWS CloudTrail + Athena: analiza qué acciones se invocan realmente
