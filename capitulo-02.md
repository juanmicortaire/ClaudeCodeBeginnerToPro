<!--
author:   Juan Miguel (Juanmi) — Curso Claude Code Profesional
email:    curso-claude-code@example.com
version:  0.1.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 2 — Memoria y Gestión de Contexto.

-->

# Módulo 2: Memoria y Gestión de Contexto

> **"La diferencia entre un usuario casual de Claude Code y un profesional se mide en cómo gestiona el contexto."**

Cada sesión de Claude Code arranca con la ventana de contexto vacía. Si cada mañana re-explicas tu stack, tu arquitectura y tus convenciones, estás quemando tiempo y tokens antes de hacer trabajo real. Este módulo te enseña a construir la memoria que convierte a Claude Code en un compañero de equipo que *recuerda*.

    {{1}}
**Objetivo del módulo:** Dominar las tres capas de memoria, escribir un CLAUDE.md profesional, configurar Auto Memory y aplicar estrategias de gestión de contexto que maximicen la calidad de las respuestas.

    {{2}}
**Tiempo estimado:** 60-75 minutos

    {{3}}
**Prerrequisito:** Haber completado el Módulo 1 (Claude Code instalado y primera sesión realizada).

---

## 2.1 Las Tres Capas de Memoria

Claude Code tiene un sistema de memoria de tres capas. Todas se cargan al inicio de cada conversación. Entenderlas es la base de todo lo que sigue.

``` ascii
  Persistencia                    Quién lo escribe
       │                                │
       ▼                                ▼

  ┌──────────────────────────────────────────┐
  │  Capa 3: Memoria de Sesión              │  ◄── Se genera automáticamente
  │  (contexto activo de la conversación)    │      durante la conversación
  │  ► Tus prompts                           │
  │  ► Respuestas de Claude                  │  Vida: solo la sesión actual
  │  ► Resultados de herramientas            │  Se pierde al cerrar (salvo -c)
  └──────────────────────────────────────────┘
  ┌──────────────────────────────────────────┐
  │  Capa 2: Auto Memory (MEMORY.md)        │  ◄── Claude lo escribe solo
  │  (~/.claude/projects/<proyecto>/memory/) │
  │  ► Patrones observados                   │  Vida: indefinida
  │  ► Preferencias aprendidas               │  Local por máquina
  │  ► Insights de debugging                 │
  └──────────────────────────────────────────┘
  ┌──────────────────────────────────────────┐
  │  Capa 1: CLAUDE.md                       │  ◄── TÚ lo escribes
  │  (archivos en el proyecto y globales)     │
  │  ► Convenciones del proyecto              │  Vida: indefinida
  │  ► Comandos de build/test                 │  Compartido vía git
  │  ► Decisiones de arquitectura             │
  └──────────────────────────────────────────┘
```

La relación entre capas es complementaria: **CLAUDE.md** contiene tus requisitos explícitos, **Auto Memory** captura lo que Claude observa de tu forma de trabajar, y la **Memoria de Sesión** es el contexto vivo de la conversación actual.

#### Quiz: Capas de memoria

Relaciona cada capa con su característica principal:

[[CLAUDE.md]        [Auto Memory]     [Sesión]]
[(X)                ( )               ( )    ]  Lo escribes tú manualmente
[( )                (X)               ( )    ]  Claude lo escribe solo basándose en tus correcciones
[( )                ( )               (X)    ]  Se pierde al cerrar la sesión
[(X)                ( )               ( )    ]  Se comparte con el equipo vía git
[( )                (X)               ( )    ]  Es local por máquina

---

## 2.2 CLAUDE.md — La Constitución del Proyecto

CLAUDE.md es el archivo más importante del ecosistema Claude Code. Prevalece sobre cualquier instrucción dada en conversación. Si hay conflicto entre lo que dice CLAUDE.md y lo que le pides en un prompt, CLAUDE.md gana.

> Piensa en CLAUDE.md como la **constitución** de tu proyecto: las leyes fundamentales que ninguna instrucción temporal puede anular.

