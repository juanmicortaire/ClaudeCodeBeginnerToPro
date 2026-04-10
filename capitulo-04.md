<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 4 — Agentes Especializados.

-->

# Módulo 4: Agentes Especializados

> **"Un solo agente que lo hace todo es como un desarrollador junior que intenta ser fullstack, devops, QA y security analyst a la vez. Funciona para tareas simples, pero para trabajo profesional necesitas un equipo de especialistas."**

Este módulo es el corazón del curso. Aquí es donde todo lo anterior —memoria, skills, hooks, MCP— converge para crear un equipo de agentes IA especializados que colaboran en el ciclo de vida completo del desarrollo.

    {{1}}
**Objetivo del módulo:** Dominar subagentes, custom agents y Agent Teams. Diseñar e implementar un roster de agentes especializados que cubran desde la especificación de features hasta la revisión de seguridad.

    {{2}}
**Tiempo estimado:** 90-120 minutos

    {{3}}
**Prerrequisito:** Haber completado los Módulos 1-3 (CLAUDE.md, skills, hooks y MCP configurados).

---

## 4.1 Los Tres Niveles de Delegación

Claude Code ofrece tres mecanismos de delegación, cada uno con diferente nivel de autonomía y coordinación:

``` ascii
  Complejidad / Autonomía ──────────────────────────────►

  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
  │  SUBAGENTES  │   │   CUSTOM     │   │   AGENT      │
  │              │   │   AGENTS     │   │   TEAMS      │
  │ Claude lanza │   │ Agentes con  │   │ Múltiples    │
  │ workers ad-  │   │ personalidad │   │ instancias   │
  │ hoc para     │   │ propia:      │   │ que hablan   │
  │ tareas       │   │ system prompt│   │ ENTRE SÍ     │
  │ paralelas    │   │ tools propios│   │              │
  │              │   │ modelo propio│   │ Team lead +  │
  │ Reportan al  │   │              │   │ teammates    │
  │ agente       │   │ Invocación   │   │              │
  │ principal    │   │ con @mención │   │ Comunicación │
  │              │   │              │   │ peer-to-peer │
  └──────────────┘   └──────────────┘   └──────────────┘
        │                   │                   │
        ▼                   ▼                   ▼
    1-5 tokens          Coste medio         Coste alto
    por tarea           por agente          (experimental)
```

| Característica | Subagentes | Custom Agents | Agent Teams |
|:---------------|:----------:|:-------------:|:-----------:|
| Quién los crea | Claude decide solo | Tú los defines | Tú los orquestas |
| Comunicación | Solo con el padre | Solo con el padre | Entre sí (peer-to-peer) |
| Contexto | Aislado, fresco | Aislado, con system prompt | Aislado, 1M tokens c/u |
| Modelo | Mismo que el padre | Configurable por agente | Opus 4.6 (requerido) |
| Persistencia | Solo durante la tarea | Definición en archivo `.md` | Solo durante la sesión |
| Mejor para | Tareas paralelas ad-hoc | Roles recurrentes | Investigación colaborativa |

#### Quiz: Niveles de delegación

Necesitas que dos investigadores analicen un problema desde ángulos diferentes, compartan hallazgos entre sí y lleguen a un consenso. ¿Qué mecanismo usas?

- [( )] Subagentes, porque pueden trabajar en paralelo
- [( )] Custom Agents, porque tienen personalidad propia
- [(X)] Agent Teams, porque permiten comunicación peer-to-peer entre teammates
- [( )] Cualquiera de los tres funciona igual
***
**Correcto.** Solo Agent Teams permite que los agentes se comuniquen directamente entre sí. Los subagentes y custom agents solo pueden reportar al agente padre, no hablar entre ellos. Para consenso y debate, necesitas comunicación peer-to-peer.
***

---

## 4.2 Subagentes — Delegación Ad-Hoc

Los subagentes son instancias de Claude que el agente principal lanza sobre la marcha para tareas específicas. No los defines tú: Claude decide cuándo crearlos basándose en la naturaleza de la tarea.

### Cuándo Claude Crea Subagentes

    {{1}}
**Tareas paralelas independientes:**
"Escribe tests mientras actualizas la documentación" — Claude lanza un subagente para cada tarea y las ejecuta simultáneamente.

    {{2}}
