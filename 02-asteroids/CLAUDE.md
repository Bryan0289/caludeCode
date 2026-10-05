# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Clon del clásico Asteroids en HTML5 Canvas puro. Sin frameworks, sin bundler, sin dependencias, sin `package.json`. Todo el código vive en un único archivo `game.js` (~420 líneas), cargado directamente por `index.html` vía `<script src="game.js">`.

## Cómo correr

Abrir `index.html` directamente en el navegador (doble clic) o servir localmente:

```bash
npx serve .
```

No hay build step, ni linter, ni tests configurados en el repo.

## Arquitectura

Todo en `game.js`, organizado en secciones delimitadas por comentarios `// ── Nombre ──`:

- **Input**: `keys` (estado continuo) y `justPressed` (edge-trigger, se resetea al leer vía `pressed(code)`) — usado para que el disparo no se repita mientras se mantiene `Space`.
- **Utils**: `wrap(v, max)` implementa el espacio toroidal (todo objeto que sale por un borde reaparece en el opuesto); usado por `Ship`, `Asteroid` y `Bullet`.
- **Clases de entidades** (`Bullet`, `Asteroid`, `Ship`, `Particle`): cada una expone `update(dt)` y `draw()`. `dead = true` marca una entidad para ser filtrada del array correspondiente en el siguiente ciclo — no hay pooling.
- **Estado global del juego**: variables sueltas a nivel de módulo (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) reinicializadas en `initGame()`. `state` es una máquina de tres estados: `'playing' | 'dead' | 'gameover'`.
- **Loop principal**: `requestAnimationFrame` clásico, `dt` en segundos clamped a 0.05 máx (evita saltos grandes tras un tab inactivo). Cada frame: `update(dt)` luego `draw()`.

### Detalles de gameplay que no son obvios leyendo una sola función

- `Asteroid.size` va de 3 (grande) a 1 (pequeño); `RADII`, `SPEEDS`, `POINTS` están indexados por tamaño (índice 0 sin usar). Al morir, `split()` genera 2 asteroides de `size - 1` en la misma posición; tamaño 1 no se parte más.
- Colisión bala-asteroide y nave-asteroide es por distancia simple (`dist(a,b) < radio`), no AABB ni SAT.
- La nave tiene invencibilidad temporal (`ship.invincible`, en segundos) tanto al iniciar partida como al reaparecer tras morir; se dibuja parpadeando (`draw()` corta el render en ciertos frames) y se ignoran colisiones nave-asteroide mientras `invincible > 0`.
- Nivel completa cuando `asteroids.length === 0`; `nextLevel()` incrementa `level` y genera `3 + level` asteroides nuevos.
- Coordenadas de canvas fijas: `W=800, H=600` (declaradas tanto en `index.html` como en `game.js` — mantenerlas sincronizadas si cambian).

### Discrepancia README vs código

El `README.md` menciona "power-ups especiales" y un asteroide tipo "estrella fugaz" como características incluidas. Estas **no existen en `game.js` actualmente** — si se pide implementar o modificar estas features, hay que crearlas desde cero, no asumir que ya están presentes.
