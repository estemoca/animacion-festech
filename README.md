# Efecto Finger Frame

## Descripción del Proyecto

**Efecto Finger Frame** es una aplicación web interactiva de efectos visuales en tiempo real que permite al usuario aplicar filtros artísticos sobre la cámara usando únicamente las manos como herramienta de control. Mediante la detección de manos de Google MediaPipe, el sistema rastrea 21 puntos de referencia (landmarks) en cada mano —utilizando las yemas de los dedos índice y medio de ambas manos— para formar un "marco" cuadrilateral en el video en vivo. Todo el contenido dentro de ese marco es procesado en tiempo real por un motor de renderizado WebGL2 con shaders personalizados en GLSL, que aplican **12 efectos visuales** distintos (desde estilos anime y cómic hasta efectos de glitch, VHS, térmico y pixel art). Un sofisticado sistema de estabilización suaviza el movimiento del marco para evitar temblores, y el proyecto incluye además un **modo sintético** sin cámara con una escena de demostración animada. Toda la aplicación vive en un solo archivo `index.html` autocontenido que emplea JavaScript vanilla (ES Modules), no tiene dependencias locales ni backend, y carga únicamente la librería MediaPipe desde un CDN en tiempo de ejecución.

---

## Tecnologías Utilizadas

| Categoría | Tecnología | Detalles |
|---|---|---|
| **Marcado** | HTML5 | Elementos semánticos, `<canvas>`, `<video>` |
| **Estilos** | CSS3 (inline) | Flexbox, `backdrop-filter`, transiciones |
| **Lenguaje** | JavaScript (ES Modules) | Vanilla JS, sin transpilador ni build |
| **Renderizado GPU** | **WebGL 2** (GLSL ES 3.00) | Shaders vertex + fragment personalizados (~220 líneas GLSL) |
| **IA/ML** | **MediaPipe Hand Landmarker** `v0.10.14` | Modelo de detección de manos cargado vía CDN |
| **Video** | `navigator.mediaDevices.getUserMedia` | API de acceso a cámara (requiere HTTPS o localhost) |
| **Servidor local** | XAMPP (Apache) / VS Code Live Server | Puerto 5501 configurado en `.vscode/settings.json` |

---

## Librerías Externas

El proyecto es **zero-dependency local**. La única dependencia externa se carga directamente desde CDN en tiempo de ejecución:

- **MediaPipe Tasks Vision** — `@mediapipe/tasks-vision@0.10.14`
  - CDN: `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14`
  - Modelo de mano: `https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task` (~10 MB)

---

## Estructura del Proyecto

```
efectoFestEch/
  .vscode/
    settings.json        -- Configuración de VS Code Live Server (puerto 5501)
  index.html             -- Aplicación completa (~755 líneas, autocontenido)
```

No hay `package.json`, `composer.json`, `tsconfig.json`, `.gitignore`, ni ningún otro archivo de configuración. Todo el código — HTML, CSS y JavaScript — vive dentro de un solo archivo `index.html`.

---

## Funcionalidad Principal

### Concepto

El usuario forma un **marco rectangular** con las yemas de los dedos índice y medio de ambas manos frente a la cámara. El área dentro de ese marco recibe un filtro visual en tiempo real.

### Funcionalidades Clave

#### 1. Detección y Rastreo de Manos (MediaPipe)
- Detecta **2 manos** simultáneamente vía webcam.
- Extrae 4 landmarks clave por mano: Muñeca (0), Pulgar (4), Dedo índice (8), Dedo medio (9).
- El "finger frame" se forma con las puntas del índice + medio de ambas manos.

#### 2. Estabilización del Cuadrilátero
- Sistema de estabilización avanzado para evitar jitter en el marco detectado.
- Suavizado por lerp, detección de teletransporte, fade in/out, tolerancia de pérdida (25 frames).
- Alineación por rotación y reflejo para evitar que el marco se invierta.

#### 3. 12 Filtros WebGL2
Cada filtro se aplica dentro del marco del cuadrilátero mediante máscara SDF:

| Filtro | Etiqueta | Descripción |
|---|---|---|
| `toon` | Dibujos | Colores posterizados + bordes de tinta Sobel |
| `invert` | Negativo | Inversión completa de colores |
| `comic` | Comics | Puntos Ben-Day + posterización + bordes de tinta |
| `sketch` | Lapiz | Boceto a lápiz con tramas y grano |
| `pixel` | Pixel | Efecto de mosaico/pixelación |
| `noir` | Noir | Blanco y negro de alto contraste |
| `glitch` | Glitch | Separación de canales RGB por bandas |
| `vhs` | VHS | Ondulación de cinta analógica + scanlines + ruido |
| `neon` | Neon | Borde arcoíris con brillo y halo |
| `thermal` | Termica | Paleta de falsos colores estilo cámara térmica |
| `ghibli` | Ghibli | Paleta cálida estilo Studio Ghibli + contornos suaves |
| `simpsons` | Simpsons | Cel shading amarillo estilo Los Simpson |

#### 4. Contorno Animado
- Borde punteado animado alrededor del cuadrilátero detectado con puntos de esquina con brillo.

#### 5. Modo Sintético (sin cámara)
- Escena de demostración incorporada con un personaje de dibujos animados (cara, ojos, boca, camiseta) y barras de calibración, animadas por funciones seno. Posiciones simuladas de "manos" permiten probar sin webcam.

#### 6. Controles de Interfaz
- Selector de filtros (barra inferior).
- Botón de contorno ON/OFF.
- Botón de pantalla completa.
- Barra de estado mostrando FPS, número de manos y estado del frame.
- Overlays para: instrucción inicial, estado de carga y error (con reintento).

#### 7. Detección de Protocolo
- Desactiva automáticamente el botón de cámara si no se sirve por HTTPS o localhost (requiere contexto seguro para `getUserMedia`).

---

## Arquitectura

```
index.html (archivo único, ~755 líneas)
  |
  |-- <style> block          (~30 líneas CSS)
  |
  |-- <body> HTML            (~30 líneas DOM)
  |
  |-- <script type="module">
       |-- Importa MediaPipe desde CDN
       |-- Constantes: lista de filtros, parámetros del estabilizador
       |-- Referencias DOM y handlers de UI
       |-- Módulo estabilizador: detección de cuadrilátero, suavizado, alineación
       |-- Renderizador WebGL2: shaders vertex/fragment, pipeline de texturas
       |-- Escena sintética: generador de patrones de prueba en canvas
       |-- Bucle principal: requestAnimationFrame tick
       |-- Inicialización y bindings de botones
```

La aplicación es **zero-dependency, zero-build, zero-backend** — un solo archivo HTML que carga una librería CDN en tiempo de ejecución.

---

## Requisitos

- **Navegador moderno** con soporte WebGL 2 y MediaPipe (Chrome, Edge, Firefox recientes).
- **Cámara web** para el modo interactivo (o usar el modo sintético sin cámara).
- **Servidor local** — XAMPP o VS Code Live Server en puerto 5501.

---

## Cómo Ejecutar

1. Iniciar XAMPP o VS Code Live Server.
2. Navegar a `http://localhost:5501` (o la URL que proporcione el servidor).
3. Permitir acceso a la cámara cuando el navegador lo solicite.
4. Usar los dedos índice y medio de ambas manos para formar el marco.
5. Seleccionar filtros desde la barra inferior.
# animacion-festech
