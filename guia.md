# Claude Code de cero a experto: primeros capítulos

## Capítulo 0 Antes de empezar

Claude Code es un asistente de programación que vive en tu terminal. A diferencia de un chat, puede leer tu repositorio, editar archivos, ejecutar comandos y verificar su propio trabajo, siempre dentro de los permisos que tú le des.

Esta guía está pensada para que pases de "lo instalé y le pedí algo" a usarlo como una herramienta profesional: con contexto persistente, reglas de seguridad, automatizaciones y un flujo de git ordenado. Cada capítulo termina con algo que puedes aplicar hoy mismo.

### Para quién es

- Desarrolladores que ya usan git y la terminal con soltura básica.

- Equipos que quieren estandarizar cómo usan un agente de código.

- Personas que probaron Claude Code y sienten que "a veces hace cosas raras": casi siempre es falta de contexto o de límites, y eso se arregla.

### El modelo mental correcto

Piensa en Claude Code como un colega muy rápido que acaba de llegar al equipo: sabe programar muy bien, pero no conoce tu proyecto, tus convenciones ni tus manías. Tu trabajo es darle tres cosas:

- **Contexto**: qué es el proyecto, cómo se ejecuta, qué convenciones sigue (capítulo 3).

- **Una tarea clara**: objetivo, restricciones y cómo saber que terminó (capítulo 2).

- **Límites**: qué puede hacer sin preguntar y qué nunca debe tocar (capítulos 4, 7 y 11).

Todo lo demás (skills, comandos, subagentes, MCP) son formas de empaquetar esas tres cosas para no repetirlas.

### Convenciones de esta guía

  FormatoSignificado
  `$ comando`Se escribe en tu terminal normal (bash, zsh).
  `/comando`Comando slash dentro de una sesión de Claude Code.
  `> texto`Lo que tú escribes como prompt dentro de la sesión.
  `[ENTRE_CORCHETES]`Reemplázalo por tu valor.

**Consejo**Lee los capítulos 1 a 4 en orden. Los demás puedes consultarlos cuando los necesites; están escritos para leerse de forma independiente.

## Capítulo 1 Instalación y primeros pasos

### Requisitos

- Un sistema compatible: macOS, Linux (Ubuntu, Debian y similares) o Windows (de forma nativa o a través de WSL, según indique la documentación vigente).

- Una cuenta con acceso a Claude Code: una suscripción de Claude que lo incluya o una cuenta de la Consola de Anthropic con facturación por API.

- Si instalas con npm: Node.js 18 o superior.

### Instalar

Hay dos vías habituales. El instalador nativo no depende de Node:

```
# Opción A: instalador nativo (macOS / Linux / WSL)
curl -fsSL https://claude.ai/install.sh | bash

# Opción B: vía npm (requiere Node.js 18+)
npm install -g @anthropic-ai/claude-code
```

**No uses sudo con npm**Si `npm install -g` falla por permisos, no lo arregles con `sudo`. Configura un prefijo de npm en tu carpeta personal o usa un gestor de versiones como nvm. Instalar paquetes globales como root es una fuente clásica de problemas de permisos y de seguridad.

Comprueba la instalación:

```
claude --version     # muestra la versión instalada
claude doctor        # revisa el estado de la instalación
claude update        # busca e instala actualizaciones
```

### Primera sesión

```
cd ~/proyectos/mi-api
claude
```

La primera vez te pedirá iniciar sesión (se abre el navegador) y confiar en la carpeta actual. Claude Code trabaja **sobre el directorio desde el que lo lanzas**, así que entra siempre a la raíz del proyecto antes de ejecutarlo.

Un buen primer prompt no es "arregla esto", sino uno de exploración:

```
> Dame un resumen de este repositorio: propósito, estructura de carpetas,
  cómo se ejecuta, cómo se corren los tests y qué partes te parecen frágiles.
  No modifiques nada.
```

### Modo interactivo y modo no interactivo

  UsoComando
  Sesión interactiva`claude`
  Sesión con un prompt inicial`claude "explica src/auth.ts"`
  Respuesta única y salir (scripts, CI)`claude -p "resume el último commit"`
  Pasar datos por tubería`cat error.log | claude -p "¿cuál es la causa raíz?"`
  Continuar la conversación más reciente`claude -c`
  Elegir una conversación anterior`claude -r`
  Salida en JSON (con `-p`)`claude -p "..." --output-format json`

