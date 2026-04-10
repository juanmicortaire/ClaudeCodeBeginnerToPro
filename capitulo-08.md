<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 8 — Patrones Avanzados.

-->

# Módulo 8: Patrones Avanzados

> **"Dominar las herramientas te hace competente. Dominar los patrones te hace experto. Este módulo recoge las técnicas que diferencian a quienes usan Claude Code de quienes lo maestrizan."**

Bienvenido al último módulo del curso. Aquí recogemos los patrones avanzados que emergen cuando has interiorizado todo lo anterior: coordinación compleja, razonamiento extendido, métricas del pipeline, integración con sistemas legacy y el camino hacia la mejora continua.

    {{1}}
**Objetivo del módulo:** Elevar tu práctica al nivel experto. Dominar los patrones que solo tienen sentido después de haber vivido todos los módulos anteriores. Establecer las bases para la evolución continua de tu workflow.

    {{2}}
**Tiempo estimado:** 90-120 minutos

    {{3}}
**Prerrequisito:** Haber completado los Módulos 1-7. Idealmente, haber usado Claude Code en un proyecto real durante al menos dos semanas.

---

## 8.1 Coordinator Mode en Profundidad

Vimos Coordinator Mode en el Módulo 4 como introducción. Ahora lo estudiamos como patrón arquitectónico completo.

### La Filosofía del Coordinator

En Coordinator Mode, Claude deja de ser un implementador para convertirse en un **orquestador puro**. Su rol es descomponer, delegar, coordinar y sintetizar, pero nunca escribir código directamente.

``` ascii
  ┌─────────────────────────────────────────────────────┐
  │           COORDINATOR ARCHITECTURE                  │
  ├─────────────────────────────────────────────────────┤
  │                                                     │
  │              ┌──────────────────┐                   │
  │              │   COORDINATOR    │                   │
  │              │     (Claude)     │                   │
  │              │                  │                   │
  │              │  Rol:            │                   │
  │              │  ┌─────────────┐ │                   │
  │              │  │ 1. Descomp. │ │                   │
  │              │  │ 2. Planif.  │ │                   │
  │              │  │ 3. Delegar  │ │                   │
  │              │  │ 4. Monitor. │ │                   │
  │              │  │ 5. Sintesis │ │                   │
  │              │  └─────────────┘ │                   │
  │              └────────┬─────────┘                   │
  │                       │                             │
  │       ┌───────────────┼───────────────┐             │
  │       ▼               ▼               ▼             │
  │  ┌─────────┐    ┌─────────┐    ┌─────────┐          │
  │  │ Worker  │    │ Worker  │    │ Worker  │          │
  │  │ Agent 1 │    │ Agent 2 │    │ Agent 3 │          │
  │  │         │    │         │    │         │          │
  │  │ Imple-  │    │ Testea  │    │ Docu-   │          │
  │  │ menta   │    │         │    │ menta   │          │
  │  └─────────┘    └─────────┘    └─────────┘          │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

### Cuándo Invocar Coordinator Mode

No todas las tareas se benefician del coordinator overhead. Úsalo cuando:

    {{1}}
**La tarea es grande y multifacética.** Cuando requiere tocar frontend, backend, base de datos, tests y documentación, un único agente pierde el hilo.

    {{2}}
**Quieres control explícito sobre el orden.** El coordinator te permite definir dependencias, puntos de sincronización y criterios de calidad antes de empezar.

    {{3}}
**La tarea involucra decisiones arquitectónicas.** Cuando el coordinator tiene que decidir qué componentes crear, qué patrones aplicar, y cómo dividir el trabajo.

    {{4}}
**Hay riesgo de conflictos.** Cuando múltiples agentes podrían modificar los mismos archivos, el coordinator arbitra.

### Plantilla de Coordinator

``` markdown
<!-- .claude/commands/coordinator.md -->

## Coordinator Mode: $ARGUMENTS

You are now acting as a COORDINATOR. You do NOT implement
anything directly. You delegate all implementation work.

## Your Process

### Phase 1: Decomposition
Break down the task into independent subtasks. Identify:
- Dependencies between subtasks
- Parallelizable subtasks
- Critical path

### Phase 2: Planning
Present the execution plan to the user:
- Ordered list of steps
- Agent assigned to each step
- Estimated complexity (S/M/L)
- Dependencies visualization

WAIT for user approval before proceeding.

### Phase 3: Delegation
For each subtask:
- Spawn the appropriate agent with a precise prompt
- Include all context the agent needs (no shared memory)
- Specify expected output format

### Phase 4: Monitoring
Track progress of each agent. If any agent:
- Returns an error: analyze and decide retry or abort
- Returns low-quality output: request revision
- Completes successfully: integrate result

### Phase 5: Conflict Resolution
If two agents produce conflicting outputs:
- Analyze the conflict
- Decide which is correct OR merge both
- Document the decision

