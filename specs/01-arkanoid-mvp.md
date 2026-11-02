# SPEC 01 — MVP jugable de Arkanoid

> **Status:** Aprobado
> **Depends on:** ninguna
> **Date:** 2026-11-01
> **Objective:** Crear un Arkanoid jugable con un nivel, 3 vidas y puntuación, controlado con teclado, en HTML, CSS y JavaScript puros.

## Alcance

**Dentro:**

- Un único nivel fijo: 6 filas × 12 columnas de bloques (72 bloques), un color por fila.
- Pala controlada con `←` `→` o `A` `D`.
- Una bola que rebota en paredes, techo, pala y bloques.
- Cada bloque se rompe de un golpe y desaparece al instante, sin animación.
- 3 vidas y puntuación de la partida actual (10 puntos por bloque).
- Estados de juego: inicio, bola pegada a la pala, jugando, victoria y game over.
- Lanzar la bola con `Espacio` y reiniciar con `Enter` en las pantallas de victoria y game over.
- Canvas con resolución lógica fija de 800×600, centrado y escalado con CSS manteniendo la proporción.
- Uso del spritesheet existente (`spritesheet.js`, `spritesheet-breakout.png`) para pala, bola y bloques.

**Fuera de alcance (para futuras specs):**

- Varios niveles y progresión de dificultad.
- Récord guardado entre sesiones (`localStorage`).
- Control con ratón o táctil.
- Animación de explosión de los bloques (`EXPLOSION_FRAMES`).
- Sonidos (`ball-bounce.mp3`, `break-sound.mp3`).
- Bloques resistentes (el sprite `gray` no se usa).
- Power-ups, pausa, menú de opciones y silenciar el sonido.
- Marcadores de vidas con sprites propios (se muestran como texto).

## Modelo de datos

Variables globales en `js/game.js` (sin módulos, igual que `spritesheet.js`):

```js
const GAME = { width: 800, height: 600 };

// 'start' | 'ready' | 'playing' | 'won' | 'gameover'
let state = {
  phase: 'start',
  score: 0,
  lives: 3,
};

const paddle = { x: 0, y: 560, w: 120, h: 14, speed: 420 }; // speed en px/s

const ball = { x: 0, y: 0, size: 16, vx: 0, vy: 0, speed: 360 }; // speed en px/s

// Bloque: 64×32 en canvas (sprite 32×16 al doble)
// { x, y, w: 64, h: 32, color: 'red', alive: true }
let blocks = [];

const BLOCK_COLORS = ['red', 'yellow', 'cyan', 'magenta', 'hotpink', 'green']; // fila 0 a 5
const LAYOUT = { cols: 12, rows: 6, offsetX: 16, offsetY: 80 };
```

Convenciones:

- Origen en la esquina superior izquierda; unidades en píxeles lógicos del canvas.
- Movimiento por tiempo (`deltaTime` en segundos), no por frame.
- La bola se trata como un cuadrado de 16×16 en las colisiones.
- Ángulo de salida de la pala: depende de dónde golpea la bola (centro = vertical, extremos = unos 60° respecto a la vertical).

## Plan de implementación

1. Crear `index.html` en la raíz con un `<canvas id="game" width="800" height="600">`, enlazar `style.css`, cargar `assets/spritesheet.js` y después `js/game.js`. Crear `style.css` (fondo oscuro, canvas centrado y escalado con CSS, `image-rendering: pixelated`) y un `js/game.js` que, desde el callback de `loadSpritesheet`, pinta el fondo. Prueba manual: abrir la página, ver el canvas centrado y sin errores en consola.
2. Añadir el bucle de juego con `requestAnimationFrame` y `deltaTime`, y el estado `start` con texto "Pulsa Espacio para empezar". Prueba manual: la pantalla de inicio se ve y `Espacio` pasa a `ready`.
3. Dibujar la pala con `drawSprite` y moverla con `←` `→` / `A` `D`, limitada a los bordes del canvas. Prueba manual: la pala se mueve y no sale del canvas.
4. Dibujar la bola pegada a la pala (estado `ready`) y lanzarla con `Espacio` con un ángulo inicial hacia arriba. Rebote en paredes laterales y techo. Prueba manual: la bola sale y rebota en tres lados.
5. Colisión bola-pala con ángulo según el punto de impacto. Si la bola cae por debajo del canvas, se resta una vida y se vuelve a `ready`. Prueba manual: la bola rebota en la pala y se pierde si se falla.
6. Generar los 72 bloques con `LAYOUT` y `BLOCK_COLORS`, y dibujarlos. Prueba manual: se ven 6 filas de colores.
7. Colisión bola-bloque: invertir el eje correcto, marcar el bloque como `alive = false` (deja de dibujarse y de colisionar) y sumar 10 puntos. Prueba manual: los bloques desaparecen al primer golpe y la puntuación sube.
8. HUD con puntuación y vidas como texto, y pantallas de `won` (sin bloques vivos) y `gameover` (0 vidas). `Enter` reinicia la partida. Prueba manual: ganar y perder muestran su pantalla y se puede reiniciar.

