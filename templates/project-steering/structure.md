---
# Cambiar a fileMatch o auto cuando el archivo crezca, para no gastar contexto
# en cada turno. Ejemplo:
#   inclusion: fileMatch
#   fileMatchPattern: 'infra/**/*.tf'
# o:
#   inclusion: auto
#   name: "estructura-proyecto"
#   description: "Layout de directorios, módulos y convenciones de nombrado"
inclusion: always
---

# Estructura del proyecto — <nombre del proyecto>

## Layout de directorios

```
<proyecto>/
├── src/                  # código fuente principal
├── test/                 # tests unitarios e integración
├── infra/                # Terraform / CDK
│   ├── modules/
│   └── environments/
│       ├── dev/
│       └── prod/
├── .kiro/
│   ├── steering/         # product.md, tech.md, structure.md
│   └── hooks/
└── docs/
```

## Módulos principales
<!-- Lista los módulos o servicios y una línea de qué hace cada uno. -->

## Convenciones de nombrado
- Recursos AWS: `<proyecto>-<ambiente>-<recurso>` (ej. `myapp-prod-alb`)
- Ramas git: `feature/<ticket>-descripcion`, `fix/<ticket>-descripcion`
- Commits: Conventional Commits (`feat:`, `fix:`, `chore:`)

## Dependencias externas
<!-- APIs de terceros, servicios compartidos, bases de datos compartidas. -->

## Decisiones de arquitectura
<!-- ADR-001: elegimos Fargate sobre EC2 porque... -->