### Phase 6: Synthesis
Integrate all agent outputs into a coherent final result.
Present to user with a summary of what each agent did.

## Rules
- NEVER implement code yourself. Always delegate.
- ALWAYS get user approval on the plan before execution.
- ALWAYS document decisions made during conflict resolution.
```

#### Quiz: Coordinator Mode

¿Cuál es el error más común al usar Coordinator Mode?

- [( )] Invocarlo para tareas demasiado pequeñas
- [( )] Olvidarse de asignar agentes
- [(X)] Que el coordinator termine implementando código directamente en lugar de delegar, rompiendo la separación de responsabilidades
- [( )] Usar demasiados workers
***
**Correcto.** El coordinator debe ser un orquestador puro. Cuando empieza a implementar código directamente, pierde la visión global y se contamina con detalles que deberían quedarse en los workers. El system prompt debe reforzar explícitamente: "NEVER implement code yourself. Always delegate."
***

---

## 8.2 Builder-Validator Chains Avanzados

Vimos el patrón básico en el Módulo 4. Ahora lo llevamos al siguiente nivel.

### El Ciclo Builder-Validator-Improver

``` ascii
                  ┌─────────┐
                  │ BUILDER │
                  │ agent   │
                  └────┬────┘
                       │
                       │ código inicial
                       ▼
                  ┌─────────┐
                  │VALIDATOR│
                  │ agent   │
                  └────┬────┘
                       │
              ┌────────┴────────┐
              │                 │
          APROBADO          RECHAZADO
              │                 │
              ▼                 ▼
         ┌─────────┐      ┌──────────┐
         │  FINAL  │      │ IMPROVER │
         └─────────┘      │  agent   │
                          └─────┬────┘
                                │
                                │ código mejorado
                                │ + justificación
                                ▼
                          ┌─────────┐
                          │VALIDATOR│
                          │ (retry) │
                          └─────────┘
                                │
                       ┌────────┴────────┐
                       │                 │
                   APROBADO           RECHAZADO
                                         │
                                         ▼
                                  Max 3 iteraciones
                                  Si sigue fallando:
                                  Escalar a humano
```

### Los Tres Roles

    {{1}}
**Builder:** El implementador. Su system prompt lo enfoca en velocidad y funcionalidad básica. Crea una primera versión funcional del código.

    {{2}}
**Validator:** El crítico. Su system prompt lo enfoca en encontrar problemas. No propone soluciones, solo señala issues con severidad.

    {{3}}
**Improver:** El refinador. Recibe el código del builder + los issues del validator. Su system prompt lo enfoca en fixes específicos sin introducir nuevos problemas.

### Por Qué Tres Agentes Son Mejor que Dos

``` text
  Builder → Validator → Builder → Validator → ...
  ──────────────────────────────────────────
  Problema: el builder tiende a "defender" su código
           y aplica fixes incompletos porque quiere
           que su implementación inicial sea la correcta.
```

``` text
  Builder → Validator → Improver → Validator → ...
  ─────────────────────────────────────────────
  Ventaja: el improver no tiene ego sobre el código
          inicial (no lo escribió él). Es más
          dispuesto a rehacer secciones completas.
```

    {{1}}
> **Sesgo cognitivo evitado:** El efecto IKEA (sobrevalorar lo que uno ha construido) afecta a los modelos de lenguaje igual que a los humanos. Separar construcción y mejora en agentes diferentes rompe este sesgo.

### Plantilla de Implementación

``` markdown
<!-- .claude/commands/bvi.md -->

## Builder-Validator-Improver Chain: $ARGUMENTS

Execute this task using the BVI pattern.

### Step 1: Build
Invoke @developer with the task. Expected output:
working implementation with basic tests.

### Step 2: Validate
Invoke @code-reviewer with the builder's output.
Expected output: list of issues with severity.
Do NOT fix anything in this step.

### Step 3: Decision
- If validator returns 0 CRITICAL/HIGH issues: APPROVE
- Otherwise: proceed to improvement

### Step 4: Improve (if needed)
Invoke a fresh @developer instance (not the original)
with:
- The original code
- The list of issues
- Instruction: "You did NOT write this code. Improve it
  without bias. Feel free to rewrite sections entirely."

### Step 5: Re-validate
Return to Step 2 with the improved code.

### Limits
- Maximum 3 iterations
- If still failing after 3 iterations, escalate to human
  with: original code, all iterations, all validator reports
