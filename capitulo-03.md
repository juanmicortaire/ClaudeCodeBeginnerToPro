<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 3 — Extensibilidad: Skills, Slash Commands, Hooks y MCP.

-->

# Módulo 3: Extensibilidad — Skills, Slash Commands, Hooks y MCP

> **"Claude Code no es solo un asistente. Es una plataforma extensible. Lo que lo distingue de cualquier otro agente de código es que puedes enseñarle workflows, conectarlo a servicios externos y automatizar quality gates — todo con archivos de texto."**

En este módulo aprenderás los cuatro mecanismos de extensión de Claude Code y cuándo usar cada uno. Al terminar, tendrás tu propio sistema de extensiones configurado y funcionando.

    {{1}}
**Objetivo del módulo:** Dominar slash commands, skills, hooks y MCP servers. Saber cuándo usar cada uno y cómo se combinan en un workflow profesional.

    {{2}}
**Tiempo estimado:** 75-90 minutos

    {{3}}
**Prerrequisito:** Haber completado los Módulos 1 y 2 (CLAUDE.md configurado).

---

## 3.1 Mapa de Extensiones

Antes de profundizar en cada mecanismo, necesitas un mapa mental de cómo encajan entre sí:

``` ascii
  ┌─────────────────────────────────────────────────────────┐
  │                  EXTENSIONES DE CLAUDE CODE             │
  ├─────────────────────────────────────────────────────────┤
  │                                                         │
  │   SLASH COMMANDS          SKILLS          SUBAGENTES    │
  │   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   │
  │   │ /comando    │   │ Auto-invoca │   │ Contexto    │   │
  │   │ Manual      │   │ por Claude  │   │ aislado     │   │
  │   │ 1 archivo   │   │ Directorio  │   │ Workers     │   │
  │   └─────────────┘   └─────────────┘   └─────────────┘   │
  │          │                  │                 │         │
  │          └──────────────────┼─────────────────┘         │
  │                             │                           │
  │                    ┌────────┴────────┐                  │
  │                    │   MCP SERVERS   │                  │
  │                    │ (capa de tools) │                  │
  │                    └─────────────────┘                  │
  │                             │                           │
  │                    ┌────────┴────────┐                  │
  │                    │     HOOKS       │                  │
  │                    │ (automatización │                  │
  │                    │  reactiva)      │                  │
  │                    └─────────────────┘                  │
  │                                                         │
  │   PLUGINS = empaquetado de cualquier combinación        │
  │             de los anteriores para distribuir           │
  └─────────────────────────────────────────────────────────┘
```

La regla fundamental:

- **MCP** proporciona las herramientas (qué *puede* hacer Claude)
- **Skills** enseñan cómo usarlas (qué *debería* hacer Claude)
- **Hooks** automatizan reacciones (qué *debe* hacer Claude tras cada acción)
- **Slash commands** son atajos manuales (qué quieres *tú* disparar)
- **Plugins** empaquetan todo para compartir

#### Quiz: Orientación rápida

Necesitas que Claude ejecute automáticamente `prettier` cada vez que escribe un archivo `.ts`. ¿Qué mecanismo de extensión usarías?

- [( )] Un slash command `/prettier`
- [( )] Una skill de formateo
- [(X)] Un hook PostToolUse con matcher `Write(*.ts)`
- [( )] Un servidor MCP de formateo
***
**Correcto.** Los hooks son el mecanismo para reacciones automáticas a eventos del ciclo de vida. Un hook `PostToolUse` con matcher `Write(*.ts)` se dispara automáticamente cada vez que Claude escribe un archivo TypeScript, sin intervención manual.
***

---

## 3.2 Slash Commands — Atajos Manuales

Los slash commands son el punto de entrada más simple a la extensibilidad. Son plantillas de prompts almacenadas como archivos Markdown que invocas explícitamente cuando quieres.

### Anatomía de un Slash Command

Un slash command es literalmente un archivo `.md` con el texto del prompt:

