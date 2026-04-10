<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 6 — Optimización de Tokens.

-->

# Módulo 6: Optimización de Tokens

> **"Los tokens son el combustible de Claude Code. Un piloto de Fórmula 1 no gana carreras quemando más gasolina que nadie — gana optimizando cada gota. Lo mismo aplica aquí."**

La optimización de tokens no es solo ahorro económico. Un contexto más limpio produce respuestas de mayor calidad, tiempos de respuesta más rápidos y menos "alucinaciones". Este módulo te enseña a obtener más por menos.

    {{1}}
**Objetivo del módulo:** Dominar las estrategias de reducción de consumo, selección inteligente de modelo, monitorización en tiempo real y el cálculo económico de un pipeline profesional.

    {{2}}
**Tiempo estimado:** 60-75 minutos

    {{3}}
**Prerrequisito:** Haber completado los Módulos 1-5.

---

## 6.1 Por Qué Importa la Optimización

Antes de entrar en técnicas, necesitas entender por qué optimizar tokens va más allá del coste:

``` ascii
  Tokens desperdiciados
         │
         ├──► Coste económico directo ($$)
         │
         ├──► Contexto contaminado
         │         │
         │         └──► Respuestas de menor calidad
         │              Instrucciones "olvidadas"
         │              Inconsistencias
         │
         ├──► Mayor latencia
         │         │
         │         └──► Más tokens = más tiempo de procesamiento
         │
         └──► Compactaciones más frecuentes
                   │
                   └──► Pérdida de detalle
                        Interrupciones del flujo
```

    {{1}}
La relación calidad-contexto no es lineal. Claude funciona excelentemente con un contexto de 30-50K tokens bien seleccionados. Un contexto de 150K tokens con mucho ruido produce peores resultados que uno de 50K tokens limpios.

    {{2}}
> **La regla de oro:** Cada token en el contexto debe *ganarse* su lugar. Si no contribuye directamente a la tarea actual, es ruido.

#### Quiz: Fundamentos

¿Cuál de estas afirmaciones es CORRECTA sobre la relación entre tokens y calidad?

- [( )] Más tokens en el contexto siempre producen mejores respuestas
- [( )] La calidad es constante independientemente del tamaño del contexto
- [(X)] Un contexto limpio de 50K tokens puede producir mejores respuestas que uno ruidoso de 150K tokens
- [( )] La calidad solo depende del modelo usado, no del contexto
***
**Correcto.** La calidad de las respuestas depende de la *relevancia* del contexto, no del volumen. Un contexto saturado de lecturas de archivos irrelevantes diluye las instrucciones de CLAUDE.md y las preferencias del usuario, degradando las respuestas.
***

---

## 6.2 Anatomía del Consumo de Tokens

Para optimizar, primero debes saber qué consume tokens y cuánto:

``` ascii
  Consumo por tipo de operación (tokens aproximados)
  ═══════════════════════════════════════════════════

  Lectura de archivo grande (500+ líneas)
  ████████████████████████████████████████  10.000-20.000

  Resultado de bash (log verbose)
  ██████████████████████████████           8.000-15.000

  Respuesta compleja de Claude
  ████████████████                         2.000-4.000

  CLAUDE.md cargado al inicio
  ██████████                               2.000-5.000 (fijo)

  Auto Memory
  █████                                    1.000-3.000 (fijo)

  Tu prompt típico
  ███                                      100-500

  Nombre + descripción de 1 skill
  █                                        ~50
```

    {{1}}
**El dato crucial:** Las lecturas de archivos y los resultados de bash son los mayores consumidores. Una sola lectura de un archivo de 500 líneas puede consumir 10-20K tokens. Un `cat` de un directorio entero o un log verbose pueden saturar el contexto de golpe.

    {{2}}
Comparemos dos sesiones de trabajo reales:

| Sesión | Acciones | Tokens | Calidad |
|:-------|:---------|:------:|:-------:|
| **A (sin optimizar)** | Lee 8 archivos "por si acaso", ejecuta `npm test` verbose, no compacta | 120K | Baja: Claude "olvida" instrucciones |
| **B (optimizada)** | Lee solo 2 archivos relevantes, ejecuta test solo del módulo, compacta a los 45K | 55K | Alta: Claude sigue instrucciones al pie |

    {{3}}