```

#### Quiz: BVI Chain

¿Por qué usar un `@developer` fresco (nueva instancia) como improver en lugar del mismo que hizo el builder?

- [( )] Para ahorrar tokens
- [( )] Porque el modelo Sonnet es mejor para improvements
- [(X)] Para evitar el sesgo del constructor: una instancia nueva no tiene ego sobre el código inicial y está más dispuesta a rehacer secciones completas
- [( )] Porque es requerido por la arquitectura de Claude Code
***
**Correcto.** Este es el punto clave del patrón BVI avanzado. Una instancia nueva del developer no tiene apego al código inicial (no lo escribió), lo que la hace más dispuesta a rewrites agresivos cuando el validator señala problemas estructurales. El mismo agente tendería a aplicar parches superficiales para "salvar" su implementación.
***

---

## 8.3 Debate Pattern para Decisiones Arquitectónicas

Ya vimos el Debate Pattern para debugging. Ahora lo aplicamos a decisiones arquitectónicas complejas.

### El Problema de las Decisiones Arquitectónicas

Las decisiones de arquitectura suelen tener trade-offs: no hay una respuesta "correcta", hay alternativas con ventajas y desventajas. Un único agente tiende a:

    {{1}}
**Converger prematuramente** en una solución sin explorar alternativas.

    {{2}}
**Sesgo hacia lo conocido:** recomienda patrones familiares aunque no sean óptimos.

    {{3}}
**No considerar todas las restricciones:** se enfoca en aspectos técnicos y olvida los organizacionales.

### Debate Pattern Arquitectónico

``` ascii
  Requirement: "Necesitamos procesar 100K eventos/segundo
                con latencia < 50ms y coste < $1000/mes"
         │
         ▼
  Agent Team (CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)
         │
    ┌────┴────┬────────┬────────┬────────┐
    ▼         ▼        ▼        ▼        ▼
  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
  │Arch1│  │Arch2│  │Arch3│  │Arch4│  │Arch5│
  │     │  │     │  │     │  │     │  │     │
  │Kafka│  │Redis│  │AWS  │  │Event│  │Apa- │
  │+Fli-│  │Str- │  │Kine-│  │Hub +│  │che  │
  │nk   │  │eams │  │sis  │  │Flink│  │Pulsa│
  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘
     │        │        │        │        │
     ◄────────┼────────┼────────┼────────►
     │        │        │        │        │
     │   Debate: cada uno defiende su    │
     │   propuesta Y ataca las de los    │
     │   demás con críticas específicas  │
     │                                   │
     ◄────────┼────────┼────────┼────────►
              │        │        │
              ▼        ▼        ▼
     ┌──────────────────────────────┐
     │  Coordinator sintetiza:      │
     │  - Alternativas con sus      │
     │    trade-offs                │
     │  - Matriz de decisión        │
     │  - Recomendación con         │
     │    justificación             │
     └──────────────────────────────┘
```

### Prompt de Invocación

``` text
> Necesitamos decidir la arquitectura para procesar 100K
  eventos/segundo con latencia < 50ms y coste < $1000/mes
  en Azure.

  Activa Agent Team y spawn 5 arquitectos, cada uno
  defendiendo una arquitectura diferente:

  1. Architect-Kafka: Event Hubs + Stream Analytics
  2. Architect-Databricks: Delta Live Tables
  3. Architect-Functions: Azure Functions + Cosmos DB
  4. Architect-AKS: Kubernetes + Apache Flink
  5. Architect-Serverless: Logic Apps + Service Bus

  Instrucciones:
  - Cada arquitecto presenta su propuesta con números
  - Cada uno debe encontrar al menos 3 debilidades en
    cada propuesta rival
  - Debate hasta converger o identificar claramente los
    trade-offs

  Output esperado:
  - Matriz de decisión (latencia, coste, operabilidad,
    vendor lock-in, curva de aprendizaje)
  - Recomendación con justificación
  - Plan B si la primera opción no funciona
```

    {{1}}
> **Por qué funciona:** El debate fuerza a cada arquitecto a no solo defender su opción, sino a **atacar activamente** las alternativas. Esto genera una matriz de decisión mucho más honesta que preguntarle a un único agente "¿cuál es mejor?".

### Documentando la Decisión: ADR Generado

El output del Debate Pattern se traduce directamente en un ADR (Architecture Decision Record):

``` markdown
# ADR-042: Event Processing Architecture

## Status
Accepted (via Agent Team debate, 2026-04-15)

## Context
[Generado por el coordinator a partir del debate]

## Alternatives Considered
1. **Event Hubs + Stream Analytics** — presented by Architect-Kafka
   - Pros: native Azure, managed, low ops
   - Cons: [ataques de los otros arquitectos]

2. **Delta Live Tables** — presented by Architect-Databricks
   - Pros: declarative, built-in quality
   - Cons: [ataques de los otros arquitectos]

[... y así con las 5 alternativas]

## Decision
We chose: [opción ganadora]

## Rationale
[Síntesis de por qué sobrevivió al debate]

## Consequences
- Positive: ...
- Negative: ...
- Neutral: ...

