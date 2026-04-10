<!--
author:   Cortaire, Juan Miguel — Curso Claude Code From Beginner to Professional
email:    juanmicortaire@gmail.com
version:  1.0.0
language: es
narrator: Spanish Female

comment:  Curso interactivo de Claude Code: De Principiante a Profesional.
          Módulo 7 — Seguridad y QA Integrados.

-->

# Módulo 7: Seguridad y QA Integrados

> **"La seguridad y la calidad no son fases del final del pipeline. Son propiedades que se integran en cada paso. Un agente de seguridad que solo revisa el PR final es un guardia que llega después del robo."**

En este módulo construimos defensas en profundidad: el agente de seguridad como paso bloqueante del pipeline, quality gates automáticas con hooks, gestión de vulnerabilidades en dependencias, y testing avanzado con hipótesis competidoras.

    {{1}}
**Objetivo del módulo:** Integrar seguridad y QA como propiedades continuas del pipeline, no como pasos puntuales. Al terminar, tendrás un sistema de defensa multicapa que detecta problemas antes de que lleguen a producción.

    {{2}}
**Tiempo estimado:** 75-90 minutos

    {{3}}
**Prerrequisito:** Haber completado los Módulos 1-6, especialmente el roster de agentes del Módulo 4.

---

## 7.1 Defensa en Profundidad: El Modelo de Capas

En lugar de un único paso de seguridad al final, aplicamos el principio de **defense in depth**: múltiples capas independientes que detectan problemas en diferentes momentos del ciclo de vida.

``` ascii
  ┌───────────────────────────────────────────────────┐
  │              DEFENSA EN PROFUNDIDAD               │
  ├───────────────────────────────────────────────────┤
  │                                                   │
  │  CAPA 1: Preventiva (antes de escribir código)    │
  │  ┌──────────────────────────────────────────┐     │
  │  │ CLAUDE.md con restricciones explícitas   │     │
  │  │ Permisos en settings.json                │     │
  │  │ Custom agents con tools restringidos     │     │
  │  └──────────────────────────────────────────┘     │
  │                      │                            │
  │                      ▼                            │
  │  CAPA 2: Durante la ejecución (runtime checks)    │
  │  ┌──────────────────────────────────────────┐     │
  │  │ Hooks PreToolUse: bloquean acciones      │     │
  │  │ Hooks PostToolUse: lint, format, test    │     │
  │  │ Modo de aprobación apropiado             │     │
  │  └──────────────────────────────────────────┘     │
  │                      │                            │
  │                      ▼                            │
  │  CAPA 3: Reactiva (después de escribir código)    │
  │  ┌──────────────────────────────────────────┐     │
  │  │ @security-analyst: audit OWASP Top 10    │     │
  │  │ @qa-engineer: tests + edge cases         │     │
  │  │ Análisis de dependencias automático      │     │
  │  └──────────────────────────────────────────┘     │
  │                      │                            │
  │                      ▼                            │
  │  CAPA 4: CI/CD (pipeline automatizado)            │
  │  ┌──────────────────────────────────────────┐     │
  │  │ claude -p con quality gates bloqueantes  │     │
  │  │ SAST/DAST integrado                      │     │
  │  │ Dependency scanning                      │     │
  │  └──────────────────────────────────────────┘     │
  └───────────────────────────────────────────────────┘
```

    {{1}}
**Cada capa captura lo que las anteriores dejaron pasar.** Un vector de ataque tendría que vulnerar las cuatro capas para llegar a producción, lo que es estadísticamente improbable.

#### Quiz: Defense in depth

¿Por qué es mejor tener múltiples capas de seguridad en lugar de un único paso de validación al final del pipeline?

- [( )] Porque consume más tokens, demostrando que se está trabajando más
- [( )] Porque evita tener que escribir CLAUDE.md
- [(X)] Porque cada capa captura problemas que las anteriores dejaron pasar, y un ataque tendría que vulnerar todas las capas para llegar a producción
- [( )] Porque permite usar el modelo Opus en cada capa
***
**Correcto.** El principio de defensa en profundidad reconoce que ninguna capa es perfecta. Al combinar prevención (CLAUDE.md, permisos), runtime checks (hooks), análisis reactivo (agentes especializados) y CI/CD, la probabilidad de que un problema llegue a producción disminuye exponencialmente.
***

---

## 7.2 Capa 1: Prevención