**Trabajo especializado:**
"Haz una auditoría de seguridad de este módulo" — Claude puede lanzar un subagente con un system prompt enfocado en seguridad.

    {{3}}
**Exploración aislada:**
"Investiga si podemos migrar de Express a Fastify" — Claude lanza un subagente para explorar sin contaminar el contexto principal con archivos y dependencias irrelevantes.

    {{4}}
**Tareas largas en background:**
"Ejecuta los tests de integración en background mientras seguimos trabajando" — El subagente corre asíncronamente.

### Cuándo NO Crear Subagentes

Claude no crea subagentes para tareas simples, lecturas de archivos pequeñas, o cualquier cosa que pueda completar directamente en pocas llamadas de herramientas. El overhead de crear un subagente solo se justifica cuando la tarea es suficientemente compleja o parallelizable.

### Foreground vs. Background

``` ascii
  FOREGROUND (por defecto)           BACKGROUND
  ─────────────────────              ──────────
  Claude lanza subagente             Claude lanza subagente
         │                                  │
         ▼                                  ▼
  Claude ESPERA                      Claude CONTINÚA trabajando
         │                                  │
         ▼                                  │
  Recibe resultado                   ┌──────┴───────┐
         │                           │ Notificación │
         ▼                           │ al completar │
  Continúa con el resultado          └──────────────┘

  Usar cuando:                       Usar cuando:
  - Necesitas el resultado           - La tarea es larga
    para continuar                   - Puedes seguir sin
  - Es una tarea corta                 el resultado
```

### Ejecución Paralela Explícita

Puedes pedir explícitamente ejecución paralela:

``` text
> Ejecuta el linter y la suite de tests en paralelo

> Usa agentes separados para investigar cómo estas tres
  librerías manejan el problema, y resume los hallazgos

> Lanza un agente para revisar las implicaciones de seguridad
  de este cambio mientras tú sigues implementando la feature
```

    {{1}}
> **Clave:** Cuando pides ejecución "en paralelo", Claude envía múltiples llamadas Agent simultáneamente. Sé explícito sobre qué debe correr en paralelo y qué debe ser secuencial.

### Contexto Aislado

    {{1}}
Cada subagente arranca con contexto limpio. **No hereda el historial de conversación del padre.** El padre debe proporcionar toda la información necesaria en el prompt del subagente.

    {{2}}
Esto es una feature, no un bug: el aislamiento previene que el subagente se confunda con contexto irrelevante y mantiene su ventana de contexto limpia para la tarea específica.

#### Quiz: Subagentes

¿Cuál de estas tareas NO justifica crear un subagente?

- [( )] Investigar tres alternativas de librería en paralelo
- [( )] Ejecutar la suite de integración tests en background
- [(X)] Leer un archivo de configuración de 20 líneas
- [( )] Hacer una auditoría de seguridad aislada del contexto principal
***
**Correcto.** Leer un archivo pequeño es una operación trivial que Claude puede hacer directamente en una llamada de herramienta. El overhead de crear un subagente (nuevo contexto, nuevo system prompt, comunicación de ida y vuelta) no se justifica para algo tan simple.
***

---

## 4.3 Custom Agents — Tu Equipo de Especialistas

Los custom agents son el salto cualitativo: agentes con personalidad propia, system prompt dedicado, herramientas restringidas y potencialmente un modelo diferente. Los defines tú y persisten como archivos del proyecto.

### Anatomía de un Custom Agent

Un custom agent es un archivo `.md` en `.claude/agents/` (proyecto) o `~/.claude/agents/` (global):

``` markdown
<!-- .claude/agents/code-reviewer.md -->
---
name: code-reviewer
description: Reviews code for bugs, security issues,
  and style violations. Best for PR reviews and
  code quality audits.
tools: read, grep, glob
model: claude-haiku-4-5-20251001
---

You are a senior code reviewer. Your job is to find
issues, not to write code. You NEVER modify files.

## Review Process

1. Read the files or diff provided
2. Check for:
   - Logic errors and edge cases
   - Security vulnerabilities (OWASP Top 10)
   - Performance issues (N+1, memory leaks)
   - Style violations per project CLAUDE.md
3. Rate each finding: CRITICAL | HIGH | MEDIUM | LOW
4. Provide fix suggestions as code snippets (but do NOT apply them)

## Output Format

### Summary
One paragraph executive summary.

### Findings
For each issue:
- **[SEVERITY]** file:line — description
- Suggested fix: `code snippet`

### Verdict
APPROVE | REQUEST_CHANGES | NEEDS_DISCUSSION
```

    {{1}}