### Atajos dentro de la sesión

  AtajoQué hace
  `Esc`Interrumpe a Claude en cualquier momento sin perder el contexto.
  `Esc` dos vecesVuelve a un mensaje anterior para editarlo o retroceder.
  `Shift+Tab`Alterna entre modos de permisos (por ejemplo, aceptar ediciones o modo plan).
  `@ruta/archivo`Menciona un archivo o carpeta para incluirlo como contexto.
  `!comando`Ejecuta un comando de shell directamente; su salida queda en la conversación.
  `/`Abre la lista de comandos slash disponibles.

### Comandos slash esenciales

  ComandoPara qué sirve
  `/help`Ayuda y lista de comandos de tu versión.
  `/init`Analiza el proyecto y genera un `CLAUDE.md` inicial.
  `/clear`Borra la conversación actual. Úsalo entre tareas no relacionadas.
  `/compact`Resume la conversación para liberar contexto sin empezar de cero.
  `/model`Cambia el modelo de la sesión.
  `/permissions`Ver y editar reglas de permisos.
  `/memory`Abre los archivos CLAUDE.md para editarlos.
  `/agents`, `/mcp`, `/hooks`Gestionan subagentes, servidores MCP y hooks.

**Ventana de contexto**Todo lo que lees, pegas y ejecutas ocupa contexto. Una sesión larga con muchos temas mezclados empeora las respuestas. Regla práctica: **una tarea, una conversación**. Termina, haz commit y usa `/clear`.

## Capítulo 2 Cómo pedir tareas

La calidad del resultado depende mucho más de cómo describes la tarea que de cualquier truco. Un buen prompt para código responde cuatro preguntas: qué quiero, dónde, con qué restricciones y cómo sé que está terminado.

  Prompt débil
```
arregla el login
```

No dice qué falla, dónde mirar, ni cómo comprobar que quedó bien. Claude tendrá que adivinar, y puede "arreglar" algo que no estaba roto.

  Prompt sólido
```
Al iniciar sesión con un email en
mayúsculas devuelve 401. Revisa
@src/auth/login.ts y el modelo User.
Normaliza el email a minúsculas al
guardar y al buscar. Añade un test
que lo cubra y ejecuta npm test.
No cambies la API pública.
```

### La plantilla de cuatro partes

```
CONTEXTO:   [qué está pasando, dónde, qué ya intentaste]
OBJETIVO:   [el resultado concreto que quieres]
RESTRICCIONES: [qué no tocar, estilo, librerías permitidas]
TERMINADO CUANDO: [tests que pasan, comando que funciona, salida esperada]
```

No hace falta escribir las etiquetas siempre, pero repasa mentalmente las cuatro antes de enviar.

### Explorar, planificar, implementar, confirmar

Para cualquier tarea que no sea trivial, sepárala en fases y pide explícitamente que se detenga entre ellas:

- **Explorar**: "Lee los archivos relevantes para X. No escribas código todavía."

- **Planificar**: "Propón un plan en pasos numerados con los archivos que tocarás." Puedes usar el **modo plan** (capítulo 4), en el que Claude investiga y propone sin editar.

- **Implementar**: "Implementa los pasos 1 y 2. Ejecuta los tests después de cada paso."

- **Confirmar**: "Muéstrame el diff resumido y propón un mensaje de commit."

Este ciclo evita el error más caro: que Claude se lance a escribir código basándose en una suposición equivocada.

### Dale una forma de verificarse

Claude rinde mucho mejor cuando puede comprobar su propio trabajo. Siempre que puedas, indícale el comando que define el éxito:

- Tests: `npm test -- users`, `pytest tests/test_pagos.py -q`, `go test ./...`

- Tipos y linters: `tsc --noEmit`, `ruff check .`, `cargo clippy`

- Un comando reproducible: "ejecuta `curl localhost:3000/health` y debe responder 200".

### Desarrollo guiado por tests con Claude

```
> Vamos a hacer TDD. Primero escribe tests en tests/test_descuentos.py para
  calcular_descuento(): cliente nuevo 0%, cliente frecuente 10%, cupón válido
  se suma, máximo 30%. Ejecútalos y confirma que fallan. No implementes aún.

> Ahora implementa calcular_descuento() hasta que pasen. No modifiques los tests.
```

La frase "no modifiques los tests" es importante: sin ella, un agente puede "resolver" un test ajustando la aserción.

### Más técnicas que marcan la diferencia

- **Señala archivos con `@`** en lugar de describirlos. Ahorra búsquedas y errores.

- **Pega errores completos**, con el stack trace, no un resumen tuyo.

- **Pega capturas** de interfaces o diagramas: Claude Code acepta imágenes (en muchas terminales, pegando desde el portapapeles o arrastrando el archivo).

- **Pide alternativas** cuando no estés seguro: "dame dos enfoques con pros y contras antes de decidir".