La capa más eficiente es la que evita que el problema ocurra. Antes de que Claude escriba una sola línea de código, ya hemos definido qué puede y qué no puede hacer.

### CLAUDE.md con Restricciones Explícitas

``` markdown
# Restricciones de Seguridad

## NUNCA hacer
- Nunca commitear sin aprobación explícita del usuario
- Nunca modificar archivos en /infrastructure/production/
- Nunca ejecutar comandos con sudo
- Nunca instalar dependencias nuevas sin justificación en el PR
- Nunca almacenar secrets en archivos .env commiteados
- Nunca desactivar reglas de ESLint/ruff sin comentario justificativo
- Nunca hacer git push --force en ramas main o develop

## SIEMPRE hacer
- Siempre ejecutar tests antes de proponer commit
- Siempre revisar vulnerabilidades con npm audit / pip-audit tras añadir deps
- Siempre usar parameterized queries (nunca string concatenation en SQL)
- Siempre validar input del usuario antes de procesarlo
- Siempre usar HttpOnly + Secure + SameSite en cookies sensibles
- Siempre loggear intentos de autenticación fallidos
```

    {{1}}
> Estas reglas se cargan automáticamente en cada sesión. Claude las respeta como restricciones inviolables, no como sugerencias.

### Permisos Granulares en settings.json

Combina `allowedTools` y `deny` para crear un sandbox apropiado:

``` json
{
  "permissions": {
    "allowedTools": [
      "Read",
      "Write",
      "Grep",
      "Glob",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(npm test *)",
      "Bash(npm run lint)",
      "Bash(ruff *)",
      "Bash(pytest *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./**/id_rsa*)",
      "Write(./production.config.*)",
      "Write(./infrastructure/prod/**)",
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(curl * | bash)",
      "Bash(wget * | sh)",
      "Bash(git push --force *)",
      "Bash(git push * --force*)",
      "Bash(chmod 777 *)"
    ]
  }
}
```

    {{2}}
**Observa los patrones importantes:**

- `Read(./.env)` y `Read(./.env.*)` protegen archivos de entorno
- `Bash(curl * | bash)` y `Bash(wget * | sh)` previenen la ejecución de scripts remotos
- `Bash(git push --force *)` previene sobrescritura de historial compartido
- `Write(./infrastructure/prod/**)` protege infraestructura de producción

### Custom Agents con Tools Restringidos

Los agentes de revisión y análisis NO necesitan escribir ni ejecutar comandos. Configúralos read-only:

``` markdown
---
name: security-analyst
description: Performs OWASP Top 10 security audit. Read-only.
tools: read, grep, glob, bash
model: claude-sonnet-4-6-20250514
---

IMPORTANT: You are READ-ONLY. You never modify files.
You only run analysis commands (npm audit, pip-audit,
git log, git diff). You never run commands that modify
state (install, write, git commit, etc.).
```

    {{3}}
> **Por qué esto importa:** Si por algún error Claude invoca al security-analyst con una intención maliciosa o malinterpretada, las herramientas restringidas previenen daño. El agente *físicamente no puede* escribir archivos o ejecutar comandos destructivos.

#### Quiz: Prevención

Estás configurando un agente de code review que analizará PRs de contribuidores externos. ¿Qué configuración de `tools` es la más segura?

- [( )] `tools: read, write, bash, grep, glob` (acceso completo)
- [(X)] `tools: read, grep, glob` (solo lectura, sin ejecución)
- [( )] `tools: read, bash` (lectura y ejecución)
- [( )] Sin restricciones de tools
***
**Correcto.** Para analizar código de contribuidores externos, el principio de menor privilegio dicta que el agente solo necesita leer. No debe poder ejecutar código (que podría ser malicioso), ni modificar archivos. `tools: read, grep, glob` es la configuración mínima y más segura.
***

---

## 7.3 Capa 2: Runtime Checks con Hooks

Los hooks son tu sistema de defensa en tiempo real. Cada acción de Claude pasa por ellos antes o después de ejecutarse.

### Hooks Defensivos: PreToolUse

Estos hooks bloquean acciones peligrosas antes de que ocurran:

``` json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(rm -rf *)",
        "hooks": [{
          "type": "command",
          "command": "echo 'BLOCKED: rm -rf is forbidden' && exit 1"
        }]
      },
      {
        "matcher": "Bash(git push --force*)",
        "hooks": [{
          "type": "command",
          "command": "echo 'BLOCKED: force push is forbidden' && exit 1"
        }]
      },
      {
        "matcher": "Write(./infrastructure/production/*)",
        "hooks": [{
          "type": "command",
          "command": "echo 'BLOCKED: production config requires manual approval' && exit 1"
        }]
      },
      {
        "matcher": "Bash(curl * | *sh*)",
        "hooks": [{
          "type": "command",
          "command": "echo 'BLOCKED: piping remote scripts to shell is forbidden' && exit 1"
        }]
      }
    ]
  }
}
```

    {{1}}
Un hook `PreToolUse` que retorna `exit 1` bloquea la acción. Claude ve el mensaje de error y puede ajustar su enfoque.

### Hooks Ofensivos: PostToolUse

Estos hooks detectan y corrigen problemas después de cada acción:

``` json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix \"$file\""
          },
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
          },
          {
            "type": "command",
            "command": "eslint --fix \"$file\""
          }
        ]
      },
      {
        "matcher": "Write(package.json)",
        "hooks": [{
          "type": "command",
          "command": "npm audit --audit-level=high"
        }]
      },
      {
        "matcher": "Write(requirements.txt)",
        "hooks": [{
          "type": "command",
          "command": "pip-audit"
        }]
      },
      {
        "matcher": "Write(src/auth/*)",
        "hooks": [{
          "type": "command",
          "command": "echo '⚠️ AUTH MODULE MODIFIED' && semgrep --config=auto src/auth/"
        }]
      }
    ]
  }
}
```

    {{2}}
**Observa el patrón crítico:** Cada vez que Claude modifica `package.json` o `requirements.txt`, se ejecuta automáticamente el auditor de dependencias. Cada modificación al módulo de autenticación dispara un scan con Semgrep. Todo sin intervención humana.

### El Pattern "Secret Scanner"

Un hook específico para prevenir commits de secrets:

``` json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit*)",
        "hooks": [{
          "type": "command",
          "command": "gitleaks protect --staged --verbose || (echo 'BLOCKED: secrets detected in staged files' && exit 1)"
        }]
      }
    ]
  }
}
```

    {{3}}
Cada intento de commit pasa por `gitleaks` para detectar tokens, API keys, passwords y otros secrets. Si se encuentra alguno, el commit se bloquea automáticamente.

#### Quiz: Hooks defensivos

Quieres que cada vez que Claude añada una nueva dependencia en `package.json`, se ejecute automáticamente `npm audit` y el proceso se detenga si hay vulnerabilidades HIGH o CRITICAL. ¿Cuál es la configuración correcta?

- [( )] Un PreToolUse con matcher `Bash(npm install *)`
- [(X)] Un PostToolUse con matcher `Write(package.json)` que ejecute `npm audit --audit-level=high`
- [( )] Un hook PermissionRequest para aprobar cada install
- [( )] Un Agent Team de 3 agentes de seguridad
***
**Correcto.** `PostToolUse` con matcher `Write(package.json)` se dispara automáticamente cuando Claude modifica el archivo de dependencias. El comando `npm audit --audit-level=high` sale con código de error si hay vulnerabilidades HIGH o CRITICAL, deteniendo el flujo. Claude ve el error y puede revertir o actualizar la dependencia.
***

---

## 7.4 Capa 3: Análisis Reactivo con Agentes Especializados

Aquí es donde el roster del Módulo 4 demuestra su valor. Dos agentes merecen atención especial en este módulo.

### El Agente de Seguridad en Detalle

Mejoramos el `@security-analyst` del Módulo 4 con un enfoque OWASP Top 10 exhaustivo:

``` markdown
<!-- .claude/agents/security-analyst.md -->
---
name: security-analyst
description: Performs comprehensive OWASP Top 10 security
  audit. Analyzes authentication, authorization, input
  validation, injection, crypto, access control, logging,
  and dependency vulnerabilities. Use for security reviews
  of code changes, PR audits, and pre-deployment checks.
tools: read, grep, glob, bash
model: claude-sonnet-4-6-20250514
---

You are a senior application security engineer with
expertise in OWASP Top 10 and secure coding practices.
You think like an attacker to defend like a guardian.

## Audit Process

### A01:2021 - Broken Access Control
- Verify authorization check on EVERY endpoint
- Check for IDOR (Insecure Direct Object Reference)
- Verify proper role-based access control (RBAC)
- Check CORS configuration for over-permissive origins
- Verify principle of least privilege in DB queries

### A02:2021 - Cryptographic Failures
- Check encryption at rest for sensitive data
- Verify TLS configuration and certificate validation
- Check for hardcoded cryptographic keys
- Verify no sensitive data in logs, URLs, or error messages
- Check password hashing (must be bcrypt/argon2, not MD5/SHA1)

### A03:2021 - Injection
- SQL Injection: verify parameterized queries everywhere
- Command Injection: check Bash calls for user input
- XSS: verify template escaping and CSP headers
- LDAP/XPath Injection: check external system queries

### A04:2021 - Insecure Design
- Check for missing rate limiting on auth endpoints
- Verify no sensitive business logic client-side
- Check for proper session management

### A05:2021 - Security Misconfiguration
- Default credentials check
- Verbose error messages in production
- Unnecessary features enabled
- Missing security headers (HSTS, CSP, X-Frame-Options)

### A06:2021 - Vulnerable Components
- Run `npm audit` or `pip-audit`
- Check versions against known CVEs
- Verify no abandoned/unmaintained dependencies

### A07:2021 - Authentication Failures
- JWT validation: expiry, signature, audience, issuer
- Password policy enforcement
- Rate limiting on auth endpoints (login, reset, refresh)
- Multi-factor authentication support
- Session timeout configuration

### A08:2021 - Software and Data Integrity
- Verify dependency integrity (lockfiles)
- Check for insecure deserialization
- Verify CI/CD pipeline security

### A09:2021 - Logging and Monitoring
- Verify security events are logged
- Check no sensitive data in logs
- Verify log injection prevention

### A10:2021 - Server-Side Request Forgery (SSRF)
- Check all URL inputs for SSRF vulnerability
- Verify allowlist for outbound requests

## Output Format

### Executive Summary
- Total findings: N
- By severity: CRITICAL X | HIGH Y | MEDIUM Z | LOW W

### Findings (ordered by severity)
For each finding:
- **[SEVERITY]** A-OWASP category — short title
- **Location:** file:line
- **Description:** what's wrong and why it matters
- **Impact:** what could an attacker do
- **Remediation:** specific fix with code example
- **References:** OWASP link, CVE if applicable

### Verdict
- PASS: No CRITICAL or HIGH findings
- FAIL: CRITICAL or HIGH findings require remediation

### Metrics
- Files scanned: N
- Lines analyzed: N
- Time: Xm
```

### Análisis de Dependencias Automatizado

Las vulnerabilidades en dependencias son una de las vías de ataque más comunes. Integra análisis automático:

``` bash
# Para proyectos Node.js
npm audit --json | claude -p "Analyze this npm audit output.
  Identify which CVEs are exploitable in our codebase context.
  For each HIGH or CRITICAL, provide a remediation plan."

# Para proyectos Python
pip-audit --format json | claude -p "Analyze these Python
  vulnerabilities. Check if our code uses the vulnerable
  functions. Prioritize actual exposure, not just presence."
```

    {{1}}
> **Clave:** No todas las vulnerabilidades reportadas son explotables en tu contexto. Claude puede revisar si tu código realmente usa las funciones vulnerables, evitando ruido y falsos positivos que desgastan al equipo.

### Revisión de Secrets con Historial

``` bash
# Busca secrets accidentalmente commiteados en el historial
git log --all --full-history -p | claude -p "Review this git
  history for accidentally committed secrets: API keys,
  tokens, passwords, private keys. Report with commit hash
  and file. This is read-only analysis." \
  --allowedTools "Read,Grep,Glob" \
  --max-budget-usd 3.00
```

#### Quiz: Agente de seguridad

¿Por qué el agente `@security-analyst` tiene `bash` en sus tools pero es considerado seguro?

- [( )] Porque `bash` no es realmente peligroso
- [(X)] Porque el system prompt le instruye explícitamente a solo ejecutar comandos de análisis (npm audit, pip-audit, git log, git diff), no comandos que modifiquen estado
- [( )] Porque Claude nunca ejecuta comandos destructivos
- [( )] Porque tiene el modelo Sonnet en lugar de Opus
***
**Correcto.** El agente necesita `bash` para ejecutar herramientas de análisis como `npm audit`, `pip-audit`, `git log` y `git diff`. La seguridad viene del system prompt que le restringe explícitamente a comandos read-only, y de la combinación con hooks PreToolUse que bloquean acciones destructivas. La restricción es de comportamiento, no solo de herramientas.
***

---

## 7.5 El Agente de QA Avanzado

El QA engineer no es solo "generador de tests". Es un detector de edge cases y un cazador de bugs latentes.