## Plan B
If this architecture fails at scale: [segunda opción del ranking]
```

#### Quiz: Debate arquitectónico

¿Qué diferencia al Debate Pattern arquitectónico de simplemente preguntarle a Claude "¿cuál es la mejor arquitectura para mi caso?"?

- [( )] Es más rápido
- [( )] Usa menos tokens
- [(X)] Fuerza a considerar múltiples alternativas con sus debilidades, evitando la convergencia prematura y el sesgo hacia patrones familiares del agente único
- [( )] Permite usar modelos diferentes
***
**Correcto.** Un único agente tiende a converger en la primera solución razonable que encuentra, típicamente la más familiar. El debate obliga a que cada alternativa tenga un "defensor" dedicado y a que cada una sea atacada por las otras. El resultado es una matriz de decisión mucho más completa y honesta, con trade-offs explícitos en lugar de un único recomendación sesgada.
***

---

## 8.4 Extended Thinking y Effort Level

Claude Code en modelos Opus y Sonnet 4.6 soporta **razonamiento adaptativo**: Claude puede "pensar más" antes de responder cuando la tarea lo requiere.

### Cuándo Activar Extended Thinking

``` ascii
  Tarea trivial (comando simple)
         │
         └──► Respuesta inmediata, sin thinking

  Tarea de complejidad media
         │
         └──► Thinking corto (~500-2000 tokens)

  Tarea compleja (arquitectura, debugging)
         │
         └──► Thinking profundo (~5000-20000 tokens)

  Tarea crítica (seguridad, algoritmo complejo)
         │
         └──► Thinking extendido (~30000+ tokens)
```

### Activación Explícita

En modelos compatibles, puedes forzar thinking extendido:

``` text
> Analiza este algoritmo de encriptación para encontrar
  vulnerabilidades criptográficas. Piensa paso a paso.
  Considera cada primitiva criptográfica individualmente.

> Antes de responder, considera todos los edge cases del
  sistema de concurrencia. No te apresures.
```

    {{1}}
**Palabras clave que activan más thinking:** "paso a paso", "considera cada", "no te apresures", "analiza en profundidad", "piensa cuidadosamente".

### Visualización del Thinking

``` text
  # Toggle de visibilidad del thinking
  Ctrl+O    # modo verbose
  Option+T  # alternar visualización

  Cuando está visible:
  ┌─────────────────────────────────────────┐
  │ [THINKING]                              │
  │ Let me analyze this step by step:       │
  │ 1. First, I need to check if the input  │
  │    validation is sufficient...          │
  │ 2. The issue could be in the token      │
  │    rotation logic...                    │
  │ ...                                     │
  │ [/THINKING]                             │
  │                                         │
  │ Based on my analysis, I found:          │
  │ 1. A race condition in...               │
  └─────────────────────────────────────────┘
```

### Cuándo NO Forzar Extended Thinking

    {{2}}
**Tareas simples:** Formateo, commits rutinarios, lecturas de archivos. El thinking extra es puro desperdicio de tokens.

    {{3}}
**Cuando el contexto es pobre:** Más thinking sobre información insuficiente no mejora la respuesta, solo la retrasa.

    {{4}}
**Cuando necesitas iteración rápida:** Si estás probando rápidamente varias ideas, el thinking lento te ralentiza.

#### Quiz: Extended Thinking

¿En cuál de estos escenarios el Extended Thinking aporta MÁS valor?

- [( )] Formatear un archivo con prettier
- [( )] Hacer un commit con mensaje convencional
- [( )] Leer un archivo de configuración
- [(X)] Analizar un algoritmo criptográfico en busca de vulnerabilidades sutiles
***
**Correcto.** El análisis criptográfico es exactamente el tipo de tarea donde el razonamiento profundo marca la diferencia: requiere considerar múltiples primitivas, sus interacciones, ataques conocidos, y edge cases sutiles. Las otras tareas son mecánicas y el thinking extra es gasto puro.
***

---

## 8.5 Integración con Sistemas Legacy

Claude Code brilla en proyectos nuevos, pero la realidad profesional es que la mayoría del trabajo ocurre en sistemas legacy. Estos son los patrones específicos para ese contexto.

### El Patrón "Exploración Antes de Acción"

En código legacy, asumir que sabes cómo funciona algo es peligroso. Siempre explora primero:

``` text
> ANTES de hacer ningún cambio, necesito entender cómo
  funciona el sistema de autenticación actual.

  1. Mapea todos los endpoints de auth y sus dependencias
  2. Identifica los puntos de integración con otros sistemas
  3. Lista las convenciones usadas (aunque no estén documentadas)
  4. Identifica los tests existentes que cubren esta área
  5. Genera un documento @docs/legacy-auth-exploration.md

  Solo después de esto, procederemos con cambios.
