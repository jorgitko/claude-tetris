# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Comandos

No hay build, ni bundler, ni tests, ni linter. Sin `package.json` ni dependencias.

```bash
start index.html            # abrir directo (Windows)
python -m http.server 8000  # servidor local (recomendado)
```

Verificación = abrir en navegador y jugar. No existe suite de tests.

## Arquitectura

Tres archivos: `index.html` (DOM + dos canvas), `style.css` (dark theme), `game.js` (toda la lógica).

`game.js` es un único script global con `'use strict'` — sin módulos, sin clases, sin `export`. El estado del juego vive en variables `let` de módulo (`board, current, next, score, lines, level, paused, gameOver, dropInterval, dropAccum, animId`), no en un objeto de estado. `init()` las reinicia todas; el botón Reiniciar llama a `init()` directamente.

Puntos clave para no romper nada:

- **Tablero**: matriz `ROWS × COLS`; cada celda es `0` o índice `1–7` que indexa a la vez `COLORS` y `PIECES`. Los índices deben mantenerse alineados entre esos dos arrays (ambos empiezan con `null` en la posición 0).
- **Rotación**: `rotateCW` transpone + invierte filas. No hay SRS real; `tryRotate` prueba wall kicks `[0,-1,1,-2,2]` en horizontal. Las piezas se definen en matrices cuadradas (I en 4×4, O en 2×2, resto 3×3) precisamente para que esa rotación funcione.
- **Game loop**: `requestAnimationFrame` con acumulador de tiempo (`dropAccum >= dropInterval`). Pausa/reanudación cancela y relanza el `rAF` reseteando `lastTime` — cualquier cambio ahí debe evitar un salto de `dt` gigante.
- **`collide(shape, ox, oy)`** es el único chequeo de validez: lo usan mover, rotar, soft/hard drop, `ghostY` y la detección de game over en `spawn()`. Permite `ny < 0` (pieza asomando por arriba).
- **Acoplamiento canvas↔constantes**: `COLS × BLOCK` y `ROWS × BLOCK` deben coincidir con `width`/`height` del `<canvas id="board">` en `index.html` (300×600). Igual para `next-canvas` (120×120 = 4×`NB`, con `NB = 30` hardcodeado en `drawNext`).
- **HUD**: `updateHUD()` escribe en el DOM; se llama tras cada keydown y en `clearLines()`. `draw()` solo pinta canvas.

## Convenciones

Idioma: código e identificadores en inglés; texto de UI, comentarios y README en español.