### QA Engineer con Enfoque en Edge Cases

``` markdown
<!-- .claude/agents/qa-engineer.md (versión avanzada) -->
---
name: qa-engineer
description: Senior QA engineer obsessed with edge cases
  and failure modes. Generates comprehensive test suites,
  analyzes coverage, identifies untested scenarios, and
  hunts for latent bugs. Use for test generation, coverage
  audits, and pre-release verification.
tools: read, write, bash, grep, glob
model: claude-sonnet-4-6-20250514
---

You are a senior QA engineer. Your mission is to find
what developers missed. You think adversarially.

## Edge Case Categories (Always Check)

### Null/Empty/Missing
- null, undefined, empty string, empty array, empty object
- Missing required fields
- Optional fields set to null vs undefined vs missing

### Boundary Values
- Minimum - 1, minimum, minimum + 1
- Maximum - 1, maximum, maximum + 1
- Zero, negative, very large numbers
- Off-by-one in loops and ranges

### Type Confusion
- String instead of number ("5" vs 5)
- Number instead of string
- Boolean instead of object
- Date as string vs Date object

### Encoding and Unicode
- Non-ASCII characters
- Emojis and multibyte UTF-8
- RTL (right-to-left) text
- Null bytes in strings
- SQL keywords in string inputs (but parameterized!)

### Timing and Concurrency
- Race conditions on shared resources
- Deadlocks in concurrent operations
- Timeouts (too short, too long, zero)
- Clock skew between systems
- Daylight saving time transitions

### Network Failures
- Total network loss
- Slow responses (partial timeouts)
- HTTP 500, 503, 429 responses
- Partial responses (connection dropped mid-stream)
- DNS failures

### State Transitions
- Invalid state transitions
- Concurrent state changes
- Idempotency of operations
- Rollback of partial operations

### Permissions and Auth
- Expired tokens
- Tokens from different users (IDOR)
- Missing permissions
- Revoked permissions mid-session

## Output Format

### Test Plan Summary
- Unit tests: N | Integration tests: N | E2E tests: N
- Coverage: X% lines, Y% branches

### Edge Cases Discovered
List of scenarios NOT in the original spec that you
identified as risky.

### Implementation
Actual test code using the project's framework,
following the naming conventions in CLAUDE.md.

### Execution Report
- Tests passing: X/Y
- Failures: detailed analysis of why and suggested fixes
- Flaky tests identified
```

### Testing con Hipótesis Competidoras (Debate Pattern)

Para bugs particularmente difíciles, usa el Debate Pattern del Módulo 4 con Agent Teams:

``` text
> Los usuarios reportan que la app crashea intermitentemente
  después de varios minutos de uso. Activa Agent Team y
  spawn 5 agentes para investigar hipótesis diferentes:
  - Agent 1: Memory leak
  - Agent 2: Event listener accumulation
  - Agent 3: Unhandled promise rejection
  - Agent 4: Race condition en state management
  - Agent 5: Integration con third-party service

  Que debatan entre sí intentando refutar las teorías
  de los demás. El que sobreviva al debate es probablemente
  la causa real.
```

    {{1}}
> **Por qué funciona:** Los bugs intermitentes son notoriamente difíciles porque la investigación secuencial se sesga hacia la primera hipótesis. Múltiples agentes desafiándose mutuamente evitan el sesgo de anclaje y convergen en la causa real mucho más rápido.

---

## 7.6 Capa 4: CI/CD con Quality Gates Bloqueantes

La última línea de defensa: pipelines de CI que bloquean merges si las validaciones fallan.

### Pipeline de GitHub Actions Completo

