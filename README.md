# miguelgrasso-aws-standards

Kiro Power para revisión de infraestructura AWS en Terraform y generación de
diagramas de arquitectura con draw.io.

Sigue el estándar [Agent Plugins 1.0.0](https://agent-plugins.org/), lo que lo
hace compatible con Kiro, VS Code, Cursor y GitHub Copilot sin reempaquetado.

---

## Qué incluye

### Skills

| Skill | Activa cuando mencionas… | Qué hace |
|---|---|---|
| `revision-terraform` | revisar terraform, checklist seguridad, iam review, tags aws, audit iac | Revisa archivos `.tf` contra checklist de seguridad IAM, cifrado, secretos, TLS, tags obligatorios y estima costos por hora en us-east-1 |
| `diagrama-arquitectura` | diagrama, arquitectura aws, drawio, visualizar infraestructura | Lee los `.tf`, mapea los recursos a shapes AWS oficiales y genera un archivo `.drawio` editable |

### MCP Servers activados

| Servidor | Paquete | Para qué se usa |
|---|---|---|
| `aws-docs` | `awslabs.aws-documentation-mcp-server@latest` | Validar configuraciones contra documentación oficial AWS |
| `aws-pricing` | `awslabs.aws-pricing-mcp-server@latest` | Obtener precios actualizados para la estimación de costos |
| `drawio` | `@drawio/mcp` | Crear y abrir diagramas draw.io desde el agente |

---

## Instalación

### En Kiro IDE

```bash
cd ~/.kiro/powers
git clone https://github.com/miguelgrasso/miguelgrasso-aws-standards.git
```

Reiniciar Kiro o recargar desde el panel de Powers.

### En Kiro CLI

```
/powers install ~/.kiro/powers/miguelgrasso-aws-standards
```

### MCPs duplicados

Este Power activa sus propios procesos para `aws-docs`, `aws-pricing` y `drawio`.
Si ya tienes esos servidores en `~/.kiro/settings/mcp.json`, Kiro los arrancará
dos veces y el agente verá las mismas herramientas duplicadas (aunque con nombres
distintos, porque Kiro prefixa los del Power).

Recomendación: una vez que compruebes que el Power funciona, elimina esas tres
entradas de `~/.kiro/settings/mcp.json` y deja que el Power los gestione.

### Requisitos previos

Los MCPs `aws-docs` y `aws-pricing` se ejecutan con `uvx` (incluido con `uv`).
El MCP `drawio` requiere Node.js con `npx`.

Verificar que estén disponibles:
```bash
uvx --version
npx --version
```

### Credenciales AWS

El Power no incluye configuración de perfil AWS para no asumir el entorno de
quien lo usa. Antes de invocar cualquier skill que llame a `aws-pricing` o
`aws-docs`, exporta tu perfil:

```bash
export AWS_PROFILE=tu-perfil
```

O añádelo a tu `~/.zshrc` / `~/.bashrc` para que esté disponible en cada sesión.

---

## Uso

La activación es automática por palabras clave. Ejemplos de prompts:

```
Revisa el archivo eks.tf contra nuestros estándares de seguridad.

Audita los tags en todos los módulos de cluster/terraform/.

Genera un diagrama de arquitectura a partir de cluster/terraform/.

¿Cuánto cuesta por hora la infraestructura definida en vpc.tf?
```

---

## Estructura del repositorio

```
miguelgrasso-aws-standards/
├── plugin.json                                  ← manifiesto Agent Plugins 1.0.0
├── mcp.json                                     ← MCP servers
├── skills/
│   ├── revision-terraform/
│   │   ├── SKILL.md                             ← instrucciones principales
│   │   └── references/
│   │       ├── iam-checklist.md                 ← checklist IAM mínimo privilegio
│   │       ├── cost-checklist.md                ← recursos de alto impacto y precios ref.
│   │       └── tags-policy.md                   ← tags obligatorios y cómo aplicarlos
│   └── diagrama-arquitectura/
│       ├── SKILL.md                             ← workflow Terraform → draw.io
│       └── references/
│           └── aws-shapes.md                    ← shapes AWS y convenciones de layout
└── README.md
```

---

## Licencia

MIT