### La Jerarquía de Archivos CLAUDE.md

Claude Code lee múltiples archivos CLAUDE.md organizados en niveles. Los niveles se **combinan** (no se reemplazan). En caso de conflicto, el nivel más específico prevalece.

``` ascii
  Prioridad
  (más alta)
      │
      ▼
  ┌─────────────────────────────────────────────────────┐
  │ Nivel 4: Enterprise Policy                          │
  │ (Managed Policy — no se puede hacer override)       │
  │ Configurado por IT corporativo                      │
  └─────────────────────────────────────────────────────┘
  ┌─────────────────────────────────────────────────────┐
  │ Nivel 3: Directorio específico                      │
  │ /proyecto/frontend/CLAUDE.md                        │
  │ Reglas que solo aplican al frontend                 │
  └─────────────────────────────────────────────────────┘
  ┌─────────────────────────────────────────────────────┐
  │ Nivel 2: Proyecto                                   │
  │ /proyecto/CLAUDE.md                                 │
  │ Compartido vía git — todo el equipo lo ve           │
  └─────────────────────────────────────────────────────┘
  ┌─────────────────────────────────────────────────────┐
  │ Nivel 1: Global + Local                             │
  │ ~/.claude/CLAUDE.md (todos tus proyectos)           │
  │ /proyecto/CLAUDE.local.md (personal, gitignored)    │
  └─────────────────────────────────────────────────────┘
```

    {{1}}
**Global** (`~/.claude/CLAUDE.md`): Preferencias que aplican a todos tus proyectos. Ejemplo: "Siempre escribe mensajes de commit en inglés" o "Usa TypeScript strict mode".

    {{2}}
**Proyecto** (`/raíz/CLAUDE.md`): Contexto compartido con todo el equipo vía git. Contiene stack, convenciones, comandos de build, decisiones de arquitectura.

    {{3}}
**Local** (`/raíz/CLAUDE.local.md`): Preferencias personales que no quieres commitear. Está en `.gitignore` por defecto.

    {{4}}
**Directorio** (`/raíz/frontend/CLAUDE.md`): Reglas específicas para un subdirectorio. Útil en monorepos donde frontend y backend tienen convenciones distintas.

#### Quiz: Jerarquía

Tu CLAUDE.md de proyecto dice: "Usa tabs para indentación". Tu CLAUDE.md de directorio `/frontend/` dice: "Usa 2 espacios para indentación". Estás editando un archivo en `/frontend/`. ¿Qué regla aplica Claude?

- [( )] Tabs, porque el proyecto tiene prioridad sobre el directorio
- [(X)] 2 espacios, porque el nivel más específico prevalece
- [( )] Claude pregunta cuál usar
- [( )] Depende del modo de aprobación
***
**Correcto.** En la jerarquía de CLAUDE.md, el nivel más específico siempre gana. El archivo de directorio es más específico que el de proyecto, así que Claude usará 2 espacios cuando trabaje en archivos del frontend.
***

### Qué Incluir en CLAUDE.md

Un buen CLAUDE.md es conciso, específico y accionable. Aquí tienes una plantilla profesional:

``` markdown
# Proyecto: WeatherForecast Platform

## Stack
- Backend: Python 3.11 + FastAPI
- Frontend: React 18 + TypeScript strict
- Base de datos: Azure SQL + Azure Data Explorer
- Infra: Azure Kubernetes Service (AKS)
- CI/CD: Azure DevOps Pipelines

## Comandos
- Build: `npm run build` (frontend), `pip install -e .` (backend)
- Test: `pytest -x --tb=short` (backend), `npm test` (frontend)
- Lint: `ruff check .` (backend), `eslint .` (frontend)
- Format: `ruff format .` (backend), `prettier --write .` (frontend)

## Convenciones de Código
- Nombres de variables y funciones: snake_case (Python), camelCase (TS)
- Commits: Conventional Commits en inglés (feat:, fix:, docs:, etc.)
- Tests: Siempre escribir tests antes de implementar (TDD)
- Documentación: Docstrings Google-style en Python, JSDoc en TypeScript
- Manejo de errores: Nunca silenciar excepciones; siempre log + re-raise

## Arquitectura
- Ver @docs/architecture.md para detalles completos
- Seguir los patrones existentes en @src/features/auth/ para nuevas features
- Microservicios comunican vía Azure Service Bus (async)
- Datos meteorológicos en formato GRIB2/NetCDF procesados con xarray

## Restricciones
- NO commitear sin aprobación explícita del usuario
- NO modificar archivos en /infrastructure/production/
- NO instalar dependencias nuevas sin justificación documentada
- Siempre ejecutar tests antes de proponer un commit
```

    {{1}}