```

    {{1}}
Este patrón previene el error clásico de "pequeño cambio que rompe 15 cosas inesperadas" típico de sistemas legacy.

### El Patrón "Characterization Tests"

Para código legacy sin tests, genera primero tests que capturen el comportamiento actual:

``` text
> Este módulo no tiene tests. Antes de refactorizar:

  1. @qa-engineer genera "characterization tests": tests
     que documentan el comportamiento ACTUAL del código,
     incluso si parece incorrecto.
  2. Estos tests son el "snapshot" del comportamiento.
  3. Solo después empezamos a refactorizar, asegurándonos
     de que los characterization tests siguen pasando.
  4. Si durante el refactor queremos cambiar el
     comportamiento, actualizamos el test explícitamente
     y lo documentamos como cambio intencional.
```

    {{2}}
> **La clave:** Los characterization tests no juzgan si el comportamiento es correcto. Solo documentan qué *hace* el código. Así, durante el refactor, sabes inmediatamente si has cambiado el comportamiento (intencionalmente o no).

### El Patrón "Strangler Fig"

Para migraciones graduales de sistemas legacy, aplica el Strangler Fig Pattern con Claude:

``` ascii
  ┌────────────────────────────────────────────┐
  │         FASE 1: Fachada                    │
  │                                            │
  │   Cliente ──► [Fachada] ──► Legacy         │
  │                                            │
  │   Claude crea la fachada sin cambiar       │
  │   nada del legacy                          │
  └────────────────────────────────────────────┘
         │
         ▼
  ┌────────────────────────────────────────────┐
  │         FASE 2: Migración gradual          │
  │                                            │
  │   Cliente ──► [Fachada] ─┬─► Legacy        │
  │                          └─► Nuevo código  │
  │                                            │
  │   Claude migra feature por feature         │
  └────────────────────────────────────────────┘
         │
         ▼
  ┌────────────────────────────────────────────┐
  │         FASE 3: Estrangulación             │
  │                                            │
  │   Cliente ──► [Fachada] ──► Nuevo código   │
  │                                            │
  │   Legacy eliminado                         │
  └────────────────────────────────────────────┘
```

#### Quiz: Legacy

¿Cuál es el primer paso al trabajar con un módulo legacy sin tests que necesitas refactorizar?

- [( )] Empezar a refactorizar directamente, confiando en la revisión manual
- [( )] Reescribir el módulo desde cero
- [(X)] Generar characterization tests que capturen el comportamiento actual, aunque parezca incorrecto, antes de tocar nada
- [( )] Eliminar el módulo y usar una librería externa
***
**Correcto.** Los characterization tests son tu red de seguridad. Documentan qué hace el código actualmente, sin juzgar si es correcto. Una vez que tienes esa red, puedes refactorizar con confianza: si los tests siguen pasando, no has cambiado el comportamiento; si fallan, has introducido un cambio que debes decidir si es intencional o un bug.
***

---

## 8.6 Métricas del Pipeline y Mejora Continua

Un pipeline profesional debe ser **medible y mejorable**. Estas son las métricas clave a monitorizar.

### Métricas de Productividad

| Métrica | Cómo medir | Objetivo |
|:--------|:-----------|:---------|
| **Tiempo por feature** | Desde spec hasta PR | Reducción trimestre a trimestre |
| **Iteraciones de review** | Commits por PR | ≤ 2 iteraciones por review |
| **Cobertura de tests** | Line + branch coverage | ≥ 80% |
| **Tasa de bugs en QA** | Bugs/feature encontrados en QA | Decreciente |
| **Tasa de bugs en prod** | Bugs/feature que llegan a prod | < 5% |

### Métricas de Coste

| Métrica | Cómo medir | Objetivo |
|:--------|:-----------|:---------|
| **Coste por feature** | Tokens consumidos × precio | Decreciente |
| **Coste por PR review** | `/cost` al terminar review | < $0.50 |
| **Coste mensual/desarrollador** | Agregado del dashboard | Dentro de presupuesto |
| **Ratio Opus/Sonnet/Haiku** | Distribución de modelo | Haiku ≥ 40% |

### Métricas de Calidad del Agente

| Métrica | Cómo medir | Objetivo |
|:--------|:-----------|:---------|
| **Tasa de aprobación BVI** | % builds aprobados a la 1ª | ≥ 60% |
| **Hallazgos de security post-merge** | CVEs descubiertos después del merge | = 0 |
| **Falsos positivos del validator** | Issues marcados incorrectamente | < 10% |
| **Skills auto-activadas** | Skills invocadas sin `/comando` | Creciente |

### Dashboard de Pipeline

Configura un loop continuo que genere un dashboard diario:

``` text
> /loop 24h Genera el reporte diario del pipeline:

  1. Número de PRs procesados hoy
  2. Tiempo promedio por fase del pipeline
  3. Coste total del día (suma de /cost)
  4. Hallazgos de @security-analyst (por severidad)
  5. Cobertura de tests promedio
  6. Comparativa con ayer y promedio semanal

  Guarda en docs/metrics/pipeline-YYYY-MM-DD.md
