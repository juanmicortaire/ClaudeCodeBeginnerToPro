# Curso Claude Code: De Principiante a Profesional

Curso interactivo creado con [LiaScript](https://liascript.github.io) para aprender Claude Code desde la instalación básica hasta la orquestación profesional de agentes especializados.

## Previsualizar el curso

### Opción 1: Online (sin instalar nada)

Abre este enlace en tu navegador (reemplaza `TU_USUARIO` con tu usuario de GitHub tras subir el repo):

```
https://liascript.github.io/course/?https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/main/capitulo-01.md
```

### Opción 2: Extensión VS Code

1. Instala la extensión **LiaScript Preview** desde el marketplace de VS Code
2. Abre `capitulo-01.md`
3. Abre la paleta de comandos (`Cmd+Shift+P` / `Ctrl+Shift+P`)
4. Busca "LiaScript: Preview"
5. El preview se actualiza en tiempo real al guardar

### Opción 3: Servidor local de desarrollo

```bash
npm install -g @liascript/devserver
liadev --input capitulo-01.md
```

Se abrirá en `http://localhost:3000` con hot-reload.

## Exportar a SCORM (para Moodle u otro LMS)

```bash
# Instalar el exportador
npm install -g @liascript/exporter

# Exportar a SCORM 1.2
liaex -i capitulo-01.md --format scorm1.2 -o curso-modulo1 \
  --scorm-organization "Curso Claude Code Profesional"

# Exportar a SCORM 2004
liaex -i capitulo-01.md --format scorm2004 -o curso-modulo1

# Exportar a web standalone
liaex -i capitulo-01.md --format web -o curso-modulo1-web

# Exportar a PDF
liaex -i capitulo-01.md --format pdf -o curso-modulo1
```

El archivo `.zip` resultante se puede subir directamente a Moodle, ILIAS, Blackboard u otro LMS compatible con SCORM.

## Estructura del curso

```
curso-claude-code/
├── README.md                  ← Este archivo
├── capitulo-01.md             ← Módulo 1: Fundamentos y Setup
├── capitulo-02.md             ← Módulo 2: Memoria y Contexto (próximo)
├── capitulo-03.md             ← Módulo 3: Skills, Hooks y MCP
├── capitulo-04.md             ← Módulo 4: Agentes Especializados
├── capitulo-05.md             ← Módulo 5: Workflows Profesionales
├── capitulo-06.md             ← Módulo 6: Optimización de Tokens
├── capitulo-07.md             ← Módulo 7: Seguridad y QA
├── capitulo-08.md             ← Módulo 8: Patrones Avanzados
└── assets/                    ← Imágenes y recursos multimedia
```

## Workflow con Claude Code

Este curso está diseñado para ser creado y mantenido con Claude Code. Skill sugerida para generar capítulos:

```bash
# Desde Claude Code, usa:
> Lee el glosario en curso-claude-code-glosario.md y genera el capítulo 2
> en formato LiaScript siguiendo el estilo del capítulo 1
```

## Tecnologías

- **Autoría:** LiaScript (Markdown extendido)
- **Preview:** VS Code Extension / devserver
- **Exportación:** LiaScript Exporter CLI (SCORM, PDF, Web)
- **LMS:** Moodle (con MCP Server para gestión desde Claude Code)
- **Repositorio:** GitHub
- **Generación:** Claude Code con skills personalizadas

## Licencia

CC BY-SA 4.0 — Puedes compartir y adaptar el contenido citando la fuente.
