# AWS Shapes — Referencia para draw.io

Guía de los shapes AWS oficiales disponibles en el MCP `drawio`.
Usar `search_shapes` con los términos de búsqueda indicados para obtener
el identificador exacto antes de construir el diagrama.

## Términos de búsqueda por servicio

| Servicio AWS | Término para `search_shapes` | Shape family |
|---|---|---|
| VPC | `vpc` | AWS Networking |
| Subnet | `subnet` | AWS Networking |
| Internet Gateway | `internet gateway` | AWS Networking |
| NAT Gateway | `nat gateway` | AWS Networking |
| Application Load Balancer | `application load balancer` | AWS Networking |
| Network Load Balancer | `network load balancer` | AWS Networking |
| Security Group | `security group` | AWS Networking |
| Route 53 | `route 53` | AWS Networking |
| CloudFront | `cloudfront` | AWS Networking |
| EC2 Instance | `ec2 instance` | AWS Compute |
| ECS | `elastic container service` | AWS Compute |
| EKS | `elastic kubernetes service` | AWS Compute |
| Lambda | `lambda` | AWS Compute |
| Fargate | `fargate` | AWS Compute |
| Auto Scaling | `auto scaling` | AWS Compute |
| S3 | `simple storage service` | AWS Storage |
| EBS | `elastic block store` | AWS Storage |
| EFS | `elastic file system` | AWS Storage |
| RDS | `relational database service` | AWS Database |
| Aurora | `aurora` | AWS Database |
| DynamoDB | `dynamodb` | AWS Database |
| ElastiCache | `elasticache` | AWS Database |
| API Gateway | `api gateway` | AWS App Integration |
| SQS | `simple queue service` | AWS App Integration |
| SNS | `simple notification service` | AWS App Integration |
| EventBridge | `eventbridge` | AWS App Integration |
| IAM | `identity and access management` | AWS Security |
| KMS | `key management service` | AWS Security |
| Secrets Manager | `secrets manager` | AWS Security |
| CloudWatch | `cloudwatch` | AWS Management |
| CloudTrail | `cloudtrail` | AWS Management |
| Cognito | `cognito` | AWS Security |

## Convenciones de layout

### Colores de contenedor recomendados

```
VPC:              fondo #f0f7ff, borde #0066cc
Subnet pública:   fondo #e6f3e6, borde #339933
Subnet privada:   fondo #fff3e0, borde #ff9900
AZ (opcional):    fondo #fafafa, borde #999999 (línea discontinua)
```

### Flechas de conexión

```
Tráfico de entrada (usuario → infra): flecha sólida azul
Tráfico interno (entre servicios):    flecha sólida gris
Referencia de datos (lectura):        flecha discontinua
Cifrado / KMS:                        flecha punteada naranja
```

### Tamaños estándar

- Shapes de servicio: 60×60 px
- Contenedores VPC: ajustar al contenido con padding de 20px
- Texto de etiqueta: debajo del shape, 11pt

## Ejemplo mínimo de XML draw.io

```xml
<mxGraph>
  <root>
    <mxCell id="0"/>
    <mxCell id="1" parent="0"/>
    <!-- VPC container -->
    <mxCell id="2" value="VPC (10.0.0.0/16)" style="points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],[1,0.25],[1,0.5],[1,0.75],[1,1],[0.75,1],[0.5,1],[0.25,1],[0,1],[0,0.75],[0,0.5],[0,0.25]];shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_vpc;fillColor=#f0f7ff;strokeColor=#0066cc;fontStyle=1;" vertex="1" parent="1">
      <mxGeometry x="40" y="40" width="600" height="400" as="geometry"/>
    </mxCell>
    <!-- EC2 instance -->
    <mxCell id="3" value="Web Server" style="outlineConnect=0;fontColor=#232F3E;gradientColor=none;strokeColor=none;fillColor=#ED7100;labelBackgroundColor=#ffffff;align=center;labelPosition=center;verticalLabelPosition=bottom;verticalAlign=top;shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.ec2;" vertex="1" parent="1">
      <mxGeometry x="200" y="150" width="60" height="60" as="geometry"/>
    </mxCell>
  </root>
</mxGraph>
```
