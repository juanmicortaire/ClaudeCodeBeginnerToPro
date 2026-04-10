<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 5 — Workflows Profesionales.

-->

# Módulo 5: Workflows Profesionales

> **"Conocer las herramientas no es lo mismo que saber usarlas juntas. Este módulo transforma tu conocimiento de componentes individuales en pipelines integrados que resuelven problemas reales."**

En los módulos anteriores aprendiste cada pieza por separado: memoria, extensiones, agentes. Ahora las conectamos en workflows de producción que cubren desde el primer commit hasta el deploy, pasando por testing, security review y documentación — todo orquestado desde Claude Code.

    {{1}}
**Objetivo del módulo:** Diseñar e implementar workflows end-to-end: el ciclo de vida completo de una feature, integración con Git, automatización en CI/CD y monitorización continua.

    {{2}}
**Tiempo estimado:** 75-90 minutos

    {{3}}
**Prerrequisito:** Haber completado los Módulos 1-4 (roster de agentes implementado).

---

## 5.1 El Workflow Feature Lifecycle

Este es el pipeline maestro del curso. Conecta los 7 agentes del roster en un flujo secuencial donde cada fase produce el input de la siguiente.

``` ascii
  ┌────────────────────────────────────────────────────────┐
  │          FEATURE LIFECYCLE PIPELINE                    │
  │                                                        │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐            │
  │  │ FASE 1   │──►│ FASE 2   │──►│ FASE 3   │            │
  │  │ Spec     │   │ Diseño   │   │ Implement│            │
  │  │          │   │          │   │          │            │
  │  │@spec-    │   │@architect│   │@developer│            │
  │  │ writer   │   │          │   │          │            │
  │  │          │   │          │   │          │            │
  │  │Output:   │   │Output:   │   │Output:   │            │
  │  │spec.md   │   │design.md │   │código    │            │
  │  └──────────┘   └──────────┘   └────┬─────┘            │
  │                                     │                  │
  │                   ┌─────────────────┼──────────┐       │
  │                   ▼                 ▼          ▼       │
  │             ┌──────────┐   ┌──────────┐   ┌─────────┐  │
  │             │ FASE 4   │   │ FASE 5   │   │ FASE 6  │  │
  │             │ Testing  │   │ Security │   │ Review  │  │
  │             │          │   │          │   │ + Docs  │  │
  │             │@qa-      │   │@security-│   │@code-   │  │
  │             │ engineer │   │ analyst  │   │ reviewer│  │
  │             │          │   │          │   │@doc-    │  │
  │             │          │   │          │   │ writer  │  │
  │             └────┬─────┘   └────┬─────┘   └────┬────┘  │
  │                  │              │               │      │
  │                  └──────────────┼───────────────┘      │
  │                                 ▼                      │
  │                         ┌──────────────┐               │
  │                         │   FASE 7     │               │
  │                         │   PR + Merge │               │
  │                         │              │               │
  │                         │ Git MCP +    │               │
  │                         │ PR skill     │               │
  │                         └──────────────┘               │
  └────────────────────────────────────────────────────────┘
```

    {{1}}
**Observa que las Fases 4, 5 y 6 pueden correr en paralelo.** Testing, security review y code review + docs son independientes entre sí. Esto es exactamente donde los subagentes paralelos o un Agent Team brillan.

### Implementación como Slash Command

El pipeline completo se puede encapsular en un slash command que orqueste todas las fases:

``` markdown
<!-- .claude/commands/feature-pipeline.md -->

## Feature Pipeline: $ARGUMENTS

Execute the full feature lifecycle for the requirement
described below. Follow each phase in order, using the
specialized agents.

### Phase 1: Specification
Use @spec-writer to create a detailed spec from the
requirement. Save to docs/specs/$ARGUMENTS.md

### Phase 2: Architecture
Use @architect to design the technical solution based
on the spec. Save to docs/design/$ARGUMENTS.md

### Phase 3: Implementation
Use @developer to implement the feature following the
spec and design. Use TDD: tests first, then code.

### Phase 4-6: Parallel Validation
Run these three in parallel:
- @qa-engineer: Generate additional tests, check coverage
- @security-analyst: Audit new code for vulnerabilities
- @code-reviewer: Final quality review
- @doc-writer: Update API docs and README

### Phase 7: PR
After all validations pass, create a PR with a structured
description including Summary, Changes, Test Plan, and
Security Considerations.

### Quality Gates
- ALL tests must pass before Phase 7
- NO CRITICAL or HIGH security findings allowed
- Code review must return APPROVE verdict
- If any gate fails, fix and re-validate
```