> **Observa las referencias con @path:** La línea `Ver @docs/architecture.md` le dice a Claude que cargue ese archivo cuando necesite contexto de arquitectura. Esto permite mantener CLAUDE.md conciso mientras la información detallada vive en archivos separados. Soporta hasta 5 niveles de recursión.

### Qué NO Incluir en CLAUDE.md

    {{1}}
**Información que cambia frecuentemente.** Si cambia cada semana, usa prompts en conversación, no CLAUDE.md. CLAUDE.md es para lo estable.

    {{2}}
**Datos sensibles.** CLAUDE.md se commitea en git. Nunca incluyas tokens, passwords, API keys o datos personales.

    {{3}}
**Instrucciones excesivamente largas.** Archivos de más de 200 líneas consumen demasiado contexto y reducen la adherencia de Claude a las instrucciones. Mueve el detalle a archivos separados con `@path`.

    {{4}}
**Descripciones del proyecto obvias desde el código.** No describas cada archivo del proyecto; Claude puede leerlos. Enfócate en lo que *no es evidente* del código: decisiones, restricciones y convenciones.

#### Quiz: Contenido de CLAUDE.md

¿Cuáles de los siguientes elementos deberían ir en CLAUDE.md? (Selecciona todos los correctos)

- [[X]] El comando para ejecutar los tests del proyecto
- [[ ]] La API key de producción de Azure
- [[X]] La convención de nombrado de variables
- [[ ]] La documentación completa de la API (500 líneas)
- [[X]] La restricción "No commitear sin aprobación"
- [[ ]] El changelog completo del último mes
- [[X]] Una referencia a `@docs/architecture.md` para detalles de arquitectura
***
**Correcto.** CLAUDE.md debe contener comandos de build/test, convenciones, restricciones y referencias a documentación detallada. Nunca debe incluir datos sensibles ni documentos extensos que se pueden referenciar con `@path`.
***

### Inicialización Automática con /init

Si estás empezando con un proyecto existente, Claude Code puede generar un CLAUDE.md inicial automáticamente:

``` bash
claude
> /init
```

Claude escanea el proyecto (package.json, requirements.txt, estructura de directorios, README existente) y genera un borrador. **Siempre revísalo y refínalo manualmente** — es un punto de partida, no el resultado final.

---

## 2.3 Auto Memory — Claude Aprende de Ti

Auto Memory es el sistema por el que Claude Code acumula conocimiento entre sesiones sin que tú escribas nada. Es el complemento perfecto de CLAUDE.md: tú defines los requisitos, Auto Memory captura lo que Claude observa.

### Cómo Funciona

``` ascii
  Sesión 1                    Sesión 2                    Sesión 3
  ─────────                   ─────────                   ─────────
  Tú: "No uses var,           Tú: "Recuerda: siempre      Claude detecta que
   usa const/let"              usa bun en vez de npm"      siempre corriges los
                                                           imports relativos a
  Claude observa ──►           Claude guarda ──►            absolutos ──►
  Guarda en MEMORY.md          Guarda en MEMORY.md          Guarda en MEMORY.md

                    ┌──────────────────────────────┐
                    │  ~/.claude/projects/          │
                    │    <proyecto>/memory/          │
                    │    ├── MEMORY.md    (índice)  │
                    │    ├── debugging.md            │
                    │    ├── api-conventions.md      │
                    │    └── workflow-habits.md      │
                    └──────────────────────────────┘
                              │
                              ▼
                    Se carga automáticamente
                    al inicio de cada sesión
                    (primeras 200 líneas de MEMORY.md)
```

    {{1}}
