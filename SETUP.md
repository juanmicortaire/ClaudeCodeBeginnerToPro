# Guía de Publicación del Curso

Este documento explica cómo publicar y distribuir el **Curso Claude Code: De Principiante a Profesional** creado con LiaScript.

## Estructura de Archivos

```
curso-claude-code-liascript/
├── README.md            ← Archivo maestro (índice del curso con imports)
├── SETUP.md             ← Este archivo (instrucciones de publicación)
├── capitulo-01.md       ← Módulo 1: Fundamentos y Setup
├── capitulo-02.md       ← Módulo 2: Memoria y Contexto
├── capitulo-03.md       ← Módulo 3: Skills, Hooks y MCP
├── capitulo-04.md       ← Módulo 4: Agentes Especializados
├── capitulo-05.md       ← Módulo 5: Workflows Profesionales
├── capitulo-06.md       ← Módulo 6: Optimización de Tokens
├── capitulo-07.md       ← Módulo 7: Seguridad y QA
└── capitulo-08.md       ← Módulo 8: Patrones Avanzados
```

## Cómo Funciona el Índice Maestro

El archivo `README.md` contiene en su cabecera una directiva `import:` que le indica a LiaScript que debe cargar los 8 capítulos como parte del mismo curso. El resultado es una experiencia de navegación unificada: los alumnos ven un único curso con un menú lateral que lista los 8 módulos, pueden navegar entre ellos con los botones `<` y `>`, y el progreso se guarda de forma conjunta.

``` ascii
  README.md (punto de entrada)
       │
       │  import:
       ▼
  ┌──────────────────────────────────┐
  │  Capítulo 1                       │
  │  Capítulo 2                       │
  │  Capítulo 3                       │
  │  Capítulo 4  ◄── Todos cargados   │
  │  Capítulo 5      como un único    │
  │  Capítulo 6      curso navegable  │
  │  Capítulo 7                       │
  │  Capítulo 8                       │
  └──────────────────────────────────┘
```

---

## Opción 1: Publicación en GitHub (Recomendada)

Esta es la forma más sencilla y profesional de publicar el curso.

### Paso 1: Crear el Repositorio

```bash
# Crear directorio local
mkdir curso-claude-code
cd curso-claude-code

# Copiar todos los archivos del curso
cp /ruta/a/curso-claude-code-liascript/*.md .

# Inicializar git
git init
git add .
git commit -m "feat: curso Claude Code completo (8 módulos)"

# Crear repo en GitHub (ajusta la URL a tu usuario)
git remote add origin https://github.com/TU_USUARIO/curso-claude-code.git
git branch -M main
git push -u origin main
```

### Paso 2: Actualizar las URLs del Import

Edita `README.md` y reemplaza `TU_USUARIO` con tu nombre de usuario de GitHub real en las 8 líneas del bloque `import:`:

```markdown
import:   https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/main/capitulo-01.md
          https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/main/capitulo-02.md
          ...
```

Commit y push:

```bash
git add README.md
git commit -m "docs: actualizar URLs de import con usuario real"
git push
```

### Paso 3: Acceder al Curso

El curso ya está publicado. La URL de acceso es:

```
https://liascript.github.io/course/?https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/main/README.md
```

**Reemplaza `TU_USUARIO`** con tu nombre de usuario real. Esta URL:

- Carga el README.md como punto de entrada
- Los `import:` traen los 8 capítulos automáticamente
- Los alumnos ven un único curso navegable con menú lateral
- Funciona offline después del primer acceso (es una PWA)
- El progreso se guarda en IndexedDB del navegador

### Paso 4: Generar un Badge para Compartir

Añade este badge a tu `README.md` del repositorio en GitHub (no el del curso) para que los visitantes puedan acceder fácilmente:

```markdown
[![LiaScript](https://raw.githubusercontent.com/LiaScript/LiaScript/master/badges/course.svg)](https://liascript.github.io/course/?https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/main/README.md)
```

---

## Opción 2: Publicación Local para Desarrollo

Para previsualizar y editar el curso antes de publicarlo:

### Con la Extensión VS Code (Recomendada)

1. Instala la extensión **LiaScript Preview** desde el marketplace de VS Code
2. Abre el directorio del curso en VS Code
3. Abre `README.md`
4. Usa `Cmd+Shift+P` / `Ctrl+Shift+P` y busca "LiaScript: Preview"
5. El preview se actualiza en tiempo real al guardar

**Nota importante:** En modo local, los `import:` pueden no funcionar directamente porque apuntan a URLs de GitHub. Para previsualizar el curso completo localmente, usa el devserver (opción siguiente).

### Con el Devserver Local

```bash
# Instalar el devserver
npm install -g @liascript/devserver

# Lanzar el servidor en el directorio del curso
cd curso-claude-code-liascript
liadev --input README.md
```

