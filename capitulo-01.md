<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 1 — Fundamentos y Setup Inicial.

link:     https://cdn.jsdelivr.net/chartist.js/latest/chartist.min.css
script:   https://cdn.jsdelivr.net/chartist.js/latest/chartist.min.js

-->

# Curso Claude Code: De Principiante a Profesional

![Claude Code Logo](https://img.shields.io/badge/Claude_Code-Curso_Profesional-blueviolet?style=for-the-badge&logo=anthropic)

> Este curso te llevará desde la instalación básica de Claude Code hasta la orquestación profesional de agentes especializados que colaboran entre sí para gestionar el ciclo de vida completo del desarrollo de software.

**Módulos del curso:**

1. **Fundamentos y Setup Inicial** ← _Estás aquí_
2. Memoria y Gestión de Contexto
3. Extensibilidad: Skills, Hooks y MCP
4. Agentes Especializados
5. Workflows Profesionales
6. Optimización de Tokens
7. Seguridad y QA Integrados
8. Patrones Avanzados

---

## Módulo 1: Fundamentos y Setup Inicial

    {{1}}
> **Objetivo del módulo:** Al finalizar, tendrás Claude Code instalado, configurado y habrás completado tu primera sesión interactiva entendiendo los conceptos fundamentales de la herramienta.

    {{2}}
**Tiempo estimado:** 45-60 minutos

    {{3}}
**Prerrequisitos:** Conocimientos básicos de terminal/línea de comandos y experiencia general en desarrollo de software.

### 1.1 ¿Qué es Claude Code?

Claude Code no es un chatbot que sugiere código en aislamiento. Es un **agente de desarrollo de IA** que opera directamente dentro de tu repositorio, con acceso completo al sistema de archivos, ejecución de comandos bash y herramientas de edición de código.

    {{1}}
**Piensa en la diferencia así:**

    {{1}}
| Característica | Chatbot tradicional | Claude Code |
|:---------------|:-------------------:|:-----------:|
| Acceso al código | Copia-pega | Lee el repo completo |
| Ejecución de comandos | No | Sí (bash, git, npm...) |
| Edición de archivos | No | Sí (con diffs) |
| Gestión de Git | No | Commits, PRs, branches |
| Tests | No | Ejecuta y valida |
| Contexto del proyecto | Lo que le pegas | Lee CLAUDE.md + auto-memoria |

    {{2}}
> **Evolución rápida:** Claude Code se lanzó en febrero de 2025 como herramienta de terminal. En abril de 2026, es una plataforma extensible con hooks, skills, subagentes, agent teams y un ecosistema completo de plugins.

#### Quiz: Concepto fundamental

¿Cuál es la diferencia principal entre Claude Code y un chatbot de IA convencional?

- [( )] Claude Code genera código más rápido
- [(X)] Claude Code opera directamente dentro del repositorio con acceso al sistema de archivos
- [( )] Claude Code usa un modelo de IA más avanzado
- [( )] Claude Code solo funciona con JavaScript
***
**Correcto.** La diferencia fundamental es que Claude Code no es un asistente externo al que le pegas código. Es un agente que vive *dentro* de tu proyecto: lee archivos, ejecuta comandos, crea commits y gestiona PRs de forma autónoma.
***

### 1.2 Superficies de Acceso

Claude Code está disponible en múltiples formatos. Todos comparten el mismo motor, pero cada uno ofrece integraciones distintas con tu entorno de trabajo.

    {{1}}
**CLI (Terminal):**
La interfaz original y más potente. Se instala globalmente con npm y se ejecuta directamente en cualquier repositorio. Ideal para desarrolladores que viven en la terminal.

    {{2}}
**Desktop App:**
Aplicación de escritorio independiente con diffs visuales, preview en vivo de aplicaciones, computer use y tareas programadas.

    {{3}}
**Extensión VS Code:**
Se lanza con `Cmd+Esc` (Mac) o `Ctrl+Esc` (Windows/Linux). Comparte selección de texto, pestañas abiertas y diagnósticos automáticamente con Claude.

    {{4}}
**Plugin JetBrains:**
Compatible con IntelliJ, PyCharm, WebStorm, CLion y Rider. Incluye visor interactivo de diffs.

#### Quiz: Superficies

Conecta cada superficie con su característica principal:

[[CLI]     [Desktop]  [VS Code]  [JetBrains]]
[(X)       ( )        ( )        ( )       ]  Se instala con `npm install -g`
[( )       (X)        ( )        ( )       ]  Incluye preview en vivo de aplicaciones
[( )       ( )        (X)        ( )       ]  Se activa con Cmd+Esc
[( )       ( )        ( )        (X)       ]  Compatible con PyCharm y WebStorm

### 1.3 Modelos Disponibles

Claude Code soporta varios modelos de la familia Claude. La selección del modelo correcto para cada tarea es una de las claves de la optimización profesional que veremos en módulos avanzados.

| Modelo | Fortaleza | Coste relativo | Caso de uso |
|:-------|:----------|:--------------:|:------------|
| **Opus 4.6** | Razonamiento profundo | $$$ | Arquitectura, debugging complejo, Agent Teams |
| **Sonnet 4.6** | Equilibrio velocidad/capacidad | $$ | Desarrollo cotidiano, code review, refactoring |
| **Haiku 4.5** | Máxima eficiencia | $ | Linting, docs boilerplate, tareas rutinarias |

> **Clave profesional:** No todas las tareas requieren el modelo más potente. Un agente de code review puede usar Haiku (barato, read-only) mientras el agente de arquitectura usa Opus. Aprenderemos esto en el Módulo 6.

Para cambiar de modelo durante una sesión, usa el comando:

``` bash
/model
```

#### Quiz: Selección de modelo

Estás diseñando un sistema multi-agente. ¿Qué modelo asignarías a un agente cuya única tarea es ejecutar `git diff` y verificar estilo de código?

- [( )] Opus 4.6, porque necesita máxima inteligencia
- [( )] Sonnet 4.6, porque es el modelo por defecto
- [(X)] Haiku 4.5, porque es una tarea rutinaria de solo lectura
***
**Correcto.** Para tareas repetitivas de solo lectura como verificación de estilo, Haiku es la elección óptima: consume menos tokens y es más rápido, sin sacrificar calidad en tareas simples y bien definidas.
***

### 1.4 Instalación Paso a Paso

Sigue estos pasos para tener Claude Code operativo en tu máquina.

#### Paso 1: Verificar prerrequisitos

Antes de instalar, verifica que tienes Node.js 18+ y npm:

``` bash
node --version   # Debe ser >= 18.0.0
npm --version    # Cualquier versión reciente
```

> **Si no tienes Node.js:** Descárgalo desde [nodejs.org](https://nodejs.org) o usa un gestor de versiones como `nvm`.

#### Paso 2: Instalar Claude Code

``` bash
npm install -g @anthropic-ai/claude-code
```

Verifica la instalación:

``` bash
claude --version
```

#### Paso 3: Primera autenticación

Ejecuta `claude` en cualquier directorio. Se abrirá el navegador para vincular tu cuenta de Anthropic:

``` bash
cd tu-proyecto
claude
```

> **Nota:** Necesitas un plan de Anthropic que incluya Claude Code (Pro, Max o Enterprise).

#### Paso 4: Tu primera interacción

Una vez autenticado, Claude Code muestra un prompt interactivo. Prueba con algo simple:

```
> ¿Cuántos archivos .js hay en este proyecto?
```

Claude ejecutará un comando como `find . -name "*.js" | wc -l` (pidiéndote aprobación primero) y te dará el resultado.

#### Quiz: Instalación

¿Cuál es el comando correcto para instalar Claude Code de forma global?

[[npm install -g @anthropic-ai/claude-code]]
[[?]] El paquete está en el registry de npm bajo el scope @anthropic-ai
[[?]] Recuerda que la flag -g instala de forma global
***
**Correcto.** Claude Code se distribuye como paquete npm bajo el scope `@anthropic-ai`. La instalación global (`-g`) hace que el comando `claude` esté disponible en cualquier directorio de tu sistema.
***

### 1.5 El Sistema de Permisos

    {{0-1}}
Claude Code solicita aprobación antes de ejecutar acciones. Esto es fundamental para la seguridad. Antes de que Claude borre un archivo, ejecute un comando destructivo o modifique producción, te lo preguntará.

    {{1}}
Existen tres modos de aprobación:

    {{1}}
**Manual (por defecto):**
Apruebas cada acción individualmente. Máximo control, menor velocidad.

    {{2}}
**Auto Mode:**
Un clasificador IA decide qué acciones son seguras. Permite automáticamente lecturas, `git status` y ejecución de tests. Bloquea force push, borrado masivo y deploys a producción.

    {{3}}
**YOLO Mode:**
Todo aprobado automáticamente. Solo usar en entornos desechables como containers de CI.

    {{4}}
> **Recomendación profesional:** Para trabajo cotidiano, usa Auto Mode. Para producción y CI/CD, configura permisos explícitos en `settings.json`.

#### Configuración de permisos en settings.json

Puedes definir listas explícitas de herramientas permitidas y denegadas:

``` json
{
  "permissions": {
    "allowedTools": [
      "Read",
      "Write",
      "Bash(git *)",
      "Bash(npm test)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Write(./production.config.*)"
    ]
  }
}
```

> **Lectura del código:** Este JSON permite a Claude leer y escribir archivos, ejecutar comandos git y npm test, pero le prohíbe leer archivos `.env`, ejecutar `rm -rf`, usar `sudo` o escribir en archivos de configuración de producción.

#### Quiz: Permisos

¿Qué modo de aprobación usarías en un pipeline de CI/CD automatizado con un container desechable?

- [( )] Manual, porque la seguridad es lo primero
- [( )] Auto Mode, porque es el equilibrio perfecto
- [(X)] Permisos explícitos en settings.json con `--allowedTools`
- [( )] YOLO Mode sin restricciones
***
**Correcto.** En CI/CD, YOLO Mode es tentador pero peligroso incluso en containers. La práctica profesional es usar `--allowedTools` para definir exactamente qué puede hacer Claude: `claude -p "run tests and fix failures" --allowedTools "Read,Write,Bash(npm test)"`. Esto da autonomía controlada.
***

### 1.6 Anatomía de una Sesión

Entender el flujo de una sesión de Claude Code te permite trabajar de forma eficiente desde el primer momento.

```
┌─────────────────────────────────────────────┐
│           INICIO DE SESIÓN                  │
├─────────────────────────────────────────────┤
│ 1. Lee CLAUDE.md (si existe)                │
│ 2. Carga Auto Memory (si existe)            │
│ 3. Carga Skills disponibles (solo nombres)  │
│ 4. Conecta servidores MCP configurados      │
│ 5. Muestra el prompt interactivo            │
├─────────────────────────────────────────────┤
│           DURANTE LA SESIÓN                 │
├─────────────────────────────────────────────┤
│ • Tu prompt → Claude razona → Usa tools     │
│ • Cada tool use pide aprobación (o auto)    │
│ • Los resultados se añaden al contexto      │
│ • El contexto crece con cada interacción    │
├─────────────────────────────────────────────┤
│           GESTIÓN DE CONTEXTO               │
├─────────────────────────────────────────────┤
│ /cost    → Ver tokens consumidos            │
│ /compact → Comprimir contexto               │
│ /clear   → Reinicio limpio                  │
├─────────────────────────────────────────────┤
│           FIN DE SESIÓN                     │
├─────────────────────────────────────────────┤
│ Auto Memory guarda aprendizajes relevantes  │
│ El contexto de sesión se pierde             │
│ CLAUDE.md persiste en disco                 │
└─────────────────────────────────────────────┘
```

#### Comandos esenciales de sesión

| Comando | Función | Cuándo usar |
|:--------|:--------|:------------|
| `/cost` | Muestra tokens consumidos | Cada 30-45 min de trabajo |
| `/compact` | Resume el contexto preservando lo esencial | Cuando `/cost` muestra >50K tokens |
| `/clear` | Limpia todo el contexto | Al cambiar de tarea |
| `/model` | Cambia el modelo activo | Cuando necesitas más/menos potencia |
| `/help` | Lista todos los comandos disponibles | Cuando estés perdido |
| `claude -c` | Continúa la última sesión | Al día siguiente, para retomar |

#### Quiz: Gestión de sesión

Llevas 45 minutos trabajando en una feature de autenticación. `/cost` muestra 62K tokens. Ahora necesitas cambiar a un bug completamente diferente en el módulo de pagos. ¿Qué haces?

- [( )] `/compact` y continuar en la misma sesión
- [(X)] `/clear` para limpiar el contexto contaminado y empezar fresco
- [( )] Cerrar la terminal y abrir una nueva sesión con `claude`
- [( )] No hacer nada, 62K tokens está bien
***
**Correcto.** Cuando cambias radicalmente de tarea, `/clear` es mejor que `/compact`. Compactar preserva un resumen de la conversación anterior, que en este caso es irrelevante y contaminaría el contexto del nuevo bug. `/clear` te da un inicio limpio pero mantiene CLAUDE.md y Auto Memory, que sí son relevantes para cualquier tarea del proyecto.
***

### 1.7 Ejercicio Práctico: Tu Primera Sesión Completa

    {{0}}
Es hora de poner en práctica lo aprendido. Sigue estos pasos en tu terminal:

    {{1}}
**1. Crea un proyecto de prueba:**

``` bash
mkdir mi-primer-proyecto-claude
cd mi-primer-proyecto-claude
git init
npm init -y
```

    {{2}}
**2. Crea un archivo simple:**

``` bash
echo 'function suma(a, b) { return a + b; }' > math.js
echo 'module.exports = { suma };' >> math.js
```

    {{3}}
**3. Inicia Claude Code:**

``` bash
claude
```

    {{4}}
**4. Prueba estas interacciones (en orden):**

``` text
> Lee el archivo math.js y dime qué hace

> Añade una función de multiplicación al archivo

> Crea tests unitarios para todas las funciones

> Ejecuta los tests

> /cost
```

    {{5}}
**5. Observa:**

- Cómo Claude pide permiso antes de leer, escribir y ejecutar
- Cómo el contexto crece con cada interacción
- El coste en tokens de cada operación

    {{6}}
> **Reflexión:** Si este ejercicio simple consume ~5K tokens, imagina un proyecto real con cientos de archivos. La gestión de contexto que veremos en el Módulo 2 no es un lujo, es una necesidad.

### 1.8 Resumen del Módulo

En este primer módulo has aprendido:

- [X] Qué es Claude Code y en qué se diferencia de un chatbot convencional
- [X] Las cuatro superficies de acceso (CLI, Desktop, VS Code, JetBrains)
- [X] Los tres modelos disponibles y cuándo usar cada uno
- [X] Cómo instalar y autenticarte por primera vez
- [X] El sistema de permisos y los tres modos de aprobación
- [X] La anatomía de una sesión y los comandos esenciales
- [X] Tu primera sesión práctica completa

### Evaluación Final del Módulo 1

Completa esta evaluación para verificar tu comprensión antes de avanzar al Módulo 2.

**Pregunta 1:** ¿Qué archivo lee Claude Code automáticamente al inicio de cada sesión para obtener contexto del proyecto?

[[CLAUDE.md]]
[[?]] Es un archivo Markdown en la raíz del proyecto
[[?]] Su nombre es el mismo que la herramienta, en mayúsculas
***
**Correcto.** `CLAUDE.md` es la "constitución" del proyecto. Claude lo lee automáticamente al arrancar cada sesión. En el Módulo 2 aprenderemos a escribir un CLAUDE.md eficaz.
***

**Pregunta 2:** ¿Cuántos tokens de contexto soporta la ventana de Claude Code?

- [( )] 50.000 tokens
- [( )] 100.000 tokens
- [(X)] 200.000 tokens
- [( )] 1.000.000 tokens
***
**Correcto.** Claude Code usa la ventana de contexto completa del modelo: 200K tokens. Pero como veremos en el Módulo 2, la gestión eficiente importa más que el tamaño bruto.
***

**Pregunta 3:** Relaciona cada comando con su función:

[[/cost]     [/compact]   [/clear]     [/model]]
[(X)         ( )          ( )          ( )     ]  Ver tokens consumidos en la sesión
[( )         (X)          ( )          ( )     ]  Resumir el contexto preservando lo esencial
[( )         ( )          (X)          ( )     ]  Limpiar todo el contexto para nueva tarea
[( )         ( )          ( )          (X)     ]  Cambiar el modelo de IA activo

**Pregunta 4:** Un compañero de equipo configura Claude Code con YOLO Mode activado en su máquina de desarrollo local conectada a la base de datos de producción. ¿Qué le dirías?

- [( )] Perfecto, YOLO Mode es más productivo
- [( )] Está bien si tiene backups de la base de datos
- [(X)] Es peligroso: YOLO Mode + acceso a producción = riesgo real de daño
- [( )] No importa, Claude Code nunca ejecuta comandos destructivos
***
**Correcto.** YOLO Mode aprueba *todas* las acciones sin preguntar. Con acceso a producción, Claude podría ejecutar migraciones, borrar datos o modificar configuración sin confirmación. La práctica profesional es usar Auto Mode o permisos explícitos, y **nunca** dar acceso directo a producción sin restricciones.
***

### Siguiente: Módulo 2 — Memoria y Gestión de Contexto

En el próximo módulo aprenderás:

- Las tres capas de memoria de Claude Code
- Cómo escribir un CLAUDE.md profesional
- Auto Memory y cómo Claude aprende de tus correcciones
- Estrategias de gestión de la ventana de contexto
- La rutina de sesión óptima

> **Tarea antes del Módulo 2:** Crea un archivo `CLAUDE.md` básico en tu proyecto de prueba con el stack tecnológico y las convenciones de tu equipo. En el próximo módulo lo revisaremos y optimizaremos juntos.