Claude no guarda algo en cada sesión. **Decide qué vale la pena recordar** basándose en si la información sería útil en una conversación futura.

    {{2}}
Lo que Claude suele guardar: comandos de build que descubre, insights de debugging, patrones de código que prefieres, convenciones de estilo observadas, y hábitos de workflow.

### Gestión Manual de Auto Memory

Puedes interactuar directamente con Auto Memory usando lenguaje natural:

| Acción | Ejemplo |
|:-------|:--------|
| **Añadir** | `Recuerda: siempre usa bun en vez de npm en este proyecto` |
| **Eliminar** | `Olvida la preferencia sobre usar tabs` |
| **Inspeccionar** | `/memory` (abre el directorio para ver qué ha guardado) |

    {{1}}
> **Importante:** Auto Memory requiere Claude Code v2.1.59 o posterior. Verifica con `claude --version`. Está activado por defecto; puedes togglearlo con `/memory` > auto memory toggle.

### Limitaciones de Auto Memory

    {{1}}
**Es local por máquina.** Si trabajas desde tu portátil y desde un servidor, cada uno tendrá su propia memoria. Para contexto compartido en equipo, usa CLAUDE.md commiteado en git.

    {{2}}
**Las primeras 200 líneas de MEMORY.md se cargan en cada sesión.** Si supera esa longitud, el contenido extra queda disponible pero no se inyecta automáticamente. Mantén MEMORY.md como un índice conciso que referencia archivos temáticos.

    {{3}}
**No sobrevive al borrado de `~/.claude/`.** Haz backup si es valioso. O mejor aún: si un aprendizaje es crítico, promuévelo a CLAUDE.md.

#### Quiz: Auto Memory

Llevas dos meses trabajando con Claude Code en un proyecto. Auto Memory ha acumulado insights valiosos sobre patrones de debugging. Ahora un nuevo compañero se une al equipo. ¿Cómo le pasas ese conocimiento?

- [( )] Le envías tu carpeta `~/.claude/projects/<proyecto>/memory/`
- [( )] Auto Memory se sincroniza automáticamente entre máquinas
- [(X)] Promueves los insights más valiosos a CLAUDE.md y lo commiteas en git
- [( )] No es posible compartir Auto Memory
***
**Correcto.** Auto Memory es local y personal. La forma profesional de compartir conocimiento acumulado es revisar `/memory`, identificar los insights más valiosos, y añadirlos a CLAUDE.md (que se commitea en git). Así todo el equipo se beneficia.
***

---

## 2.4 Reglas Modulares con .claude/rules/

Para proyectos grandes, un CLAUDE.md monolítico se vuelve difícil de mantener. La alternativa profesional son las reglas modulares.

``` text
.claude/
└── rules/
    ├── testing.md          # Reglas de testing
    ├── api-design.md       # Convenciones de API
    ├── security.md         # Políticas de seguridad
    ├── git-workflow.md     # Convenciones de git
    └── code-style.md       # Estilo de código
```

Cada archivo es un fragmento de CLAUDE.md que se carga automáticamente. Las ventajas son claras:

    {{1}}
**Separación de responsabilidades.** El lead de seguridad mantiene `security.md`; el lead de QA mantiene `testing.md`. Cada uno es dueño de sus reglas.

    {{2}}
**Code review granular.** Un cambio en reglas de testing no contamina el diff de reglas de API.

    {{3}}
**Activación condicional futura.** El roadmap de Claude Code incluye activación de reglas por patrón de archivo (similar a `.gitattributes`).

#### Quiz: Organización de reglas

Tienes un monorepo con 15 microservicios, un frontend React y una app móvil Flutter. ¿Qué estructura de CLAUDE.md sería más mantenible?