**Observa los detalles clave:**

- **`tools: read, grep, glob`** — Solo herramientas de lectura. Este agente no puede escribir ni ejecutar comandos. Seguridad por diseño.
- **`model: claude-haiku-4-5-20251001`** — Usa Haiku, el modelo más barato. Para solo leer y analizar, no necesita Opus.
- **El system prompt** define la personalidad: "Tu trabajo es encontrar problemas, no escribir código. NUNCA modificas archivos."

### Invocación con @mención

Desde abril de 2026, puedes invocar agents con typeahead:

``` text
> @code-reviewer revisa los cambios en src/auth/

> @doc-writer genera la documentación de la API de pagos

> @security-analyst audita el módulo de autenticación
```

Claude también puede delegar automáticamente a un custom agent si la descripción del agente encaja con la tarea.

### Prioridad de Resolución

Si existe un agente con el mismo nombre en ambas ubicaciones:

``` text
  .claude/agents/code-reviewer.md    ← GANA (proyecto)
  ~/.claude/agents/code-reviewer.md  ← Pierde (global)
```

El agente de proyecto siempre tiene prioridad sobre el global.

---

## 4.4 Diseño del Roster: Tu Equipo Completo

Esta es la sección central del módulo. Vamos a diseñar un equipo completo de agentes especializados que cubra el ciclo de vida del desarrollo.

``` ascii
  ┌─────────────────────────────────────────────────────┐
  │              CICLO DE VIDA DE UNA FEATURE           │
  ├─────────────────────────────────────────────────────┤
  │                                                     │
  │  Requisito ──► @spec-writer ──► Especificación      │
  │                     │                               │
  │                     ▼                               │
  │              @architect ──► Diseño técnico          │
  │                     │                               │
  │                     ▼                               │
  │              @developer ──► Implementación          │
  │                     │                               │
  │                     ▼                               │
  │              @qa-engineer ──► Tests                 │
  │                     │                               │
  │                     ▼                               │
  │           @security-analyst ──► Security review     │
  │                     │                               │
  │                     ▼                               │
  │           @code-reviewer ──► Code review final      │
  │                     │                               │
  │                     ▼                               │
  │            @doc-writer ──► Documentación            │
  │                     │                               │
  │                     ▼                               │
  │                  PR + Deploy                        │
  └─────────────────────────────────────────────────────┘
```

### Agente 1: Spec Writer

``` markdown
<!-- .claude/agents/spec-writer.md -->
---
name: spec-writer
description: Transforms high-level requirements into
  detailed technical specifications with user stories,
  acceptance criteria, and edge cases. Use when starting
  a new feature or refining requirements.
tools: read, grep, glob
model: claude-sonnet-4-6-20250514
---

You are a senior product engineer who bridges business
requirements and technical implementation.

## Process

1. Read the requirement provided by the user
2. Analyze the existing codebase for context
3. Produce a specification with:

### Output Structure

#### User Stories
- As a [role], I want [action], so that [benefit]
- Include primary flow AND alternative flows

#### Acceptance Criteria
- Given/When/Then format
- Cover happy path, edge cases, error scenarios

#### Technical Notes
- Affected components and interfaces
- Data model changes needed
- API contract changes
- Migration requirements

#### Open Questions
- List ambiguities that need product decision

#### Dependencies
- External services, other features, or data sources

Save the spec as docs/specs/[feature-name].md
```

### Agente 2: Architect

