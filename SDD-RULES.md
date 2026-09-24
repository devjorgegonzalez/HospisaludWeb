# Estructura de proyecto para Spec-Driven Development con Claude Code

> Documento de referencia. Describe la estructura de archivos, el propósito de cada
> pieza, sus formatos, y ejemplos listos para copiar, orientado a desarrollar software
> con Claude Code (u otro agente) usando *spec-driven development* (SDD).

**Versión:** 1.0 · **Fecha:** 2026-07-23

---

## Índice

1. [Qué es SDD y por qué esta estructura](#1-qué-es-sdd-y-por-qué-esta-estructura)
2. [Estructura completa de archivos](#2-estructura-completa-de-archivos)
3. [Grado de estandarización de cada pieza](#3-grado-de-estandarización-de-cada-pieza)
4. [AGENTS.md — contexto del proyecto](#4-agentsmd--contexto-del-proyecto)
5. [CLAUDE.md — contexto específico de Claude](#5-claudemd--contexto-específico-de-claude)
6. [La carpeta `specs/` — el flujo de 4 fases](#6-la-carpeta-specs--el-flujo-de-4-fases)
7. [La constitución del proyecto](#7-la-constitución-del-proyecto)
8. [Notación EARS para requisitos](#8-notación-ears-para-requisitos)
9. [La carpeta `.claude/`](#9-la-carpeta-claude)
10. [Buenas prácticas y errores comunes](#10-buenas-prácticas-y-errores-comunes)
11. [Herramientas del ecosistema](#11-herramientas-del-ecosistema)
12. [Checklist de arranque](#12-checklist-de-arranque)

---

## 1. Qué es SDD y por qué esta estructura

Spec-driven development invierte el flujo tradicional: **la especificación es el
artefacto fuente de verdad y el código es el resultado de compilarla**, de forma
análoga a como un `.c` produce un binario. En vez de describir un cambio en el chat y
revisar lo que salga (*vibe coding*), primero se escribe qué debe hacer el cambio, se
convierte en un plan de tareas numeradas y recién entonces el agente implementa.

El motivo es concreto: sin guía estructurada, la tasa de acierto de un agente al primer
intento en cambios pequeños o medianos ronda apenas un tercio. Una feature típica
implica muchas decisiones; si el agente acierta el 80 % de cada una y hay 20 decisiones,
la probabilidad de acertar las 20 es `0.8²⁰ ≈ 1 %`. Una spec revisada por un humano lleva
cada decisión cerca de la certeza porque la decisión ya está tomada antes de escribir
código.

El flujo canónico tiene **cuatro fases con un checkpoint humano entre cada una**:

```
Especificar  →  Planificar  →  Tareas  →  Implementar
   spec.md        plan.md      tasks.md     código
     ▲              ▲            ▲            ▲
   revisión      revisión     revisión    revisión
```

Cada fase produce un archivo persistido en disco (no solo en el chat), de modo que
sobrevive entre sesiones y queda versionado en Git junto al código.

---

## 2. Estructura completa de archivos

```
mi-proyecto/
├── AGENTS.md                       # Contexto del proyecto (cross-tool, estándar abierto)
├── CLAUDE.md                       # Contexto específico de Claude (opcional)
├── README.md                       # Documentación para humanos
│
├── .claude/                        # Configuración de Claude Code
│   ├── commands/                   # Slash commands personalizados
│   │   ├── specify.md
│   │   ├── plan.md
│   │   └── tasks.md
│   ├── skills/                     # Skills reutilizables (SKILL.md)
│   │   └── generate-endpoint/
│   │       └── SKILL.md
│   ├── hooks/                      # Hooks determinísticos (validaciones)
│   │   └── pre-commit.sh
│   └── settings.json               # Permisos y configuración del agente
│
├── .specify/                       # Metadatos de SDD (si usas Spec Kit)
│   ├── memory/
│   │   └── constitution.md         # Principios no negociables del proyecto
│   └── templates/                  # Plantillas de spec/plan/tasks
│
├── specs/                          # Una carpeta por feature
│   ├── 2026-07-22-auth-usuarios/
│   │   ├── spec.md                 # QUÉ se construye
│   │   ├── plan.md                 # CÓMO se construye
│   │   └── tasks.md                # Lista ordenada de tareas
│   │
│   └── 2026-07-15-pagos/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
│
├── docs/                           # Documentación de referencia estable
│   ├── architecture.md
│   └── decisions/                  # ADRs (Architecture Decision Records)
│       └── 0001-elegir-postgres.md
│
├── src/                            # Código fuente
├── tests/                          # Pruebas
└── package.json / pyproject.toml   # Manifiesto del proyecto
```

En **monorepos**, se puede colocar un `AGENTS.md` en cada paquete: el agente lee el
archivo más cercano al fichero que está editando, así cada subproyecto lleva
instrucciones a medida.

---

## 3. Grado de estandarización de cada pieza

Conviene tener claro qué es un estándar formal y qué es una convención popular:

| Pieza | Estatus | Custodia |
|---|---|---|
| `AGENTS.md` | **Especificación abierta formal** | Linux Foundation (Agentic AI Foundation), desde dic. 2025 |
| Notación **EARS** | Estándar de facto en ingeniería de requisitos | Presentado en IEEE RE09 (2009) |
| Flujo de 4 fases y layout `specs/` | Convención de facto | Popularizada por GitHub Spec Kit |
| `CLAUDE.md` | Convención propietaria | Anthropic (Claude Code) |
| `SKILL.md` | Especificación propietaria | Anthropic |

Implicación práctica: para máxima portabilidad, pon el contexto del proyecto en
`AGENTS.md` (Claude Code también lo lee) y reserva `CLAUDE.md` solo para lo que sea
exclusivo de Claude.

---

## 4. AGENTS.md — contexto del proyecto

**Uso:** archivo raíz que todo agente lee al iniciar una tarea. Es el "README para
agentes": stack, comandos, convenciones y límites. Su objetivo es unificar en un solo
archivo lo que antes se duplicaba en `.cursorrules`, `CLAUDE.md`,
`copilot-instructions.md`, etc.

**Formato:** Markdown plano. No tiene campos obligatorios, pero la comunidad converge en
una estructura **WHAT / WHY / HOW**: qué es el proyecto, por qué existe, cómo se
construye.

**Regla de oro sobre el tamaño:** mantenerlo **corto** (recomendación habitual: por
debajo de ~150–300 líneas). Los modelos siguen de forma fiable un número limitado de
instrucciones antes de que el cumplimiento empiece a degradarse. Un archivo
auto-generado con todo dentro es un antipatrón: en estudios empíricos, los `AGENTS.md`
generados por IA llegaron a **reducir la tasa de éxito y aumentar el coste** porque
duplicaban información ya presente en el repo.

### Secciones recomendadas

- **Project overview:** qué es, lenguaje y framework principal con versiones.
- **Build & test commands:** comandos exactos con sus flags, no nombres vagos de tools.
- **Code style:** solo reglas que difieran de los defaults del lenguaje.
- **Do / Don't:** listas explícitas de lo permitido y lo prohibido.
- **Permission boundaries:** qué puede tocar el agente y qué no.

### Ejemplo

```markdown
# AGENTS.md

## Project overview
API REST de gestión de reservas. TypeScript 5.4, Node 20, Fastify 4, PostgreSQL 16.
Arquitectura hexagonal: dominio en `src/domain`, adaptadores en `src/adapters`.

## Build & test commands
- Instalar:      `pnpm install`
- Desarrollo:    `pnpm dev`
- Tests:         `pnpm test`            # Vitest; deben pasar antes de cualquier commit
- Test único:    `pnpm test <archivo>`
- Lint + format: `pnpm check`           # Biome; corre en pre-commit hook
- Migraciones:   `pnpm db:migrate`

## Code style
- Usar `Result<T, E>` para errores esperados; nunca lanzar excepciones en el dominio.
- Validación de entrada siempre con esquemas Zod en la capa de adaptadores.
- Sin `any`. Sin `console.log` en `src/` (usar el logger inyectado).

## Do / Don't
- ✅ Escribir un test de integración por cada endpoint nuevo.
- ✅ Leer la spec relevante en `specs/` antes de implementar.
- ❌ No modificar `src/domain` para resolver un problema de infraestructura.
- ❌ No añadir dependencias sin justificarlo en el plan.

## Permission boundaries
- No ejecutar migraciones contra la base de datos de producción.
- No commitear secretos ni archivos `.env`.
```

---

## 5. CLAUDE.md — contexto específico de Claude

**Uso:** contexto que Claude Code carga en cada sesión. Si ya tienes `AGENTS.md`, usa
`CLAUDE.md` **solo** para lo exclusivo de Claude (por ejemplo, referencias a skills
propias o comandos de Claude Code), y evita duplicar.

**Importante — es advisory, no determinístico:** Claude lee `CLAUDE.md` e *intenta*
seguirlo, pero no hay garantía de cumplimiento estricto, sobre todo con instrucciones
vagas o contradictorias. Se han documentado tres modos de "drift":

1. Reglas ignoradas durante la ejecución.
2. Reglas olvidadas a mitad de sesión, cuando el contexto se llena de código.
3. Reglas saltadas por considerarlas innecesarias.

**Conclusión:** para reglas que *no se pueden romper* (cobertura de tests, formato,
prohibición de secretos), no confíes en el texto de `CLAUDE.md` → usa **hooks**
(sección 9), que sí son determinísticos.

### Ejemplo (mínimo, complementando AGENTS.md)

```markdown
# CLAUDE.md

Este proyecto usa spec-driven development. Antes de escribir código:
1. Lee `AGENTS.md` para el contexto general.
2. Lee la spec activa en `specs/<feature>/spec.md`.
3. Sigue el plan en `specs/<feature>/plan.md`.

Skills disponibles en `.claude/skills/`. Usa `generate-endpoint` para nuevos endpoints.

Al terminar una tarea de `tasks.md`, márcala como completada y ejecuta `pnpm test`.
```

---

## 6. La carpeta `specs/` — el flujo de 4 fases

Cada feature vive en su propia carpeta fechada (`AAAA-MM-DD-nombre`) con tres archivos
que corresponden a las tres primeras fases del flujo.

### 6.1 `spec.md` — el QUÉ (fase Especificar)

Es el paso más importante y el que más gente salta. Define **qué** se construye sin
entrar en el cómo. Debe incluir: historias de usuario, criterios de aceptación (en
EARS), modelo de datos, endpoints requeridos, restricciones de seguridad y casos límite.

> **Regla económica:** cambiar una spec toma 10 minutos; cambiar una base de código
> construida sobre una spec equivocada toma días. Revisa la spec antes de que se escriba
> una sola línea de código.

**Ejemplo:**

```markdown
# Spec: Autenticación de usuarios

**Creado:** 2026-07-22 · **Estado:** Aprobado · **Autor:** @maria

## Problema
Los usuarios no pueden crear cuentas ni iniciar sesión. Necesitamos autenticación
por email + contraseña con sesiones basadas en tokens.

## Historias de usuario
- Como visitante, quiero registrarme con email y contraseña para tener una cuenta.
- Como usuario, quiero iniciar sesión para acceder a mis reservas.
- Como usuario, quiero cerrar sesión para proteger mi cuenta en equipos compartidos.

## Criterios de aceptación (EARS)
- El sistema deberá rechazar registros con emails ya existentes (respuesta 409).
- CUANDO un usuario envía credenciales válidas, el sistema deberá devolver un JWT
  con expiración de 24 h.
- CUANDO un usuario envía una contraseña incorrecta 5 veces en 15 min, el sistema
  deberá bloquear el login de esa cuenta durante 30 min.
- MIENTRAS un token esté expirado, el sistema deberá rechazar las peticiones con 401.
- El sistema deberá almacenar contraseñas hasheadas con Argon2id, nunca en texto plano.

## Modelo de datos
- `users`: id (uuid), email (único), password_hash, created_at, locked_until (nullable)

## Endpoints
- `POST /auth/register`  → 201 | 409
- `POST /auth/login`     → 200 {token} | 401 | 429
- `POST /auth/logout`    → 204

## Casos límite
- Email con mayúsculas/minúsculas → normalizar a minúsculas antes de comparar.
- Registro concurrente con el mismo email → la constraint de unicidad debe ganar.

## Fuera de alcance
- OAuth / login social (feature futura).
- Recuperación de contraseña (spec separada).
```

### 6.2 `plan.md` — el CÓMO (fase Planificar)

Traduce la spec en diseño técnico: qué archivos se crean o modifican, qué patrones se
siguen, qué dependencias se añaden. Aquí es donde se capturan las decisiones de
implementación para que la revisión detecte patrones conflictivos o soluciones a medias.

**Ejemplo:**

```markdown
# Plan: Autenticación de usuarios

## Enfoque
Servicio de dominio `AuthService` sin dependencias de infraestructura. El hashing y la
generación de JWT se inyectan como puertos. Rate limiting con un contador en Redis.

## Archivos
- `src/domain/auth/auth-service.ts`       (nuevo) — lógica de registro/login.
- `src/domain/auth/ports.ts`              (nuevo) — interfaces Hasher, TokenIssuer.
- `src/adapters/http/auth-routes.ts`      (nuevo) — rutas Fastify + validación Zod.
- `src/adapters/crypto/argon2-hasher.ts`  (nuevo) — implementa Hasher.
- `migrations/0007_create_users.sql`      (nuevo) — tabla users.

## Dependencias nuevas
- `argon2` — hashing de contraseñas (justificación: estándar recomendado por OWASP).

## Decisiones
- JWT firmado con clave simétrica desde `env.JWT_SECRET` (rotación futura, fuera de alcance).
- El bloqueo por intentos se persiste en `users.locked_until`, no en Redis, para que
  sobreviva a reinicios.
```

### 6.3 `tasks.md` — la lista ejecutable (fase Tareas)

Descompone el plan en **tareas ordenadas, pequeñas y testables**, idealmente con
trazabilidad hacia los requisitos de la spec. El agente ejecuta una por una.

**Ejemplo:**

```markdown
# Tasks: Autenticación de usuarios

- [ ] 1. Crear migración `0007_create_users.sql` con la tabla `users`.
- [ ] 2. Definir puertos `Hasher` y `TokenIssuer` en `src/domain/auth/ports.ts`.
- [ ] 3. Implementar `Argon2Hasher` (satisface criterio: hashing Argon2id).
- [ ] 4. Implementar `AuthService.register` (satisface: rechazo de email duplicado 409).
- [ ] 5. Implementar `AuthService.login` (satisface: JWT 24 h, 401 en credencial mala).
- [ ] 6. Añadir bloqueo por 5 intentos fallidos (satisface: bloqueo 30 min, 429).
- [ ] 7. Exponer rutas HTTP con validación Zod en `auth-routes.ts`.
- [ ] 8. Escribir tests de integración para register, login y logout.
- [ ] 9. Verificar contra la spec: correr `pnpm test` y revisar cada criterio.
```

### 6.4 La cuarta fase: Implementar + Validar

El agente ejecuta las tareas dentro de las restricciones de la spec, el plan, el
`AGENTS.md` y la constitución. Una práctica común es que Claude Code cree una **rama de
Git por feature** (`feature/auth-usuarios`), de modo que la implementación queda aislada
y revisable: el reviewer abre el PR y contrasta el código contra el `spec.md` en el mismo
repositorio, sin documentos externos.

La fase de **validación** (confirmar que el código cumple la spec) es la que más
herramientas tratan como opcional y donde el método se debilita si se descuida. No la
saltes: es el checkpoint que atrapa planes que se leían bien pero fallan en el primer
test.

---

## 7. La constitución del proyecto

**Uso:** un archivo de principios no negociables que *toda* acción del agente debe
respetar, transversal a todas las features. Es esencialmente una lista de sentencias
EARS "ubicuas" (siempre activas) sobre el proyecto en sí.

**Ubicación:** típicamente `AGENTS.md` en la raíz, o `.specify/memory/constitution.md`
si usas Spec Kit. Se commitea a control de versiones.

**Ejemplo:**

```markdown
# Constitución del proyecto

- El sistema deberá usar TypeScript en modo estricto.
- El sistema deberá rechazar PRs que reduzcan la cobertura de tests.
- El sistema deberá evitar dependencias en runtime sobre paquetes sin mantenimiento.
- El sistema deberá mantener la capa de dominio libre de dependencias de framework.
- Toda entrada externa deberá validarse antes de entrar al dominio.
```

Un buen punto de partida es escribir la constitución codificando las convenciones que ya
existen en el proyecto (un ejercicio de ~1 hora con el agente en modo entrevista).

---

## 8. Notación EARS para requisitos

**EARS** (Easy Approach to Requirements Syntax) convierte requisitos difusos en
sentencias testables y parseables por una IA. Un agente puede leer un requisito EARS,
generar el código y escribir un test que lo verifique, todo sin adivinar. Tiene cinco
patrones:

| Patrón | Plantilla | Ejemplo |
|---|---|---|
| **Ubicuo** (siempre activo) | El sistema deberá `<respuesta>`. | El sistema deberá cifrar los datos en reposo. |
| **Dirigido por evento** | CUANDO `<disparador>`, el sistema deberá `<respuesta>`. | CUANDO un usuario envía el formulario, el sistema deberá guardar el borrador. |
| **Dirigido por estado** | MIENTRAS `<estado>`, el sistema deberá `<respuesta>`. | MIENTRAS la sesión esté activa, el sistema deberá refrescar el token cada hora. |
| **Opcional (por feature)** | DONDE `<feature esté presente>`, el sistema deberá `<respuesta>`. | DONDE el plan sea premium, el sistema deberá permitir exportar a PDF. |
| **Comportamiento no deseado** | SI `<condición no deseada>`, ENTONCES el sistema deberá `<respuesta>`. | SI el pago es rechazado, ENTONCES el sistema deberá revertir la reserva. |

Escribir los criterios de aceptación en EARS es lo que hace que `tasks.md` pueda mapear
1:1 tareas ↔ requisitos ↔ tests.

---

## 9. La carpeta `.claude/`

Configuración específica de Claude Code.

### 9.1 `commands/` — slash commands

Archivos Markdown que definen comandos personalizados invocables con `/nombre`. Son la
forma de encapsular el flujo de las 4 fases.

**Ejemplo — `.claude/commands/specify.md`:**

```markdown
Genera una especificación en `specs/<fecha>-<slug>/spec.md` para la feature que describa
el usuario. Sigue esta estructura: Problema, Historias de usuario, Criterios de
aceptación en EARS, Modelo de datos, Endpoints, Casos límite, Fuera de alcance.
No escribas código. Al terminar, pídeme que revise la spec antes de continuar.
```

### 9.2 `skills/` — skills reutilizables

`SKILL.md` es la especificación de Anthropic para empaquetar capacidades reutilizables
del agente, con frontmatter YAML y, opcionalmente, scripts y referencias.

**Ejemplo — `.claude/skills/generate-endpoint/SKILL.md`:**

```markdown
---
name: generate-endpoint
description: Genera un nuevo endpoint siguiendo las convenciones del proyecto.
---
Al generar un endpoint nuevo:
1. Lee la spec relevante en `specs/`.
2. Sigue el patrón de autenticación de `src/adapters/http/auth-routes.ts`.
3. Valida la entrada con esquemas Zod.
4. Escribe tests de integración.
5. Valida contra la constitución del proyecto.
```

### 9.3 `hooks/` — validaciones determinísticas

A diferencia de `AGENTS.md`/`CLAUDE.md` (advisory), los hooks se ejecutan de forma
determinística. Son el mecanismo correcto para reglas que no se pueden romper.

**Ejemplo — `.claude/hooks/pre-commit.sh`:**

```bash
#!/usr/bin/env bash
set -euo pipefail

pnpm check          # lint + format; falla si hay errores
pnpm test           # falla si algún test no pasa

# Bloquea secretos accidentales
if git diff --cached --name-only | grep -qE '\.env$'; then
  echo "ERROR: no commitear archivos .env" >&2
  exit 1
fi
```

### 9.4 `settings.json` — permisos y configuración

Define qué comandos puede ejecutar el agente sin pedir permiso, qué está vetado, etc.
(consulta la documentación de Claude Code para el esquema exacto y actualizado).

---

## 10. Buenas prácticas y errores comunes

**Hacer:**

- Mantener `AGENTS.md` corto (< ~150–300 líneas) y con estructura hub-and-spoke: enlaza
  a documentos detallados en vez de meterlo todo dentro.
- Revisar la spec y el plan **antes** de implementar (los checkpoints humanos son el
  corazón del método).
- Usar EARS para criterios de aceptación → tests derivables directamente.
- Usar hooks para reglas no negociables; usar los archivos de contexto para guía.
- Versionar specs, planes y tareas junto al código.
- Pinnear en el trace de cada run qué versiones de spec/plan/skills la moldearon
  (facilita el post-mortem: pasa de "excavación" a "búsqueda").

**Evitar:**

- Auto-generar un `AGENTS.md` gigante con todo dentro: reduce éxito y sube coste al
  duplicar lo que ya está en el repo.
- Documentar rutas de archivos concretas en los archivos de contexto: se pudren.
- Usar el archivo de contexto como si fuera un linter: para eso están los hooks.
- Saltar la fase de validación.

**Expectativa realista:** la fase de spec cuesta horas de escritura antes de que corra
código, y las primeras features se sienten más lentas que el *vibe coding*. El punto de
equilibrio suele llegar tras varias features; a partir de ahí el retorno es alto y
compuesto.

---

## 11. Herramientas del ecosistema

| Herramienta | Qué es | Ideal para |
|---|---|---|
| **GitHub Spec Kit** | Implementación de referencia open-source. CLI `specify` + plantillas y slash commands. Agent-agnostic (Claude Code, Copilot, Cursor, Codex, Gemini…). | Empezar rápido con la convención más adoptada. |
| **AWS Kiro** | IDE agéntico con SDD integrado; popularizó EARS para specs de IA. | Equipos que quieren un IDE-first con SDD nativo. |
| **OpenSpec** | Librería ligera y agnóstica de framework: CLI + formato de spec (Markdown + YAML frontmatter). | SDD sin atarse a un vendor. |
| **BMAD-METHOD** | Metodología comunitaria (convenciones + prompt packs) previa a Spec Kit e influyó en su diseño. | Equipos que prefieren convenciones sobre tooling. |
| **Claude Code skills / Plan Mode** | Mecanismos nativos de Claude para planificar y encapsular capacidades. | Flujos centrados en Claude Code. |

Recomendación: si quieres el camino más transitado y portable, arranca con **GitHub Spec
Kit** y `AGENTS.md`.

---

## 12. Checklist de arranque

```
[ ] Crear AGENTS.md con overview, comandos de build/test, code style, do/don't, límites
[ ] (Opcional) Crear CLAUDE.md solo con lo específico de Claude
[ ] Escribir la constitución (principios no negociables, en EARS ubicuo)
[ ] Crear la carpeta specs/
[ ] Configurar .claude/commands/ con /specify, /plan, /tasks
[ ] Configurar .claude/hooks/ para reglas no negociables (tests, lint, secretos)
[ ] (Opcional) Instalar GitHub Spec Kit: pipx install specify-cli && specify init
[ ] Primera feature: /specify → revisar → /plan → revisar → /tasks → implementar → validar
```

---

*Nota sobre estándares: `AGENTS.md` es una especificación abierta bajo custodia de la
Linux Foundation; EARS es un estándar de facto de ingeniería de requisitos (IEEE RE09);
el layout de `specs/` y el flujo de 4 fases son convenciones de facto popularizadas por
GitHub Spec Kit; `CLAUDE.md` y `SKILL.md` son convenciones propietarias de Anthropic.*
