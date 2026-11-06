# 🌐 Select Language / Seleccionar Idioma

[🇪🇸 Español](#-español---lrc-checkedgen) | [🇬🇧 English](#-english---lrc-checkedgen)

---

[![Docker Image Version](https://img.shields.io/docker/v/oct4vyus/lrc-checkedgen?sort=semver&label=Docker%20Hub)](https://hub.docker.com/r/oct4vyus/lrc-checkedgen/tags)
[![Docker Pulls](https://img.shields.io/docker/pulls/oct4vyus/lrc-checkedgen?logo=docker)](https://hub.docker.com/r/oct4vyus/lrc-checkedgen)
[![GitHub Release](https://img.shields.io/github/v/release/Oct4vyus/LRC-CheckEdGen)](https://github.com/Oct4vyus/LRC-CheckEdGen/releases/latest)
[![License](https://img.shields.io/github/license/Oct4vyus/LRC-CheckEdGen)](https://github.com/Oct4vyus/LRC-CheckEdGen/blob/main/LICENSE)

# 🇪🇸 Español - LRC-CheckEdGen

🏷️ **Docker Hub Frontend:** `oct4vyus/lrc-checkedgen:latest` (o `1.3.0`)  
🏷️ **Docker Hub Backend:** `oct4vyus/lrcgen-container:latest` (o `1.0.0`)

**LRC-CheckEdGen** es una herramienta pensada para usuarios y administradores de servidores multimedia (**Jellyfin, Plex, Navidrome**). Te permite **auditar tu colección musical, identificar las canciones sin letras y generarlas o editarlas automáticamente** utilizando inteligencia artificial local.

## 💡 Flujo de trabajo modular

1. **Generación intensiva:** ejecuta LRC-CheckEdGen junto con LRCGen (Whisper + UVR5) para generar letras sincronizadas en los álbumes que no tienen archivos `.lrc`.
2. **Auditoría y mantenimiento:** cuando termine el procesamiento por lotes, puedes detener el backend de IA y usar el contenedor liviano LRC-Checker v6.2.2 para seguir auditando y editando manualmente con menos consumo de recursos.

## 📜 Historial de versiones

- **LRC-Checker v6.2.1 / v6.2.2:** auditoría de estados `missing`, `unsynced`, `desynced` y `empty`, edición con reproductor HTML5 e incorporación de la categoría LRC-GEN.
- **LRC-Checker v7.5.1:** integración con LRCGen por red Docker, cola SQLite persistente y tarjeta de progreso interactiva.
- **LRC-CheckEdGen v1.1.0:** unificación de la suite y publicación de imágenes en Docker Hub.
- **LRC-CheckEdGen v1.2.0:** nuevo logo, filtrado de idiomas `es`/`en`, estado `no_lyrics` para pistas sin voz y resumen de finalización por álbum.
- **LRC-CheckEdGen v1.3.0:** selector de idioma, optimizaciones del editor `.lrc`, transmisión de audio por bloques y regeneración agrupada de reportes.

### Novedades de v1.2.0 y v1.3.0

- **Selector de idioma por álbum (v1.3.0):** permite elegir español (`es`), inglés (`en`), portugués (`pt`), autodetección (`auto`) u otro código ISO-639-1 de dos letras. La selección explícita evita la autodetección y se conserva en el navegador.
- **Editor y audio optimizados (v1.3.0):** virtualización de filas, actualizaciones focalizadas y transmisión progresiva del audio desde `/archivo`.
- **Reportes agrupados (v1.3.0):** las actualizaciones HTML/JSON se agrupan mediante `REPORT_REFRESH_DELAY_SEC` (1 segundo por defecto).
- **Detección de pistas instrumentales (v1.2.0):** cuando no se detecta voz, la pista puede quedar como `no_lyrics` sin marcar el trabajo como fallido.
- **Resumen al finalizar:** informa letras guardadas, errores, omisiones por idioma y pistas instrumentales.

---

## ✨ ¿Qué puedes hacer con LRC-CheckEdGen?

- **📊 Auditar tu biblioteca musical:** Escanea tus álbumes y descubre al instante qué canciones ya tienen archivo de letra sincronizada (`.lrc`) y cuáles están pendientes.
- **🤖 Generar letras sincronizadas con IA:** Si tus plugins o proveedores no encontraron la letra de una canción, la IA (Whisper + aislamiento vocal UVR5) la transcribirá y sincronizará línea por línea.
- **🌍 Seleccionar el idioma antes de procesar:** Elige el idioma del álbum (español, inglés, portugués, etc.) antes de iniciar la generación para evitar errores de autodetección y obtener transcripciones más precisas.
- **🎛️ Editor web integrado:** Revisa, ajusta los tiempos y edita manualmente las letras directamente desde tu navegador web.
- **⏱️ Seguimiento en tiempo real:** Observa una tarjeta flotante de estado que muestra qué pista se está procesando y el avance del trabajo.
- **🛡️ Cuidado de tu colección:** Los archivos `.lrc` existentes en tu biblioteca nunca son sobrescritos. Tu música se mantiene intacta.
- **🔄 Uso flexible y a tu medida:**
  - **Modo Completo (IA):** Enciende la suite completa cuando quieras generar letras automáticamente para canciones faltantes.
  - **Modo Liviano (Auditoría / Edición):** Si solo necesitas auditar tu colección o editar letras manualmente sin consumir recursos de IA en tu servidor, puedes usar únicamente el contenedor ligero (LRC-Checker v6.2.2).

---

## 🚀 Despliegue con Docker Compose

Puedes instalar **LRC-CheckEdGen** en tu servidor doméstico o NAS (**OpenMediaVault, Synology, Unraid, TrueNAS, Portainer**) fácilmente.

Crea tu archivo `docker-compose.yml` adaptando las rutas de tus carpetas:

```yaml
version: '3.8'

services:
  # 1. Servidor de Auditoría y Editor Web
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
      - SERVER_URL=http://TU_IP_SERVIDOR:9080   # Reemplaza TU_IP_SERVIDOR por la IP local de tu servidor
      - LRCGEN_URL=http://lrcgen-whisper:8000     # Conexión privada interna entre contenedores
      - LRCGEN_QUEUE_DB=/reports/lrcgen_queue.sqlite3
      - LRCGEN_UPLOADS_DIR=/lrcgen-uploads
    volumes:
      # Adapta las rutas del lado izquierdo a tus carpetas reales en el servidor/NAS:
      - /ruta/a/tu/musica:/music:rw
      - /ruta/a/tus/reportes:/reports:rw
      - /ruta/a/tus/cargas_temporales:/lrcgen-uploads:rw
    ports:
      - "9080:8080"

  # 2. Motor de Inteligencia Artificial (Whisper + UVR)
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
      - WHISPER_MODEL=medium                      # Modelos disponibles: tiny, base, small, medium, large
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

> **Pasos previos:**
> Crea la red compartida de Docker una sola vez en la consola de tu servidor:
> ```bash
> docker network create lrc_network
> ```

### Variables de entorno principales

| Variable | Descripción | Ejemplo |
| :--- | :--- | :--- |
| `SERVER_URL` | URL de acceso a la interfaz desde el navegador. | `http://TU_IP_SERVIDOR:9080` |
| `LRCGEN_URL` | URL privada de Docker para el backend LRCGen. | `http://lrcgen-whisper:8000` |
| `LRCGEN_QUEUE_DB` | Ruta persistente de la cola SQLite. | `/reports/lrcgen_queue.sqlite3` |
| `LRCGEN_UPLOADS_DIR` | Directorio compartido para archivos temporales. | `/lrcgen-uploads` |
| `REPORT_REFRESH_DELAY_SEC` | Tiempo para agrupar actualizaciones de reportes. | `1.0` |
| `WHISPER_MODEL` | Modelo Whisper: `tiny`, `base`, `small`, `medium` o `large`. | `medium` |

---

## 💡 Guía Rápida de Uso

1. **Ingresa desde tu navegador:** Abre `http://TU_IP_SERVIDOR:9080`.
2. **Revisa el reporte:** Verás tus discos y canciones. Las marcadas como `missing` son las que no tienen letra.
3. **Genera con un clic:** Presiona el botón azul **GENERAR LRC** en el álbum deseado.
4. **Elige el idioma:** Selecciona el idioma del álbum en la ventana emergente para que la IA trabaje con mayor exactitud.
5. **Disfruta del resultado:** Al finalizar el proceso, la letra `.lrc` se guardará automáticamente junto al archivo de audio en tu servidor multimedia.

## 🔍 Solución de problemas

- **Idioma incorrecto:** selecciona explícitamente el idioma del álbum antes de iniciar la generación.
- **LRCGen no disponible:** confirma que ambos contenedores estén conectados a `lrc_network` y que `LRCGEN_URL` apunte a `http://lrcgen-whisper:8000`.
- **Error de permisos al guardar `.lrc`:** verifica que los volúmenes tengan permisos de lectura y escritura y que `PUID`/`PGID` correspondan al usuario adecuado.

## 📄 Créditos y licencia

- **Base de auditoría:** proyecto LRC-Checker.
- **Motor de IA:** OpenAI Whisper y UVR5 mediante LRCGen.
- **Suite de integración:** LRC-CheckEdGen.
- Consulta el archivo [LICENSE](LICENSE) para los términos de uso.

---

<br><br>

---

# 🇬🇧 English - LRC-CheckEdGen

🏷️ **Docker Hub Frontend:** `oct4vyus/lrc-checkedgen:latest` (or `1.3.0`)  
🏷️ **Docker Hub Backend:** `oct4vyus/lrcgen-container:latest` (or `1.0.0`)

**LRC-CheckEdGen** is a user-friendly tool built for music lovers and home media server managers (**Jellyfin, Plex, Navidrome**). It allows you to **audit your music library, find missing lyrics, and automatically generate or edit synchronized `.lrc` files** using local AI.

## 💡 Modular workflow

1. **Intensive generation:** run LRC-CheckEdGen with LRCGen (Whisper + UVR5) to generate synchronized lyrics for albums without `.lrc` files.
2. **Auditing and maintenance:** after batch processing, stop the AI backend and use the lightweight LRC-Checker v6.2.2 container for ongoing audits and manual editing with lower resource usage.

## 📜 Version history

- **LRC-Checker v6.2.1 / v6.2.2:** audits `missing`, `unsynced`, `desynced`, and `empty` states; HTML5-player editing; and the LRC-GEN category.
- **LRC-Checker v7.5.1:** Docker-network integration with LRCGen, a persistent SQLite queue, and an interactive progress card.
- **LRC-CheckEdGen v1.1.0:** unified suite and Docker Hub image releases.
- **LRC-CheckEdGen v1.2.0:** refreshed logo, `es`/`en` language filtering, `no_lyrics` state for tracks without vocals, and album completion summaries.
- **LRC-CheckEdGen v1.3.0:** language selector, `.lrc` editor optimizations, chunked audio streaming, and batched report regeneration.

### v1.2.0 and v1.3.0 highlights

- **Album language selector (v1.3.0):** choose Spanish (`es`), English (`en`), Portuguese (`pt`), autodetect (`auto`), or another two-letter ISO-639-1 code. Explicit choices bypass autodetection and are remembered in the browser.
- **Editor and audio optimizations (v1.3.0):** virtualized rows, targeted updates, and progressive audio streaming from `/archivo`.
- **Batched reports (v1.3.0):** HTML/JSON report refreshes are grouped using `REPORT_REFRESH_DELAY_SEC` (1 second by default).
- **Instrumental detection (v1.2.0):** tracks without detected vocals can be marked `no_lyrics` without treating the job as failed.
- **Completion summary:** reports saved lyrics, errors, language skips, and instrumental tracks.

---

## ✨ Key Features

- **📊 Audit Your Music Collection:** Instantly scan your albums to see which tracks have synchronized lyrics (`.lrc`) and which ones need them.
- **🤖 AI Lyric Generation:** When standard metadata fetchers miss lyrics, local AI (Whisper + UVR5 vocal isolation) transcribes and syncs them line by line.
- **🌍 Pre-Generation Language Selector:** Choose the album's language (English, Spanish, Portuguese, etc.) before starting generation to eliminate auto-detection errors and improve accuracy.
- **🎛️ Built-in Web Editor:** Review, fine-tune timing, and manually edit lyrics directly in your browser.
- **⏱️ Live Progress Tracker:** Monitor track-by-track progress in real time with an interactive status card.
- **🛡️ Safe & Non-Destructive:** Existing `.lrc` files are never overwritten. Your original audio collection remains untouched.
- **🔄 Adaptable & Modular:**
  - **Full AI Mode:** Run both containers when you want to batch-generate lyrics for your missing tracks.
  - **Lightweight Audit Mode:** Run only the audit container if you just want to scan your collection or edit lyrics manually with minimal RAM/CPU usage (LRC-Checker v6.2.2).

---

## 🚀 Quick Start with Docker Compose

Easily deploy **LRC-CheckEdGen** on your home server or NAS (**OpenMediaVault, Synology, Unraid, TrueNAS, Portainer**).

Create your `docker-compose.yml` file and adjust folder paths for your environment:

```yaml
version: '3.8'

services:
  # 1. Audit Server & Web Editor Frontend
  lrc-checkedgen:
    image: oct4vyus/lrc-checkedgen:latest
    container_name: lrc-checkedgen
    restart: unless-stopped
    networks:
      - lrc_network
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/New_York
      - MUSIC_DIR=/music
      - OUTPUT_DIR=/reports
      - CHECK_ORPHANS=true
      - VERBOSE=true
      - REVIEW_THRESHOLD_SEC=40.0
      - REPORT_REFRESH_DELAY_SEC=1.0
      - SERVER_URL=http://YOUR_SERVER_IP:9080   # Replace YOUR_SERVER_IP with your local server IP
      - LRCGEN_URL=http://lrcgen-whisper:8000     # Private container-to-container connection
      - LRCGEN_QUEUE_DB=/reports/lrcgen_queue.sqlite3
      - LRCGEN_UPLOADS_DIR=/lrcgen-uploads
    volumes:
      # Map the left side of paths to your actual server/NAS folders:
      - /path/to/your/music:/music:rw
      - /path/to/your/reports:/reports:rw
      - /path/to/your/temp_uploads:/lrcgen-uploads:rw
    ports:
      - "9080:8080"

  # 2. Artificial Intelligence Engine (Whisper + UVR)
  lrcgen:
    image: oct4vyus/lrcgen-container:1.0.0
    container_name: lrcgen-whisper
    restart: unless-stopped
    networks:
      - lrc_network
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/New_York
      - WHISPER_MODEL=medium                      # Available models: tiny, base, small, medium, large
      - DEFAULT_LANGUAGE=en
      - UVR_ENABLED=true
    volumes:
      - /path/to/your/music:/music:rw
      - /path/to/your/reports:/reports:rw
      - /path/to/your/cache/whisper:/cache/whisper
      - /path/to/your/cache/uvr5:/cache/audio-separator
      - /path/to/your/temp_uploads:/app/uploads:rw

networks:
  lrc_network:
    external: true
    name: lrc_network
```

> **Initial Step:**
> Create the shared Docker network once in your server terminal:
> ```bash
> docker network create lrc_network
> ```

### Key environment variables

| Variable | Description | Example |
| :--- | :--- | :--- |
| `SERVER_URL` | Browser-accessible URL for the web interface. | `http://YOUR_SERVER_IP:9080` |
| `LRCGEN_URL` | Private Docker URL for the LRCGen backend. | `http://lrcgen-whisper:8000` |
| `LRCGEN_QUEUE_DB` | Persistent path for the SQLite queue. | `/reports/lrcgen_queue.sqlite3` |
| `LRCGEN_UPLOADS_DIR` | Shared directory for temporary uploads. | `/lrcgen-uploads` |
| `REPORT_REFRESH_DELAY_SEC` | Delay used to batch report updates. | `1.0` |
| `WHISPER_MODEL` | Whisper model: `tiny`, `base`, `small`, `medium`, or `large`. | `medium` |

---

## 💡 Simple Usage Guide

1. **Access via browser:** Open `http://YOUR_SERVER_IP:9080`.
2. **Review your report:** Check your albums. Tracks marked as `missing` need lyrics.
3. **One-click generation:** Click the blue **GENERAR LRC** button on any album.
4. **Choose language:** Select the album language in the popup window for best accuracy.
5. **Enjoy synced lyrics:** Once completed, your new `.lrc` file will be automatically saved right next to your music files for Jellyfin or Plex to read.

## 🔍 Troubleshooting

- **Wrong language:** explicitly select the album language before starting generation.
- **LRCGen unavailable:** confirm both containers are connected to `lrc_network` and `LRCGEN_URL` points to `http://lrcgen-whisper:8000`.
- **Permission errors saving `.lrc` files:** check that mounted volumes are read/write and that `PUID`/`PGID` match the intended user.

## 📄 Credits and license

- **Auditing base:** LRC-Checker project.
- **AI engine:** OpenAI Whisper and UVR5 via LRCGen.
- **Integration suite:** LRC-CheckEdGen.
- See [LICENSE](LICENSE) for license terms.