> La sesión B consume menos de la mitad de tokens y produce mejores resultados. La optimización es un multiplicador de calidad, no solo de coste.

---

## 6.3 Siete Estrategias de Reducción

### Estrategia 1: Progressive Disclosure en Skills

Ya lo vimos en el Módulo 3, pero merece repetición por su impacto:

``` ascii
  SIN progressive disclosure:
  10 skills × ~5.000 tokens cada una = 50.000 tokens al inicio
  ────────────────────────────────────────────────────────────

  CON progressive disclosure:
  10 skills × ~50 tokens (nombre+descripción) = 500 tokens al inicio
  + 1 skill activa × ~5.000 tokens = 5.500 tokens total
  ────────────────────────────────────────────────────────────

  Ahorro: ~90% en el mejor caso, hasta 98% con muchas skills
```

    {{1}}
**Acción:** Migra tus prompts repetitivos de CLAUDE.md o conversación a skills con SKILL.md y frontmatter YAML. Solo pagarás tokens cuando se activen.

### Estrategia 2: MCP Tool Search y Lazy Loading

Los servidores MCP exponen definiciones de herramientas que ocupan contexto. MCP Tool Search carga las herramientas bajo demanda:

``` ascii
  SIN lazy loading:
  5 MCP servers × ~50 tools × ~200 tokens/tool = 50.000 tokens
  ────────────────────────────────────────────────────────────

  CON MCP Tool Search:
  5 MCP servers × ~5 tokens (metadata) = 25 tokens
  + tools usadas bajo demanda ≈ 2.500 tokens
  ────────────────────────────────────────────────────────────

  Ahorro: ~95%
```

    {{2}}
**Acción:** Habilita MCP Tool Search. Con esta funcionalidad puedes tener docenas de MCP servers conectados sin impacto significativo en el contexto.

### Estrategia 3: CLAUDE.md Lean

Cada sesión de Claude Code carga CLAUDE.md completo. Un archivo inflado consume contexto en *todas* las sesiones, incluso las que no lo necesitan.

| Práctica | Tokens | Impacto |
|:---------|:------:|:--------|
| CLAUDE.md de 400 líneas con API docs embebidos | ~8.000 | Consume 8K en cada sesión, muchas no usan API |
| CLAUDE.md de 100 líneas con `@docs/api.md` | ~2.000 | API docs solo se cargan cuando Claude las necesita |

    {{3}}
**Acción:** Mantén CLAUDE.md bajo 200 líneas. Mueve contenido detallado a archivos referenciados con `@path`. Usa `.claude/rules/` para reglas modulares.

### Estrategia 4: Prompts Específicos

La forma en que formulas tus peticiones tiene impacto directo en tokens consumidos:

    {{1}}
**Mal prompt (vago, genera exploración innecesaria):**

``` text
> Revisa el proyecto y dime qué se puede mejorar
```

Claude lee docenas de archivos, genera un reporte de 3.000 tokens sobre mejoras genéricas. Coste: ~40K tokens.

    {{2}}
**Buen prompt (específico, dirige la acción):**

``` text
> Revisa src/auth/token-service.ts buscando:
> 1. Manejo de errores de JWT expirado
> 2. Race conditions en refresh concurrente
> Solo esos dos puntos.
```

Claude lee un archivo, analiza dos aspectos concretos. Coste: ~5K tokens. Misma calidad en lo que importa.

    {{3}}
**Acción:** Incluye los archivos relevantes con `@path` en el prompt. Delimita exactamente qué quieres. "Solo estos puntos" o "máximo 10 líneas de respuesta" acotan el output.

### Estrategia 5: /compact Proactivo

No esperes a que el contexto se sature. Compacta preventivamente:

``` ascii
  ┌──────────────────────────────────────────┐
  │   REGLA DE COMPACTACIÓN                  │
  │                                           │
  │   /cost muestra...     Acción             │
  │   ─────────────────    ──────             │
  │   < 30K tokens         Sigue trabajando   │
  │   30K - 50K tokens     Plantea compactar  │
  │   > 50K tokens         /compact AHORA     │
  │   > 80K tokens         /clear si cambias  │
  │                        de tarea           │
  └──────────────────────────────────────────┘
```

    {{4}}
