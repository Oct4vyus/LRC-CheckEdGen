# 🌐 Select Language / Seleccionar Idioma

[🇪🇸 Español](#-español---lrc-checkedgen) | [🇬🇧 English](#-english---lrc-checkedgen)

---

# 🇪🇸 Español - LRC-CheckEdGen

🏷�?**Docker Hub Frontend:** `oct4vyus/lrc-checkedgen:latest` (o `1.3.0`)  
🏷�?**Docker Hub Backend:** `oct4vyus/lrcgen-container:latest` (o `1.0.0`)

**LRC-CheckEdGen** es una herramienta pensada para usuarios y administradores de servidores multimedia (**Jellyfin, Plex, Navidrome**). Te permite **auditar tu colección musical, identificar las canciones sin letras y generarlas o editarlas automáticamente** utilizando inteligencia artificial local.
https://img.shields.io/docker/v/oct4vyus/lrc-checkedgen?sort=semver&label=Imagen](https://hub.docker.com/r/oct4vyus/lrc-checkedgen/tags) https://img.shields.io/docker/pulls/oct4vyus/lrc-checkedgen?logo=docker&label=Descargas](https://hub.docker.com/r/oct4vyus/lrc-checkedgen) https://img.shields.io/docker/image-size/oct4vyus/lrc-checkedgen/latest?logo=docker&label=Tama?o](https://hub.docker.com/r/oct4vyus/lrc-checkedgen) https://img.shields.io/github/v/release/Oct4vyus/LRC-CheckEdGen?display_name=tag&label=Lanzamiento](https://github.com/Oct4vyus/LRC-CheckEdGen/releases/latest) https://img.shields.io/github/license/Oct4vyus/LRC-CheckEdGen?label=Licencia](https://github.com/Oct4vyus/LRC-CheckEdGen/blob/main/LICENSE)
---

## �?¿Qué puedes hacer con LRC-CheckEdGen?

- **📊 Auditar tu biblioteca musical:** Escanea tus álbumes y descubre al instante qué canciones ya tienen archivo de letra sincronizada (`.lrc`) y cuáles están pendientes.
- **🤖 Generar letras sincronizadas con IA:** Si tus plugins o proveedores no encontraron la letra de una canción, la IA (Whisper + aislamiento vocal UVR5) la transcribirá y sincronizará línea por línea.
- **🌍 Seleccionar el idioma antes de procesar:** Elige el idioma del álbum (español, inglés, portugués, etc.) antes de iniciar la generación para evitar errores de autodetección y obtener transcripciones más precisas.
- **🎛�?Editor web integrado:** Revisa, ajusta los tiempos y edita manualmente las letras directamente desde tu navegador web.
- **⏱️ Seguimiento en tiempo real:** Observa una tarjeta flotante de estado que muestra qué pista se está procesando y el avance del trabajo.
- **🛡�?Cuidado de tu colección:** Los archivos `.lrc` existentes en tu biblioteca nunca son sobrescritos. Tu música se mantiene intacta.
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

---

## 💡 Guía Rápida de Uso

1. **Ingresa desde tu navegador:** Abre `http://TU_IP_SERVIDOR:9080`.
2. **Revisa el reporte:** Verás tus discos y canciones. Las marcadas como `missing` son las que no tienen letra.
3. **Genera con un clic:** Presiona el botón azul **GENERAR LRC** en el álbum deseado.
4. **Elige el idioma:** Selecciona el idioma del álbum en la ventana emergente para que la IA trabaje con mayor exactitud.
5. **Disfruta del resultado:** Al finalizar el proceso, la letra `.lrc` se guardará automáticamente junto al archivo de audio en tu servidor multimedia.

---

<br><br>

---

# 🇬🇧 English - LRC-CheckEdGen

🏷�?**Docker Hub Frontend:** `oct4vyus/lrc-checkedgen:latest` (or `1.3.0`)  
🏷�?**Docker Hub Backend:** `oct4vyus/lrcgen-container:latest` (or `1.0.0`)

**LRC-CheckEdGen** is a user-friendly tool built for music lovers and home media server managers (**Jellyfin, Plex, Navidrome**). It allows you to **audit your music library, find missing lyrics, and automatically generate or edit synchronized `.lrc` files** using local AI.

---

## �?Key Features

- **📊 Audit Your Music Collection:** Instantly scan your albums to see which tracks have synchronized lyrics (`.lrc`) and which ones need them.
- **🤖 AI Lyric Generation:** When standard metadata fetchers miss lyrics, local AI (Whisper + UVR5 vocal isolation) transcribes and syncs them line by line.
- **🌍 Pre-Generation Language Selector:** Choose the album's language (English, Spanish, Portuguese, etc.) before starting generation to eliminate auto-detection errors and improve accuracy.
- **🎛�?Built-in Web Editor:** Review, fine-tune timing, and manually edit lyrics directly in your browser.
- **⏱️ Live Progress Tracker:** Monitor track-by-track progress in real time with an interactive status card.
- **🛡�?Safe & Non-Destructive:** Existing `.lrc` files are never overwritten. Your original audio collection remains untouched.
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

---

## 💡 Simple Usage Guide

1. **Access via browser:** Open `http://YOUR_SERVER_IP:9080`.
2. **Review your report:** Check your albums. Tracks marked as `missing` need lyrics.
3. **One-click generation:** Click the blue **GENERAR LRC** button on any album.
4. **Choose language:** Select the album language in the popup window for best accuracy.
5. **Enjoy synced lyrics:** Once completed, your new `.lrc` file will be automatically saved right next to your music files for Jellyfin or Plex to read.