``` markdown
<!-- .claude/agents/architect.md -->
---
name: architect
description: Designs technical solutions from specifications.
  Proposes component architecture, interfaces, patterns,
  and produces Architecture Decision Records (ADRs).
  Use for system design, refactoring plans, or technical decisions.
tools: read, grep, glob, write
model: claude-sonnet-4-6-20250514
---

You are a senior software architect. You design systems
that are maintainable, scalable, and aligned with existing
project patterns.

## Process

1. Read the specification from docs/specs/
2. Analyze existing architecture (@docs/architecture.md)
3. Propose design that respects existing patterns

## Output Structure

### Component Design
- New components with responsibilities
- Modified interfaces
- Sequence diagrams (Mermaid format)

### ADR (Architecture Decision Record)
- Title: ADR-NNN: [Decision]
- Status: Proposed
- Context: Why this decision is needed
- Decision: What we decided
- Alternatives: What we considered and rejected
- Consequences: Trade-offs accepted

### Implementation Plan
- Ordered list of steps
- Estimated complexity per step (S/M/L)
- Parallelizable steps identified

Save as docs/design/[feature-name].md
```

### Agente 3: Developer

``` markdown
<!-- .claude/agents/developer.md -->
---
name: developer
description: Implements features following specifications
  and architecture designs. Writes production code that
  follows project conventions. Use for coding tasks.
tools: read, write, bash, grep, glob
model: claude-sonnet-4-6-20250514
---

You are a senior developer. You write clean, tested,
production-ready code.

## Rules

1. ALWAYS read the spec and design docs before coding
2. Follow conventions in CLAUDE.md strictly
3. Write tests FIRST (TDD), then implementation
4. Run tests after every significant change
5. Never commit without explicit user approval
6. Keep changes minimal and focused
7. If something is unclear, ask before implementing
```

### Agente 4: QA Engineer

``` markdown
<!-- .claude/agents/qa-engineer.md -->
---
name: qa-engineer
description: Generates and executes comprehensive tests.
  Covers unit, integration, and e2e tests. Analyzes
  coverage and identifies untested edge cases. Use after
  implementation for quality verification.
tools: read, write, bash, grep, glob
model: claude-sonnet-4-6-20250514
---

You are a senior QA engineer obsessed with edge cases.

## Process

1. Read the acceptance criteria from the spec
2. Read the implementation code
3. Design test plan covering:
   - Happy path for each acceptance criterion
   - Edge cases (null, empty, boundary values, overflow)
   - Error handling (network failures, invalid input, timeouts)
   - Concurrency issues if applicable
4. Implement tests using the project's test framework
5. Execute full suite and report results

## Output Format

### Test Plan Summary
- Total tests: N
- By type: unit (X) | integration (Y) | e2e (Z)

### Coverage Analysis
- Lines covered: X%
- Branches covered: Y%
- Untested paths identified

### Edge Cases Found
- List of scenarios NOT in the original spec
```

### Agente 5: Security Analyst

``` markdown
<!-- .claude/agents/security-analyst.md -->
---
name: security-analyst
description: Performs security audits against OWASP Top 10.
  Reviews authentication, authorization, input validation,
  secrets exposure, and dependency vulnerabilities.
  Use for security reviews and compliance checks.
tools: read, bash, grep, glob
model: claude-sonnet-4-6-20250514
---

You are a senior application security engineer.
You think like an attacker to protect like a defender.

## Audit Checklist

### A01 - Broken Access Control
- Verify authorization on all endpoints
- Check for IDOR vulnerabilities
- Review CORS configuration

### A02 - Cryptographic Failures
- Check encryption at rest and in transit
- Verify no sensitive data in logs
- Review token/session management

### A03 - Injection
- Verify parameterized queries everywhere
- Check for command injection in Bash calls
- Review template rendering for XSS

### A07 - Authentication Failures
- Review JWT validation (expiry, signature, audience)
- Check password policies
- Verify rate limiting on auth endpoints

### Dependencies
- Run `npm audit` / `pip-audit` or equivalent
- Flag known CVEs

## Output Format
Rate findings: CRITICAL | HIGH | MEDIUM | LOW | INFO
CRITICAL and HIGH require immediate remediation.
```

### Agente 6: Code Reviewer

(Ya lo definimos en la sección 4.3 — usa herramientas read-only y Haiku.)

### Agente 7: Doc Writer