```

    {{1}}
> **Mejora continua:** Revisa estas métricas semanalmente. Si la cobertura baja, refuerza al @qa-engineer. Si el coste sube, revisa la selección de modelos. Si los bugs en prod suben, añade una capa de seguridad.

#### Quiz: Métricas

¿Cuál de estas métricas es el mejor indicador de que tu pipeline está degradándose y necesita revisión?

- [( )] El coste mensual aumenta
- [( )] El número de features implementadas aumenta
- [(X)] La tasa de bugs que llegan a producción aumenta, especialmente si afecta a áreas críticas como autenticación
- [( )] Los desarrolladores usan más Haiku
***
**Correcto.** El aumento de bugs en producción es la señal definitiva de que alguna capa del pipeline está fallando. Las otras métricas pueden tener explicaciones benignas (más coste porque se procesan más features, más Haiku porque se optimiza correctamente). Pero bugs en producción significan que las 4 capas de defensa del Módulo 7 están dejando pasar problemas, y hay que investigar cuál refuerza.
***

---

## 8.7 El Camino al Siguiente Nivel

Este curso te ha dado las bases para ser un usuario profesional de Claude Code. Pero el dominio verdadero viene de la práctica continua. Estas son las recomendaciones finales:

### Las Tres Reglas del Practicante

    {{1}}
**Regla 1: Mide antes de optimizar.**
No intentes optimizar algo que no estás midiendo. Establece tu baseline con `/cost`, métricas del pipeline y observación de calidad antes de cambiar nada.

    {{2}}
**Regla 2: Itera en incrementos pequeños.**
No reescribas toda tu configuración de golpe. Añade una skill, observa el impacto durante una semana, añade otra. Los cambios pequeños te permiten identificar qué funciona y qué no.

    {{3}}
**Regla 3: Comparte con la comunidad.**
Las skills, hooks y agentes que funcionan en tu equipo probablemente funcionarán en otros. Contribuye a `awesome-claude-code`, publica tus plantillas, comparte tus insights.

### Roadmap de Crecimiento Post-Curso

``` ascii
  Mes 1: Consolidación
  ─────────────────────
  - Aplica los módulos 1-4 en un proyecto real
  - Mide tu baseline de tokens y calidad
  - Documenta tus insights personales

  Mes 2: Optimización
  ─────────────────────
  - Aplica los módulos 5-6 (workflows y tokens)
  - Reduce tu coste por pipeline un 50%
  - Implementa al menos 3 skills propias

  Mes 3: Defensa
  ─────────────────────
  - Aplica el módulo 7 (seguridad)
  - Configura las 4 capas de defensa
  - Integra quality gates en CI/CD

  Mes 4+: Patrones avanzados
  ─────────────────────
  - Aplica el módulo 8 según necesidad
  - Establece métricas y mejora continua
  - Comparte con tu equipo y comunidad