**Acción:** Incorpora `/cost` a tu rutina cada 30-45 minutos. Compacta antes de que el contexto degrade la calidad.

### Estrategia 6: /clear Agresivo al Cambiar de Tarea

El coste de re-cargar contexto (Claude re-lee CLAUDE.md + Auto Memory, ~3-5K tokens) es mucho menor que el coste de trabajar con 50K tokens de contexto irrelevante de la tarea anterior.

    {{5}}
**Acción:** Siempre `/clear` al cambiar de tarea. No intentes "aprovechar" el contexto existente si la nueva tarea no tiene relación con la anterior.

### Estrategia 7: Selección de Modelo por Tarea

Esta es la palanca de optimización con mayor impacto económico:

``` ascii
  Coste relativo por modelo (por millón de tokens)
  ═════════════════════════════════════════════════

  Opus 4.6
  ████████████████████████████████████████  $$$$$

  Sonnet 4.6
  ████████████████                          $$

  Haiku 4.5
  ████                                      $
```

    {{6}}
La diferencia de coste entre Opus y Haiku puede ser de 10-20x por las mismas operaciones. Usar Opus para todo es como enviar un paquete por correo urgente cuando el ordinario llega igual de bien.

#### Quiz: Estrategias

¿Cuál de estas acciones tiene el MAYOR impacto en ahorro de tokens por sesión?

- [( )] Escribir prompts más cortos
- [( )] Usar `/compact` cada 10 minutos
- [(X)] Migrar instrucciones repetitivas de CLAUDE.md a skills con progressive disclosure
- [( )] Desactivar Auto Memory
***
**Correcto.** Progressive disclosure en skills puede ahorrar hasta un 98% de los tokens que esas instrucciones consumirían si estuvieran permanentemente en el contexto. Un CLAUDE.md inflado con instrucciones que solo se usan ocasionalmente es el mayor desperdicio sistémico, porque afecta a *todas* las sesiones.
***

---

## 6.4 Selección de Modelo por Rol de Agente

En el Módulo 4 diseñamos el roster de agentes. Ahora asignamos modelos con criterio económico:

``` ascii
  TAREA                           MODELO          JUSTIFICACIÓN
  ─────                           ──────          ─────────────

  Orquestación de Agent Team      Opus 4.6        Requiere razonamiento
  Diseño de arquitectura          Opus/Sonnet     complejo y coordinación

  Implementación de features      Sonnet 4.6      Equilibrio velocidad/
  Testing y QA                    Sonnet 4.6      calidad para tareas
  Security review                 Sonnet 4.6      que requieren criterio

  Code review (read-only)         Haiku 4.5       Tarea bien definida,
  Generación de docs              Haiku 4.5       no requiere creatividad
  Linting y formateo              Haiku 4.5       Patrón repetitivo
  Descripción de PR               Haiku 4.5       Template-driven
```

### Impacto Económico Real

Calculemos el coste de un pipeline Feature Lifecycle con y sin optimización de modelo:

    {{1}}
**Sin optimización (todo Opus):**

| Fase | Agente | Tokens | Modelo | Coste estimado |
|:-----|:-------|:------:|:------:|:--------------:|
| Spec | spec-writer | ~8K | Opus | $0.30 |
| Diseño | architect | ~10K | Opus | $0.38 |
| Implementación | developer | ~25K | Opus | $0.95 |
| Testing | qa-engineer | ~15K | Opus | $0.57 |
| Security | security-analyst | ~10K | Opus | $0.38 |
| Review | code-reviewer | ~8K | Opus | $0.30 |
| Docs | doc-writer | ~5K | Opus | $0.19 |
| **Total** | | **~81K** | | **~$3.07** |

    {{2}}
**Con optimización de modelo:**

