# ops-generator

Sistema de generación de artefactos de software usando equipos de agentes Claude Code.

Está inspirado en `connector-generator-ops` de Nvisionx, pero generalizado para cualquier tipo de artefacto: endpoints, servicios, módulos, librerías — en cualquier stack.

## ¿Cómo funciona?

Un equipo de 6 agentes especializados construye el artefacto siguiendo el mismo flujo que usaría un equipo de ingeniería:

```
research_codebase → research_target → architect → implementer → test-writer → quality-reviewer → tester
```

1. **Investigar el codebase** — documenta los patrones del proyecto de referencia
2. **Investigar el target** — documenta qué hay que construir y cómo
3. **Architect** — diseña la arquitectura, elige estrategias, requiere aprobación humana
4. **Implementer** — escribe código siguiendo el diseño, pregunta al researcher cuando necesita info
5. **Test-writer** — escribe tests de integración basados en la arquitectura
6. **Quality-reviewer** — audita conformidad con los patrones del codebase
7. **Tester** — corre todos los tests, routea fallos, hace loop hasta que todo pasa

## Uso rápido

### Paso 1: Generar los documentos de investigación

Antes de correr el equipo, ejecuta los prompts de research. Edita los placeholders en cada archivo antes de usarlos.

```
# En Claude Code:
@prompts/research_codebase.md
# → genera generated/<artifact>/research/codebase-guide.md

@prompts/research_library.md   ← opcional, solo si hay una librería compartida
# → genera generated/<artifact>/research/library-guide.md

@prompts/research_target.md
# → genera generated/<artifact>/research/target-guide.md
```

### Paso 2: Escribir el prompt de creación

Copia un ejemplo de `prompts/examples/` y personalízalo:
- Describe el artefacto a construir con precisión
- Especifica el contrato de la interfaz (endpoints, campos, eventos)
- Incluye credenciales de test si aplica
- Links a docs externas relevantes

### Paso 3: Correr el equipo

```
(pega tu prompt de creación en Claude Code)
# El prompt referencia @prompts/create_team.md
# El equipo de 6 agentes se encarga del resto
# Output → generated/<artifact>/
```

### Paso 4: Monitorear

- Revisar el status de los agentes cada ~15 segundos
- Aprobar el plan del architect cuando lo solicite
- Esperar hasta que el tester reporte que todos los tests pasan

## Estructura del repo

```
ops-generator/
├── prompts/
│   ├── create_team.md          # Equipo de 6 agentes (template)
│   ├── research_codebase.md    # Fase 1: documentar patrones del codebase
│   ├── research_library.md     # Fase 2 (opcional): documentar API de la librería compartida
│   ├── research_target.md      # Fase 3: investigar qué construir
│   └── examples/
│       ├── create_endpoint.md  # Ejemplo: nuevo endpoint REST
│       └── create_service.md   # Ejemplo: nuevo microservicio
└── generated/                  # Output de cada generación
    └── <artifact>/
        ├── research/
        │   ├── codebase-guide.md    # Patrones del codebase de referencia
        │   ├── library-guide.md     # API de la librería compartida (si aplica)
        │   ├── target-guide.md      # Investigación del target
        │   ├── quality-review.md    # Auditoría de conformidad
        │   └── test-results.md      # Resultados de tests
        └── <artifact>/              # Código generado
            └── ARCHITECTURE.md
```

## Skills

- `min-prompt-tool-use` — reduce permission prompts de los agentes

Para proyectos específicos podés agregar skills en `.claude/skills/`:
- Go → copia `golang-patterns` de `connector-generator-ops`
- Node/TypeScript → crea tu propio `typescript-patterns`

## Configuración de `.claude/settings.json`

El `settings.json` incluye permisos comunes para múltiples stacks. Ajustá según tu proyecto:
- Para Go: agregar `GOPRIVATE`, `GOPROXY` en `env`
- Para Node: agregar permisos para `npm`, `npx`
- Para Python: configurar el path del virtualenv