- [( )] Un único CLAUDE.md de 500 líneas en la raíz
- [( )] Un CLAUDE.md por microservicio (15 archivos) sin reglas compartidas
- [(X)] CLAUDE.md raíz conciso + `.claude/rules/` para reglas compartidas + CLAUDE.md en cada subdirectorio para reglas específicas
- [( )] No usar CLAUDE.md; confiar solo en Auto Memory
***
**Correcto.** La estructura óptima combina un CLAUDE.md raíz lean (stack general, restricciones globales) con reglas modulares en `.claude/rules/` para convenciones compartidas, y CLAUDE.md de directorio donde las tecnologías difieren (Flutter vs React vs Python tienen convenciones distintas).
***

---

## 2.5 Gestión de la Ventana de Contexto

Claude Code usa una ventana de contexto de 200K tokens. Pero el tamaño bruto importa menos que la gestión eficiente. Un contexto saturado degrada la calidad de las respuestas de forma medible.

### Distribución del Presupuesto de Contexto

``` ascii
  Ventana de contexto (200K tokens)
  ══════════════════════════════════════════════════

  ┌─────────────────────────┐
  │ System prompt +         │  ~2-5K tokens (fijo)
  │ CLAUDE.md               │
  ├─────────────────────────┤
  │ Auto Memory (MEMORY.md) │  ~1-3K tokens (fijo)
  ├─────────────────────────┤
  │                         │
  │ Historial de            │  Crece con cada
  │ conversación            │  interacción:
  │  • Tus prompts          │   • Un prompt: ~100-500
  │  • Respuestas de Claude │   • Una respuesta: ~500-2K
  │  • Resultados de tools  │   • Una lectura de
  │                         │     archivo: 1-20K ◄──────
  │                         │                    CUIDADO
  ├─────────────────────────┤
  │                         │
  │ Espacio libre para      │  Lo que queda para
  │ trabajo nuevo           │  razonamiento y respuesta
  │                         │
  └─────────────────────────┘
```

    {{1}}
> **El dato clave:** Los resultados de herramientas son los mayores consumidores de contexto. Una sola lectura de un archivo grande puede consumir 10-20K tokens. Un `cat` de un directorio entero puede llenar el contexto de golpe.

### Los Tres Comandos de Supervivencia

#### /cost — "¿Cuánto llevo gastado?"

``` text
> /cost

  Session cost:
  Input tokens:  45,230
  Output tokens: 12,108
  Total cost:    $0.47
  Time active:   32 min
  Time waiting:  18 min
```

**Cuándo usarlo:** Cada 30-45 minutos de trabajo intenso. Si el input supera 50K tokens, es hora de actuar.

#### /compact — "Resume sin perder lo importante"

`/compact` comprime la conversación: Claude resume el historial preservando la información esencial y re-inyecta CLAUDE.md fresco desde disco.

    {{1}}
**Lo que sobrevive a /compact:**

- CLAUDE.md (se re-lee desde disco, siempre intacto)
- Un resumen del historial de conversación
- Auto Memory

    {{2}}
**Lo que se puede perder:**

- Detalles específicos de lecturas de archivos
- Resultados exactos de comandos bash
- Instrucciones que diste solo en conversación (no en CLAUDE.md)

    {{3}}
> **Regla de oro:** Si una instrucción es importante para que sobreviva a `/compact`, ponla en CLAUDE.md. Lo que solo existe en conversación es efímero.

#### /clear — "Borrón y cuenta nueva"

`/clear` elimina todo el contexto de sesión. CLAUDE.md y Auto Memory persisten (son archivos en disco, no contexto de conversación).

**Cuándo usarlo:** Al cambiar radicalmente de tarea. Arrastrar contexto de autenticación mientras depuras el módulo de pagos solo confunde a Claude.

#### Quiz: Gestión de contexto

Estás trabajando en una feature compleja. Has leído 8 archivos, ejecutado tests y acumulado 85K tokens según `/cost`. Todavía necesitas seguir con la misma feature. ¿Qué comando usas?

