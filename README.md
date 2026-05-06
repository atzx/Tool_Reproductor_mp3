# Tool Player mp3

Reproductor de música en el navegador construido con vanilla HTML, CSS y JavaScript. Sin dependencias externas.

## Características

### Reproducción
- Carga de archivos via input file o drag & drop
- Control de reproducción: play/pause, siguiente, anterior
- Barra de progreso con tooltip de tiempo al hacer hover
- Control de volumen
- Velocidad ajustable: 0.5x, 1x, 1.5x, 2x
- Fade in al iniciar la reproducción

### Playlist
- Lista de reproducción con scroll y scrollbar personalizada
- Eliminar canciones individuales
- Limpiar playlist completa
- Ordenar por nombre A-Z / Z-A
- Búsqueda y filtro en tiempo real
- Contador de reproducciones por canción
- Exportar playlist como archivo JSON

### Modos de reproducción
- Modo aleatorio (shuffle) con animación visual
- Repetición: desactivada / toda la lista / una canción
- Sleep timer: 5min, 10min, 30min, 1h

### Visuales
- Disco de vinilo animado que gira al reproducir
- Ecualizador visual con barras de frecuencia en tiempo real
- Tooltips en todos los botones
- Micro-interacciones: escalado al click, desplazamiento en hover
- Tema oscuro por defecto con toggle a tema claro
- Diseño responsive para móviles
- Notificaciones toast

### Extras
- Persistencia de preferencias en localStorage (volumen, modo, tema, velocidad)
- Picture-in-Picture (PiP)
- Media Session API (controles desde lock screen / notificaciones del SO)
- Atajos de teclado

## Atajos de teclado

| Tecla           | Acción                  |
|-----------------|-------------------------|
| `Space`         | Play / Pause            |
| `→`             | Adelantar 5s            |
| `←`             | Retroceder 5s           |
| `↑`             | Subir volumen           |
| `↓`             | Bajar volumen           |
| `N`             | Siguiente canción       |
| `P`             | Canción anterior        |
| `S`             | Alternar shuffle        |
| `R`             | Alternar repetición     |
| `T`             | Cambiar tema            |

## Cómo usar

1. Abre `index.html` en cualquier navegador moderno.
2. Arrastra archivos de audio al área punteada o usa el botón "Seleccionar archivos".
3. Usa los controles para reproducir, pausar, navegar y ajustar la reproducción.

## Stack técnico

- HTML5 semántico
- CSS3 (Flexbox, CSS Custom Properties, animaciones, media queries)
- JavaScript vanilla (Web Audio API, Canvas, Media Session API, localStorage)

## Especificaciones

Ver [SPEC.md](SPEC.md) para documentación técnica detallada.
