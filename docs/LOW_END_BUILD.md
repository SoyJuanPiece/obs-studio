# OBS Studio — Perfil para PCs antiguas (Low-End)

Este perfil de compilación reduce la carga de OBS en equipos con poca CPU,
poca RAM o sin GPU dedicada, manteniendo las funciones necesarias para
transmitir y grabar.

## Compilar

```sh
cmake -S . -B build-lowend -DENABLE_LOW_END=ON
cmake --build build-lowend
```

La opción `ENABLE_LOW_END` está desactivada por defecto, así que una
compilación normal no cambia.

## Qué cambia

**Módulos que no se compilan** (solo con `ENABLE_LOW_END=ON`):

- `aja`, `aja-output-ui` — tarjetas de captura/salida AJA.
- `decklink`, `decklink-captions`, `decklink-output-ui` — tarjetas Blackmagic DeckLink.
- `obs-vst` — host de plugins VST.
- `vlc-video` — fuente VLC.

**Módulos que se mantienen** a propósito: `obs-browser` (el frontend lo usa
para el actualizador, el diálogo "Novedades" y el inicio de sesión OAuth),
`obs-websocket`, `obs-webrtc`, `obs-x264`, `obs-ffmpeg`, `obs-nvenc`, `obs-qsv11`
y los módulos de captura de tu plataforma.

**Valores por defecto para el primer arranque** (solo si el perfil está activo):

| Ajuste | Estándar | Low-End |
|---|---|---|
| Resolución base (canvas) | Resolución del monitor (máx. 1920x1080) | 1280x720 |
| Vista previa | Activada | Desactivada |
| Bitrate de transmisión (modo simple) | 6000 kbps | 2500 kbps |
| Preset x264 (modo simple) | veryfast | ultrafast |

Estos valores solo son defaults: el usuario puede cambiarlos en Ajustes en
cualquier momento, y una configuración existente no se modifica.

## Consejos para equipos muy limitados

- Usa un codificador por hardware si tu PC lo tiene (NVENC, QuickSync o AMF).
  Es la mejora más grande de CPU.
- Mantén la FPS en 30. Cambia a 60 solo si la CPU lo aguanta.
- Desactiva la vista previa (clic derecho en la vista previa → "Disable Preview") mientras transmites.
- Quita fuentes de navegador y filtros que no uses.

## Limitación

Codificar video siempre consume CPU o GPU. Con este perfil el consumo baja
mucho, pero no llega a 1 % mientras se transmite; ese número solo es posible
con el sistema en reposo.