``` markdown
<!-- .claude/commands/review.md -->

Revisa el código del archivo $ARGUMENTS buscando:

1. Problemas de seguridad (OWASP Top 10)
2. Problemas de rendimiento (complejidad, N+1 queries, memory leaks)
3. Violaciones de estilo según nuestras convenciones

Formato de output:
- Lista priorizada de issues (crítico > alto > medio > bajo)
- Para cada issue: ubicación, descripción y fix sugerido
- Resumen ejecutivo al final
```

Para invocarlo:

``` text
> /project:review src/auth/token-service.ts
```

Claude recibe el contenido del archivo con `$ARGUMENTS` reemplazado por `src/auth/token-service.ts`.

### Dos Ámbitos: Proyecto y Personal

| Ámbito | Ubicación | Invocación | Compartido |
|:-------|:----------|:-----------|:----------:|
| **Proyecto** | `.claude/commands/` | `/project:nombre` | Sí (vía git) |
| **Personal** | `~/.claude/commands/` | `/user:nombre` | No |

    {{1}}
**Comandos de proyecto** se commitean en git y están disponibles para todo el equipo. Ideales para workflows estandarizados como code review, generación de PRs o auditorías de seguridad.

    {{2}}
**Comandos personales** son tuyos. Ideales para atajos que solo tú usas: tu estilo de commit, tu forma de depurar, tu checklist de final de sprint.

### Subdirectorios para Organización

Puedes organizar comandos en subdirectorios:

``` text
.claude/commands/
├── review/
│   ├── security.md        →  /project:review/security
│   ├── performance.md     →  /project:review/performance
│   └── style.md           →  /project:review/style
├── generate/
│   ├── test.md            →  /project:generate/test
│   ├── docs.md            →  /project:generate/docs
│   └── migration.md       →  /project:generate/migration
└── deploy/
    ├── staging.md         →  /project:deploy/staging
    └── production.md      →  /project:deploy/production
```

### Ejercicio: Crea tu Primer Slash Command

    {{1}}
**Paso 1:** Crea el directorio de comandos:

``` bash
mkdir -p .claude/commands
```

    {{2}}
**Paso 2:** Crea un comando de generación de tests:

``` bash
cat > .claude/commands/test.md << 'EOF'
Genera tests unitarios completos para $ARGUMENTS.

Reglas:
- Framework: Jest (o el que use el proyecto según CLAUDE.md)
- Cobertura: happy path + edge cases + error handling
- Naming: describe("NombreDelModulo", () => { it("should...") })
- Mocks: mockear dependencias externas, no lógica interna
- Assertions: ser específico, no usar toBeTruthy genérico

Ejecuta los tests al terminar para verificar que pasan.
EOF
```

    {{3}}
**Paso 3:** Úsalo en sesión:

``` text
> /project:test src/services/payment-service.ts
```

#### Quiz: Slash commands

¿Cuál es la diferencia entre `/project:review` y `/user:review`?

- [( )] No hay diferencia, son sinónimos
- [(X)] `/project:review` está en `.claude/commands/` (compartido vía git); `/user:review` está en `~/.claude/commands/` (solo tuyo)
- [( )] `/project:review` solo funciona en modo headless
- [( )] `/user:review` tiene más prioridad que `/project:review`
***
**Correcto.** Los comandos de proyecto viven en el repositorio y se comparten con el equipo. Los comandos personales son globales para ti pero invisibles para otros. Si ambos existen con el mismo nombre, puedes invocar cada uno con su prefijo explícito.
***

---

## 3.3 Skills — Automatización Inteligente

Las skills son la evolución de los slash commands. La diferencia clave: además de invocarse manualmente con `/nombre`, **se activan automáticamente cuando Claude detecta que la tarea encaja con su descripción**.

### Anatomía de una Skill

Una skill es un directorio con un archivo `SKILL.md` que contiene frontmatter YAML y contenido Markdown:

``` markdown
<!-- ~/.claude/skills/code-review/SKILL.md -->
---
name: code-review
description: Reviews code for bugs, security issues, and style
  violations. Use when reviewing code, auditing changes, or
  when the user asks "how does this look" about code.
---

When reviewing code, always follow this process:

1. **Security scan first**: Check OWASP Top 10 vulnerabilities
2. **Logic review**: Look for off-by-one errors, null handling,
   race conditions
3. **Style check**: Verify naming conventions per CLAUDE.md
4. **Performance**: Identify N+1 queries, unnecessary loops,
   missing indexes
5. **Test coverage**: Note untested paths

Output format:
- Severity: CRITICAL | HIGH | MEDIUM | LOW
- Location: file:line
- Issue: description
- Fix: suggested code change

Always end with an executive summary.
```

### La Arquitectura de Progressive Disclosure

Este es el concepto que hace a las skills dramáticamente más eficientes que cargar instrucciones en CLAUDE.md o en el prompt:

``` ascii
  ┌──────────────────────────────────────────────────┐
  │          AL INICIO DE SESIÓN                     │
  │                                                  │
  │  Claude carga SOLO:                              │
  │  ┌─────────────────────────────────┐             │
  │  │ name: code-review               │ ~50 tokens  │
  │  │ description: Reviews code for...│             │
  │  └─────────────────────────────────┘             │
  │  ┌─────────────────────────────────┐             │
  │  │ name: generate-tests            │ ~50 tokens  │
  │  │ description: Generates unit...  │             │
  │  └─────────────────────────────────┘             │
  │  ┌─────────────────────────────────┐             │
  │  │ name: deploy-staging            │ ~50 tokens  │
  │  │ description: Deploys current... │             │
  │  └─────────────────────────────────┘             │
  │                                                  │
  │  Total: ~150 tokens para 3 skills                │
  │                                                  │
  ├──────────────────────────────────────────────────┤
  │          CUANDO SE ACTIVA UNA SKILL              │
  │                                                  │
  │  Claude carga las instrucciones completas:       │
  │  ┌─────────────────────────────────┐             │
  │  │ When reviewing code, always...  │             │
  │  │ 1. Security scan first...       │ ~500 tokens │
  │  │ 2. Logic review...              │             │
  │  │ ...                             │             │
  │  └─────────────────────────────────┘             │
  │                                                  │
  │  Ahorro: 98% respecto a cargar todo siempre      │
  └──────────────────────────────────────────────────┘
```

    {{1}}
Con 10 skills instaladas, pagas ~1.000 tokens al inicio (nombres + descripciones). Sin progressive disclosure, pagarías ~50.000+ tokens cargando todas las instrucciones completas siempre.

    {{2}}
> **Implicación práctica:** Puedes tener docenas de skills instaladas sin impacto en rendimiento. Solo la skill activa consume tokens significativos.

### Skill vs. Slash Command: ¿Cuándo Usar Cada Uno?

| Criterio | Slash Command | Skill |
|:---------|:-------------:|:-----:|
| Invocación | Solo manual (`/nombre`) | Manual + automática |
| Estructura | 1 archivo `.md` | Directorio con `SKILL.md` |
| Progressive disclosure | No (siempre cargado si referenciado) | Sí (solo nombre+descripción hasta activación) |
| Recursos adicionales | No | Sí (scripts, templates en el directorio) |
| Mejor para | Atajos rápidos de prompt | Workflows complejos y reutilizables |

    {{1}}
> **Desde Claude Code 2.1**, las skills aparecen en el menú de slash commands automáticamente. Puedes invocar `/code-review` directamente. La fusión es casi completa en la invocación, pero la diferencia arquitectónica (progressive disclosure, recursos adicionales) sigue siendo importante.

### Dónde Viven las Skills

| Ubicación | Alcance | Prioridad |
|:----------|:--------|:---------:|
| `.claude/skills/` | Solo este proyecto | Máxima |
| `~/.claude/skills/` | Todos tus proyectos | Normal |

En caso de conflicto de nombres, la skill de proyecto gana.

### Ejercicio: Crea tu Primera Skill

    {{1}}
**Paso 1:** Crea el directorio:

``` bash
mkdir -p .claude/skills/security-audit
```

    {{2}}
**Paso 2:** Crea `SKILL.md`:

``` bash
cat > .claude/skills/security-audit/SKILL.md << 'EOF'
---
name: security-audit
description: Performs a security audit on code changes.
  Activates when reviewing PRs, checking authentication code,
  or when the user mentions security, vulnerabilities, or OWASP.
---

## Security Audit Process

When performing a security audit:

### 1. Input Validation
- Check all user inputs for sanitization
- Verify parameterized queries (no string concatenation in SQL)
- Check file upload restrictions

### 2. Authentication & Authorization
- Verify JWT validation (expiry, signature, audience)
- Check role-based access control on all endpoints
- Verify CORS configuration

### 3. Data Protection
- Check for exposed secrets in code or config
- Verify encryption at rest and in transit
- Check PII handling compliance

### 4. Dependencies
- Run `npm audit` or equivalent
- Flag known vulnerable packages

### Output Format
Rate each finding: CRITICAL | HIGH | MEDIUM | LOW | INFO
Provide remediation steps for CRITICAL and HIGH findings.
EOF
```

    {{3}}
**Paso 3:** Pruébalo — ahora puedes invocarlo de dos formas:

``` text
> /security-audit                    # Invocación manual
> Revisa la seguridad de src/auth/   # Claude lo activa solo
```

#### Quiz: Skills

¿Por qué la arquitectura de progressive disclosure ahorra un 98% de tokens?

- [( )] Porque las skills se comprimen antes de cargar
- [( )] Porque Claude solo lee las skills que tú invocas manualmente
- [(X)] Porque al inicio solo se cargan nombres y descripciones (~50 tokens por skill), y las instrucciones completas se inyectan solo cuando la skill se activa
- [( )] Porque las skills se almacenan en MCP servers externos
***
**Correcto.** El truco es que Claude solo necesita saber qué skills existen (nombre + descripción) para decidir cuándo activar cada una. Las instrucciones detalladas se inyectan en el contexto únicamente cuando la skill se activa, ya sea por invocación manual o por detección automática.
***

---

## 3.4 Hooks — Automatización Reactiva

Los hooks son scripts que se ejecutan automáticamente en puntos específicos del ciclo de vida de Claude Code. Son la pieza que permite automatizar quality gates sin intervención humana.

### Puntos de Enganche del Ciclo de Vida

``` ascii
  Tú escribes un prompt
        │
        ▼
  ┌─────────────┐
  │ Claude      │
  │ razona...   │
  │             │
  │ Decide usar │
  │ una tool    │
  └──────┬──────┘
         │
         ▼
  ┌──────────────┐     ┌──────────────────────┐
  │ PreToolUse   │────►│ Hook: ¿Permitir?     │
  │              │     │ ¿Modificar input?    │
  └──────┬───────┘     └──────────────────────┘
         │
         ▼
  ┌──────────────┐
  │ Tool se      │
  │ ejecuta      │
  │ (Read, Write,│
  │  Bash, etc.) │
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐     ┌──────────────────────┐
  │ PostToolUse  │────►│ Hook: Formatear?     │
  │              │     │ Lint? Test? Notificar│
  └──────────────┘     └──────────────────────┘
```

Los hooks principales son:

    {{1}}
**PreToolUse:** Se ejecuta *antes* de que la herramienta actúe. Puede bloquear la acción o modificar su input. Útil para prevenir operaciones peligrosas.

    {{2}}
**PostToolUse:** Se ejecuta *después* de que la herramienta haya actuado. Útil para formateo automático, linting, ejecución de tests o notificaciones.

    {{3}}
**PermissionRequest:** Se ejecuta cuando Claude pide aprobación al usuario. Solo funciona en modo interactivo (no en headless/CI).

### Configuración en settings.json

Los hooks se definen en `settings.json` con un matcher que filtra por herramienta y patrón:

``` json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [
          {
            "type": "command",
            "command": "ruff format \"$file\""
          }
        ]
      },
      {
        "matcher": "Write(*.ts)",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$file\""
          }
        ]
      },
      {
        "matcher": "Write(*.test.*)",
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --testPathPattern=\"$file\""
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash(rm -rf *)",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'BLOCKED: rm -rf not allowed' && exit 1"
          }
        ]
      }
    ]
  }
}
```

    {{1}}
