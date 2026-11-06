# 🎵 LRC-CheckEdGen

[ 🇪🇸 Versión en Español ](#-español---lrc-checkedgen) | [ 🇬🇧 English Version ](#-english---lrc-checkedgen)

---

# 🇪🇸 Español - LRC-CheckEdGen

**LRC-CheckEdGen** es una solución integral para la auditoría de bibliotecas musicales, gestión de reportes y generación inteligente de letras sincronizadas (.lrc) basada en inteligencia artificial (OpenAI Whisper + UVR5 Vocal Isolation).

---

## 💡 Filosofía y Flujo de Trabajo Modular

LRC-CheckEdGen ha sido concebido para operar bajo un esquema de trabajo de dos fases optimizado para servidores domésticos (NAS / OpenMediaVault):

1. **Fase de Generación Intensiva (LRC-CheckEdGen v1.3.0 + IA):**
   Despliega la suite completa integrada con el contenedor pesado **LRCGen** (Whisper + UVR5). Se utiliza temporalmente para auditar y generar automáticamente letras sincronizadas en todos aquellos álbumes cuyos archivos .lrc no hayan sido encontrados por Jellyfin o gestores de metadatos.
2. **Fase de Auditoría y Mantenimiento Ligero (LRC-Checker v6.2.2):**
   Una vez completada la generación masiva de la biblioteca, se puede detener el backend de IA y volver al contenedor liviano **v6.2.2**. Esto permite mantener una auditoría continua, cuantificación rápida y edición manual con un consumo mínimo de RAM, CPU y VRAM.

---

## 📜 Historial de Evolución

- **LRC-Checker v6.2.1 / v6.2.2:** Base ligera de auditoría de bibliotecas musicales, escaneo de estados (missing, unsynced, desynced, empty), edición manual con reproductor HTML5 e introducción de la categoría LRC-GEN ([by:LRC Generator]).
- **LRC-Checker v7.5.1:** Primera integración mediante red interna Docker con el contenedor LRCGen, agregando la cola persistente SQLite y la tarjeta azul de progreso interactivo.
- **LRC-CheckEdGen v1.1.0:** Unificación oficial de la suite y publicación de imágenes preconstruidas en Docker Hub (oct4vyus/lrc-checkedgen y oct4vyus/lrcgen-container).
- **LRC-CheckEdGen v1.2.0:** Actualización de identidad visual (nuevo logo 96x96 PNG transparente), filtro explícito de idiomas no aceptados (es/en), estado no_lyrics para temas instrumentales/sin voz y resumen detallado al finalizar álbumes.
- **LRC-CheckEdGen v1.3.0 (Versión Actual):** Selector interactivo de idioma previo a la generación, optimizaciones masivas en el editor .lrc (virtualización DOM, content-visibility, streaming por bloques /archivo), regeneración agrupada en segundo plano de reportes (REPORT_REFRESH_DELAY_SEC) y compatibilidad con publicaciones forzadas por idioma elegido.

---

## 🚀 Novedades de v1.2.0 y v1.3.0

### 🌍 Selector de Idioma por Álbum (v1.3.0)
- **Cuadro Modal Previo:** Al presionar **GENERAR LRC**, se abre un selector interactivo con opciones predefinidas: **Español (es)**, **Inglés (en)**, **Portugués (pt)**, **Autodetectar (auto)** u **Otro** (con validación estricta de 2 letras ISO-639-1).
- **Publicación Garantizada:** Al seleccionar un idioma explícito, Whisper omite la autodetección y se desactiva el filtro restrictivo de idioma, publicando directamente el .lrc generado.
- **Memoria de Selección:** Guarda la última preferencia en el navegador y registra el idioma activo en la tarjeta de seguimiento y en la tabla SQLite (jobs.language).

### ⚡ Optimizaciones de Rendimiento (v1.3.0)
- **Virtualización del Editor .lrc:** Renderizado unitario de filas, actualización focalizada de la línea activa, delegación de eventos y uso de content-visibility para fluidificar temas extensos.
- **Streaming de Audio por Bloques:** El endpoint /archivo transmite el audio de forma fraccionada sin cargar la pista completa en RAM.
- **Regeneración de Reportes Agrupada:** Pospone y agrupa la reconstrucción de reportes HTML/JSON (~1.0 s por defecto con REPORT_REFRESH_DELAY_SEC) al realizar ediciones múltiples o firmas manuales.

### 🤖 Gestión Inteligente de Pistas (v1.2.0)
- **Detección de Instrumentales (no_lyrics):** Las pistas donde Whisper no detecta voces o segmentos de tiempo pasan automáticamente al estado no_lyrics ("posible instrumental o sin voz") sin considerarse un error de ejecución.
- **Resumen Completo de Fin de Trabajo:** Informa con exactitud cantidad de letras guardadas, fallos reales, omisiones por idioma y temas instrumentales.

---

## 🏗️ Arquitectura del Sistema

```text
[ Navegador Web / LAN ]
          │
          ▼ (HTTP :9080)
┌─────────────────────────────────────────────────────────────┐
│ Contenedor: lrc-checkedgen (v1.3.0)                         │
│ ─ Orquestador y Servidor Web (Puerto interno 8080)          │
│ ─ Cola SQLite en /reports/lrcgen_queue.sqlite3              │
└──────────────────────────────┬──────────────────────────────┘
                               │ (HTTP interno en lrc_network)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Contenedor: lrcgen-whisper                                  │
│ ─ Backend OpenAI Whisper + Separación Vocal UVR5            │
│ ─ Puerto interno 8000                                       │
└──────────────────────────────┴──────────────────────────────┘
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
 [ Bibliotecas musicales (rw) ]        [ Directorios /reports y /uploads ]
```

---

## 🛠️ Requisitos y Despliegue

### 1. Preparar la Red Docker Externa
```bash
docker network create lrc_network
```

### 2. Archivo docker-compose.yml (Docker Hub)

```yaml
version: '3.8'

services:
  # Contenedor 1: LRC-CheckEdGen (Frontend & Orquestador)
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
      - SERVER_URL=http://IP-DE-TU-SERVIDOR:9080   # Ajustar a la IP/Hostname de tu servidor
      - LRCGEN_URL=http://lrcgen-whisper:8000     # Comunicación privada interna
      - LRCGEN_QUEUE_DB=/reports/lrcgen_queue.sqlite3
      - LRCGEN_UPLOADS_DIR=/lrcgen-uploads
    volumes:
      - /ruta/host/musica/jellyfin:/music/jellyfin:rw
      - /ruta/host/musica/pub-local:/music/Pub-Local:rw
      - /ruta/host/reportes:/reports:rw
      - /ruta/host/uploads:/lrcgen-uploads:rw
    ports:
      - "9080:8080"
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M

  # Contenedor 2: LRCGen (Backend Whisper + UVR5)
  lrcgen:
    image: oct4vyus/lrcgen-container:latest
    container_name: lrcgen-whisper
    restart: unless-stopped
    networks:
      - lrc_network
    ports:
      - "8000:8000"                               # Opcional para acceso directo
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Argentina/Buenos_Aires
      - WHISPER_MODEL=medium                      # tiny, base, small, medium, large
      - DEFAULT_LANGUAGE=es
      - UVR_ENABLED=true
    volumes:
      - /ruta/host/musica/jellyfin:/music/jellyfin:rw
      - /ruta/host/musica/pub-local:/music/Pub-Local:rw
      - /ruta/host/reportes:/reports:rw
      - /ruta/host/models/whisper:/root/.cache/whisper
      - /ruta/host/models/uvr5:/root/.cache/audio-separator
      - /ruta/host/uploads:/app/uploads:rw
    command: python -m uvicorn app:app --host 0.0.0.0 --port 8000

networks:
  lrc_network:
    external: true
    name: lrc_network
```

---

## 🔍 Variables de Entorno Clave

| Variable | Descripción | Valor por Defecto / Ejemplo |
| :--- | :--- | :--- |
| SERVER_URL | URL pública de acceso a la interfaz web desde el navegador. | http://IP-DE-TU-SERVIDOR:9080 |
| LRCGEN_URL | URL de red privada Docker hacia el motor LRCGen. | http://lrcgen-whisper:8000 |
| LRCGEN_QUEUE_DB | Ruta persistente a la base SQLite de la cola. | /reports/lrcgen_queue.sqlite3 |
| LRCGEN_UPLOADS_DIR | Punto de montaje compartido para la limpieza de temporales por UUID. | /lrcgen-uploads |
| REPORT_REFRESH_DELAY_SEC | Tiempo de espera (s) para agrupar regeneraciones del reporte tras cambios. | 1.0 |
| WHISPER_MODEL | Modelo de IA para transcripción (tiny, base, small, medium, large). | medium |

---

## ❓ Solución de Problemas Frecuentes

- **Error de autodetección o letra en idioma incorrecto:**  
  En v1.3.0, selecciona explícitamente el idioma (es, en, pt, etc.) en el modal antes de enviar el álbum a generar.
- **El reporte indica que LRCGen no está disponible:**  
  Verifica que ambos contenedores pertenezcan a la red lrc_network (docker network inspect lrc_network) y que LRCGEN_URL coincida con el container_name de LRCGen (http://lrcgen-whisper:8000).
- **Permisos de escritura al guardar .lrc:**  
  Asegúrate de que los volúmenes montados tengan privilegios de lectura y escritura (:rw) para los identificadores PUID/PGID configurados.

---
---

# 🇬🇧 English - LRC-CheckEdGen

**LRC-CheckEdGen** is an all-in-one solution for music library auditing, report management, and AI-powered synchronized lyrics generation (.lrc) leveraging OpenAI Whisper and UVR5 vocal isolation.

---

## 💡 Modular Workflow Philosophy

LRC-CheckEdGen is designed to operate in a flexible two-stage workflow tailored for home media servers (NAS / OpenMediaVault):

1. **Intensive Generation Phase (LRC-CheckEdGen v1.3.0 + AI):**  
   Deploy the full suite integrated with the heavy **LRCGen** container (Whisper + UVR5). Use it temporarily to audit and automatically generate synchronized lyrics for albums missing .lrc files.
2. **Lightweight Auditing & Maintenance Phase (LRC-Checker v6.2.2):**  
   Once batch lyrics generation is complete, stop the AI backend and revert to the lightweight **v6.2.2** container. This keeps ongoing library checks, quick metrics, and manual editing running with minimal CPU, RAM, and VRAM usage.

---

## 📜 Version Evolution History

- **LRC-Checker v6.2.1 / v6.2.2:** Lightweight auditing engine for music libraries, scanning states (missing, unsynced, desynced, empty), manual editing via HTML5 player, and introduction of the LRC-GEN category ([by:LRC Generator]).
- **LRC-Checker v7.5.1:** First internal Docker network integration with LRCGen, introducing SQLite persistent queuing and an interactive blue progress tracker card.
- **LRC-CheckEdGen v1.1.0:** Unified suite release and pre-built image publishing to Docker Hub (oct4vyus/lrc-checkedgen and oct4vyus/lrcgen-container).
- **LRC-CheckEdGen v1.2.0:** Branding refresh (new 96x96 transparent PNG logo), strict language filtering (es/en), no_lyrics state for instrumental/voiceless tracks, and detailed album completion alerts.
- **LRC-CheckEdGen v1.3.0 (Current Version):** Pre-generation interactive language selection modal, massive .lrc editor performance optimizations (DOM virtualization, content-visibility, chunked /archivo streaming), background batched report regeneration (REPORT_REFRESH_DELAY_SEC), and forced publishing for user-selected languages.

---

## 🚀 Key Features in v1.2.0 & v1.3.0

### 🌍 Album Language Selector (v1.3.0)
- **Pre-Generation Modal:** Clicking **GENERATE LRC** opens an interactive dialog with options: **Spanish (es)**, **English (en)**, **Portuguese (pt)**, **Autodetect (auto)**, or **Other** (validated 2-letter ISO-639-1 code).
- **Guaranteed Publishing:** When an explicit language is chosen, Whisper bypasses autodetection and disables language restrictions, directly saving the generated .lrc.
- **Preference Persistence:** Stores your last selection in the browser and logs the language in the progress card and SQLite database (jobs.language).

### ⚡ Performance Enhancements (v1.3.0)
- **Editor Virtualization:** Row-by-row rendering, active line targeted DOM updates, event delegation, and content-visibility for smooth navigation on long tracks.
- **Audio Chunk Streaming:** The /archivo endpoint streams audio progressively without loading full files into system memory.
- **Batched Report Regeneration:** Delays and batches HTML/JSON report updates (~1.0s default via REPORT_REFRESH_DELAY_SEC) during rapid manual edits or sign-offs.

### 🤖 Smart Track Management (v1.2.0)
- **Instrumental Detection (no_lyrics):** Tracks where Whisper detects no vocal segments are automatically assigned to no_lyrics ("possible instrumental or voiceless track") without marking the job as failed.
- **Comprehensive Completion Summary:** Displays accurate totals for saved lyrics, actual errors, language skips, and instrumental tracks upon completion.

---

## 🏗️ System Architecture

```text
[ Web Browser / LAN ]
          │
          ▼ (HTTP :9080)
┌─────────────────────────────────────────────────────────────┐
│ Container: lrc-checkedgen (v1.3.0)                          │
│ ─ Orchestrator & Web Server (Internal Port 8080)            │
│ ─ SQLite Queue at /reports/lrcgen_queue.sqlite3             │
└──────────────────────────────┬──────────────────────────────┘
                               │ (Internal HTTP over lrc_network)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Container: lrcgen-whisper                                   │
│ ─ OpenAI Whisper Backend + UVR5 Vocal Isolation             │
│ ─ Internal Port 8000                                        │
└──────────────────────────────┴──────────────────────────────┘
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
 [ Music Libraries (rw) ]              [ /reports & /uploads Directories ]
```

---

## 🛠️ Requirements & Deployment

### 1. Create External Docker Network
```bash
docker network create lrc_network
```

### 2. docker-compose.yml Configuration (Docker Hub)

```yaml
version: '3.8'

services:
  # Container 1: LRC-CheckEdGen (Frontend & Orchestrator)
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
      - SERVER_URL=http://YOUR-SERVER-IP:9080        # Set to your server IP/Hostname
      - LRCGEN_URL=http://lrcgen-whisper:8000       # Private internal communication
      - LRCGEN_QUEUE_DB=/reports/lrcgen_queue.sqlite3
      - LRCGEN_UPLOADS_DIR=/lrcgen-uploads
    volumes:
      - /path/host/music/jellyfin:/music/jellyfin:rw
      - /path/host/music/pub-local:/music/Pub-Local:rw
      - /path/host/reports:/reports:rw
      - /path/host/uploads:/lrcgen-uploads:rw
    ports:
      - "9080:8080"
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M

  # Container 2: LRCGen (Whisper + UVR5 Backend)
  lrcgen:
    image: oct4vyus/lrcgen-container:latest
    container_name: lrcgen-whisper
    restart: unless-stopped
    networks:
      - lrc_network
    ports:
      - "8000:8000"                               # Optional for direct access
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Argentina/Buenos_Aires
      - WHISPER_MODEL=medium                      # tiny, base, small, medium, large
      - DEFAULT_LANGUAGE=es
      - UVR_ENABLED=true
    volumes:
      - /path/host/music/jellyfin:/music/jellyfin:rw
      - /path/host/music/pub-local:/music/Pub-Local:rw
      - /path/host/reports:/reports:rw
      - /path/host/models/whisper:/root/.cache/whisper
      - /path/host/models/uvr5:/root/.cache/audio-separator
      - /path/host/uploads:/app/uploads:rw
    command: python -m uvicorn app:app --host 0.0.0.0 --port 8000

networks:
  lrc_network:
    external: true
    name: lrc_network
```

---

## 🔍 Key Environment Variables

| Variable | Description | Default / Example |
| :--- | :--- | :--- |
| SERVER_URL | Public URL accessible from browsers for reports. | http://YOUR-SERVER-IP:9080 |
| LRCGEN_URL | Private Docker network URL to the LRCGen container. | http://lrcgen-whisper:8000 |
| LRCGEN_QUEUE_DB | Path to persistent SQLite queue database. | /reports/lrcgen_queue.sqlite3 |
| LRCGEN_UPLOADS_DIR | Shared mount point for UUID temporary file cleanup. | /lrcgen-uploads |
| REPORT_REFRESH_DELAY_SEC | Delay (s) for batching report updates after edits. | 1.0 |
| WHISPER_MODEL | OpenAI Whisper model size (tiny, base, small, medium, large). | medium |

---

## ❓ Troubleshooting

- **Language autodetection errors or wrong language output:**  
  In v1.3.0, explicitly select the album language (es, en, pt, etc.) in the modal dialog before starting generation.
- **Report indicates LRCGen is unavailable:**  
  Verify both containers are joined to lrc_network (docker network inspect lrc_network) and LRCGEN_URL points to http://lrcgen-whisper:8000.
- **Permission errors writing .lrc files:**  
  Ensure host volume mounts have read/write permissions (:rw) for configured PUID/PGID.

---

## 📜 License & Credits

Developed for music library auditing, synchronization, and automated lyrics generation.
- **Auditing Base:** LRC-Checker Project (v6.2.1 / v6.2.2).
- **AI Engine:** OpenAI Whisper & UVR5 (LRCGen).
- **Integration Suite:** LRC-CheckEdGen (v1.3.0).