Para ejecutarlo:

``` text
> /project:feature-pipeline auth-refresh-token
```

    {{1}}
Claude orquesta todo el pipeline, invocando cada agente en secuencia (y en paralelo donde es posible), gestionando las dependencias entre fases y aplicando los quality gates.

#### Quiz: Feature Lifecycle

En el pipeline, ¿por qué las fases 4, 5 y 6 pueden ejecutarse en paralelo?

- [( )] Porque usan el mismo agente
- [( )] Porque no producen output
- [(X)] Porque son independientes entre sí: testing, security y review no necesitan el resultado de las otras para trabajar
- [( )] Porque Agent Teams las paraleliza automáticamente
***
**Correcto.** Testing (Fase 4), security review (Fase 5) y code review + docs (Fase 6) operan sobre el mismo código de la Fase 3, pero no dependen del resultado de las demás. Esto las hace candidatas perfectas para ejecución paralela con subagentes.
***

### Implementación como Skill (Auto-activación)

Para equipos que quieren que el pipeline se sugiera automáticamente cuando alguien inicia una nueva feature:

``` markdown
<!-- .claude/skills/feature-pipeline/SKILL.md -->
---
name: feature-pipeline
description: Orchestrates the full feature lifecycle from
  specification to PR. Activates when the user mentions
  "new feature", "implement feature", "feature pipeline",
  or provides a requirement that needs end-to-end development.
---

[Mismo contenido que el slash command anterior]
```

    {{1}}
La ventaja de la skill sobre el slash command: Claude detecta cuándo necesitas el pipeline y lo sugiere o activa automáticamente, sin que tengas que recordar el nombre del comando.

---

## 5.2 Git Workflows con Claude Code

Claude Code tiene una integración profunda con Git que va mucho más allá de simples commits. Entender estos mecanismos desbloquea workflows que no son posibles con otros agentes.

### Commits con Conventional Commits

``` text
> Commitea los cambios con un mensaje descriptivo

Claude ejecuta:
  git add -A
  git commit -m "feat(auth): add refresh token rotation with 24h expiry

  - Implement token rotation on each refresh
  - Store refresh tokens as HttpOnly cookies
  - Add rate limiting on refresh endpoint (10/min)
  - Include migration for refresh_tokens table

  Closes #347"
```

Para estandarizar esto como slash command:

``` markdown
<!-- .claude/commands/commit.md -->

Stage all changes and create a commit with Conventional
Commits format. Analyze the diff to determine:

- Type: feat|fix|docs|style|refactor|perf|test|chore
- Scope: affected module in parentheses
- Subject: imperative, lowercase, no period
- Body: bullet list of what changed and why
- Footer: reference any related issues

DO NOT commit without showing me the message first.
```

### Creación y Gestión de PRs

Con MCP de GitHub conectado, Claude puede gestionar el ciclo completo:

``` text
> Crea un PR para esta feature branch

Claude ejecuta:
  1. Lee git log para entender los cambios
  2. Genera descripción estructurada con la skill pr-description
  3. Crea el PR vía MCP de GitHub
  4. Asigna reviewers si están configurados
```

Una skill de PR bien diseñada:

``` markdown
<!-- .claude/skills/pr-description/SKILL.md -->
---
name: pr-description
description: Generates structured PR descriptions from
  git log. Activates when creating pull requests or when
  the user asks to prepare a PR.
---

When creating a PR description, follow this structure:

## Summary
One paragraph explaining WHAT changed and WHY.

## Changes
Bullet list of specific changes grouped by area:
- **Backend**: ...
- **Frontend**: ...
- **Database**: ...
- **Config**: ...

## Test Plan
How to verify these changes work:
1. Unit tests: `npm test`
2. Manual verification steps
3. Edge cases to check

## Security Considerations
- Authentication impact: yes/no
- New dependencies added: list
- Secrets/config changes: list

## Related
- Spec: docs/specs/[feature].md
- Design: docs/design/[feature].md
- Issue: #NNN
```

### Checkpoints Automáticos

Claude Code crea un checkpoint en cada prompt del usuario, lo que te permite "viajar en el tiempo":

``` ascii
  Prompt 1 ──► [Checkpoint 1] ──► Respuesta de Claude
  Prompt 2 ──► [Checkpoint 2] ──► Respuesta de Claude
  Prompt 3 ──► [Checkpoint 3] ──► Respuesta de Claude
                     ▲
                     │
              Puedes volver aquí
```

    {{1}}