Este ejemplo configura:

- Auto-formateo con `ruff` tras escribir archivos Python
- Auto-formateo con `prettier` tras escribir archivos TypeScript
- Ejecución automática de tests tras escribir archivos de test
- Bloqueo de `rm -rf` en cualquier contexto

### Casos de Uso Profesionales de Hooks

    {{1}}
**Quality gate de linting:** Cada escritura de archivo pasa por el linter. Si falla, Claude recibe el error y corrige automáticamente, sin que tú intervengas.

    {{2}}
**Tests post-edición:** Tras editar código de producción, el hook ejecuta los tests relevantes. Si fallan, Claude ve el output y corrige.

    {{3}}
**Prevención de operaciones peligrosas:** `PreToolUse` hooks que bloquean `sudo`, `rm -rf`, escritura en archivos de producción o force push.

    {{4}}
**Notificaciones:** Un hook que envía un mensaje a Slack cuando Claude completa una tarea larga.

### Hooks en CI/CD

    {{1}}
> **Importante:** En modo headless (`claude -p`), los hooks `PermissionRequest` **no se disparan**. Usa `PreToolUse` para decisiones de permisos automáticas en CI. Esta es una trampa común que causa que pipelines se queden colgados esperando una aprobación que nunca llega.

#### Quiz: Hooks

Quieres que cada vez que Claude escriba un archivo en el directorio `src/auth/`, se ejecute automáticamente un scan de seguridad. ¿Cómo configurarías el hook?

- [( )] Un PreToolUse con matcher `Read(src/auth/*)`
- [(X)] Un PostToolUse con matcher `Write(src/auth/*)`
- [( )] Un PermissionRequest con matcher `Write(src/auth/*)`
- [( )] No es posible con hooks; necesitas una skill
***
**Correcto.** `PostToolUse` con matcher `Write(src/auth/*)` se dispara *después* de que Claude escribe cualquier archivo en el directorio de autenticación. El hook puede ejecutar entonces el scan de seguridad. Usamos Post (no Pre) porque necesitamos que el archivo ya esté escrito para analizarlo.
***

---

## 3.5 MCP — Model Context Protocol

MCP es el protocolo estándar abierto que conecta Claude Code con herramientas, APIs y fuentes de datos externas. Si los hooks son automatización reactiva dentro de Claude, MCP es la puerta al mundo exterior.

### El Concepto: USB-C para IA

``` ascii
  ┌──────────────────┐
  │   Claude Code    │
  │   (MCP Client)   │
  │                  │
  │  "Obtén los PRs  │          ┌─────────────────────┐
  │   abiertos del   │─────────►│  GitHub MCP Server  │
  │   repo"          │          └─────────────────────┘
  │                  │
  │  "Ejecuta esta   │          ┌─────────────────────┐
  │   query SQL"     │─────────►│  PostgreSQL MCP     │
  │                  │          └─────────────────────┘
  │                  │
  │  "Busca bugs     │          ┌─────────────────────┐
  │   en Jira"       │─────────►│  Jira MCP Server    │
  │                  │          └─────────────────────┘
  │                  │
  │  "Abre esta URL  │          ┌─────────────────────┐
  │   en el browser" │─────────►│  Browser-use MCP    │
  │                  │          └─────────────────────┘
  └──────────────────┘
```

    {{1}}
Cada MCP server añade herramientas (tools) que Claude puede invocar con lenguaje natural. No necesitas invocaciones explícitas: simplemente dices lo que necesitas y Claude elige la herramienta correcta.

### Claude Code: Cliente Y Servidor MCP

    {{1}}
**Como cliente:** Se conecta a múltiples servidores MCP simultáneamente, accediendo a herramientas externas mientras mantiene conexiones aisladas por seguridad.

    {{2}}
**Como servidor:** Expone sus herramientas internas (View, Edit, LS) a otras aplicaciones, permitiendo acceso programático a las capacidades de Claude Code desde fuera.

