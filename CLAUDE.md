# CLAUDE.md

Este archivo ofrece orientación a Claude Code (claude.ai/code) cuando trabaja con el código de este repositorio.

## Idioma

Responde siempre en español en este proyecto.

## Estado del proyecto

Juego de Arkanoid en HTML, CSS y JavaScript puros, **cero dependencias** (sin npm, sin bundler, sin paso de build). Según el README, el juego **todavía no está implementado**: solo existen los assets. No hay `index.html`, ni configuración de tests, ni de lint. Tampoco es un repositorio git todavía.

Cuando exista un `index.html`, se ejecuta abriéndolo en el navegador o sirviendo la carpeta de forma estática (p. ej. `python -m http.server`). Conviene servirlo por HTTP porque el spritesheet se dibuja en un canvas fuera de pantalla.

## Assets (`assets/`)

- `spritesheet-breakout.png` — único spritesheet con todos los gráficos.
- `spritesheet.js` — script de ámbito global (sin módulos). Cárgalo con una etiqueta `<script>` antes del código del juego. Proporciona:
  - `SPRITES` — rectángulos de origen (`sx, sy, sw, sh`) para `paddle`, `ball` y `blocks.<color>` (gray, red, yellow, cyan, magenta, hotpink, green).
  - `EXPLOSION_FRAMES` / `EXPLOSION_DURATION` (150) — animación de rotura de 4 frames por color de bloque. `gray` reutiliza los frames de `red`.
  - `loadSpritesheet(cb)` — cargador asíncrono; copia la imagen a un canvas fuera de pantalla y luego ejecuta todos los callbacks en cola. Inicia siempre el juego desde este callback, porque las funciones de dibujo no hacen nada (sin error) hasta que la imagen se carga.
  - `drawSprite(ctx, name, x, y, w, h)` — `name` es `'paddle'`, `'ball'` o `'block_<color>'`. `drawFrame(ctx, frame, x, y, w, h)` dibuja un frame explícito, como los de explosión.
  - La ruta de la imagen está fija en `assets/spritesheet-breakout.png`, **relativa a la página HTML**. Por eso `index.html` debe estar en la raíz del proyecto.
- `sounds/ball-bounce.mp3`, `sounds/break-sound.mp3` — efectos de sonido para los rebotes de la bola y la rotura de bloques.

## Flujo spec-driven (`.agents/skills/`)

Dos skills definen el proceso para funcionalidades grandes. Ambas son solo manuales (`disable-model-invocation`). Provienen de `Klerith/fernando-skills`, fijadas en `skills-lock.json`.

- `/spec <descripción>` — hace preguntas de aclaración y luego escribe `specs/NN-slug.md` en estado `Draft`. Nunca escribe código. `.agents/skills/spec/template.md` da la estructura.
- `/spec-impl <NN-slug>` — solo se ejecuta sobre specs cuyo estado sea "Approved" (Aprobado). El cambio de estado lo hace la persona, no el agente. Crea la rama `spec-NN-slug`, implementa un paso del plan a la vez y se detiene para revisar el diff. **Nunca hace commit por su cuenta**. La configuración está en `specs/.spec-config.yml` (`AutoCreateBranch`).

Las skills esperan un repositorio git (`spec-impl` usa `git status` y ramas), y esta carpeta aún no lo es. Ejecuta `git init` antes de usar `/spec-impl`. El idioma de las specs debe coincidir con el de las existentes; el README y el `CLAUDE.md` del directorio padre están en español.