**Tres opciones de rewind:**

| Opción | Qué deshace | Qué mantiene |
|:-------|:------------|:-------------|
| Solo conversación | Historial de chat | Cambios en código |
| Solo código | Cambios en archivos | Historial de chat |
| Ambos | Todo | Nada (vuelves al estado del checkpoint) |

    {{2}}
> **Consejo profesional:** Usa Git como complemento para historial real. Los checkpoints de Claude Code son útiles para errores inmediatos, pero Git proporciona historial persistente y compartido.

### Git Worktrees para Aislamiento

Los worktrees de Git permiten ejecutar múltiples sesiones de Claude Code en paralelo, cada una con su propia copia de trabajo:

``` bash
# Crear un worktree para una feature
git worktree add ../mi-proyecto-feature-auth feature/auth

# Abrir Claude Code en el worktree
cd ../mi-proyecto-feature-auth
claude

# Mientras tanto, en el directorio principal, otra sesión:
cd ../mi-proyecto
claude
```

    {{1}}
Cada sesión opera sobre su propia copia del código. No hay conflictos de archivos entre sesiones. Al terminar, merge el worktree de vuelta y elimínalo:

``` bash
cd ../mi-proyecto
git merge feature/auth
git worktree remove ../mi-proyecto-feature-auth
```

#### Quiz: Git workflows

¿Cuál es la ventaja principal de usar git worktrees con Claude Code?

- [( )] Permite usar diferentes modelos en cada sesión
- [( )] Comparte el contexto entre sesiones
- [(X)] Permite ejecutar múltiples sesiones en paralelo sin conflictos de archivos
- [( )] Reduce el consumo de tokens
***
**Correcto.** Cada worktree es una copia independiente del código en su propia carpeta. Puedes tener una sesión de Claude Code implementando una feature mientras otra sesión depura un bug, sin que se pisen. Al terminar, mergeas como cualquier branch.
***

---

## 5.3 Modo Headless y CI/CD

El modo headless (`claude -p`) transforma Claude Code de herramienta interactiva a componente de un pipeline automatizado. Este es el puente entre desarrollo local y CI/CD.

### Uso Básico

``` bash
# Ejecutar un prompt y terminar
claude -p "list all TODO comments in the codebase"

# Pipe de input
git diff main | claude -p "review for security issues"

# Output estructurado para scripts
claude -p "list all exported functions in src/" \
  --output-format json

# Con schema de validación
claude -p "extract function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}}}'
```

### Controles de Seguridad y Coste

| Flag | Función | Ejemplo |
|:-----|:--------|:--------|
| `--max-budget-usd` | Límite de gasto | `--max-budget-usd 5.00` |
| `--max-turns` | Límite de turnos | `--max-turns 3` |
| `--allowedTools` | Herramientas permitidas | `--allowedTools "Read,Write,Bash(git *)"` |
| `--disallowedTools` | Herramientas bloqueadas | `--disallowedTools "Bash(rm:*),Bash(sudo:*)"` |
| `--fallback-model` | Modelo alternativo si el principal falla | `--fallback-model haiku` |
| `--strict-mcp-config` | Solo MCP servers de la config dada | Ver Módulo 3 |

### Pipeline de CI/CD Ejemplo

Un workflow de GitHub Actions que usa Claude Code para review automático:

``` yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  security-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Security Review
        run: |
          git diff origin/main...HEAD | claude -p \
            "Review this diff for security vulnerabilities.
             Focus on OWASP Top 10. Output as JSON with
             fields: severity, file, line, description, fix." \
            --output-format json \
            --max-budget-usd 2.00 \
            --max-turns 3 \
            --allowedTools "Read,Grep,Glob" \
            > security-report.json
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

      - name: Check for Critical Findings
        run: |
          CRITICAL=$(jq '[.[] | select(.severity=="CRITICAL")] | length' security-report.json)
          if [ "$CRITICAL" -gt 0 ]; then
            echo "::error::Found $CRITICAL critical security issues"
            cat security-report.json | jq '.[] | select(.severity=="CRITICAL")'
            exit 1
          fi

  test-coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Generate Missing Tests
        run: |
          claude -p \
            "Analyze test coverage for src/. Identify untested
             functions and generate tests for them. Run the
             full suite to verify." \
            --max-budget-usd 5.00 \
            --allowedTools "Read,Write,Bash(npm test *),Grep,Glob"
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

    {{1}}
**Observa los controles de seguridad:**

- `--max-budget-usd 2.00` limita el gasto por ejecución
- `--max-turns 3` limita los turnos para que no entre en loops
- `--allowedTools` restringe a solo lectura en el job de security
- El job de tests permite `Write` y `Bash(npm test *)` pero nada más

    {{2}}
> **Trampa CI/CD:** Recuerda del Módulo 3 que los hooks `PermissionRequest` no se disparan en modo headless. Si tu pipeline se queda colgado, probablemente está esperando una aprobación interactiva que nunca llegará. Usa `--allowedTools` para pre-aprobar las herramientas necesarias.

#### Quiz: CI/CD

Tu pipeline de CI tiene un job que ejecuta `claude -p "fix all lint errors"`. El job lleva 30 minutos y no termina. ¿Cuál es la causa más probable?

- [( )] Claude Code no funciona en containers de CI
- [( )] El modelo es demasiado lento
- [(X)] No tiene `--max-turns` ni `--max-budget-usd`, y Claude está en un loop intentando fixes que generan nuevos errores
- [( )] Falta el flag `--headless`
***
**Correcto.** Sin límites de turnos o presupuesto, Claude puede entrar en un ciclo: corrige un error de lint, introduce otro, intenta corregirlo, introduce uno nuevo... Los flags `--max-turns` y `--max-budget-usd` son obligatorios en CI/CD para prevenir loops infinitos y costes descontrolados.
***

---

## 5.4 Loop Continuo

La funcionalidad `/loop` permite ejecutar tareas periódicamente dentro de una sesión:

``` text
# Cada 5 minutos, verifica si el deploy completó
> /loop 5m check if the deploy on staging is complete

# Ejecuta un slash command cada 10 minutos (intervalo por defecto)
> /loop /test

# Cada 2 minutos, ejecuta el security review
> /loop 2m /project:review/security src/
```

    {{1}}
**Aislamiento de contexto:** Cada iteración del loop tiene su propio contexto. Las iteraciones no inflan la conversación, lo que hace viable loops de larga duración.

    {{2}}
**Casos de uso:**

- Monitorización de deploys en staging
- Ejecución periódica de tests durante refactoring
- Verificación de estado de servicios externos
- Polling de resultados de procesos largos

---

## 5.5 Workflow Completo: De Idea a Deploy

Combinemos todo en un ejemplo realista. Imagina que recibes un requisito de producto:

> "Los tokens de refresh deben rotar automáticamente en cada uso, expirar a las 24h, y almacenarse como HttpOnly cookies."

### Paso 1: Arranque de Sesión

``` text
$ claude
> Hoy implementamos refresh token rotation. El requisito es:
  "Los tokens de refresh deben rotar automáticamente en cada
  uso, expirar a las 24h, y almacenarse como HttpOnly cookies."
  Ejecuta el pipeline completo.
```

### Paso 2: Spec (Claude invoca @spec-writer)

``` text
  @spec-writer analiza el requisito y genera:
  → docs/specs/auth-refresh-token.md

  Contenido:
  - User stories con criterios de aceptación
  - Edge cases: token expirado, token reusado, concurrent refresh
  - Dependencias: tabla refresh_tokens, middleware de cookies
```

### Paso 3: Diseño (Claude invoca @architect)

``` text
  @architect lee la spec y genera:
  → docs/design/auth-refresh-token.md

  Contenido:
  - ADR: "Usar rotación con familia de tokens"
  - Diagrama de secuencia del flujo de refresh
  - Cambios en la base de datos (migración)
  - Interfaces de API modificadas
```

### Paso 4: Implementación (Claude invoca @developer)

``` text
  @developer implementa siguiendo spec + design:
  1. Crea migración para tabla refresh_tokens
  2. Implementa RefreshTokenService
  3. Modifica AuthController.refresh()
  4. Configura HttpOnly cookies en middleware
  5. Escribe tests unitarios (TDD)
  6. Ejecuta tests → PASS

  Hook PostToolUse → prettier formatea cada archivo escrito
  Hook PostToolUse → jest ejecuta tests tras cada .test.ts
```

### Paso 5: Validación Paralela

``` text
  Claude lanza 3 subagentes en paralelo:

  @qa-engineer:
  - Genera tests de integración
  - Test de concurrent refresh race condition
  - Test de token reusado (debe fallar)
  - Cobertura: 94%

  @security-analyst:
  - Verifica HttpOnly + Secure + SameSite flags ✓
  - Verifica rate limiting en /refresh ✓
  - Detecta: falta logging de eventos de rotación → MEDIUM
  - No hay CRITICAL ni HIGH → PASS

  @code-reviewer + @doc-writer:
  - Review: APPROVE con 2 sugerencias MEDIUM
  - Documentación de API actualizada
  - README actualizado con nueva sección de auth