``` markdown
<!-- .claude/agents/doc-writer.md -->
---
name: doc-writer
description: Generates JSDoc/TSDoc comments, README files,
  API documentation, and changelogs from source code.
  Use for documentation tasks.
tools: read, write, grep, glob
model: claude-haiku-4-5-20251001
---

You are a technical writer. You write for developers
who have never seen this codebase before.

## Standards

- Every public function: @param, @returns, @throws, @example
- At least one working @example per public function
- README sections: Overview, Installation, Usage, API, Contributing
- Changelog: Conventional Commits format (Added/Changed/Fixed)
- Be concise. No filler text. Every sentence must add value.
```

### Tabla Resumen del Roster

| Agente | Modelo | Herramientas | Coste relativo |
|:-------|:-------|:-------------|:--------------:|
| spec-writer | Sonnet | read, grep, glob | $$ |
| architect | Sonnet | read, grep, glob, write | $$ |
| developer | Sonnet | read, write, bash, grep, glob | $$ |
| qa-engineer | Sonnet | read, write, bash, grep, glob | $$ |
| security-analyst | Sonnet | read, bash, grep, glob | $$ |
| code-reviewer | **Haiku** | read, grep, glob | **$** |
| doc-writer | **Haiku** | read, write, grep, glob | **$** |

    {{1}}
> **Optimización de coste:** Los agentes de code review y documentación usan Haiku porque sus tareas son más rutinarias y bien definidas. Esto reduce significativamente el coste del pipeline completo sin sacrificar calidad en esas tareas.

#### Quiz: Diseño de agentes

Estás diseñando un agente cuya única tarea es ejecutar `git diff main` y generar una descripción estructurada del PR. ¿Qué configuración es la más apropiada?

- [( )] Modelo: Opus, Tools: read, write, bash, grep, glob
- [( )] Modelo: Sonnet, Tools: read, write, bash
- [(X)] Modelo: Haiku, Tools: read, bash, grep, glob (sin write)
- [( )] Modelo: Sonnet, Tools: todos los disponibles
***
**Correcto.** Este agente solo necesita leer (`read`) y ejecutar `git diff` (`bash`), con `grep`/`glob` para buscar contexto. No necesita escribir archivos (`write`). Y como la tarea es bien definida y repetitiva, Haiku es suficiente y mucho más barato que Sonnet u Opus.
***

---

## 4.5 Agent Teams — Colaboración Multi-Agente

Agent Teams es la funcionalidad más avanzada de Claude Code. Lanzada como experimental con Opus 4.6 en febrero de 2026, permite orquestar múltiples instancias que trabajan en paralelo y **se comunican directamente entre sí**.

### La Diferencia Fundamental

``` ascii
  SUBAGENTES                     AGENT TEAMS
  ──────────                     ───────────

     ┌──────┐                    ┌──────┐
     │ Lead │                    │ Lead │
     └──┬───┘                    └──┬───┘
        │                           │
   ┌────┼────┐              ┌──────┼──────┐
   ▼    ▼    ▼              ▼      ▼      ▼
  ┌─┐  ┌─┐  ┌─┐            ┌──┐   ┌──┐   ┌──┐
  │A│  │B│  │C│            │A │◄─►│B │◄─►│C │
  └─┘  └─┘  └─┘            └──┘   └──┘   └──┘
   │    │    │
   └────┼────┘            Los teammates hablan
        │                 ENTRE SÍ directamente
        ▼                 vía sistema de buzón
  Solo reportan           (mailbox)
  al padre
```

    {{1}}
**Subagentes:** Reportan al padre. No pueden comunicarse entre sí. El padre es el intermediario obligatorio.

    {{2}}
**Agent Teams:** Los teammates comparten descubrimientos, desafían conclusiones de los demás y se coordinan directamente a través de un sistema de buzón (mailbox). Cada uno tiene su propia ventana de contexto de 1M tokens.

### Habilitación

Agent Teams es experimental y está deshabilitado por defecto:

``` bash
# En variable de entorno
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# O en settings.json
{
  "experimental": {
    "agentTeams": true
  }
}
```

    {{1}}
> **Requisito:** Agent Teams requiere Opus 4.6 como modelo. No funciona con Sonnet ni Haiku.

### Casos de Uso Óptimos

    {{1}}
**Investigación y revisión:** Múltiples teammates investigan diferentes aspectos de un problema simultáneamente y comparten hallazgos.

    {{2}}
**Nuevos módulos o features:** Cada teammate posee una pieza del desarrollo (frontend, backend, tests) sin pisarse.

    {{3}}