| Fase | Agente | Tokens | Modelo | Coste estimado |
|:-----|:-------|:------:|:------:|:--------------:|
| Spec | spec-writer | ~8K | Sonnet | $0.07 |
| Diseño | architect | ~10K | Sonnet | $0.09 |
| Implementación | developer | ~25K | Sonnet | $0.22 |
| Testing | qa-engineer | ~15K | Sonnet | $0.13 |
| Security | security-analyst | ~10K | Sonnet | $0.09 |
| Review | code-reviewer | ~8K | **Haiku** | **$0.02** |
| Docs | doc-writer | ~5K | **Haiku** | **$0.01** |
| **Total** | | **~81K** | | **~$0.63** |

    {{3}}
> **Ahorro: ~80% ($3.07 → $0.63) por pipeline.** Con 5 features por semana, eso es ~$12/semana vs ~$61/semana. A lo largo de un año: ~$624 vs ~$3.172 por desarrollador.

    {{4}}
> **Nota:** Estos costes son estimaciones ilustrativas basadas en precios públicos. Los costes reales dependen del plan, la complejidad de la tarea y la longitud real de las conversaciones. Lo importante es la proporción: la optimización de modelo reduce el coste del pipeline de forma dramática.

#### Quiz: Selección de modelo

¿Cuál es el ahorro porcentual aproximado de usar Haiku en vez de Opus para el agente de code review (read-only)?

- [( )] ~20%
- [( )] ~50%
- [(X)] ~90% o más
- [( )] No hay diferencia significativa
***
**Correcto.** La diferencia de precio entre Opus y Haiku es de aproximadamente 10-20x. Para una tarea de solo lectura bien definida como code review, Haiku produce resultados equivalentes a una fracción del coste. El ahorro es del ~90-95%.
***

---

## 6.5 Morph MCP: Ediciones y Búsquedas Optimizadas

Morph MCP es un servidor MCP de terceros que optimiza dos de las operaciones más costosas:

    {{1}}
**FastApply:** Ediciones de código un 35% más rápidas. En lugar de que Claude genere el archivo completo modificado (caro en tokens), FastApply aplica diffs incrementales.

    {{2}}
**WarpGrep:** Búsqueda paralela en el codebase que ahorra un 40% de tokens de entrada respecto al grep estándar de Claude Code.

``` text
  Operación             Sin Morph          Con Morph         Ahorro
  ─────────             ─────────          ─────────         ──────
  Editar archivo        Regenera completo  Diff incremental   ~35%
  500 líneas            (~5.000 tokens)    (~3.250 tokens)

  Buscar en codebase    Grep secuencial    WarpGrep paralelo  ~40%
  50K líneas            (~8.000 tokens)    (~4.800 tokens)
```

    {{3}}
**Instalación:**

``` bash
claude mcp add morph @morphllm/morphmcp
```

---

## 6.6 Monitorización y Control de Costes

### Herramientas de Monitorización

    {{1}}
**`/cost` en sesión:** Tu herramienta principal. Úsala cada 30-45 minutos.

``` text
> /cost

  Session cost:
  Input tokens:  45,230
  Output tokens: 12,108
  Total cost:    $0.47
  Time active:   32 min
  Time waiting:  18 min
```

    {{2}}
**`--max-budget-usd` en headless:** Límite de gasto por ejecución. Imprescindible en CI/CD.

``` bash
claude -p "refactor the auth module" --max-budget-usd 5.00
```

Si Claude alcanza el límite, se detiene limpiamente en lugar de seguir acumulando coste.

    {{3}}
**CC Usage (herramienta de terceros):** CLI que analiza los logs locales de Claude Code y presenta dashboards de consumo:

``` bash
npx cc-usage
```

Muestra costes por sesión, por proyecto, tokens consumidos por tipo de operación y tendencias temporales.

    {{4}}
**Statuslines personalizadas:** Extensiones como `claude-powerline` o `claudia-statusline` que muestran en el terminal, en tiempo real: modelo activo, tokens usados, rama git, coste acumulado y progreso del contexto.

### Presupuesto por Equipo

Para gestionar costes a nivel de equipo:

``` text
  Presupuesto mensual del equipo: $500
  ──────────────────────────────────────

  Desarrollador senior (uso intensivo):     $150/mes
  Desarrollador mid (uso moderado):          $80/mes
  QA/Tester (uso específico):               $40/mes
  CI/CD pipelines (automatizado):           $100/mes
  Buffer para picos:                        $130/mes
```

    {{5}}