```

### Recursos para Seguir Aprendiendo

- **Documentación oficial:** `code.claude.com/docs` — referencia canónica actualizada.
- **awesome-claude-code:** Lista curada con cientos de recursos de la comunidad.
- **Repositorio oficial de skills de Anthropic:** Ejemplos de producción.
- **Discord y foros:** Comunidad activa para preguntas y discusión.

---

## 8.8 Resumen del Módulo

En este módulo final has aprendido:

- [X] Coordinator Mode como patrón arquitectónico puro
- [X] Builder-Validator-Improver chains con separación de sesgos
- [X] Debate Pattern para decisiones arquitectónicas complejas
- [X] Extended Thinking y cuándo activarlo
- [X] Patrones específicos para sistemas legacy
- [X] Characterization tests y Strangler Fig Pattern
- [X] Métricas del pipeline en tres dimensiones
- [X] Las tres reglas del practicante y el roadmap post-curso

---

## Evaluación Final del Módulo 8

**Pregunta 1:** En el patrón Builder-Validator-Improver, ¿por qué el improver debe ser una instancia nueva del desarrollador y no el builder original?

- [( )] Para usar un modelo más barato
- [( )] Porque el builder está ocupado
- [(X)] Para evitar el efecto IKEA: el builder original tiene apego al código que escribió y tiende a aplicar fixes superficiales; una instancia nueva no tiene ese sesgo
- [( )] Es un requisito técnico de Claude Code
***
**Correcto.** El efecto IKEA (sobrevalorar lo que uno mismo ha construido) afecta a los modelos de lenguaje igual que a los humanos. Una instancia fresca del developer no tiene apego emocional al código inicial y está más dispuesta a rewrites agresivos cuando el validator señala problemas estructurales.
***

**Pregunta 2:** ¿Qué tipo de test generarías PRIMERO al trabajar con un módulo legacy sin cobertura de tests?

[[characterization tests]]
[[?]] Son tests que documentan el comportamiento actual del código, sin juzgar si es correcto
[[?]] El término empieza por "characterization"
***
**Correcto.** Los characterization tests capturan el comportamiento actual del código como un "snapshot", independientemente de si ese comportamiento es correcto. Son tu red de seguridad durante el refactor: si siguen pasando, no has cambiado el comportamiento; si fallan, has introducido un cambio que debes validar como intencional.
***

**Pregunta 3:** Relaciona cada patrón con su caso de uso óptimo:

[[Coordinator]    [BVI Chain]    [Debate Pattern]    [Strangler Fig]]
[(X)              ( )            ( )                 ( )           ]  Tarea grande multifacética con riesgo de conflictos entre agentes
[( )              (X)            ( )                 ( )           ]  Implementación con ciclo de mejora iterativa separando construcción y refinamiento
[( )              ( )            (X)                 ( )           ]  Decisión arquitectónica con múltiples alternativas y trade-offs
[( )              ( )            ( )                 (X)           ]  Migración gradual de sistema legacy a nuevo código

**Pregunta 4:** ¿En cuál de estos escenarios Extended Thinking NO aporta valor significativo?

- [(X)] Formatear un archivo TypeScript con prettier
- [( )] Analizar la seguridad de un algoritmo criptográfico
- [( )] Diseñar la arquitectura de un sistema de alta concurrencia
- [( )] Debuggear un race condition complejo
***
**Correcto.** Formatear con prettier es una tarea mecánica y determinista. Forzar thinking extendido solo añade latencia y consume tokens sin mejorar el resultado. Extended Thinking es valioso para problemas que requieren razonamiento profundo: seguridad, arquitectura, debugging complejo.
***

**Pregunta 5:** ¿Cuál es la métrica más importante para detectar degradación del pipeline?

- [( )] Coste mensual en tokens
- [( )] Número de skills configuradas
- [(X)] Tasa de bugs que llegan a producción
- [( )] Velocidad de respuesta del modelo
***
**Correcto.** Los bugs en producción son el indicador definitivo de que el pipeline está fallando: significa que las 4 capas de defensa del Módulo 7 (prevención, runtime, análisis reactivo, CI/CD) están dejando pasar problemas. Las otras métricas pueden variar por razones benignas, pero los bugs en producción siempre requieren investigación y refuerzo de alguna capa.
***

---

## Cierre del Curso

Enhorabuena, has completado el **Curso Claude Code: De Principiante a Profesional**.

### Lo Que Has Aprendido

``` ascii
  ┌──────────────────────────────────────────────────┐
  │                  VIAJE COMPLETADO                │
  ├──────────────────────────────────────────────────┤
  │                                                  │
  │  Módulo 1: Fundamentos y Setup                   │
  │  └─► Sabes instalar, configurar y manejar        │
  │      permisos de Claude Code                     │
  │                                                  │
  │  Módulo 2: Memoria y Contexto                    │
  │  └─► Dominas CLAUDE.md, Auto Memory y gestión    │
  │      profesional del contexto                    │
  │                                                  │
  │  Módulo 3: Extensibilidad                        │
  │  └─► Creas skills, hooks, slash commands y       │
  │      conectas con MCP servers                    │
  │                                                  │
  │  Módulo 4: Agentes Especializados                │
  │  └─► Diseñas y orquestas rosters de agentes      │
  │      con Agent Teams                             │
  │                                                  │
  │  Módulo 5: Workflows Profesionales               │
  │  └─► Implementas pipelines end-to-end con Git,   │
  │      CI/CD y headless mode                       │
  │                                                  │
  │  Módulo 6: Optimización de Tokens                │
  │  └─► Reduces costes hasta un 80% sin sacrificar  │
  │      calidad                                     │
  │                                                  │
  │  Módulo 7: Seguridad y QA                        │
  │  └─► Aplicas defense in depth con 4 capas        │
  │      de protección                               │
  │                                                  │
  │  Módulo 8: Patrones Avanzados                    │
  │  └─► Dominas BVI chains, Debate Pattern y        │
  │      métricas del pipeline                       │
  └──────────────────────────────────────────────────┘
