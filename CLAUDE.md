# CLAUDE.md

Este archivo da guía a Claude Code (claude.ai/code) al trabajar en este repositorio.

## Proyecto

Clon jugable de Tetris en JavaScript vanilla, HTML5 Canvas y CSS. Sin dependencias, sin build, sin gestor de paquetes, sin suite de tests.

## Comandos

No hay herramientas de build/lint/test. Para ejecutar el juego:

```bash
open index.html        # macOS, o abre el archivo directo en el navegador
```

O sírvelo local (necesario si agregas features que requieran `fetch`/módulos con restricciones CORS):

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

No hay suite de tests automatizada — verifica cambios jugando en el navegador.

## Arquitectura

Tres archivos, sin módulos/bundler — todo carga vía un único tag `<script src="game.js">`.

- `index.html` — estructura DOM: canvas principal `<canvas id="board">` (300×600, o sea `COLS*BLOCK` × `ROWS*BLOCK`), un `<canvas id="next-canvas">` para preview de pieza, elementos HUD (`#score`, `#lines`, `#level`), y un div `#overlay` compartido reusado para estados PAUSE y GAME OVER.
- `style.css` — solo tema visual dark/retro arcade; sin lógica.
- `game.js` — toda la lógica del juego, archivo único, sin clases, funciones planas + estado mutable a nivel módulo (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.).

### Mecánicas clave en `game.js`

- **Modelo del tablero**: matriz `ROWS × COLS`, cada celda es `0` (vacía) o índice de color de pieza `1–7`.
- **Piezas**: array `PIECES` de matrices cuadradas. Rotación (`rotateCW`) es transposición + reverso de filas, no guarda estados de rotación por pieza.
- **Colisión** (`collide`): comprueba límites del tablero y solape con celdas ya fijadas.
- **Wall kicks** (`tryRotate`): tras rotar, intenta offsets `[0, -1, 1, -2, 2]` columnas antes de descartar la rotación.
- **Game loop** (`loop`): impulsado por `requestAnimationFrame`, acumula `dt` y baja la pieza una fila cuando `dropAccum >= dropInterval`.
- **Limpieza de líneas** (`clearLines`): recorre de abajo hacia arriba, elimina filas completas e inserta filas vacías arriba.
- **Puntuación**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplicado por `level` actual; hard drop suma 2 puntos por celda caída, soft drop suma 1 punto por fila.
- **Nivel/velocidad**: nivel sube cada 10 líneas; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): proyecta la pieza actual hacia abajo hasta su fila de aterrizaje, dibujada con `globalAlpha = 0.2`.

### Flujo

`init()` crea el tablero, siembra `next` vía `randomPiece()`, llama `spawn()` (promueve `next` a `current`, genera nuevo `next`), luego inicia el loop de `requestAnimationFrame`. Si una pieza recién generada colisiona de inmediato, dispara `endGame()` y muestra overlay GAME OVER. Input de teclado (listener `keydown`) maneja movimiento/rotación/soft-drop/hard-drop/pausa; `P` alterna pausa independiente del estado game-over.

### Constantes ajustables (todas en `game.js`)

`COLS`, `ROWS`, `BLOCK` (px por celda), `COLORS`, `LINE_SCORES`, `dropInterval`. Si cambias `COLS`/`ROWS`/`BLOCK`, actualiza también los atributos `width`/`height` de `<canvas id="board">` en `index.html` para que coincidan (`COLS*BLOCK` × `ROWS*BLOCK`).