> **Consejo:** Configura alertas en el dashboard de tu cuenta de Anthropic para recibir notificación cuando el consumo alcance el 80% del presupuesto mensual.

#### Quiz: Monitorización

Tu equipo tiene un presupuesto de $500/mes para Claude Code. Un desarrollador junior está consumiendo $300/mes él solo. Al investigar, descubres que sus sesiones promedian 150K tokens. ¿Cuál es la acción más probable de mayor impacto?

- [( )] Limitar su acceso a Claude Code
- [( )] Cambiar todo su uso a Haiku
- [(X)] Revisar su CLAUDE.md (probablemente inflado) y enseñarle a usar `/compact`, `/clear` y prompts específicos
- [( )] Aumentar el presupuesto del equipo
***
**Correcto.** 150K tokens por sesión indica un contexto sin gestión: archivos leídos innecesariamente, sin compactación, prompts vagos que generan exploración amplia. Enseñarle las prácticas del Módulo 2 (CLAUDE.md lean, `/compact` proactivo, `/clear` al cambiar de tarea) y del Módulo 6 (prompts específicos) puede reducir su consumo un 50-70% sin limitar su productividad.
***

---

## 6.7 Checklist de Optimización

Usa esta checklist como referencia rápida. Cada ítem tiene un impacto estimado:

| # | Acción | Impacto | Esfuerzo |
|:-:|:-------|:-------:|:--------:|
| 1 | Migrar instrucciones repetitivas a skills | ★★★★★ | Medio |
| 2 | Asignar Haiku a agentes rutinarios | ★★★★★ | Bajo |
| 3 | Mantener CLAUDE.md < 200 líneas con `@path` | ★★★★ | Bajo |
| 4 | Habilitar MCP Tool Search (lazy loading) | ★★★★ | Bajo |
| 5 | `/compact` proactivo cuando `/cost` > 50K | ★★★★ | Bajo |
| 6 | `/clear` al cambiar de tarea | ★★★ | Bajo |
| 7 | Prompts específicos con archivos explícitos | ★★★ | Medio |
| 8 | Instalar Morph MCP para ediciones/búsquedas | ★★★ | Bajo |
| 9 | `--max-budget-usd` en todo pipeline de CI | ★★★ | Bajo |
| 10 | Monitorizar con `/cost` cada 30-45 min | ★★ | Bajo |

    {{1}}
> **Priorización:** Implementa primero los ítems 1-4 (alto impacto, bajo-medio esfuerzo). Solo esos cuatro pueden reducir tu consumo un 60-70%.

---

## 6.8 Ejercicio Práctico: Auditoría de Consumo

Realiza esta auditoría en tu proyecto actual:

    {{1}}
**Paso 1: Mide tu baseline**

``` text
$ claude
> Lee src/auth/token-service.ts y dime qué hace
> /cost
```

Anota los tokens de input. Este es tu coste de "una lectura + un análisis".

    {{2}}
**Paso 2: Revisa tu CLAUDE.md**

``` bash
wc -l CLAUDE.md
# Si > 200 líneas, necesitas adelgazar
```

    {{3}}
**Paso 3: Identifica skills candidatas**

Revisa tu historial de comandos: ¿hay prompts que repites en múltiples sesiones? Cada uno es candidato a skill.

    {{4}}
**Paso 4: Revisa tus agentes**

¿Hay agentes usando Sonnet u Opus que podrían usar Haiku? Revisa la tabla de la sección 6.4.

    {{5}}
**Paso 5: Ejecuta una sesión optimizada**

Aplica las estrategias 1-7 durante una sesión de trabajo real. Al terminar, compara `/cost` con tu sesión típica.

---

## 6.9 Resumen del Módulo

En este módulo has aprendido:

- [X] Por qué la optimización de tokens mejora calidad además de reducir coste
- [X] La anatomía del consumo: lecturas de archivos y bash son los mayores consumidores
- [X] Siete estrategias de reducción con impacto estimado
- [X] Selección de modelo por rol con cálculo económico real (~80% ahorro)
- [X] Morph MCP para ediciones y búsquedas optimizadas
- [X] Herramientas de monitorización: `/cost`, `--max-budget-usd`, CC Usage, statuslines
- [X] Gestión de presupuesto por equipo
- [X] Checklist de optimización priorizada