**Debugging con hipótesis competidoras:** Teammates prueban teorías diferentes en paralelo y convergen en la causa raíz.

    {{4}}
**Coordinación cross-layer:** Cambios que abarcan frontend, backend y tests, cada uno con su dueño dedicado.

### El Debate Pattern

Este es el patrón más poderoso de Agent Teams y merece atención especial:

``` ascii
  ┌─────────────────────────────────────────────────┐
  │ Prompt:                                         │
  │ "Los usuarios reportan que la app sale después  │
  │  de un mensaje. Spawn 5 agentes para investigar │
  │  hipótesis diferentes. Que debatan entre sí     │
  │  intentando refutar las teorías de los demás.   │
  │  Actualiza el doc de hallazgos con el consenso."│
  └──────────────┬──────────────────────────────────┘
                 │
     ┌───────────┼───────────────┐
     ▼           ▼               ▼
  ┌──────┐   ┌──────┐        ┌──────┐
  │Agent1│   │Agent2│  ...   │Agent5│
  │Memory│   │Event │        │Race  │
  │leak? │   │loop? │        │cond? │
  └──┬───┘   └──┬───┘        └──┬───┘
     │          │               │
     ◄──────────┼───────────────►
     │    Debate: intentan      │
     │    REFUTAR las teorías   │
     │    de los demás          │
     ◄──────────┼───────────────►
                │
                ▼
        ┌───────────────┐
        │   CONSENSO    │
        │ La hipótesis  │
        │ que sobrevive │
        │ al debate es  │
        │ probablemente │
        │ correcta      │
        └───────────────┘
```

    {{1}}
**¿Por qué funciona?** La investigación secuencial sufre de *anclaje*: una vez que se explora una primera teoría, la investigación posterior está sesgada hacia ella. Con múltiples investigadores independientes intentando activamente refutar las teorías de los demás, la hipótesis que sobrevive tiene mucha más probabilidad de ser la causa raíz real.

### Cuándo NO Usar Agent Teams

    {{1}}
**Tareas secuenciales:** Si el paso B depende del resultado del paso A, Agent Teams no aporta nada.

    {{2}}
**Ediciones en el mismo archivo:** Múltiples agentes editando el mismo archivo genera conflictos de merge y coordinación que consumen más tokens que una sesión única.

    {{3}}
**Trabajo con muchas dependencias entre agentes:** Si los agentes necesitan constantemente leer el output de los demás, el overhead de coordinación supera el beneficio.

    {{4}}
> **Coste:** Agent Teams consume significativamente más tokens que una sesión única. En una demo de Anthropic, 16 agentes construyeron un compilador C de 100.000 líneas en Rust en dos semanas con ~$20.000 en tokens. Úsalos cuando la exploración paralela genuinamente añade valor.

#### Quiz: Agent Teams

¿Cuál de estos escenarios es el PEOR candidato para Agent Teams?

- [( )] Tres agentes investigando la causa de un bug, cada uno con una hipótesis diferente
- [( )] Cuatro agentes implementando módulos independientes de una nueva feature
- [(X)] Cinco agentes editando secuencialmente el mismo archivo de configuración
- [( )] Dos agentes haciendo revisión cruzada de seguridad y rendimiento
***
**Correcto.** Ediciones secuenciales al mismo archivo es el peor escenario para Agent Teams. No hay paralelismo posible (cada edición depende de la anterior), y múltiples agentes accediendo al mismo archivo genera conflictos. Una sesión única lo resolvería más rápido y barato.
***

---

## 4.6 Coordinator Mode

En Coordinator Mode, Claude actúa exclusivamente como orquestador: planifica, divide el trabajo, asigna a subagentes y sintetiza resultados, pero no implementa nada directamente.

