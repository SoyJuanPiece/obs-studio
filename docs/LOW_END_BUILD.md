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
| FPS | 30 | 60 |
| Vista previa | Activada | Desactivada |
| Codificador (modo simple) | NVENC si existe, si no x264 | NVENC si existe, si no QuickSync (Intel), si no x264 |
| Bitrate de transmisión (modo simple) | 6000 kbps | 3500 kbps |
| Preset x264 (modo simple, solo si se usa x264) | veryfast | ultrafast |

Estos valores solo son defaults: el usuario puede cambiarlos en Ajustes en
cualquier momento, y una configuración existente no se modifica.

## Cambios de código (además de los valores por defecto)

- **Límite de FPS en la vista previa** (30 FPS con el perfil activo). Nueva
  función `obs_display_set_max_fps()` en libobs (`libobs/obs-display.c`): los
  cuadros que llegan antes del intervalo se saltan y se mantiene la última
  imagen. Solo la vista previa principal lo usa; proyectores y otros displays
  no cambian.
- **Medidores de audio a ~30 Hz** en vez de ~60 Hz
  (`frontend/components/VolumeMeter.cpp`).
- **Escalado bilineal** por defecto (solo afecta cuando hay que escalar).
- **Preset de QuickSync corregido.** El menú de preset del modo simple
  (Speed / Balanced / Quality) guardaba el valor en `preset`, pero el encoder
  de QuickSync lee `target_usage`, así que el menú no tenía efecto. Ahora se
  pasa a `target_usage`. Esto afecta a todas las compilaciones, no solo al
  perfil ligero; con la configuración por defecto (balanced) el resultado no
  cambia. El perfil ligero usa `speed` (TU7) por defecto.

Estos cambios no quitan funciones: la vista previa sigue funcionando, solo se
actualiza menos seguido.

## Requisitos de sistema

OBS 28 en adelante requiere **Windows 10 u 11 de 64 bits**. Para PCs con
Windows 7 u 8 hay que usar OBS 27.2.4 (última versión con Qt5), que no es
este código base.

QuickSync requiere una CPU Intel con gráficos integrados y el controlador Intel
instalado. El H.264 por QuickSync existe desde las CPUs Core de 2.ª generación,
pero la calidad y el rendimiento varían por modelo: pruébalo en la PC real. Si
no está disponible, el perfil usa x264 por software.

## Consejos para equipos muy limitados

- Usa un codificador por hardware si tu PC lo tiene (NVENC, QuickSync o AMF).
  Es la mejora más grande de CPU.
- Si 720p60 se congela, baja a 30 FPS en Ajustes → Video.
- Desactiva la vista previa (clic derecho en la vista previa → "Disable Preview") mientras transmites.
- Quita fuentes de navegador y filtros que no uses.

## Limitación

Codificar video siempre consume CPU o GPU. Con este perfil el consumo baja
mucho, pero no llega a 1 % mientras se transmite; ese número solo es posible
con el sistema en reposo.