---

## Evaluación Final del Módulo 6

**Pregunta 1:** ¿Cuál es el mayor consumidor de tokens en una sesión típica de Claude Code?

- [( )] Los prompts del usuario
- [( )] Las respuestas de Claude
- [(X)] Las lecturas de archivos y resultados de bash
- [( )] El CLAUDE.md cargado al inicio
***
**Correcto.** Una sola lectura de un archivo de 500+ líneas puede consumir 10-20K tokens. Los resultados de comandos bash verbose pueden consumir aún más. Tus prompts (~100-500 tokens) son insignificantes en comparación.
***

**Pregunta 2:** ¿Cuántas líneas como máximo debería tener un CLAUDE.md bien optimizado?

[[200]]
[[?]] Es el umbral donde la adherencia empieza a degradarse
***
**Correcto.** Por encima de 200 líneas, el consumo de contexto es excesivo y la adherencia de Claude a las instrucciones se degrada. Mueve el detalle a archivos referenciados con `@path`.
***

**Pregunta 3:** Ordena estas operaciones de MAYOR a MENOR consumo de tokens:

- [[1]] Lectura de un archivo de 800 líneas (~15K tokens)
- [[2]] Resultado de `npm test` verbose (~8K tokens)
- [[3]] CLAUDE.md de 150 líneas cargado al inicio (~3K tokens)
- [[4]] Tu prompt pidiendo una revisión de código (~300 tokens)
- [[5]] Nombre + descripción de una skill (~50 tokens)

**Pregunta 4:** Tu pipeline Feature Lifecycle cuesta ~$3 por ejecución con todos los agentes en Opus. Aplicas las optimizaciones del módulo: Haiku para code-reviewer y doc-writer, Sonnet para el resto. ¿Cuál es el coste aproximado optimizado?

- [( )] ~$2.50 (ahorro del ~15%)
- [(X)] ~$0.60 (ahorro del ~80%)
- [( )] ~$1.50 (ahorro del ~50%)
- [( )] ~$0.10 (ahorro del ~97%)
***
**Correcto.** Como vimos en la sección 6.4, la selección inteligente de modelo reduce el coste del pipeline de ~$3.07 a ~$0.63, un ahorro de aproximadamente el 80%. La mayor parte del ahorro viene de usar Sonnet en lugar de Opus para la mayoría de tareas, no solo del switch a Haiku en dos agentes.
***

**Pregunta 5:** Un colega dice: "Voy a poner todas las instrucciones de mis 15 skills directamente en CLAUDE.md para que Claude las tenga siempre disponibles. Así no perderá tiempo cargándolas." ¿Qué le explicas?

- [( )] Es buena idea, más contexto es mejor
- [( )] Solo debería poner las 5 skills más usadas
- [(X)] Las skills con progressive disclosure cargan solo ~50 tokens cada una al inicio (750 tokens total) vs ~75.000 tokens si estuvieran todas en CLAUDE.md, y la calidad de las respuestas sería peor con un contexto tan saturado
- [( )] Es indiferente, los tokens son baratos
***
**Correcto.** 15 skills × ~5.000 tokens = ~75.000 tokens permanentemente en el contexto de *cada sesión*. Con progressive disclosure: 15 × ~50 = 750 tokens al inicio + ~5.000 solo cuando se activa una. El ahorro es del 99%, y la calidad mejora porque el contexto está limpio.
***

---

## Siguiente: Módulo 7 — Seguridad y QA Integrados

En el próximo módulo profundizaremos en:

- El agente de seguridad como paso bloqueante del pipeline
- Quality gates automatizadas con hooks
- Análisis de dependencias y gestión de vulnerabilidades
- Testing con hipótesis competidoras usando Agent Teams
- El agente de QA como generador de edge cases

> **Tarea antes del Módulo 7:** Configura un hook PostToolUse que ejecute `npm audit` (o el equivalente de tu proyecto) cada vez que Claude escriba en `package.json`. Así tendrás una quality gate de seguridad de dependencias automática antes del próximo módulo.