- [(X)] `/compact` para comprimir sin perder el hilo
- [( )] `/clear` para empezar limpio
- [( )] Nada, 85K está dentro del límite de 200K
- [( )] Cerrar y abrir con `claude -c`
***
**Correcto.** Como sigues trabajando en la misma feature, `/compact` es la elección: resume el contexto manteniendo el hilo de la conversación. `/clear` sería apropiado si cambiaras de tarea. Y aunque 85K está dentro del límite técnico, a partir de ~100K la calidad de las respuestas empieza a degradarse porque Claude tiene menos "espacio mental" para razonar.
***

---

## 2.6 La Rutina de Sesión Óptima

Después de experimentar con decenas de proyectos, esta es la rutina que maximiza la productividad:

``` ascii
  ┌─ INICIO DE SESIÓN ──────────────────────────────┐
  │                                                   │
  │  1. Abrir sesión (claude, o claude -c)            │
  │  2. Contexto: "¿Qué hicimos ayer en [feature]?"  │
  │  3. Definir tarea: "Hoy implementamos [X]"        │
  │                                                   │
  ├─ TRABAJO (bloques de 30-45 min) ─────────────────┤
  │                                                   │
  │  4. Trabajar en la tarea                          │
  │  5. /cost cada 30-45 minutos                      │
  │     Si >50K tokens ──► /compact                   │
  │     Si cambias de tema ──► /clear                 │
  │                                                   │
  ├─ FIN DE SESIÓN ──────────────────────────────────┤
  │                                                   │
  │  6. "¿Cuáles fueron las decisiones clave?"        │
  │  7. Actualizar CLAUDE.md con decisiones nuevas    │
  │  8. Auto Memory guardará el resto solo            │
  │                                                   │
  └──────────────────────────────────────────────────┘
```

    {{1}}
> **El hábito de mayor impacto:** Dedicar 2 minutos al final de cada sesión a actualizar CLAUDE.md con las decisiones del día. Esto compone. Después de un mes, tienes un registro buscable de todas las decisiones. Después de seis meses, Claude puede responder preguntas sobre el historial del proyecto que tú mismo has olvidado.

### Ejemplo Real: Flujo de una Mañana Productiva

    {{1}}
**8:30 — Inicio**

``` text
$ claude -c
> Resúmeme qué hicimos ayer en la feature de autenticación
```

    {{2}}
**8:35 — Definir tarea**

``` text
> Hoy necesitamos implementar el endpoint de refresh token.
> La spec está en @docs/specs/auth-refresh.md
```

    {{3}}
**9:15 — Check de contexto**

``` text
> /cost
  Input: 38K tokens
```

Bien, seguimos.

    {{4}}
**10:00 — Segundo check**

``` text
> /cost
  Input: 67K tokens
> /compact
```

    {{5}}
**10:45 — Cambio de tarea**

``` text
> /clear
> Ahora necesito depurar el bug #347 en el módulo de pagos
```

    {{6}}
**12:00 — Cierre**

``` text
> ¿Cuáles fueron las decisiones clave de esta mañana?
> Añade a CLAUDE.md: "Refresh tokens expiran a las 24h,
>   usan rotación automática, se almacenan en HttpOnly cookies"
```

---

## 2.7 Antipatrones: Lo Que NO Hacer

Estos son los errores más comunes que vemos en equipos que adoptan Claude Code:

    {{1}}
**Antipatrón 1: CLAUDE.md vacío o inexistente.**
Claude arranca cada sesión sin saber nada de tu proyecto. Re-explicas lo mismo cada vez. Coste: cientos de tokens desperdiciados diariamente.

    {{2}}
**Antipatrón 2: CLAUDE.md de 400+ líneas.**
Intentas documentar cada decisión del proyecto en CLAUDE.md. Claude pierde foco, las instrucciones se contradicen, y el archivo consume demasiado contexto. Solución: mantenerlo bajo 200 líneas, usar `@path` para detalles y `.claude/rules/` para reglas modulares.

    {{3}}
