# 🎵 LRC-CheckEdGen (v1.1.0)

**LRC-CheckEdGen** es una solución integral y automatizada para la auditoría de bibliotecas musicales y la generación inteligente de letras sincronizadas (`.lrc`) utilizando modelos de inteligencia artificial (Whisper / UVR).

Esta versión **v1.1.0** representa la evolución y consolidación unificada de lo que anteriormente fue **LRC-Checker v6.2.1**, integrando de forma nativa la gestión de reportes, colas persistentes en SQLite y la comunicación interna con el motor de transcripción **LRCGen**.

---

## 🚀 Novedades y Mejoras en v1.1.0

- **Evolución desde LRC-Checker v6.2.1:** Consolidación de la base de auditoría v6.2.1 con la nueva arquitectura integrada de generación automatizada.
- **Despliegue Directo desde Docker Hub:** Las imágenes de ambos contenedores no se construyen en local. Se descargarán preconstruidas directamente desde Docker Hub.
- **Tarjeta de Seguimiento en Tiempo Real:** Interfaz enriquecida con una tarjeta interactiva (azul) que muestra el progreso del trabajo activo de LRCGen (en cola, procesando pista X/Y, finalizado o con error).
- **Navegación Interactiva al Álbum:** Al hacer clic en la tarjeta de estado, el reporte desplaza automáticamente la vista y enfoca el encabezado del álbum correspondiente.
- **Cola Persistente e Idempotente:** Orquestación secuencial respaldada por SQLite (`lrcgen_queue.sqlite3`), garantizando resistencia a reinicios y prevención de trabajos duplicados.
- **Publicación Inteligente por Idioma:** Publicación automática de archivos `.lrc` para contenidos en español (`es`) e inglés (`en`). Idiomas adicionales se marcan para revisión manual (`LRC-GEN`).
- **Protección Antisobrescritura:** Los archivos `.lrc` existentes en la biblioteca musical nunca son sobrescritos.
- **Limpieza Automática por UUID:** Eliminación segura de cargas temporales e intermediarios de aislamiento de voz (UVR) vinculados al UUID único del trabajo activo.

---

## 🏗️ Arquitectura del Sistema

```text
[ Navegador Web / LAN ]
          │
          ▼ (HTTP :9080)
┌─────────────────────────────────────────────────────────────┐
│ contenedor: lrc-checkedgen (v1.1.0)                         │
│ ─ Orquestador y Servidor Web (Puerto interno 8080)          │
│ ─ Cola SQLite en /reports/lrcgen_queue.sqlite3              │
└──────────────────────────────┬──────────────────────────────┘
                               │ (HTTP interno en lrc_network)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ contenedor: lrcgen-whisper                                  │
│ ─ Motor de Transcripción Whisper + Separación UVR           │
│ ─ Puerto interno 8000                                       │
└──────────────────────────────┴──────────────────────────────┘
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
 [ Bibliotecas de Música ]              [ Directorios /reports y /uploads ]
 (Montaje rw compartido)               (Persistencia y limpieza de temporales)
```

### Principales Componentes

1. **LRC-CheckEdGen (Frontend & Orquestador):**
   - Escanea la biblioteca musical y genera reportes interactivos HTML/JSON.
   - Mantiene la cola de trabajos persistente en SQLite.
   - Consume la API interna de LRCGen sin exponerla al cliente web.
2. **LRCGen (Backend Whisper):**
   - Procesa la extracción de voz (UVR5) y realiza la transcripción fonética/temporal con OpenAI Whisper.

---

## 🛠️ Requisitos Previos

1. **Docker Engine** y **Docker Compose** (o Portainer sobre OpenMediaVault / Linux).
2. **Red Docker Externa:** Debe crearse una red compartida llamada `lrc_network` para permitir la comunicación privada DNS entre contenedores:
   ```bash
   docker network create lrc_network
   ```
3. **Directorios en el Host:**
   - Ruta(s) a la biblioteca de música (con permisos de lectura y escritura `rw`).
   - Ruta para los reportes y base de datos SQLite.
   - Ruta temporal compartida para cargas.

---

## 📦 Despliegue con Docker Compose (Docker Hub)

Ambas imágenes se obtienen directamente de **Docker Hub**. No se requiere Dockerfile ni compilación local.

Crea o actualiza tu archivo `docker-compose.yml` (o configura el Stack en Portainer) con la siguiente estructura:

```yaml
version: '3.8'

services:
  # Contenedor 1: LRC-CheckEdGen (Checker + Orquestador)
  lrc-checkedgen:
    image: tu-usuario/lrc-checkedgen:1.1.0
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
      - SERVER_URL=http://IP-DE-TU-SERVIDOR:9080   # Reemplazar con la IP/Hostname de tu servidor
      - LRCGEN_URL=http://lrcgen-whisper:8000     # URL privada interna entre contenedores
      - LRCGEN_QUEUE_DB=/reports/lrcgen_queue.sqlite3
      - LRCGEN_UPLOADS_DIR=/lrcgen-uploads
    volumes:
      # Montajes de Bibliotecas de Música
      - /ruta/host/musica/jellyfin:/music/jellyfin:rw
      - /ruta/host/musica/pub-local:/music/Pub-Local:rw
      # Directorio persistente de reportes y cola SQLite
      - /ruta/host/reportes:/reports:rw
      # Directorio temporal de cargas (compartido con LRCGen)
      - /ruta/host/uploads:/lrcgen-uploads:rw
    ports:
      - "9080:8080"
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  # Contenedor 2: LRCGen (Whisper + UVR)
  lrcgen:
    image: tu-usuario/lrcgen:latest
    container_name: lrcgen-whisper
    restart: unless-stopped
    networks:
      - lrc_network
    ports:
      - "8000:8000"                               # Opcional si se requiere acceso externo directo
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Argentina/Buenos_Aires
      - WHISPER_MODEL=medium                      # tiny, base, small, medium, large
      - DEFAULT_LANGUAGE=es
      - UVR_ENABLED=true
    volumes:
      # Las mismas bibliotecas musicales
      - /ruta/host/musica/jellyfin:/music/jellyfin:rw
      - /ruta/host/musica/pub-local:/music/Pub-Local:rw
      # Directorio de reportes
      - /ruta/host/reportes:/reports:rw
      # Caché de modelos Whisper y UVR
      - /ruta/host/models/whisper:/cache/whisper
      - /ruta/host/models/uvr5:/cache/audio-separator
      # Directorio compartido de cargas temporales
      - /ruta/host/uploads:/app/uploads:rw
    command: python -m uvicorn app:app --host 0.0.0.0 --port 8000

networks:
  lrc_network:
    external: true
    name: lrc_network
```

---

## ⚙️ Variables de Entorno Clave

| Variable | Descripción | Valor por Defecto / Ejemplo |
| :--- | :--- | :--- |
| `SERVER_URL` | URL pública accesible desde el navegador web para los reportes. | `http://IP-DE-TU-SERVIDOR:9080` |
| `LRCGEN_URL` | URL interna privada de red Docker hacia el contenedor de LRCGen. | `http://lrcgen-whisper:8000` |
| `LRCGEN_QUEUE_DB` | Ruta interna de la base SQLite para la cola secuencial. | `/reports/lrcgen_queue.sqlite3` |
| `LRCGEN_UPLOADS_DIR` | Punto de montaje para la limpieza compartida de temporales por UUID. | `/lrcgen-uploads` |
| `WHISPER_MODEL` | Modelo de OpenAI Whisper a utilizar (`tiny`, `base`, `small`, `medium`, `large`). | `medium` |
| `CHECK_ORPHANS` | Activa la verificación de archivos `.lrc` huérfanos sin audio. | `true` |

---

## 🖥️ Despliegue en Portainer / OMV (Paso a Paso)

1. **Crear la red Docker externa:**
   En la consola SSH de tu servidor OMV:
   ```bash
   docker network create lrc_network
   ```
2. **Descargar y desplegar las imágenes desde Docker Hub:**
   Ejecuta mediante Docker Compose:
   ```bash
   docker compose pull
   docker compose up -d --force-recreate
   ```
   *O desde la interfaz web de Portainer:*
   - Ve a **Stacks -> Add stack** (o edita el Stack existente).
   - Copia el bloque Compose con los nombres de imagen de Docker Hub (`tu-usuario/lrc-checkedgen:1.1.0` y `tu-usuario/lrcgen:latest`).
   - Activa la casilla **Re-pull image and update** y presiona **Deploy the stack**.

3. **Verificación de estado de los contenedores:**
   ```bash
   docker ps --format 'table {{.Names}}	{{.Image}}	{{.Status}}	{{.Ports}}'
   ```
   Ambos contenedores (`lrc-checkedgen` y `lrcgen-whisper`) deben aparecer activos (`Up`).

---

## 📖 Guía de Uso

1. Accede desde tu navegador a la dirección configurada en `SERVER_URL` (ej. `http://IP-DE-TU-SERVIDOR:9080`).
2. El sistema mostrará el reporte interactivo de tu biblioteca de música.
3. En cualquier álbum que presente canciones en estado `missing`, haz clic en **GENERAR LRC** (botón azul).
4. Se mostrará la **tarjeta azul de seguimiento** informando el avance en tiempo real de la cola y la pista actual.
5. Puedes pulsar la tarjeta azul para desplazarte inmediatamente al álbum en proceso.
6. Al finalizar, el estado cambiará a **Terminado** y el archivo `.lrc` se escribirá de forma segura en tu directorio de música.

---

## 🔍 Solución de Problemas Frecuentes

- **El reporte indica que LRCGen no está disponible:**
  - Verifica que `lrcgen-whisper` esté en ejecución (`docker ps`).
  - Confirma que ambos contenedores compartan la red `lrc_network`.
  - Verifica que la variable `LRCGEN_URL` apunte a `http://lrcgen-whisper:8000`.
- **Error de permisos al guardar los `.lrc`:**
  - Asegúrate de que los montajes de tus volúmenes de música tengan permisos de escritura (`:rw`).
- **Incompatibilidad de la cola SQLite:**
  - Al migrar desde versiones anteriores de LRC-Checker, descarga la versión `1.1.0` desde Docker Hub y recrea el contenedor conservando el directorio `/reports`.

---

## 📜 Licencia y Créditos

Desarrollado para la auditoría, sincronización y automatización de bibliotecas musicales.
- **Base de Auditoría:** Proyecto LRC-Checker (v6.2.1).
- **Motor de Transcripción:** OpenAI Whisper & UVR5 (LRCGen).
- **Orquestación e Integración:** Proyecto LRC-CheckEdGen (v1.1.0).
