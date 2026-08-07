# Tetris - Teste de Integração com Claude Code para editar código

## Utilizando @claude para chamar o claude code e solicitar que analise e atualize o código a partir de uma Issue

Implementación del clásico **Tetris** en JavaScript vanilla, usando HTML5 Canvas y CSS. Sin dependencias externas, sin frameworks, sin proceso de build: solo abrir y jugar.

![Tech](https://img.shields.io/badge/HTML5-Canvas-orange)
![Tech](https://img.shields.io/badge/CSS3-blueviolet)
![Tech](https://img.shields.io/badge/JavaScript-Vanilla-yellow)

---

## Tabla de contenidos

- [Tetris](#tetris)
  - [Tabla de contenidos](#tabla-de-contenidos)
  - [Qué hace el proyecto](#qué-hace-el-proyecto)
  - [Cómo ejecutar el juego](#cómo-ejecutar-el-juego)
    - [Opción 1: abrir el archivo directamente](#opción-1-abrir-el-archivo-directamente)
    - [Opción 2: servidor local (recomendado)](#opción-2-servidor-local-recomendado)
  - [Controles](#controles)
  - [Tabla de récords local](#tabla-de-récords-local)
  - [Cómo funciona](#cómo-funciona)
    - [1. `index.html`](#1-indexhtml)
    - [2. `style.css`](#2-stylecss)
    - [3. `game.js`](#3-gamejs)
    - [Flujo del juego](#flujo-del-juego)
  - [Tecnologías](#tecnologías)
  - [Estructura del proyecto](#estructura-del-proyecto)
  - [Personalización](#personalización)
  - [Licencia](#licencia)

---

## Qué hace el proyecto

Es una versión jugable del Tetris clásico con todas las mecánicas que esperarías:

- Tablero de **10 × 20** celdas.
- Las **7 piezas estándar** (I, O, T, S, Z, J, L) con colores diferenciados.
- Pieza especial **"tuerca"** (3×3 con un hueco vacío en el centro): aparece raramente (~8% de las veces) y deja un hueco imposible de rellenar en la fila donde encaja, para mayor desafío.
- **Rotación** con _wall kicks_ básicos (pequeños desplazamientos para que la pieza pueda rotar pegada a la pared).
- **Soft drop** (bajada acelerada) y **hard drop** (caída instantánea).
- **Pieza fantasma** (_ghost piece_): muestra dónde aterrizará la pieza actual.
- **Vista previa** de la siguiente pieza.
- **Sistema de puntuación** clásico de Tetris (100 / 300 / 500 / 800 multiplicado por nivel).
- **Niveles** que aumentan cada 10 líneas y aceleran la caída.
- **Pausa** y **Game Over** con opción de reinicio.
- **Tabla de récords local**: guarda el Top 5 de puntuaciones (con nombre de jugador) en `localStorage`, visible en la pantalla de inicio y en el overlay de Game Over.
- **Temas visuales / skins**: selector con cuatro estilos (Retro, Neon, Pastel, Pixel art) que cambia colores y forma de dibujar los bloques al vuelo, sin recargar la página. La preferencia se guarda en `localStorage`.

---

## Cómo ejecutar el juego

No hay nada que instalar ni compilar. Tienes dos opciones:

### Opción 1: abrir el archivo directamente

```bash
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

### Opción 2: servidor local (recomendado)

Cualquier servidor estático funciona. Algunos ejemplos:

```bash
# Con Python 3
python3 -m http.server 8000

# Con Node.js (npx)
npx serve .

# Con PHP
php -S localhost:8000
```

Después abre `http://localhost:8000` en el navegador.

---

## Controles

| Tecla     | Acción                            |
| --------- | --------------------------------- |
| `←` / `→` | Mover la pieza horizontalmente    |
| `↑` o `X` | Rotar la pieza en sentido horario |
| `↓`       | Soft drop (bajar más rápido)      |
| `Espacio` | Hard drop (caída instantánea)     |
| `P` / `Esc` | Pausar / reanudar               |

---

## Menú de pausa

Al pausar (`P` o `Esc`) se abre un menú con varias opciones, y todos los controles del juego quedan bloqueados hasta cerrarlo (para evitar movimientos accidentales al volver):

- **Reanudar** — cierra el menú y continúa la partida donde estaba.
- **Reiniciar** — empieza una partida nueva sin recargar la página.
- **Ver controles** — muestra dentro del propio menú la lista de teclas (con un botón **Volver** para regresar a las opciones).
- **Nivel inicial** — selector (1–10) para elegir con qué nivel empezará la **próxima** partida (al pulsar "Reiniciar" o tras un Game Over).

---

## Tabla de récords local

El juego guarda automáticamente el **Top 5** de puntuaciones en `localStorage` del navegador (clave `tetris-highscores`), sin necesidad de servidor ni base de datos:

- Se muestra en la **pantalla de inicio** (antes de pulsar "Jugar") y en el **overlay de Game Over**.
- Si la puntuación de la partida entra en el Top 5, al perder aparece un campo de texto para introducir el nombre del jugador (máx. 12 caracteres); al guardar, la entrada se resalta en la lista.
- Junto a la lista se muestra también el **mejor combo** (líneas eliminadas en jugadas consecutivas sin fallar) y las **líneas máximas** conseguidas entre las partidas guardadas en el Top 5.
- Un botón **"Resetear récords"** (disponible en ambas pantallas) borra todos los récords guardados, previa confirmación.

El combo se calcula en `game.js`: cada vez que `clearLines()` elimina al menos una línea, `combo` se incrementa (y `maxCombo` guarda el pico de la partida); si una pieza se fija sin eliminar ninguna línea, `combo` vuelve a `0`.

---

## Cómo funciona

El juego se compone de tres archivos que cooperan:

### 1. `index.html`

Define la estructura visual:

- Un `<canvas id="board">` de **300 × 600** píxeles donde se renderiza el tablero.
- Un panel lateral con `SCORE`, `LINES`, `LEVEL`, vista de la siguiente pieza, un `<select id="skin-select">` para elegir el tema visual y la lista de controles.
- Un overlay para los estados **PAUSA** (menú completo con reanudar, reiniciar, ver controles y nivel inicial) y **GAME OVER** (que añade el formulario de nombre, si la puntuación entra en el Top 5, y la tabla de récords).
- Un overlay de **pantalla de inicio** (`#start-overlay`), visible al cargar la página, con la tabla de récords y el botón "Jugar" que arranca la partida.

### 2. `style.css`

Aporta el aspecto visual con estética _dark / retro arcade_: fondo oscuro, tipografía monoespaciada para los marcadores y _backdrop blur_ en los overlays.

### 3. `game.js`

Contiene toda la lógica del juego. A grandes rasgos:

- **Modelo del tablero**: una matriz `ROWS × COLS` donde cada celda guarda `0` (vacía) o un índice de color (1–8) que identifica la pieza.
- **Piezas**: definidas como matrices cuadradas. Para rotar se calcula la transposición + reverso de filas (`rotateCW`). La pieza especial "tuerca" (índice 8) es un anillo 3×3 con el centro en `0`; su rotación es un no-op visual y su hueco central nunca puede colisionar ni rellenarse, dejando un agujero permanente en la fila.
- **Detección de colisiones** (`collide`): comprueba que ninguna celda de la pieza salga del tablero ni se solape con bloques ya fijados.
- **Wall kicks** (`tryRotate`): si la rotación choca, intenta desplazar la pieza ±1 y ±2 columnas antes de descartar el giro.
- **Game loop** (`loop`): basado en `requestAnimationFrame`, acumula el tiempo transcurrido y baja la pieza una fila cuando se supera `dropInterval`.
- **Limpieza de líneas** (`clearLines`): recorre el tablero de abajo hacia arriba; cada fila completa se elimina y se inserta una vacía en la cima.
- **Puntuación**: usa la tabla clásica `[0, 100, 300, 500, 800]` multiplicada por el nivel actual; el hard drop suma 2 puntos por celda recorrida y el soft drop 1 punto por fila.
- **Nivel y velocidad**: el nivel sube cada 10 líneas; la velocidad de caída se calcula como `max(100, 1000 − (level − 1) × 90)` milisegundos.
- **Ghost piece** (`ghostY`): proyecta la posición final de la pieza actual hacia abajo y la dibuja con `globalAlpha = 0.2`.
- **Récords** (`loadHighScores` / `addHighScore` / `renderHighScores`): el Top 5 se guarda como JSON en `localStorage` bajo la clave `HIGH_SCORES_KEY`; cada entrada tiene `{ id, name, score, lines, combo }`. `qualifiesForHighScore` decide si la puntuación actual merece pedir el nombre del jugador.
- **Temas visuales** (`THEMES`, `currentSkin`): cada tema define una paleta de colores (`colors`), un color de fondo del canvas (`background`) y un color de grilla (`gridColor`). El dibujo de cada bloque delega en una función por tema (`drawBlockRetro`, `drawBlockNeon`, `drawBlockPastel`, `drawBlockPixel`) seleccionada vía `SKIN_DRAWERS[currentSkin]`. El `<select id="skin-select">` cambia `currentSkin`, lo persiste en `localStorage` y fuerza un redibujado inmediato (`draw()` + `drawNext()`) mientras la partida está en curso, sin recargar la página ni afectar el estado de la partida.

### Flujo del juego

```
carga de página
  └─ renderAllHighScores()          → pinta el Top 5 en la pantalla de inicio

"Jugar" → init()
  ├─ createBoard()                  → matriz vacía
  ├─ next = randomPiece()
  ├─ spawn()                        → mueve next a current y genera nueva next
  └─ requestAnimationFrame(loop)
        ↓
   loop(timestamp)
     ├─ acumula dt
     ├─ si dt ≥ dropInterval → baja la pieza o llama a lockPiece()
     ├─ draw()  (grid + tablero + ghost + pieza actual)
     └─ requestAnimationFrame(loop)

   keydown → mover / rotar / soft-drop / hard-drop / pausa
```

Cuando una pieza recién generada ya colisiona al aparecer (`spawn`), se dispara `endGame()` y se muestra el overlay de **Game Over**. Si la puntuación entra en el Top 5 (`qualifiesForHighScore`), se pide el nombre del jugador antes de guardar el récord (`addHighScore`) y refrescar la tabla en ambas pantallas.

---

## Tecnologías

- **HTML5** — marcado y dos elementos `<canvas>` (tablero y vista previa).
- **CSS3** — _flexbox_, variables de color, `backdrop-filter` y `box-shadow`.
- **JavaScript (ES6+) vanilla** — `const`/`let`, _arrow functions_, _spread operator_, `Array.from`, _template literals_…
- **Canvas 2D API** — para todo el renderizado del juego.
- **`requestAnimationFrame`** — para el bucle de juego sincronizado con el navegador.

**Sin dependencias.** No hay `package.json`, ni bundler, ni transpilador.

---

## Estructura del proyecto

```
03-tetris/
├── index.html      # Estructura del DOM y canvas
├── style.css       # Estilos del juego (dark theme)
├── game.js         # Toda la lógica del Tetris (~300 líneas)
└── README.md
```

---

## Personalización

Algunos parámetros fáciles de tunear en `game.js`:

| Constante      | Significado                              | Por defecto           |
| -------------- | ---------------------------------------- | --------------------- |
| `COLS`         | Columnas del tablero                     | `10`                  |
| `ROWS`         | Filas del tablero                        | `20`                  |
| `BLOCK`        | Tamaño en píxeles de cada celda          | `30`                  |
| `COLORS`       | Paleta de colores por tipo de pieza      | 8 colores             |
| `LINE_SCORES`  | Puntos por 1, 2, 3 o 4 líneas eliminadas | `[0,100,300,500,800]` |
| `dropInterval` | Velocidad inicial de caída en ms         | `1000`                |
| `NUT_CHANCE`   | Probabilidad de que salga la pieza "tuerca" | `0.08`             |
| `HIGH_SCORES_KEY` | Clave de `localStorage` para los récords | `'tetris-highscores'` |
| `MAX_HIGH_SCORES` | Cantidad de puntuaciones guardadas en el Top | `5`                |
| `THEMES`       | Paletas y renderizadores por tema visual (Retro/Neon/Pastel/Pixel art) | 4 temas |

> Si cambias `COLS`, `ROWS` o `BLOCK`, recuerda ajustar también `width` y `height` del `<canvas id="board">` en `index.html` para que coincida (`COLS × BLOCK` × `ROWS × BLOCK`).

---

## Licencia

Proyecto de uso libre con fines educativos y de práctica.
