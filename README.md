# LRC-CheckEdGen

[Español](#espanol) | [English](#english)

[![Docker Hub version](https://img.shields.io/docker/v/oct4vyus/lrc-checkedgen?sort=semver&label=frontend)](https://hub.docker.com/r/oct4vyus/lrc-checkedgen/tags)
[![Docker Hub pulls](https://img.shields.io/docker/pulls/oct4vyus/lrc-checkedgen?logo=docker)](https://hub.docker.com/r/oct4vyus/lrc-checkedgen)
[![Latest release](https://img.shields.io/github/v/release/Oct4vyus/LRC-CheckEdGen?display_name=tag)](https://github.com/Oct4vyus/LRC-CheckEdGen/releases/latest)
[![License](https://img.shields.io/github/license/Oct4vyus/LRC-CheckEdGen)](https://github.com/Oct4vyus/LRC-CheckEdGen/blob/main/LICENSE)

<a id="espanol"></a>
## Español

**LRC-CheckEdGen** ayuda a usuarios y administradores de servidores multimedia como **Jellyfin, Plex y Navidrome** a auditar su biblioteca musical, encontrar canciones sin letras sincronizadas y generar o editar archivos `.lrc` con inteligencia artificial local.

Imágenes disponibles en Docker Hub:

- Frontend: [`oct4vyus/lrc-checkedgen:latest`](https://hub.docker.com/r/oct4vyus/lrc-checkedgen) (actualmente `1.3.0`)
- Backend: [`oct4vyus/lrcgen-container:1.0.0`](https://hub.docker.com/r/oct4vyus/lrcgen-container)

### Funciones principales

- Audita álbumes y muestra qué pistas tienen letras sincronizadas y cuáles están pendientes.
- Genera letras con OpenAI Whisper y aislamiento vocal UVR5.
- Permite elegir el idioma del álbum antes de generar, para reducir errores de autodetección.
- Incluye un editor web para revisar letras y ajustar sus tiempos.
- Muestra el progreso de generación por pista.
- Conserva los archivos `.lrc` existentes: no los sobrescribe.
- Puede utilizarse con ambos contenedores para generación con IA o solo con LRC-Checker v6.2.2 para auditoría y edición cuando no se necesita la IA.

### Novedades de v1.2.0 y v1.3.0

- El selector de idioma ofrece español, inglés, portugués, autodetección y un código ISO-639-1 personalizado. La última selección se recuerda y el idioma se muestra durante el progreso.
- Las pistas sin segmentos vocales pueden marcarse como `no_lyrics` en lugar de tratarse como fallos; el resumen final distingue letras guardadas, errores, omisiones por idioma y pistas instrumentales.
- El editor virtualiza las filas para manejar letras extensas con mayor fluidez. El endpoint `/archivo` transmite audio por bloques y la regeneración de informes puede agruparse con `REPORT_REFRESH_DELAY_SEC`.

### Despliegue con Docker Compose

Este ejemplo despliega la interfaz y el backend de IA. Adapta las rutas del host, la zona horaria, el idioma predeterminado y la URL del servidor a tu instalación.

```yaml
version: "3.8"

services:
  lrc-checkedgen:
    image: oct4vyus/lrc-checkedgen:latest
    container_name: lrc-checkedgen
    restart: unless-stopped
    networks:
      - lrc_network
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Argentina/Cordoba
      - MUSIC_DIR=/music
      - OUTPUT_DIR=/reports
      - CHECK_ORPHANS=true
      - VERBOSE=true
      - REVIEW_THRESHOLD_SEC=40.0
      - REPORT_REFRESH_DELAY_SEC=1.0
      - SERVER_URL=http://TU_IP_SERVIDOR:9080
      - LRCGEN_URL=http://lrcgen-whisper:8000
      - LRCGEN_QUEUE_DB=/reports/lrcgen_queue.sqlite3
      - LRCGEN_UPLOADS_DIR=/lrcgen-uploads
    volumes:
      - /ruta/a/tu/musica:/music:rw
      - /ruta/a/tus/reportes:/reports:rw
      - /ruta/a/tus/cargas_temporales:/lrcgen-uploads:rw
    ports:
      - "9080:8080"

  lrcgen:
    image: oct4vyus/lrcgen-container:1.0.0
    container_name: lrcgen-whisper
    restart: unless-stopped
    networks:
      - lrc_network
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Argentina/Cordoba
      - WHISPER_MODEL=medium
      - DEFAULT_LANGUAGE=es
      - UVR_ENABLED=true
    volumes:
      - /ruta/a/tu/musica:/music:rw
      - /ruta/a/tus/reportes:/reports:rw
      - /ruta/a/tu/cache/whisper:/cache/whisper
      - /ruta/a/tu/cache/uvr5:/cache/audio-separator
      - /ruta/a/tus/cargas_temporales:/app/uploads:rw

networks:
  lrc_network:
    external: true
    name: lrc_network
```

1. Crea la red externa una sola vez: `docker network create lrc_network`.
2. Guarda el ejemplo como `docker-compose.yml` y reemplaza las rutas de ejemplo por rutas reales.
3. Cambia `TU_IP_SERVIDOR` por la IP o el nombre del servidor accesible desde tu navegador.
4. Inicia los servicios: `docker compose up -d`.
5. Abre `http://TU_IP_SERVIDOR:9080`.

### Variables de entorno

| Variable | Descripción / Description | Ejemplo / Example |
| :--- | :--- | :--- |
| `SERVER_URL` | URL de acceso a la interfaz desde el navegador. / Browser-accessible URL for the interface. | `http://TU_IP_SERVIDOR:9080` |
| `LRCGEN_URL` | Dirección privada de Docker para LRCGen. / Private Docker URL for LRCGen. | `http://lrcgen-whisper:8000` |
| `LRCGEN_QUEUE_DB` | Ruta persistente de la cola SQLite. / Persistent path to the SQLite queue. | `/reports/lrcgen_queue.sqlite3` |
| `LRCGEN_UPLOADS_DIR` | Directorio compartido para archivos temporales. / Shared directory for temporary files. | `/lrcgen-uploads` |
| `REPORT_REFRESH_DELAY_SEC` | Espera para agrupar actualizaciones de informes. / Delay for batching report updates. | `1.0` segundos / seconds |
| `REVIEW_THRESHOLD_SEC` | Umbral de revisión en segundos. / Review threshold in seconds. | `40.0` |
| `WHISPER_MODEL` | Modelo de Whisper: `tiny`, `base`, `small`, `medium` o `large`. / Whisper model size. | `medium` |
| `DEFAULT_LANGUAGE` | Idioma predeterminado para la transcripción. / Default transcription language. | `es` |

### Guía rápida de uso

1. Abre la interfaz en el navegador y revisa el informe de la biblioteca.
2. Las pistas marcadas como `missing` no tienen letra.
3. Pulsa **GENERAR LRC** en el álbum y selecciona el idioma en el cuadro de diálogo.
4. Al terminar, el archivo `.lrc` se guardará junto al audio.

<a id="troubleshooting"></a>
### Solución de problemas / Troubleshooting

- **Idioma incorrecto o autodetección imprecisa / Wrong language or inaccurate autodetection:** selecciona explícitamente el idioma del álbum antes de generar / select the album language before generation.
- **LRCGen no está disponible / LRCGen is unavailable:** comprueba que ambos contenedores estén en `lrc_network` y que `LRCGEN_URL` apunte a `http://lrcgen-whisper:8000` / check that both containers are on `lrc_network` and `LRCGEN_URL` points to `http://lrcgen-whisper:8000`.
- **No se pueden guardar archivos `.lrc` / Cannot save `.lrc` files:** verifica los permisos de lectura y escritura para `PUID` y `PGID` / check read/write permissions for `PUID` and `PGID`.

<a id="english"></a>
## English

**LRC-CheckEdGen** helps music lovers and home media server administrators (**Jellyfin, Plex, and Navidrome**) audit their music libraries, find tracks without synchronized lyrics, and generate or edit `.lrc` files with local AI.

Docker Hub images: [`oct4vyus/lrc-checkedgen:latest`](https://hub.docker.com/r/oct4vyus/lrc-checkedgen) and [`oct4vyus/lrcgen-container:1.0.0`](https://hub.docker.com/r/oct4vyus/lrcgen-container).

### Key features

- Audits albums and reports which tracks have synchronized lyrics and which are missing them.
- Generates lyrics with OpenAI Whisper and UVR5 vocal isolation.
- Lets you select an album's language before generation to reduce autodetection errors.
- Includes a web editor for reviewing lyrics and adjusting timestamps.
- Shows track-by-track generation progress.
- Preserves existing `.lrc` files instead of overwriting them.
- Supports a full AI workflow with both containers, or a lightweight audit/editing workflow using LRC-Checker v6.2.2 without the AI backend.

### v1.2.0 and v1.3.0 highlights

- The language selector offers Spanish, English, Portuguese, autodetection, and a custom ISO-639-1 code. It remembers the last selection and displays the active language during progress.
- Tracks without vocal segments can be marked `no_lyrics` rather than treated as failures; completion summaries distinguish saved lyrics, errors, language skips, and instrumental tracks.
- The editor virtualizes rows for smoother handling of long lyrics. The `/archivo` endpoint streams audio in chunks, and report regeneration can be batched with `REPORT_REFRESH_DELAY_SEC`.

### Docker Compose and quick start

Use the single Compose example in the [Spanish deployment section](#despliegue-con-docker-compose); it configures both the frontend and AI backend and avoids maintaining duplicate configuration blocks. Replace the sample host paths and server URL with your own. For an English-first setup, you may also change `TZ` to your timezone and `DEFAULT_LANGUAGE` to `en`.

1. Create the external network once: `docker network create lrc_network`.
2. Save and customize the Compose example as `docker-compose.yml`.
3. Start the services with `docker compose up -d`.
4. Open `http://YOUR_SERVER_IP:9080`, review tracks marked `missing`, and click **GENERAR LRC** on an album.
5. Choose the album language in the dialog. The generated `.lrc` file is saved next to its audio file.

The [bilingual environment variable reference](#variables-de-entorno) and [troubleshooting guide](#troubleshooting) above apply to both languages. Ensure the Docker network and read/write volume permissions are correct if the backend is unavailable or lyrics cannot be saved.

---

**Créditos / Credits:** LRC-CheckEdGen integrates the LRC-Checker auditing project with the LRCGen AI engine, based on OpenAI Whisper and UVR5.