- **Corrige pronto**: si ves que va por mal camino, pulsa `Esc` y redirige. Es más barato que dejarlo terminar.

- **Sé específico con el alcance**: "solo en `src/billing/`", "sin dependencias nuevas", "compatible con Python 3.10".

- **Pide que pregunte**: "si algo es ambiguo, pregúntame antes de suponer".

### Prompts útiles para el día a día

```
> ¿Por qué existe esta función? Revisa el historial de git de
  @src/legacy/tarifas.py y resume las decisiones importantes.

> Encuentra dónde se valida el token JWT y explícame el flujo completo
  desde que llega la petición hasta que se rechaza.

> Este test falla de forma intermitente. Ejecútalo 10 veces, identifica el
  patrón y propón la causa antes de cambiar nada.

> Refactoriza @src/utils/fechas.js para eliminar duplicación, sin cambiar
  el comportamiento. Los tests actuales deben seguir pasando sin tocarlos.
```

## Capítulo 3 CLAUDE.md: la memoria de tu proyecto

`CLAUDE.md` es un archivo Markdown que Claude Code carga automáticamente al iniciar la sesión. Es el lugar para todo lo que le dirías a un colega nuevo el primer día: cómo se ejecuta el proyecto, qué convenciones hay y qué errores evitar.

### Dónde puede vivir

  UbicaciónAlcance¿En git?
  `./CLAUDE.md` (raíz del repo)Todo el equipo, este proyectoSí
  `./subcarpeta/CLAUDE.md`Se añade cuando Claude trabaja en esa parte (útil en monorepos)Sí
  `~/.claude/CLAUDE.md`Tú, en todos tus proyectosNo

Además, las organizaciones pueden distribuir un archivo de políticas a nivel de sistema. Consulta `/memory` para ver qué archivos se están cargando en tu sesión.

### Créalo con /init y luego edítalo

Ejecuta `/init` en la raíz del proyecto: Claude analizará el repositorio y propondrá un `CLAUDE.md`. Tómalo como borrador. Lo valioso es lo que **solo tú sabes** y que no se deduce leyendo el código.

### Ejemplo de un buen CLAUDE.md

CLAUDE.md

```
# API de Reservas

API REST en Node 20 + TypeScript + Express. Base de datos PostgreSQL vía Prisma.

## Comandos
- Instalar: `pnpm install`
- Desarrollo: `pnpm dev` (puerto 3000)
- Tests: `pnpm test` (Vitest). Un archivo: `pnpm test src/reservas`
- Tipos: `pnpm typecheck` · Lint: `pnpm lint --fix`
- Migraciones: `pnpm prisma migrate dev --name `

## Arquitectura
- `src/routes/` solo valida entrada (zod) y llama a `src/services/`.
- La lógica de negocio vive en `src/services/`. Nunca accedas a Prisma desde routes.
- Errores: lanza `AppError(codigo, mensaje)`; el middleware los traduce a HTTP.

## Convenciones
- Nombres de funciones y variables en inglés; mensajes al usuario en español.
- Fechas siempre en UTC con `date-fns`. No uses `moment`.
- Cada servicio nuevo necesita tests en `src/services/__tests__/`.

## Reglas importantes
- NO edites archivos en `prisma/migrations/` ya existentes; crea una migración nueva.
- NO añadas dependencias sin preguntar.
- Antes de terminar una tarea ejecuta `pnpm typecheck && pnpm test`.
```

### Importar otros archivos

Puedes referenciar otros documentos con la sintaxis `@ruta` dentro de `CLAUDE.md`, para no duplicar información que ya existe:

```
Consulta @docs/arquitectura.md para el diagrama de módulos
y @package.json para los scripts disponibles.
```

### Buenas prácticas

- **Corto y concreto.** Cada línea se lee en cada sesión y consume contexto. Apunta a una o dos pantallas.

- **Comandos exactos**, copiables, mejor que descripciones.

- **Escribe reglas, no deseos.** "Usa `AppError`" es mejor que "intenta manejar bien los errores".

- **Actualízalo cuando Claude se equivoque.** Si repite un error, añade una línea que lo evite. Puedes pedirle: "añade a CLAUDE.md la regla que acabamos de acordar".

- **Nada de secretos.** Va a git y lo lee un modelo. Ni contraseñas, ni tokens, ni URLs internas sensibles.

- **Preferencias personales** (idioma de las respuestas, tu estilo) van en `~/.claude/CLAUDE.md`, no en el del equipo.

**Ejemplo de CLAUDE.md personal**
`~/.claude/CLAUDE.md`: "Responde siempre en español. Prefiero cambios pequeños y commits atómicos. Antes de instalar paquetes, pregúntame. Uso pnpm, no npm."