**Antipatrón 3: Nunca usar /compact ni /clear.**
Dejas que el contexto crezca sin control hasta que Claude empieza a "olvidar" instrucciones de CLAUDE.md o a dar respuestas inconsistentes. El modelo no falla, simplemente tiene demasiado ruido en el contexto.

    {{4}}
**Antipatrón 4: Confiar solo en Auto Memory.**
Auto Memory es un complemento, no un sustituto de CLAUDE.md. Los aprendizajes críticos deben estar en CLAUDE.md (compartido, versionado, explícito), no solo en Auto Memory (local, implícito, no versionado).

    {{5}}
**Antipatrón 5: No actualizar CLAUDE.md tras decisiones importantes.**
Tomas una decisión de arquitectura en conversación, pero no la persistes en CLAUDE.md. En la próxima sesión, Claude no la recuerda y propone algo contradictorio.

#### Quiz: Antipatrones

Tu compañero tiene un CLAUDE.md de 380 líneas que incluye la documentación completa de la API REST, el schema de la base de datos y todas las decisiones de arquitectura de los últimos 6 meses. ¿Qué le recomendarías?

- [( )] Está bien, más contexto es mejor
- [( )] Borrar CLAUDE.md y usar solo Auto Memory
- [(X)] Reducirlo a ~100 líneas con referencias `@path` a la documentación detallada, y mover reglas a `.claude/rules/`
- [( )] Dividirlo en 10 archivos CLAUDE.md en diferentes directorios
***
**Correcto.** CLAUDE.md debe ser un índice conciso con lo esencial para cualquier sesión. La documentación detallada (API docs, schemas) debe vivir en archivos separados referenciados con `@path`, que Claude carga bajo demanda solo cuando los necesita. Las reglas por dominio van en `.claude/rules/`.
***

---

## 2.8 Ejercicio Práctico: Construye tu CLAUDE.md

Es hora de crear un CLAUDE.md profesional para tu proyecto. Sigue estos pasos:

    {{1}}
**Paso 1: Inicializa (si no tienes uno)**

``` bash
cd tu-proyecto
claude
> /init
```

Revisa el borrador que genera Claude. Es un punto de partida.

    {{2}}
**Paso 2: Refina con esta checklist**

Verifica que tu CLAUDE.md incluye:

- [ ] Nombre y descripción breve del proyecto (1-2 líneas)
- [ ] Stack tecnológico completo
- [ ] Comandos de build, test, lint y format
- [ ] Convenciones de código (naming, patrones, estilo de commits)
- [ ] Referencias a documentación con `@path`
- [ ] Restricciones explícitas (qué NO debe hacer Claude)
- [ ] Que el total sea **menor de 200 líneas**

    {{3}}
**Paso 3: Crea reglas modulares si tu proyecto lo justifica**

``` bash
mkdir -p .claude/rules
```

Mueve a archivos separados las secciones temáticas (testing, seguridad, API design).

    {{4}}
**Paso 4: Verifica en una sesión nueva**

``` bash
claude
> ¿Qué sabes de este proyecto?
```

Claude debería poder resumir tu stack, convenciones y restricciones sin que le expliques nada. Si no puede, tu CLAUDE.md necesita más trabajo.

---

## 2.9 Resumen del Módulo

En este módulo has aprendido:

- [X] Las tres capas de memoria: CLAUDE.md (tú escribes), Auto Memory (Claude escribe) y Sesión (temporal)
- [X] La jerarquía de archivos CLAUDE.md: Global → Proyecto → Local → Directorio
- [X] Qué incluir y qué no incluir en CLAUDE.md
- [X] Cómo funciona Auto Memory y sus limitaciones
- [X] Reglas modulares con `.claude/rules/`
- [X] Distribución del presupuesto de contexto (200K tokens)
- [X] Los tres comandos de supervivencia: `/cost`, `/compact`, `/clear`
- [X] La rutina de sesión óptima
- [X] Los 5 antipatrones más comunes

---

## Evaluación Final del Módulo 2