``` yaml
# .github/workflows/security-quality.yml
name: Security & Quality Gates

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  security-gate:
    name: Security Gate (BLOCKING)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: OWASP Audit
        id: owasp
        run: |
          git diff origin/main...HEAD | claude -p \
            "Perform OWASP Top 10 audit on this diff.
             Output as JSON: {findings: [{severity, category,
             file, line, description}], verdict: 'PASS'|'FAIL'}" \
            --output-format json \
            --allowedTools "Read,Grep,Glob" \
            --max-budget-usd 2.00 \
            --max-turns 3 \
            > security-report.json
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

      - name: Check Verdict
        run: |
          VERDICT=$(jq -r '.verdict' security-report.json)
          CRITICAL=$(jq '[.findings[] | select(.severity=="CRITICAL")] | length' security-report.json)
          HIGH=$(jq '[.findings[] | select(.severity=="HIGH")] | length' security-report.json)

          echo "Verdict: $VERDICT"
          echo "Critical: $CRITICAL"
          echo "High: $HIGH"

          if [ "$VERDICT" = "FAIL" ] || [ "$CRITICAL" -gt 0 ] || [ "$HIGH" -gt 0 ]; then
            echo "::error::Security gate FAILED"
            jq '.findings[] | select(.severity=="CRITICAL" or .severity=="HIGH")' security-report.json
            exit 1
          fi

      - name: Upload Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: security-report
          path: security-report.json

  dependency-gate:
    name: Dependency Audit (BLOCKING)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: NPM Audit
        run: npm audit --audit-level=high

      - name: Secrets Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  test-coverage-gate:
    name: Test Coverage (BLOCKING)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm test -- --coverage

      - name: Check coverage threshold
        run: |
          COVERAGE=$(jq '.total.lines.pct' coverage/coverage-summary.json)
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "::error::Coverage $COVERAGE% below threshold 80%"
            exit 1
          fi
```

    {{1}}
**Tres jobs bloqueantes:**

1. **Security Gate:** Claude Code hace audit OWASP del diff. Si hay CRITICAL o HIGH, falla el PR.
2. **Dependency Gate:** `npm audit` + gitleaks. Vulnerabilidades y secrets bloqueantes.
3. **Test Coverage Gate:** Cobertura mínima del 80%.

    {{2}}
Ninguna de estas protecciones depende de la disciplina humana. Son automatizadas, consistentes y no se pueden saltar.

### Configuración de Branch Protection

En GitHub, configura branch protection para `main`:

``` text
  ┌────────────────────────────────────────────┐
  │ Branch Protection Rules for `main`         │
  ├────────────────────────────────────────────┤
  │ ✓ Require PR before merging                │
  │ ✓ Require approvals: 1                     │
  │ ✓ Require status checks to pass:           │
  │   ✓ security-gate                          │
  │   ✓ dependency-gate                        │
  │   ✓ test-coverage-gate                     │
  │ ✓ Require branches to be up to date        │
  │ ✓ Do not allow bypassing the above         │
  │ ✓ Restrict who can push to matching        │
  └────────────────────────────────────────────┘
```

#### Quiz: CI/CD Quality Gates

¿Cuál es la ventaja principal de tener quality gates bloqueantes en CI/CD sobre confiar en que los desarrolladores ejecuten las validaciones localmente?

- [( )] Es más rápido que ejecutar localmente
- [( )] Consume menos tokens
- [(X)] No depende de la disciplina humana: las validaciones se ejecutan de forma consistente y no pueden ser saltadas por error u olvido
- [( )] Funciona sin configuración
***
**Correcto.** Los desarrolladores tienen buena intención, pero bajo presión de deadline, las validaciones "opcionales" se saltan. Las quality gates en CI/CD son inviolables: ningún PR llega a main sin pasarlas. La automatización elimina la variable humana como fuente de fallos.
***

---

## 7.7 Workflow Completo de Seguridad

Veamos cómo todas las capas trabajan juntas en un escenario real:

``` ascii
  Desarrollador: "Añade un endpoint para que los usuarios
                   descarguen sus datos personales (GDPR)"
         │
         ▼
  ┌───────────────────────────────────────────────────┐
  │ CAPA 1 (Prevención)                               │
  │ CLAUDE.md: "Siempre validar autorización en       │
  │  endpoints", "Siempre rate limit en descargas"    │
  │ Claude lee las restricciones                      │
  └───────────────────────────────────────────────────┘
         │
         ▼
  @developer implementa el endpoint
         │
         ▼
  ┌───────────────────────────────────────────────────┐
  │ CAPA 2 (Runtime hooks)                            │
  │ PostToolUse Write(src/api/*) ──► semgrep scan     │
  │ PostToolUse Write(src/api/*) ──► eslint security  │
  │ Detecta: falta rate limit ⚠️                      │
  │ Claude corrige y añade rate limit                 │
  └───────────────────────────────────────────────────┘
         │
         ▼
  ┌───────────────────────────────────────────────────┐
  │ CAPA 3 (Análisis reactivo)                        │
  │ @security-analyst ejecuta audit OWASP             │
  │ Detecta: falta verificación de propiedad del      │
  │          recurso (IDOR potencial) - HIGH          │
  │ Detecta: datos se devuelven sin logging - MEDIUM  │
  │                                                   │
  │ @qa-engineer genera tests:                        │
  │ - Test de IDOR: usuario A no puede descargar      │
  │   datos de usuario B ✓                            │
  │ - Test de rate limit: máximo 5 descargas/hora     │
  │ - Edge case: usuario borrado pero con datos       │
  │                                                   │
  │ Claude corrige el IDOR y añade logging            │
  └───────────────────────────────────────────────────┘
         │
         ▼
  Desarrollador aprueba y crea PR
         │
         ▼
  ┌───────────────────────────────────────────────────┐
  │ CAPA 4 (CI/CD Quality Gates)                       │
  │ Job 1: Security Gate ──► PASS (no CRITICAL/HIGH)   │
  │ Job 2: Dependency Gate ──► PASS                    │
  │ Job 3: Test Coverage ──► PASS (94% coverage)       │
  │                                                    │
  │ Status: All checks passed ✓                       │
  └───────────────────────────────────────────────────┘
         │
         ▼
  Human reviewer hace revisión final
         │
         ▼
  MERGE a main
```

    {{1}}