```

### Tu Próximo Paso

    {{1}}
El conocimiento sin práctica es efímero. La diferencia entre quienes completan este curso y lo olvidan, y quienes lo interiorizan y lo dominan, está en las próximas dos semanas.

    {{2}}
**Compromiso sugerido:**

- [ ] Aplica al menos 3 skills o agentes propios en un proyecto real esta semana
- [ ] Mide tu baseline de tokens con `/cost` durante 5 sesiones
- [ ] Configura al menos 2 hooks de quality gate en tu `settings.json`
- [ ] Comparte un insight del curso con tu equipo
- [ ] Revisa este curso en 3 meses para refrescar y profundizar

### Un Mensaje Final

    {{3}}
> **Recuerda:** Claude Code es una herramienta, no un reemplazo. Tu juicio, tu experiencia y tu capacidad de decisión siguen siendo lo más importante. Claude Code amplifica tus capacidades — es tu responsabilidad dirigirlas con sabiduría.

    {{4}}
> La diferencia entre un desarrollador y un desarrollador profesional no está en las herramientas que usa, sino en cómo las usa. Este curso te ha dado las técnicas. La maestría viene con la práctica consciente.

    {{5}}
**¡Éxito en tu viaje profesional con Claude Code!**

---

## Evaluación Final del Curso Completo

Esta es la evaluación de cierre del curso. Si aciertas al menos 4 de 5 preguntas, estás listo para aplicar todo lo aprendido en producción.

**Pregunta 1:** Ordena estos mecanismos de Claude Code de menor a mayor complejidad:

- [[1]] Slash Commands (atajos manuales simples)
- [[2]] Skills (con progressive disclosure y auto-activación)
- [[3]] Custom Agents (con system prompt y tools propios)
- [[4]] Agent Teams (comunicación peer-to-peer, experimental)

**Pregunta 2:** Diseña la selección de modelo para un pipeline de desarrollo con presupuesto ajustado. Selecciona las asignaciones correctas (multi-select):

- [[X]] Code reviewer read-only: Haiku
- [[X]] Doc writer: Haiku
- [[X]] Developer principal: Sonnet
- [[X]] Security analyst: Sonnet
- [[ ]] Linter automático: Opus
- [[ ]] Formateador de código: Opus
- [[X]] Arquitecto (decisiones complejas): Opus o Sonnet
***
**Correcto.** La selección inteligente asigna Haiku a tareas rutinarias y bien definidas (review, docs), Sonnet a tareas de criterio técnico (dev, security), y Opus solo a decisiones genuinamente complejas. Nunca uses Opus para tareas mecánicas como linting o formateo.
***

**Pregunta 3:** ¿Cuáles son las 4 capas de Defense in Depth del Módulo 7? (multi-select)

- [[X]] Prevención (CLAUDE.md, permisos, tools restringidos)
- [[X]] Runtime checks (hooks PreToolUse y PostToolUse)
- [[X]] Análisis reactivo (@security-analyst, @qa-engineer)
- [[X]] CI/CD quality gates bloqueantes
- [[ ]] Encriptación de comunicaciones
- [[ ]] VPN corporativa

**Pregunta 4:** Un colega te dice: "Voy a poner todos mis prompts frecuentes directamente en CLAUDE.md para que Claude los tenga siempre disponibles." ¿Qué le respondes?

- [( )] Buena idea, más contexto es mejor
- [( )] Cada prompt debería ser un slash command separado
- [(X)] Las skills con progressive disclosure son la solución: cargan solo nombre y descripción al inicio (~50 tokens cada una), y las instrucciones completas se inyectan solo cuando se activan. Así puedes tener docenas de prompts sin impacto en el contexto.
- [( )] Es indiferente, los tokens son baratos
***
**Correcto.** Este es el principio central del Módulo 3 y 6. Meter prompts repetitivos en CLAUDE.md los convierte en gasto permanente. Convertirlos en skills aprovecha progressive disclosure, ahorrando hasta el 98% de tokens mientras mantiene todo disponible bajo demanda.
***

**Pregunta 5:** Después de completar este curso, ¿cuál es el mejor siguiente paso para interiorizar lo aprendido?

- [( )] Leer todo el curso de nuevo
- [( )] Memorizar todas las tablas y diagramas
- [(X)] Aplicar lo aprendido en un proyecto real esta semana, midiendo el baseline antes de optimizar, e iterando en incrementos pequeños
- [( )] Esperar a la próxima versión del curso
***
**Correcto.** El conocimiento teórico se olvida rápidamente sin aplicación práctica. Las tres reglas del practicante del Módulo 8 son: medir antes de optimizar, iterar en incrementos pequeños, y compartir con la comunidad. El mejor momento para empezar es ahora, en un proyecto real, midiendo tu baseline primero.
***

---

## Gracias por Completar el Curso

> **"El futuro del desarrollo de software no es sobre si usaremos agentes IA, sino sobre cómo los usaremos. Quienes aprendan a orquestarlos con sabiduría, criterio y rigor profesional liderarán la próxima década del software."**

Hasta aquí llega el **Curso Claude Code: De Principiante a Profesional**. Ocho módulos. Decenas de quizzes. Diagramas, plantillas, ejercicios prácticos y patrones avanzados.

Todo este material es tuyo para revisar, adaptar y compartir. Vuelve a él cuando lo necesites. Úsalo como referencia. Constrúyelo sobre tu propia experiencia.

**El viaje apenas comienza. Ahora te toca a ti.**

---

*Curso creado con [LiaScript](https://liascript.github.io) — Markdown interactivo open-source.*

*Si este curso te ha sido útil, considera contribuir con tus propios insights, skills y patrones a la comunidad de Claude Code.*