### Configuración de un MCP Server

Los servidores MCP se configuran en `.mcp.json` en la raíz del proyecto:

``` json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_TOKEN"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    }
  }
}
```

También puedes gestionar MCP por CLI:

``` bash
# Añadir un server
claude mcp add --transport stdio github \
  npx @modelcontextprotocol/server-github

# Listar servers conectados (en sesión)
> /mcp

# Cargar config externa
claude --mcp-config ./team-mcp.json

# Modo estricto: solo la config especificada (ignora otras fuentes)
claude --mcp-config ./ci-mcp.json --strict-mcp-config
```

### MCP Tool Search: Lazy Loading

    {{1}}
Una preocupación legítima: si tengo 5 MCP servers con 50 herramientas cada uno, ¿no contamina eso mi contexto con 250 definiciones de tools?

    {{2}}
La respuesta es **MCP Tool Search**. Esta funcionalidad implementa lazy loading para MCP servers, reduciendo el uso de contexto hasta un 95%. Las herramientas se cargan bajo demanda, solo cuando Claude las necesita.

### MCP Servers Esenciales

| Server | Función | Instalación |
|:-------|:--------|:------------|
| **GitHub** | Repos, PRs, issues, CI/CD | `npx @modelcontextprotocol/server-github` |
| **PostgreSQL** | Queries SQL en lenguaje natural | `npx @modelcontextprotocol/server-postgres` |
| **SQLite** | BD local | `npx @modelcontextprotocol/server-sqlite` |
| **Filesystem** | Operaciones de archivo avanzadas | `npx @modelcontextprotocol/server-filesystem` |
| **Browser-use** | Automatización web con Playwright | Playwright-based |
| **Moodle** | Gestión de cursos LMS | `npx moodle-mcp-server` |

### La Sinergia MCP + Skills

    {{1}}
**MCP proporciona las herramientas; las skills enseñan a usarlas.** Son complementarios:

``` text
  MCP Server de GitHub ──► Herramientas: create_pr, list_issues, ...
                               │
  Skill "pr-description" ──► "Cuando crees un PR, lee el git log,
                              genera una descripción con formato:
                              ## Summary, ## Changes, ## Test Plan"
```

Sin la skill, Claude crearía PRs con descripciones genéricas. Con la skill, sigue tu workflow estandarizado de equipo usando las herramientas que MCP le da.

#### Quiz: MCP

¿Qué flag usarías en un pipeline CI/CD para garantizar que Claude Code solo usa los MCP servers definidos en tu configuración, ignorando cualquier otra fuente?

- [( )] `--mcp-only`
- [( )] `--mcp-config ./ci-mcp.json`
- [(X)] `--mcp-config ./ci-mcp.json --strict-mcp-config`
- [( )] `--no-mcp`
***
**Correcto.** `--strict-mcp-config` asegura que solo se cargan los servers de la configuración especificada, ignorando los de `.mcp.json`, user settings o cualquier otra fuente. Esto garantiza consistencia y seguridad en pipelines automatizados.
***

---

## 3.6 Plugins — Empaquetado y Distribución

Los plugins son el mecanismo para empaquetar cualquier combinación de extensiones (skills, hooks, agentes, comandos, configuración MCP) como una unidad distribuible.

### Cuándo Crear un Plugin

    {{1}}
**No necesitas plugins para tu propio setup.** Si solo configuras tu entorno personal, usa directamente skills, hooks y MCP como hemos visto.

    {{2}}
**Sí necesitas plugins cuando** quieres compartir tu configuración con el equipo, con la comunidad, o estandarizar un setup que múltiples proyectos van a usar.

### Ecosistema de la Comunidad

El ecosistema de plugins de Claude Code ya es amplio. Algunos destacados:

    {{1}}
**SuperClaude:** Framework con comandos especializados, personas cognitivas y metodologías como "Introspection" y "Orchestration".

    {{2}}
**Context Engineering Kit:** Técnicas avanzadas de ingeniería de contexto con footprint mínimo de tokens.

    {{3}}