**Cuatro capas. Cuatro oportunidades de detectar problemas.** Un bug de seguridad tendría que pasar por las cuatro para llegar a producción, lo que es exponencialmente improbable.

---

## 7.8 Gestión de Vulnerabilidades Continua

La seguridad no es un estado, es un proceso. Configura loops de monitorización continua:

### Loop de Auditoría Periódica

``` text
> /loop 24h @security-analyst audit the entire codebase
  for new vulnerabilities. Compare with the previous run.
  If new CRITICAL or HIGH findings, create a GitHub issue
  with priority P0.
```

Esto mantiene un vigilante activo que detecta regresiones y nuevas vulnerabilidades introducidas con cada merge.

### Monitorización de CVEs

``` text
> Cada semana, revisa las dependencias del proyecto contra
  la base de datos de CVEs publicados. Para cada match,
  evalúa si nuestro código usa la función vulnerable.
  Genera un reporte con acción recomendada.
```

### Dependabot + Claude Code

Combina Dependabot (que abre PRs con actualizaciones) con Claude Code (que las analiza):

``` yaml
# .github/workflows/dependabot-review.yml
name: Review Dependabot PRs

on:
  pull_request:
    types: [opened]

jobs:
  review:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Analyze update
        run: |
          claude -p \
            "Dependabot opened a PR updating a dependency.
             1. Read the changelog of the new version
             2. Analyze if we use the changed APIs
             3. Check for breaking changes affecting us
             4. Run tests after applying the update
             5. Comment on the PR with: SAFE_TO_MERGE
                or NEEDS_MANUAL_REVIEW with reasoning" \
            --max-budget-usd 2.00
```

#### Quiz: Gestión continua

¿Cuál es el propósito de combinar Dependabot con Claude Code para revisión automática de PRs de actualización?

- [( )] Reducir el coste de las actualizaciones
- [(X)] Dependabot detecta actualizaciones disponibles; Claude analiza el changelog, verifica si el código usa las APIs afectadas y ejecuta tests, aprobando automáticamente las actualizaciones seguras
- [( )] Reemplazar a los reviewers humanos completamente
- [( )] Evitar tener que configurar hooks
***
**Correcto.** Dependabot es bueno detectando actualizaciones pero no sabe si tu código específico se ve afectado. Claude Code añade la capa de análisis: lee el changelog, verifica el uso real de las APIs cambiadas, ejecuta tests, y solo requiere revisión manual cuando hay riesgo real. Esto reduce significativamente el trabajo de los reviewers humanos sin sacrificar seguridad.
***

---

## 7.9 Resumen del Módulo

En este módulo has aprendido:

- [X] Defense in depth: 4 capas de seguridad independientes
- [X] Capa 1 (Prevención): CLAUDE.md, permisos granulares, tools restringidos
- [X] Capa 2 (Runtime): hooks PreToolUse defensivos, PostToolUse ofensivos, secret scanning
- [X] Capa 3 (Análisis reactivo): `@security-analyst` con OWASP Top 10 exhaustivo
- [X] `@qa-engineer` avanzado con categorías de edge cases
- [X] Debate Pattern para bugs intermitentes con Agent Teams
- [X] Capa 4 (CI/CD): quality gates bloqueantes con Claude Code
- [X] Workflow completo integrando las 4 capas
- [X] Gestión de vulnerabilidades continua con loops y Dependabot

