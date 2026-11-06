# Prompts frecuentes

Referencia rápida. Copia el bloque de texto del prompt, ajusta los valores entre
corchetes y pégalo en el chat.

---

## 1 — Explorar un directorio Terraform antes de tocarlo

```
Lee todos los archivos .tf en [DIRECTORIO] y dime:
- qué recursos crea, en qué orden de dependencia
- qué módulos externos usa y sus versiones
- qué variables son obligatorias (sin default)
- si hay algo inusual, deprecado o que debería revisarse antes de hacer cualquier cambio

No modifiques nada. Solo explica.
```

---

## 2 — Terraform plan explicado con costos

```
Ejecuta terraform plan en [DIRECTORIO] y para cada recurso que vaya a crearse,
modificarse o destruirse explícame:
- qué hace ese recurso en la arquitectura
- por qué Terraform lo va a cambiar (causa del diff)
- costo estimado por hora usando el MCP aws-pricing en la región del proyecto
  (por defecto us-west-2). No uses estimaciones de memoria — consulta precios reales.

Muéstrame el plan completo antes de cualquier apply. No ejecutes apply.
```

---

## 3 — Revisión de seguridad con el agente reviewer

> ⚠️ **Antes de pegar este prompt**: selecciona el agente **reviewer** en el
> selector de agentes. Pegarlo en el agente por defecto no cambia de agente.

```
Analiza [ARCHIVO o DIRECTORIO .tf] y genera un reporte con:
- políticas IAM con wildcards en Action o Resource
- recursos sin cifrado en reposo
- secretos o credenciales hardcodeadas
- tags obligatorios faltantes (Project, Owner, Environment, CostCenter)
- versiones de providers sin límite superior

Formato: tabla markdown con columnas Archivo, Línea, Severidad (CRÍTICO/ADVERTENCIA/INFO),
Descripción, Recomendación.
```

---

## 4 — Diagnóstico de pod caído con sre-oncall

> ⚠️ **Antes de pegar este prompt**: selecciona el agente **sre-oncall** en el
> selector de agentes. Pegarlo en el agente por defecto no cambia de agente.

```
Diagnostica este pod caído:

Namespace: [NAMESPACE]
Pod: [NOMBRE DEL POD]
Clúster: [NOMBRE DEL CLÚSTER]

Quiero saber:
- por qué está en estado [CrashLoopBackOff / OOMKilled / Pending / otro]
- qué dicen los logs y los events del pod
- si el problema es de recursos, imagen, permisos IAM (IRSA) o red
- los pasos exactos para reproducir el diagnóstico desde la terminal

Restricciones: no uses kubectl get secret ni kubectl exec.
No apliques ningún cambio. Solo diagnóstico y recomendaciones.
```

---

## 5 — Generar diagrama draw.io de la arquitectura

```
Lee los archivos .tf en [DIRECTORIO] y genera un diagrama de arquitectura AWS en
formato .drawio.

Incluye: VPC, subnets (públicas y privadas), NAT Gateway, IGW, Load Balancers,
instancias de cómputo (EC2/ECS/EKS/Lambda), bases de datos y sus security groups.

Organiza el layout de izquierda a derecha: internet → IGW → subnets públicas → subnets privadas.
Usa los shapes AWS oficiales. Guarda el resultado como arquitectura-[NOMBRE].drawio
en el mismo directorio que los .tf.
```

---

## 6 — Cierre de sesión con notas y checklist de destroy

```
Cierra esta sesión de trabajo:

1. Guarda un resumen en notas/[AAAA-MM-DD]-[tema].md usando la plantilla de
   ~/proyectos/miguelgrasso-aws-standards/templates/notas-sesion.md con:
   - conceptos vistos hoy
   - comandos clave usados
   - errores encontrados y su solución
   - preguntas de repaso (2 preguntas sobre lo visto)

2. Lista todos los recursos AWS que se hayan creado o planificado en esta sesión
   y dime exactamente qué comandos ejecutar para destruirlos, en qué orden
   y desde qué directorio.

3. Verifica que no quedó nada cobrando ejecutando estos comandos y reporta
   cualquier recurso activo inesperado:

   aws eks list-clusters --region [REGIÓN] --profile [PROFILE]

   aws ec2 describe-nat-gateways \
     --filter Name=state,Values=available \
     --query 'NatGateways[].{Id:NatGatewayId,VPC:VpcId,Subnet:SubnetId}' \
     --region [REGIÓN] --profile [PROFILE]

   aws elbv2 describe-load-balancers \
     --query 'LoadBalancers[].{Name:LoadBalancerName,DNS:DNSName,State:State.Code}' \
     --region [REGIÓN] --profile [PROFILE]

   aws ec2 describe-volumes \
     --filters Name=status,Values=available \
     --query 'Volumes[].{Id:VolumeId,Size:Size,AZ:AvailabilityZone}' \
     --region [REGIÓN] --profile [PROFILE]

   Los NAT Gateways y Load Balancers huérfanos son los recursos que más pueden
   acumular costo sin que lo notes. Si aparece alguno no esperado, no lo destruyas
   sin confirmar primero.
```

---

## 7 — Quiz SAA sobre lo visto hoy

```
Genera 5 preguntas tipo AWS Solutions Architect Associate sobre los temas que hemos
trabajado en esta sesión.

Formato para cada pregunta:
- enunciado con contexto real (tipo caso de uso, no definición)
- 4 opciones (A, B, C, D), solo una correcta
- señala las trampas de examen si las hay (⚠️)

No me des las respuestas todavía. Espera a que yo responda las 5 y luego corrígelas
con explicación de por qué cada opción incorrecta es incorrecta.
```

---

## 8 — Chequeo previo a la sesión

> **Perfil por defecto:** `mykegrasso`. Reemplaza `[PERFIL]` con tu perfil SSO
> si trabajas en otra cuenta o rol. Si ya tienes `AWS_PROFILE=mykegrasso`
> exportado en tu shell, puedes omitir `--profile [PERFIL]` en todos los comandos.

```
Antes de empezar, ejecuta estos comandos y confirma que todo está en orden:

1. Autenticar con SSO:
   aws sso login --profile [PERFIL]

2. Verificar identidad activa:
   aws sts get-caller-identity --profile [PERFIL]

   Confirma que el Account ID y el Role son los esperados antes de continuar.
   Si el role tiene más permisos de los necesarios para lo que vamos a hacer hoy,
   dímelo.

3. Revisar presupuesto y recursos activos que puedan estar cobrando:

   aws budgets describe-budgets \
     --account-id $(aws sts get-caller-identity --query Account --output text \
       --profile [PERFIL]) \
     --profile [PERFIL] \
     --query 'Budgets[].{Nombre:BudgetName,Limite:BudgetLimit.Amount,Actual:CalculatedSpend.ActualSpend.Amount}'

   aws ec2 describe-nat-gateways \
     --filter Name=state,Values=available \
     --query 'NatGateways[].{Id:NatGatewayId,VPC:VpcId}' \
     --profile [PERFIL]

   aws elbv2 describe-load-balancers \
     --query 'LoadBalancers[].{Name:LoadBalancerName,State:State.Code}' \
     --profile [PERFIL]

   Si hay NAT Gateways o Load Balancers activos que no reconoces, no arranques
   la sesión sin investigar primero.
```