**Compound Engineering Plugin:** Sistema que convierte errores pasados en lecciones para mejora continua.

    {{4}}
**awesome-claude-code:** Lista curada en GitHub con cientos de plugins organizados por dominio.

    {{5}}
> **Antes de crear tu propio plugin**, revisa `awesome-claude-code`. Es probable que alguien ya haya resuelto tu problema.

---

## 3.7 Cómo Se Combinan en la Práctica

Veamos un workflow real que integra los cuatro mecanismos:

``` ascii
  CLAUDE.md dice:
  "No commitear sin aprobación"
  "Siempre escribir tests primero"
          │
          ▼
  ┌──────────────────────────────────────────────┐
  │ /project:new-feature auth-refresh-token      │  ◄── Slash command
  │                                              │      inicia el workflow
  │ Claude lee la spec con skill "spec-reader"   │  ◄── Skill se auto-activa
  │                                              │
  │ Implementa código...                         │
  │   PostToolUse Write(*.ts) ──► prettier       │  ◄── Hook auto-formatea
  │   PostToolUse Write(*.test.*) ──► jest       │  ◄── Hook auto-ejecuta tests
  │                                              │
  │ Revisa seguridad con skill "security-audit"  │  ◄── Skill se auto-activa
  │                                              │
  │ Crea PR vía MCP GitHub con skill "pr-desc"   │  ◄── MCP + Skill combinados
  │                                              │
  │ PreToolUse Bash(git push --force) ──► BLOCK  │  ◄── Hook previene force push
  └──────────────────────────────────────────────┘
```

    {{1}}
**Observa cómo cada mecanismo tiene su rol:**

- El **slash command** inicia el flujo (acción explícita tuya)
- Las **skills** se activan solas cuando detectan contexto relevante
- Los **hooks** automatizan calidad sin pensarlo (formateo, tests)
- **MCP** conecta con GitHub para crear el PR
- **CLAUDE.md** pone las restricciones que nadie puede violar

---

## 3.8 Tabla de Decisión Rápida

Usa esta tabla cuando no sepas qué mecanismo usar:

| Necesito... | Mecanismo | Ejemplo |
|:------------|:----------|:--------|
| Un atajo para un prompt que uso mucho | **Slash Command** | `/project:review $file` |
| Que Claude siga un proceso automáticamente | **Skill** | Skill de code review que se activa sola |
| Que algo se ejecute tras cada acción de Claude | **Hook** | Prettier tras cada Write |
| Conectar con un servicio externo | **MCP Server** | GitHub, PostgreSQL, Jira |
| Bloquear una acción peligrosa | **Hook (PreToolUse)** | Bloquear `rm -rf` |
| Compartir mi setup con el equipo | **Plugin** | Paquete con skills + hooks + MCP config |
| Instrucciones permanentes para el proyecto | **CLAUDE.md** | Convenciones, restricciones |
| Que Claude aprenda de mis correcciones | **Auto Memory** | Preferencias observadas |

#### Quiz: Decisión

Tu equipo quiere que todos los PRs generados por Claude Code sigan un formato específico con secciones "Summary", "Changes", "Test Plan" y "Security Considerations". ¿Qué combinación de mecanismos usarías?

- [( )] Solo un slash command `/create-pr`
- [( )] Solo un hook PostToolUse
- [(X)] Una skill "pr-description" que se auto-active al crear PRs + MCP de GitHub para la integración
- [( )] Solo añadir las instrucciones a CLAUDE.md
***
**Correcto.** La combinación óptima es una skill (se auto-activa cuando Claude detecta que va a crear un PR, con las instrucciones de formato) + MCP de GitHub (proporciona la herramienta `create_pr` para interactuar con la API de GitHub). Podrías añadir una línea en CLAUDE.md como referencia: "Al crear PRs, seguir la skill pr-description".
***

---

## 3.9 Resumen del Módulo

En este módulo has aprendido:

- [X] Los cuatro mecanismos de extensión y cuándo usar cada uno
- [X] Slash commands: creación, ámbitos (proyecto/personal) y parametrización
- [X] Skills: progressive disclosure, auto-activación y ahorro del 98% en tokens
- [X] Hooks: PreToolUse y PostToolUse como quality gates automáticas
- [X] MCP: conexión con servicios externos y Tool Search para lazy loading
- [X] Plugins: empaquetado y distribución de configuraciones
- [X] Cómo se combinan los mecanismos en un workflow profesional
- [X] La tabla de decisión rápida para elegir el mecanismo correcto

---

## Evaluación Final del Módulo 3

**Pregunta 1:** ¿Qué mecanismo de extensión implementa "progressive disclosure" para ahorrar tokens?

- [( )] Slash Commands
- [(X)] Skills
- [( )] Hooks
- [( )] MCP Servers
***
**Correcto.** Las skills cargan solo nombre y descripción al inicio (~50 tokens cada una). Las instrucciones completas se inyectan solo al activarse. Este es el mecanismo de progressive disclosure.
***

**Pregunta 2:** ¿En qué archivo se configuran los hooks de Claude Code?

[[settings.json]]
[[?]] Es el archivo central de configuración de Claude Code
[[?]] Tiene formato JSON
***
**Correcto.** Los hooks se configuran en `settings.json`, dentro del objeto `"hooks"` con claves como `"PostToolUse"` y `"PreToolUse"`.
***

**Pregunta 3:** Clasifica cada escenario con el mecanismo correcto:

[[Slash Command]   [Skill]   [Hook]   [MCP Server]]
[( )               ( )       (X)      ( )         ]  Ejecutar `black` automáticamente tras escribir un `.py`
[(X)               ( )       ( )      ( )         ]  Atajo `/deploy staging` que ejecutas manualmente
[( )               (X)       ( )      ( )         ]  Claude sigue un proceso de review cuando detecta que estás revisando código
[( )               ( )       ( )      (X)         ]  Claude consulta issues abiertos en Jira

**Pregunta 4:** ¿Qué ocurre con los hooks `PermissionRequest` en modo headless (`claude -p`)?

- [( )] Funcionan igual que en modo interactivo
- [( )] Aprueban todo automáticamente
- [(X)] No se disparan; hay que usar PreToolUse para permisos automáticos en CI
- [( )] Bloquean todas las herramientas
***
**Correcto.** Esta es una trampa común en CI/CD. Los hooks `PermissionRequest` requieren interacción humana que no existe en modo headless. El pipeline se quedaría colgado. Usa `PreToolUse` para decisiones de permisos automáticas en pipelines.
***

**Pregunta 5:** Diseña la configuración: quieres que Claude Code se conecte a GitHub y que cada archivo TypeScript escrito se formatee automáticamente con prettier. ¿Qué necesitas configurar?

- [( )] Solo `.mcp.json` con GitHub
- [( )] Solo `settings.json` con un hook PostToolUse
- [(X)] `.mcp.json` con GitHub MCP Server + `settings.json` con un hook PostToolUse matcher `Write(*.ts)` que ejecute prettier
- [( )] Una skill que combine ambas funciones
***
**Correcto.** Son dos necesidades distintas que requieren dos mecanismos distintos: MCP para la conexión con GitHub (definido en `.mcp.json`) y un hook para el autoformateo (definido en `settings.json`). Cada mecanismo hace lo que mejor sabe hacer.
***

---

## Siguiente: Módulo 4 — Agentes Especializados

En el próximo módulo entraremos en el corazón del curso:

- Subagentes: delegación aislada de tareas
- Custom Agents con `@mención`: tu equipo de especialistas IA
- Diseño del roster completo: agente de specs, arquitectura, desarrollo, QA, seguridad y documentación
- Agent Teams: colaboración multi-agente con comunicación peer-to-peer
- Coordinator Mode y patrones de orquestación

> **Tarea antes del Módulo 4:** Crea un custom agent básico en `.claude/agents/code-reviewer.md` con un system prompt de revisión de código y herramientas restringidas a solo lectura. Prueba invocarlo con `@code-reviewer` en tu próxima sesión. Usaremos este agente como base para construir el equipo completo en el Módulo 4.