## Criterios de aceptación

- [ ] Al abrir `index.html` (por HTTP) no hay errores en la consola del navegador.
- [ ] La pantalla inicial muestra el texto de inicio y no se mueve nada hasta pulsar `Espacio`.
- [ ] `←` y `A` mueven la pala a la izquierda; `→` y `D` a la derecha; nunca sale del canvas.
- [ ] La bola empieza pegada a la pala y solo se lanza al pulsar `Espacio`.
- [ ] La bola rebota en las paredes laterales y en el techo sin atravesarlos.
- [ ] Golpear la bola con el centro de la pala la manda casi vertical; golpearla en un extremo la manda inclinada hacia ese lado.
- [ ] Si la bola sale por debajo del canvas, las vidas bajan en 1 y la bola vuelve a estar pegada a la pala.
- [ ] Hay 72 bloques al empezar (6 filas × 12 columnas), con 6 colores distintos.
- [ ] Un bloque desaparece en el primer golpe, en el mismo frame, sin animación.
- [ ] Romper un bloque suma exactamente 10 puntos.
- [ ] Al romper los 72 bloques aparece la pantalla de victoria con la puntuación final (720).
- [ ] Al llegar a 0 vidas aparece la pantalla de game over.
- [ ] Pulsar `Enter` en victoria o game over reinicia con 3 vidas, 0 puntos y 72 bloques.
- [ ] El juego no carga ni reproduce ningún archivo de `assets/sounds/`.
- [ ] Al redimensionar la ventana, el canvas conserva la proporción 4:3 y la jugabilidad no cambia.
- [ ] El proyecto sigue sin dependencias: no hay `package.json` ni paso de build.

## Decisiones tomadas y descartadas

- **Sí:** un único nivel fijo. Es lo mínimo para que el juego sea jugable; los niveles van en otra spec.
- **Sí:** 3 vidas y puntuación de la partida actual. Dan sensación de juego completo sin tocar la persistencia.
- **No:** récord en `localStorage`. Añade versionado y manejo de errores que el MVP no necesita.
- **Sí:** teclado (`←` `→` / `A` `D`). Es lo estándar y lo más simple. **No:** ratón y táctil, para no añadir layout ni eventos extra.
- **Sí:** todos los bloques de un golpe que desaparecen al instante. **No:** animación de explosión, aunque el spritesheet ya trae los frames, para reducir el MVP; va en otra spec. **No:** bloque `gray` resistente, porque añadiría estado por bloque.
- **Sí:** `index.html` + `style.css` + `js/game.js` con scripts globales, coherente con `spritesheet.js`. **No:** varios archivos ni módulos ES, porque exigirían servir por HTTP y son excesivos para un MVP.
- **Sí:** resolución lógica fija 800×600 escalada con CSS. Mantiene la física independiente de la pantalla. **No:** canvas de tamaño variable.
- **Sí:** bloques dibujados al doble de su sprite (64×32) para que 12 columnas llenen 768 px de los 800 px de ancho (16 px de margen a cada lado).
- **No:** sonidos en el MVP. Evita la política de autoplay del navegador y el manejo de errores de audio; van en otra spec.
- **Sí:** estados `start`, `ready`, `playing`, `won` y `gameover`. **No:** pausa, que va en otra spec.
- **Sí:** valores concretos (pala 120×14, bola 16×16, velocidad de bola 360 px/s, de pala 420 px/s) como punto de partida. Se pueden ajustar al probar el juego sin cambiar la spec.
- **Sí:** vidas y puntos como texto en el HUD. **No:** iconos de vida, porque el spritesheet no los incluye.

## Riesgos identificados

| Riesgo                                                             | Mitigación                                                                                                        |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| La bola atraviesa bloques o la pala a velocidad alta (túnel)       | Mover la bola con `deltaTime` limitado (máx. 1/30 s) y velocidad moderada, de modo que no recorra más de 16 px por paso. |
| Rebote en la esquina de dos bloques con el eje equivocado          | Resolver la colisión eligiendo el eje de menor solapamiento y procesar un solo bloque por paso.                    |
| La bola queda en un bucle horizontal sin bajar                     | Garantizar un componente vertical mínimo de velocidad tras cada rebote.                                           |
| Abrir `index.html` con doble clic bloquea la lectura del spritesheet | Servir con `python -m http.server`; el README debe indicarlo (como indica `CLAUDE.md`).                           |

## Lo que **no** entra en esta spec

- Varios niveles y mayor dificultad.
- Récord persistente entre sesiones.
- Control con ratón o táctil.
- Animación de explosión de los bloques.
- Sonidos.
- Bloques resistentes, power-ups y pausa.
- Opciones y pantallas de menú.

Cada una de estas cosas, si se hace, va en su propia spec.