Se abrirá automáticamente en `http://localhost:3000` con hot-reload. El devserver carga los capítulos desde archivos locales, lo que permite probar el curso completo antes de publicarlo.

---

## Opción 3: Exportar a SCORM (para Moodle u otros LMS)

Si prefieres distribuir el curso a través de un LMS corporativo como Moodle, ILIAS, Blackboard o Canvas, puedes exportar cada módulo a SCORM.

### Instalación del Exportador

```bash
npm install -g @liascript/exporter
```

### Exportar Cada Módulo

**Importante:** El exportador SCORM funciona con un archivo markdown a la vez, no con el curso completo navegable. Tendrás que exportar cada capítulo por separado:

```bash
# Exportar el Módulo 1
liaex -i capitulo-01.md --format scorm1.2 \
  -o curso-claude-modulo1 \
  --scorm-organization "Curso Claude Code Profesional"

# Exportar el Módulo 2
liaex -i capitulo-02.md --format scorm1.2 \
  -o curso-claude-modulo2 \
  --scorm-organization "Curso Claude Code Profesional"

# ... y así hasta el Módulo 8
```

Esto genera 8 archivos `.zip` (uno por módulo) que puedes subir a tu LMS como cursos independientes.

### Script de Exportación Masiva

Para automatizar la exportación de los 8 módulos:

```bash
#!/bin/bash
# export-all.sh

for i in 01 02 03 04 05 06 07 08; do
  echo "Exportando capítulo $i..."
  liaex -i capitulo-$i.md --format scorm1.2 \
    -o curso-claude-modulo$i \
    --scorm-organization "Curso Claude Code Profesional" \
    --scorm-typicalDuration "PT1H30M0S"
done

echo "Exportación completada. Revisa los archivos .zip generados."
```

### Otros Formatos de Exportación

```bash
# Exportar a SCORM 2004 (si tu LMS lo requiere)
liaex -i capitulo-01.md --format scorm2004 -o modulo1

# Exportar a web standalone (HTML independiente)
liaex -i capitulo-01.md --format web -o modulo1-web

# Exportar a PDF (para distribución offline)
liaex -i capitulo-01.md --format pdf -o modulo1

# Exportar a IMS (Instructional Management Systems)
liaex -i capitulo-01.md --format ims -o modulo1

# Exportar a app Android (requiere Android SDK)
liaex -i capitulo-01.md --format android -o modulo1 \
  --android-sdk /path/to/android-sdk
```

---

## Opción 4: Distribución por Data-URI

Si no quieres hostear el curso en ningún sitio, puedes compartirlo como una URL de datos. Esto es útil para demos rápidas o distribución privada:

1. Ve a [LiaScript Data-URI Generator](https://liascript.github.io/data-uri/)
2. Sube tu `README.md` (con los capítulos incluidos inline, no con `import:`)
3. Genera una URL que contiene todo el curso codificado
4. Comparte esa URL con quien quieras

**Limitación:** Esta opción requiere un único archivo markdown con todo el contenido. Para usarla necesitas crear una versión "all-in-one" del curso (ver siguiente sección).

---

## Versión Todo-en-Uno (sin imports)

Si prefieres un único archivo markdown que contenga los 8 capítulos concatenados (sin usar `import:`), puedes generarlo con:

```bash
# Script para concatenar todos los capítulos en un único archivo
cat > curso-completo.md << 'HEADER'
<!--
author:   Juan Miguel (Juanmi) — Curso Claude Code Profesional
email:    curso-claude-code@example.com
version:  1.0.0
language: es
narrator: Spanish Female
-->

HEADER

# Añadir el contenido del README.md (sin las líneas del bloque import:)
sed '/^import:/,/^-->/d' README.md >> curso-completo.md

# Añadir cada capítulo
for i in 01 02 03 04 05 06 07 08; do
  echo "" >> curso-completo.md
  # Eliminar el header HTML de cada capítulo antes de concatenar
  sed '/^<!--/,/^-->/d' capitulo-$i.md >> curso-completo.md
done

echo "Archivo curso-completo.md creado."
```

Este archivo todo-en-uno es más pesado (~140 KB) pero tiene ventajas:

- Funciona sin conexión inmediatamente
- Compatible con data-URI
- Más fácil de exportar a SCORM como curso único
- No depende de URLs externas

**Trade-off:** Es más difícil de mantener. Si actualizas un capítulo, tienes que regenerar el archivo completo.

---

## Personalización

### Cambiar el Autor y la Configuración

Edita las primeras líneas del `README.md` (o del archivo todo-en-uno):

```markdown
<!--
author:   Tu Nombre Aquí
email:    tu-email@ejemplo.com
version:  1.0.0
language: es
narrator: Spanish Female   ← Voz del TTS (ver opciones abajo)
-->
```

### Voces disponibles para Text-to-Speech

LiaScript usa ResponsiveVoice para TTS. Algunas voces en español:

- `Spanish Female` — Español estándar femenino
- `Spanish Male` — Español estándar masculino
- `Spanish Latin American Female` — Latinoamericano femenino
- `Spanish Latin American Male` — Latinoamericano masculino

Para otros idiomas:

- `English US Female`, `English UK Male`, etc.
- Lista completa: [ResponsiveVoice Languages](https://responsivevoice.org/text-to-speech-languages/)

### Añadir tu Logo

```markdown
<!--
logo: https://tu-dominio.com/logo.png
-->
```

Se mostrará en la cabecera del curso cuando los alumnos lo vean.

---

## Mantenimiento del Curso

### Actualizar Contenido

Con la estructura de `import:`, actualizar el curso es simple:

1. Edita el capítulo correspondiente (`capitulo-0X.md`)
2. Haz commit y push
3. Los alumnos verán los cambios la próxima vez que abran el curso (puede requerir refresh del navegador)

### Versionado con Git Tags

Para mantener versiones estables del curso:

```bash
# Marcar una versión estable
git tag -a v1.0 -m "Versión 1.0 del curso"
git push --tags

# Los alumnos pueden acceder a una versión específica:
# https://liascript.github.io/course/?https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/v1.0/README.md
```

### Traducción del Curso

Aprovecha los branches de Git para mantener traducciones:

```bash
git checkout -b english
# Traduce los archivos
git push origin english

# Los alumnos acceden a la versión en inglés:
# https://liascript.github.io/course/?https://raw.githubusercontent.com/TU_USUARIO/curso-claude-code/english/README.md
```

---

## Integración con Moodle (con MCP Server)

Como vimos en el curso, puedes usar el MCP Server de Moodle desde Claude Code para gestionar la subida del curso directamente:

```bash
# Configura el MCP de Moodle
claude mcp add moodle \
  --command "uvx" \
  --args "moodle-mcp" \
  --env "MOODLE_URL=https://tu-moodle.com" \
  --env "MOODLE_TOKEN=tu-token"

# Desde Claude Code
> Sube los 8 archivos SCORM del curso Claude Code a Moodle.
  Crea una nueva categoría "Formación IA" si no existe.
  Los módulos deben seguir el orden numérico y bloquearse
  hasta completar el anterior (linear progression).
```

---

## Troubleshooting

### El curso no carga los capítulos

**Causa probable:** Las URLs del bloque `import:` apuntan a `TU_USUARIO` sin reemplazar.

**Solución:** Edita `README.md` y reemplaza `TU_USUARIO` con tu nombre de usuario de GitHub real en las 8 líneas.

### Los quizzes no se renderizan correctamente

**Causa probable:** Los archivos tienen encoding incorrecto (Windows CRLF en vez de Unix LF).

**Solución:**

```bash
# Convertir a LF (Linux/Mac line endings)
find . -name "*.md" -exec dos2unix {} \;
```

### El TTS no funciona

**Causa probable:** ResponsiveVoice requiere una key para uso comercial.

**Solución:** Para uso educativo no comercial, funciona sin key. Para uso comercial:

```markdown
<!--
narrator: Spanish Female
-->

Contenido del curso...

<!-- ?key=TU_RESPONSIVE_VOICE_KEY -->
```

### El progreso de los alumnos se pierde

**Causa probable:** Los alumnos usan navegación privada o borran el almacenamiento.

**Solución:** LiaScript guarda el progreso en IndexedDB del navegador. En navegación privada no persiste. Informa a los alumnos que deben usar una sesión normal si quieren mantener progreso entre visitas.

---

## Recursos Adicionales

- **Documentación oficial de LiaScript:** [liascript.github.io](https://liascript.github.io)
- **Repositorio de LiaScript:** [github.com/liascript/LiaScript](https://github.com/liascript/LiaScript)
- **Ejemplos de cursos:** [github.com/liaScript/docs](https://github.com/liaScript/docs)
- **Comunidad de LiaScript:** Disponible en Matrix y GitHub Discussions

---

## Licencia Sugerida

Se recomienda publicar el curso bajo **Creative Commons BY-SA 4.0**:

- **BY:** Otros pueden compartir y adaptar citando la autoría
- **SA:** Las adaptaciones deben mantener la misma licencia

Añade un archivo `LICENSE` al repositorio:

```
Curso Claude Code: De Principiante a Profesional
Copyright (c) 2026 Juan Miguel (Juanmi)

Licensed under the Creative Commons Attribution-ShareAlike 4.0
International License (CC BY-SA 4.0).

You are free to:
- Share — copy and redistribute the material
- Adapt — remix, transform, and build upon the material

Under the following terms:
- Attribution — You must give appropriate credit
- ShareAlike — If you remix or build upon the material,
  you must distribute your contributions under the same license

Full license text: https://creativecommons.org/licenses/by-sa/4.0/
```

---

*Cualquier duda sobre la publicación o personalización del curso, consulta la documentación de LiaScript o abre un issue en el repositorio del curso.*