``` ascii
  ┌─────────────────────────────────────────────────┐
  │              COORDINATOR (Claude)               │
  │                                                 │
  │  1. Recibe la tarea del usuario                 │
  │  2. Analiza y divide en subtareas               │
  │  3. Asigna cada subtarea a un subagente         │
  │  4. Monitoriza progreso                         │
  │  5. Resuelve conflictos entre subagentes        │
  │  6. Sintetiza resultados                        │
  │  7. Presenta resultado final al usuario         │
  │                                                 │
  │  NO implementa código directamente              │
  └──────────────────────┬──────────────────────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
      ┌──────────┐ ┌──────────┐ ┌──────────┐
      │@developer│ │@qa-eng   │ │@doc-write│
      │          │ │          │ │          │
      │Implementa│ │Genera    │ │Actualiza │
      │la feature│ │tests     │ │docs      │
      └──────────┘ └──────────┘ └──────────┘
```

### Cuándo Usar Coordinator Mode

    {{1}}
**Tareas grandes y multifacéticas:** Cuando la tarea requiere tocar múltiples capas o dominios y un solo agente no puede mantener todo en contexto.

    {{2}}
**Cuando quieres control sobre el workflow:** El coordinator te permite definir el orden de ejecución, las dependencias y los criterios de calidad antes de que empiece el trabajo.

### Builder-Validator Pattern

Una variante poderosa del Coordinator Mode es el patrón Builder-Validator:

``` ascii
  ┌──────────┐         ┌──────────────┐
  │ BUILDER  │────────►│  VALIDATOR   │
  │          │         │              │
  │@developer│         │@code-reviewer│
  │          │         │@qa-engineer  │
  │Implementa│◄────────│              │
  │          │ feedback│Revisa contra │
  │Corrige   │ si falla│criterios de  │
  │          │         │calidad       │
  └──────────┘         └──────────────┘
        │                    │
        │    Ciclo hasta     │
        │    que pase ───────┘
        │
        ▼
   ┌──────────┐
   │ APROBADO │
   └──────────┘
```

    {{1}}
**El builder** (agente desarrollador) implementa el código.

    {{2}}
**El validator** (agente reviewer + QA) revisa contra criterios de calidad: tests, linting, security, estilo.

    {{3}}
**Si falla la validación**, vuelve al builder con feedback específico.

    {{4}}
**El ciclo se repite** hasta que el validator aprueba. Esto produce código de mayor calidad que una implementación de un solo paso.

#### Quiz: Patrones de orquestación

¿Cuál es la ventaja principal del Builder-Validator pattern sobre un único agente que implementa y revisa?

- [( )] Es más rápido porque corre en paralelo
- [( )] Usa menos tokens en total
- [(X)] Evita el sesgo de confirmación: el builder tiende a validar su propio trabajo; un agente separado como validator es más crítico
- [( )] Permite usar diferentes modelos de IA
***
**Correcto.** El problema fundamental de auto-revisión es el sesgo de confirmación: el mismo agente que escribió el código tiende a validarlo favorablemente. Un agente validator separado (con system prompt enfocado en encontrar problemas, no en defenderlos) es significativamente más crítico y encuentra más issues.
***

---

## 4.7 Ejercicio Práctico: Monta tu Roster

Vamos a implementar el equipo completo en tu proyecto.

    {{1}}
**Paso 1: Crea el directorio de agentes**

``` bash
mkdir -p .claude/agents
```

    {{2}}
**Paso 2: Implementa los agentes core**

Crea al menos estos tres agentes usando las plantillas de la sección 4.4 (adaptándolas a tu stack):

``` bash
# Copia los templates del curso y adapta al stack de tu proyecto
touch .claude/agents/code-reviewer.md
touch .claude/agents/qa-engineer.md
touch .claude/agents/security-analyst.md
```

    {{3}}
**Paso 3: Prueba la invocación**

``` text
> @code-reviewer revisa los últimos cambios en src/

> @qa-engineer genera tests para el módulo de autenticación

> @security-analyst audita src/auth/
```

    {{4}}
**Paso 4: Commitea para el equipo**

``` bash
git add .claude/agents/
git commit -m "feat: add specialized AI agents for code review, QA and security"
```

    {{5}}
**Paso 5: Documenta en CLAUDE.md**

Añade una sección a tu CLAUDE.md:

``` markdown
## AI Agents
- @code-reviewer: Code quality review (read-only, Haiku)
- @qa-engineer: Test generation and coverage (Sonnet)
- @security-analyst: OWASP security audit (Sonnet)
```

---

## 4.8 Resumen del Módulo

En este módulo has aprendido:

- [X] Los tres niveles de delegación: subagentes, custom agents y Agent Teams
- [X] Cuándo Claude crea subagentes automáticamente (y cuándo no)
- [X] Foreground vs. background para subagentes
- [X] Anatomía de un custom agent: system prompt, tools, modelo
- [X] Invocación con @mención
- [X] El roster completo: 7 agentes especializados del spec al deploy
- [X] Optimización de coste: Haiku para tareas rutinarias, Sonnet para implementación
- [X] Agent Teams: comunicación peer-to-peer y debate pattern
- [X] Coordinator Mode y Builder-Validator pattern
- [X] Implementación práctica del roster

---

## Evaluación Final del Módulo 4

**Pregunta 1:** ¿Qué modelo de IA es obligatorio para usar Agent Teams?

- [( )] Sonnet 4.6
- [(X)] Opus 4.6
- [( )] Haiku 4.5
- [( )] Cualquier modelo funciona
***
**Correcto.** Agent Teams requiere Opus 4.6 como modelo. Es el único modelo con la capacidad de razonamiento necesaria para coordinar comunicación peer-to-peer entre múltiples instancias.
***

**Pregunta 2:** ¿Qué campo del frontmatter YAML de un custom agent restringe las herramientas que puede usar?

[[tools]]
[[?]] Es la propiedad que define qué herramientas (read, write, bash...) tiene disponibles el agente
***
**Correcto.** El campo `tools` en el frontmatter define exactamente qué herramientas puede usar el agente. Por ejemplo, `tools: read, grep, glob` crea un agente de solo lectura que no puede modificar archivos ni ejecutar comandos.
***

**Pregunta 3:** Asigna el modelo más eficiente a cada agente:

[[Haiku]   [Sonnet]   [Opus]]
[(X)       ( )        ( )  ]  Agente de documentación que genera JSDoc
[( )       (X)        ( )  ]  Agente desarrollador que implementa features
[( )       ( )        (X)  ]  Team lead que coordina 5 agentes en un Agent Team
[(X)       ( )        ( )  ]  Agente de code review con herramientas read-only

**Pregunta 4:** En el Debate Pattern para debugging, ¿por qué la investigación con múltiples agentes en paralelo es mejor que la investigación secuencial por un solo agente?

- [( )] Porque es más rápida
- [( )] Porque usa menos tokens
- [(X)] Porque evita el sesgo de anclaje: la investigación secuencial se sesga hacia la primera hipótesis explorada
- [( )] Porque cada agente tiene acceso a diferentes archivos
***
**Correcto.** El sesgo de anclaje es el problema fundamental de la investigación secuencial. Una vez que un agente explora una primera hipótesis, su investigación posterior está sesgada hacia confirmarla. Con múltiples agentes independientes desafiándose mutuamente, la hipótesis que sobrevive al escrutinio tiene mucha más probabilidad de ser correcta.
***

**Pregunta 5:** Tienes un pipeline de feature que pasa por spec, diseño, implementación, tests, security y review. ¿Qué combinación de mecanismos del Módulo 3 y 4 usarías para automatizarlo?

- [( )] Un único Agent Team con 7 agentes simultáneos
- [( )] Solo slash commands encadenados manualmente
- [(X)] Custom agents para cada rol + slash command o skill que orqueste el pipeline + hooks para quality gates automáticas entre pasos
- [( )] Solo subagentes ad-hoc sin agentes predefinidos
***
**Correcto.** La combinación profesional es: custom agents para los roles especializados (persistentes, con system prompts y tools optimizados), una skill o slash command que defina el pipeline y orqueste la secuencia, y hooks que actúen como quality gates automáticas (linting, formateo, tests) entre cada paso.
***

---

## Siguiente: Módulo 5 — Workflows Profesionales

En el próximo módulo pondremos todo en acción:

- El workflow Feature Lifecycle completo (de spec a deploy)
- Git workflows con Claude Code (commits, PRs, checkpoints)
- Modo headless y CI/CD
- Loop continuo para monitorización
- Integración del roster de agentes en pipelines reales

> **Tarea antes del Módulo 5:** Implementa al menos 3 de los 7 custom agents del roster en tu proyecto. Pruébalos individualmente con @mención. En el Módulo 5 los conectaremos en un pipeline completo.