---

## Evaluación Final del Módulo 7

**Pregunta 1:** ¿Cuáles son las cuatro capas del modelo de Defense in Depth visto en este módulo?

- [[X]] Prevención (CLAUDE.md, permisos, tools restringidos)
- [[X]] Runtime checks (hooks PreToolUse y PostToolUse)
- [[X]] Análisis reactivo (@security-analyst, @qa-engineer)
- [[X]] CI/CD quality gates bloqueantes
- [[ ]] Backup y disaster recovery
- [[ ]] Firewall de red
***
**Correcto.** Las 4 capas trabajan en momentos distintos del ciclo: prevención antes de escribir código, runtime durante la ejecución, análisis reactivo tras la implementación, y CI/CD como última barrera antes de producción.
***

**Pregunta 2:** ¿Qué herramienta se usa típicamente en un hook PreToolUse para detectar secrets antes de un commit?

[[gitleaks]]
[[?]] Es una herramienta open-source popular para detectar secrets en código
[[?]] Su nombre combina "git" con "leaks"
***
**Correcto.** `gitleaks` es la herramienta estándar para detectar secrets en repositorios. Integrada como hook PreToolUse en `Bash(git commit*)`, bloquea automáticamente cualquier commit que contenga API keys, tokens, passwords u otros secrets.
***

**Pregunta 3:** Relaciona cada problema de seguridad con la categoría OWASP correspondiente:

[[A01]        [A03]        [A07]        [A06]]
[(X)          ( )          ( )          ( )  ]  Un endpoint permite a un usuario leer datos de otro (IDOR)
[( )          (X)          ( )          ( )  ]  SQL query construida con concatenación de strings
[( )          ( )          (X)          ( )  ]  JWT sin validación de expiración
[( )          ( )          ( )          (X)  ]  Dependencia con CVE conocido en package.json

**Pregunta 4:** Un equipo tiene configurados los 4 niveles de defensa. Un bug de seguridad tipo SQL injection llega a producción. ¿Cuál es el orden más probable de fallo de capas?

- [( )] Las 4 capas fallaron simultáneamente
- [(X)] CLAUDE.md no tenía regla explícita sobre parameterized queries, el hook de eslint-security no detectó el patrón, el @security-analyst no auditó ese archivo, y el security-gate de CI estaba deshabilitado
- [( )] Solo la Capa 4 (CI/CD) falló
- [( )] La seguridad en capas no funciona para SQL injection
***
**Correcto.** Para que un bug llegue a producción en un sistema con 4 capas, las 4 tuvieron que fallar. Cada fallo suele tener una causa: reglas faltantes en CLAUDE.md, hooks mal configurados, agentes no invocados para esa parte del código, o gates deshabilitados bajo presión de deadline. El análisis post-mortem identifica qué reforzar en cada capa.
***

**Pregunta 5:** ¿Por qué el `@security-analyst` puede tener `bash` en su lista de tools sin comprometer la seguridad?

- [( )] Porque bash es inofensivo
- [( )] Porque Sonnet es más seguro que Opus
- [(X)] Porque su system prompt lo restringe a comandos de análisis read-only (npm audit, pip-audit, git log, git diff), y los hooks PreToolUse bloquean cualquier comando destructivo que intentara ejecutar
- [( )] Porque los agentes no pueden ejecutar comandos realmente
***
**Correcto.** La seguridad del agente viene de dos fuentes combinadas: el system prompt que lo restringe a análisis (restricción de comportamiento) y los hooks PreToolUse que bloquean `rm`, `sudo`, `git push --force`, etc. (restricción física). Incluso si el agente fuera manipulado para intentar un comando destructivo, los hooks lo detendrían.
***

---

## Siguiente: Módulo 8 — Patrones Avanzados

En el último módulo del curso entraremos en patrones de nivel experto:

- Coordinator Mode en profundidad
- Builder-Validator chains avanzados
- Debate Pattern para decisiones arquitectónicas
- Extended Thinking y Effort Level
- Patrones de integración con sistemas legacy
- Métricas y mejora continua del pipeline

> **Tarea antes del Módulo 8:** Implementa al menos 2 de las 4 capas de defensa en tu proyecto. Como mínimo, configura las restricciones de CLAUDE.md (Capa 1) y un par de hooks PreToolUse + PostToolUse (Capa 2). En el Módulo 8 aplicaremos patrones avanzados sobre esta base.