```

### Paso 6: PR

``` text
  Claude crea PR vía GitHub MCP:

  Title: feat(auth): implement refresh token rotation

  ## Summary
  Implements automatic refresh token rotation with 24h expiry
  stored as HttpOnly cookies.

  ## Changes
  - **Backend**: RefreshTokenService, token rotation logic
  - **Database**: Migration for refresh_tokens table
  - **API**: Modified POST /auth/refresh response
  - **Security**: HttpOnly + Secure + SameSite cookie config

  ## Test Plan
  - Unit: 12 tests (all passing)
  - Integration: 5 tests (concurrent refresh, reuse detection)
  - Coverage: 94%

  ## Security Considerations
  - HttpOnly cookies prevent XSS token theft
  - Token family tracking detects reuse attacks
  - Rate limiting: 10 refreshes/min per user
  - MEDIUM: Add audit logging for rotation events

  Closes #347
```

### Paso 7: Cierre de Sesión

``` text
> ¿Cuáles fueron las decisiones clave de hoy?

Claude resume:
- Rotación con familia de tokens (no single-use)
- HttpOnly + Secure + SameSite=Strict
- Expiración 24h, rate limit 10/min
- Pendiente: logging de auditoría (MEDIUM)

> Añade estas decisiones a CLAUDE.md en la sección de arquitectura
```

    {{1}}
> **Tiempo total:** Aproximadamente 45-90 minutos para un pipeline que, manualmente, tomaría un día completo de un desarrollador senior. Y con quality gates integradas que no dependen de la disciplina humana.

---

## 5.6 Patrones de Workflow Avanzados

### Patrón 1: Feature Branch Aislada

``` bash
# Crear worktree + sesión dedicada
git worktree add ../feature-auth feature/auth-refresh
cd ../feature-auth
claude

# Trabajar en la feature...
# Al terminar:
cd ../mi-proyecto
git merge feature/auth-refresh
git worktree remove ../feature-auth
```

### Patrón 2: Hotfix Urgente

``` text
> /clear
> Bug urgente en producción: los usuarios no pueden hacer login
> después de las 00:00 UTC. Diagnostica el problema en src/auth/
> y propón un fix mínimo.

Claude:
  1. Lee los archivos de auth
  2. Identifica el bug (timezone en expiración)
  3. Propone fix de 1 línea
  4. Genera test que reproduce el bug
  5. Ejecuta test → falla (confirma el bug)
  6. Aplica fix
  7. Ejecuta test → pasa (confirma el fix)
  8. Ejecuta suite completa → todos pasan
```

### Patrón 3: Refactoring Asistido

``` text
> Necesito refactorizar el módulo de pagos de Express a Fastify.
> Antes de cambiar nada, analiza el impacto:
> - Archivos afectados
> - Tests que se romperán
> - Dependencias que cambiarán
> Plan primero, ejecuta después.

Claude con @architect:
  → docs/design/payment-migration.md
  → Plan de 12 pasos con estimación de complejidad
  → 3 pasos paralelizables identificados