**Pregunta 1:** ¿Qué ocurre con CLAUDE.md cuando ejecutas `/compact`?

- [( )] Se pierde junto con el resto del contexto
- [( )] Se comprime en un resumen
- [(X)] Se re-lee desde disco y se inyecta intacto
- [( )] Se fusiona con Auto Memory
***
**Correcto.** CLAUDE.md sobrevive completamente a `/compact`. Claude lo re-lee desde el archivo en disco y lo re-inyecta fresco en la sesión. Esta es la razón por la que las instrucciones importantes deben estar en CLAUDE.md: son inmunes a la compactación.
***

**Pregunta 2:** ¿Cuántas líneas debería tener como máximo un CLAUDE.md bien diseñado?

[[200]]
[[?]] El número está asociado a la longitud máxima recomendada para mantener buena adherencia
***
**Correcto.** Archivos de más de 200 líneas consumen demasiado contexto y reducen la adherencia de Claude a las instrucciones. La información detallada debe ir en archivos separados referenciados con `@path`.
***

**Pregunta 3:** Tienes un proyecto React con 50K líneas de código. Al inicio de la sesión, la ventana de contexto se distribuye así: System prompt + CLAUDE.md (~4K), Auto Memory (~2K), espacio libre (~194K). Después de leer 5 archivos grandes y ejecutar 3 comandos, `/cost` muestra 78K tokens de input. ¿Cuánto espacio libre te queda aproximadamente para razonamiento?

- [( )] 200K - 78K = 122K tokens
- [(X)] Aproximadamente 122K tokens, pero la calidad empieza a degradarse a partir de ~100K de input
- [( )] 0 tokens, el contexto está lleno
- [( )] 194K tokens, la lectura de archivos no cuenta
***
**Correcto.** Matemáticamente quedan ~122K tokens, pero en la práctica la calidad del razonamiento degrada progresivamente cuando el input supera ~100K. Por eso la recomendación es compactar proactivamente a los 50K, no esperar a llenar la ventana.
***

**Pregunta 4:** Ordena estas acciones de mayor a menor consumo de tokens:

[[Lectura de archivo grande]   [Respuesta de Claude]   [Tu prompt]   [CLAUDE.md]]
[(X)                           ( )                     ( )           ( )        ]  10.000-20.000 tokens
[( )                           (X)                     ( )           ( )        ]  500-2.000 tokens
[( )                           ( )                     (X)           ( )        ]  100-500 tokens
[( )                           ( )                     ( )           (X)        ]  2.000-5.000 tokens (fijo)

**Pregunta 5:** Un colega nuevo se queja: "Claude Code no recuerda nada entre sesiones, tengo que re-explicar todo cada mañana". ¿Cuál es la causa más probable y la solución?

- [( )] Bug en Claude Code; hay que reinstalar
- [( )] Auto Memory no funciona en su versión
- [(X)] No tiene CLAUDE.md configurado; debe crear uno con /init y refinarlo
- [( )] Necesita usar `claude -c` para continuar sesiones
***
**Correcto.** El síntoma clásico de un CLAUDE.md inexistente es tener que re-explicar el proyecto cada sesión. La solución es crear un CLAUDE.md con `/init`, refinarlo con las convenciones del equipo, y commitearlo en git. `claude -c` solo continúa la sesión anterior (contexto de conversación), no resuelve el problema de fondo.
***

---

## Siguiente: Módulo 3 — Extensibilidad: Skills, Hooks y MCP

En el próximo módulo aprenderás:

- Slash commands para automatizar workflows manuales
- Skills con progressive disclosure (ahorro del 98% en tokens)
- Hooks como quality gates automáticas
- MCP (Model Context Protocol) para conectar con servicios externos
- Plugins para empaquetar y compartir tu setup

> **Tarea antes del Módulo 3:** Crea un slash command simple en `.claude/commands/review.md` con el contenido: `Revisa el código del archivo $ARGUMENTS buscando problemas de seguridad, rendimiento y estilo.` Prueba invocarlo con `/project:review src/auth.js` en tu próxima sesión.