> Ejecuta el plan paso a paso. Espera mi aprobación entre pasos.
```

#### Quiz: Patrones de workflow

Recibes un bug report urgente en producción a las 3 AM. Necesitas un fix rápido y seguro. ¿Cuál es la secuencia correcta de acciones en Claude Code?

- [( )] Implementar el fix directamente, commitear y desplegar
- [( )] Crear un Agent Team de 5 agentes para investigar
- [(X)] `/clear` → diagnosticar → proponer fix mínimo → escribir test que reproduzca el bug → aplicar fix → verificar que pasa → ejecutar suite completa
- [( )] Crear un slash command para hotfixes y configurar hooks
***
**Correcto.** Para hotfixes, el patrón es: limpiar contexto (`/clear` para foco), diagnosticar, fix mínimo y TDD (test que reproduce → fix → test pasa → suite completa pasa). Agent Teams y configuraciones elaboradas no se justifican para una urgencia — necesitas velocidad con seguridad.
***

---

## 5.7 Resumen del Módulo

En este módulo has aprendido:

- [X] El pipeline Feature Lifecycle de 7 fases con agentes especializados
- [X] Implementación como slash command y como skill auto-activable
- [X] Git workflows: commits convencionales, PRs estructurados, checkpoints
- [X] Git worktrees para sesiones paralelas sin conflictos
- [X] Modo headless con controles de seguridad y coste para CI/CD
- [X] Pipeline de GitHub Actions con Claude Code integrado
- [X] Loop continuo para monitorización periódica
- [X] Walkthrough completo de idea a PR
- [X] Patrones avanzados: feature branch aislada, hotfix urgente, refactoring asistido

---

## Evaluación Final del Módulo 5

**Pregunta 1:** En el Feature Lifecycle Pipeline, ¿qué fases pueden ejecutarse en paralelo y por qué?

- [( )] Fases 1 y 2 (spec y diseño), porque no dependen entre sí
- [( )] Fases 3, 4 y 5, porque son las más rápidas
- [(X)] Fases 4, 5 y 6 (testing, security, review+docs), porque operan sobre el mismo código sin depender del resultado de las otras
- [( )] Todas las fases pueden correr en paralelo
***
**Correcto.** Las fases 1→2→3 son secuenciales (cada una necesita el output de la anterior). Pero testing (4), security review (5) y code review + docs (6) operan todas sobre el código de la fase 3 sin necesitar los resultados de las demás.
***

**Pregunta 2:** ¿Qué dos flags son OBLIGATORIOS en cualquier ejecución de Claude Code en CI/CD para prevenir loops infinitos y costes descontrolados?

[[--max-turns y --max-budget-usd]]
[[?]] Son dos flags de control, una limita iteraciones y otra limita dinero
[[?]] El formato es --max-algo
***
**Correcto.** `--max-turns` previene loops infinitos (Claude intenta corregir un error, introduce otro, repite) y `--max-budget-usd` pone un techo al gasto. Sin ambos, un pipeline puede correr indefinidamente acumulando costes.
***

**Pregunta 3:** Un pipeline de CI ejecuta `claude -p "fix all lint errors"` pero se queda colgado indefinidamente. Identifica las TRES causas posibles:

- [[X]] Falta `--max-turns` para limitar iteraciones
- [[ ]] Claude Code no es compatible con CI
- [[X]] Falta `--max-budget-usd` para limitar coste
- [[X]] Puede haber un hook PermissionRequest que espera aprobación interactiva
- [[ ]] El modelo Opus es necesario para CI/CD
***
**Correcto.** Las tres causas: sin `--max-turns` Claude puede entrar en loop; sin `--max-budget-usd` el gasto es ilimitado; y un hook `PermissionRequest` en modo headless se queda esperando para siempre. Recuerda: usa `PreToolUse` para permisos automáticos en CI.
***

**Pregunta 4:** ¿Cuál es el propósito de los git worktrees en el contexto de Claude Code?

- [( )] Compartir contexto entre sesiones de Claude Code
- [( )] Reducir el consumo de tokens
- [(X)] Permitir ejecutar múltiples sesiones de Claude Code en paralelo, cada una con su propia copia del código sin conflictos de archivos
- [( )] Crear backups automáticos del código
***
**Correcto.** Cada worktree es una copia independiente del repositorio en su propia carpeta. Puedes tener una sesión trabajando en una feature mientras otra sesión depura un bug, sin que los cambios de archivos de una interfieran con la otra.
***

**Pregunta 5:** Ordena las 7 fases del Feature Lifecycle Pipeline en el orden correcto:

- [[1]] Especificación (@spec-writer)
- [[2]] Diseño y arquitectura (@architect)
- [[3]] Implementación (@developer)
- [[4]] Testing (@qa-engineer) — paralelo con 5 y 6
- [[5]] Security review (@security-analyst) — paralelo con 4 y 6
- [[6]] Code review + documentación (@code-reviewer + @doc-writer) — paralelo con 4 y 5
- [[7]] PR y merge (GitHub MCP + PR skill)

---

## Siguiente: Módulo 6 — Optimización de Tokens

En el próximo módulo aprenderás:

- Estrategias de reducción de consumo con impacto medible
- Selección de modelo por tarea para minimizar costes
- Monitorización y control de gasto en tiempo real
- Progressive disclosure, MCP lazy loading y CLAUDE.md lean
- El cálculo económico: cuánto cuesta un pipeline profesional

> **Tarea antes del Módulo 6:** Ejecuta `/cost` al final de tus próximas 5 sesiones y anota los tokens consumidos. Identifica qué operación consumió más tokens en cada sesión (pista: probablemente fue la lectura de archivos grandes). En el Módulo 6 usaremos esos datos para optimizar.
