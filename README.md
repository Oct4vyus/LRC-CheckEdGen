#  LRC-CheckEdGen (v1.1.0)

**LRC-CheckEdGen** es una soluciÃ³n integral y automatizada para la auditorÃ­a de bibliotecas musicales y la generaciÃ³n inteligente de letras sincronizadas (`.lrc`) utilizando modelos de inteligencia artificial (Whisper / UVR).

Esta versi¨®n v1.1.0 consolida la evoluci¨®n de LRC-Checker v6.2.2, integrando la orquestaci¨®n en colas persistentes SQLite y el motor de transcripci¨®n LRCGen.
?? Filosof¨ªa y Flujo de Trabajo Modular

El proyecto permite alternar entre dos modos de trabajo seg¨²n la necesidad del servidor:

    ? Modo Generaci¨®n e Inteligencia Artificial (v1.1.0 ¡ª Suite LRC-CheckEdGen):
        Objetivo: Procesar y generar autom¨¢ticamente las letras de canciones que plugins externos (como Jellyfin) no logran encontrar.
        Estructura: Se despliegan los 2 contenedores de Docker Hub ( y ). Genera los archivos e integra la voz separada mediante UVR5 y Whisper.

    ?? Modo Auditor¨ªa y Mantenimiento Ligero (v6.2.2 ¡ª LRC-Checker Standalone):
        Objetivo: Mantener una herramienta liviana de bajo consumo para cuantificar la biblioteca y realizar ediciones manuales.
        Flujo Revertible: Una vez completada la generaci¨®n de las letras faltantes en la biblioteca, puedes apagar el stack pesado de IA y regresar a la versi¨®n v6.2.2. As¨ª conservas el control y la edici¨®n manual en un ¨²nico contenedor hiper-liviano sin consumo innecesario de RAM/CPU/GPU.

?? Novedades en v1.1.0

    Despliegue Directo desde Docker Hub: Ambas im¨¢genes se descargan preconstruidas ( y ).
    Tarjeta Azul de Avance en Tiempo Real: Visualizaci¨®n en vivo del progreso de generaci¨®n (pista X/Y, en cola, completado o error).
    Navegaci¨®n Interactiva: Al pulsar la tarjeta azul, la interfaz hace scroll directo al ¨¢lbum en proceso.
    Cola Persistente SQLite: Manejo secuencial idempotente ().
    Publicaci¨®n Autom¨¢tica: Publica directamente idiomas en espa?ol () e ingl¨¦s ().
    Protecci¨®n de Archivos: No sobrescribe archivos preexistentes.
    Limpieza de Temporales por UUID: Borrado seguro de cargas compartidas en .



---

## ?? Novedades y Mejoras en v1.1.0

- **Despliegue Directo desde Docker Hub:** Las imÃ¡genes de ambos contenedores no se construyen en local. Se descargarÃ¡n preconstruidas directamente desde Docker Hub.
- **Tarjeta de Seguimiento en Tiempo Real:** Interfaz enriquecida con una tarjeta interactiva (azul) que muestra el progreso del trabajo activo de LRCGen (en cola, procesando pista X/Y, finalizado o con error).
- **NavegaciÃ³n Interactiva al ?lbum:** Al hacer clic en la tarjeta de estado, el reporte desplaza automÃ¡ticamente la vista y enfoca el encabezado del Ã¡lbum correspondiente.
- **Cola Persistente e Idempotente:** OrquestaciÃ³n secuencial respaldada por SQLite (`lrcgen_queue.sqlite3`), garantizando resistencia a reinicios y prevenciÃ³n de trabajos duplicados.
- **PublicaciÃ³n Inteligente por Idioma:** PublicaciÃ³n automÃ¡tica de archivos `.lrc` para contenidos en espaÃ±ol (`es`) e inglÃ©s (`en`). Idiomas adicionales se marcan para revisiÃ³n manual.
- **ProtecciÃ³n Antisobrescritura:** Los archivos `.lrc` existentes en la biblioteca musical nunca son sobrescritos.
- **Limpieza AutomÃ¡tica por UUID:** EliminaciÃ³n segura de cargas temporales e intermediarios de aislamiento de voz (UVR) vinculados al UUID Ãºnico del trabajo activo.

---

## ??ï¸?Arquitectura del Sistema

```text
[ Navegador Web / LAN ]
          ??          ??(HTTP :9080)
?Œâ??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€????contenedor: lrc-checkedgen (v1.1.0)                         ?????€ Orquestador y Servidor Web (Puerto interno 8080)          ?????€ Cola SQLite en /reports/lrcgen_queue.sqlite3              ???”â??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?¬â??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€??                               ??(HTTP interno en lrc_network)
                               ???Œâ??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€????contenedor: lrcgen-whisper                                  ?????€ Motor de TranscripciÃ³n Whisper + SeparaciÃ³n UVR           ?????€ Puerto interno 8000                                       ???”â??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?´â??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€??                               ??            ?Œâ??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?´â??€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€?€??            ??                                    ?? [ Bibliotecas de MÃºsica ]              [ Directorios /reports y /uploads ]
 (Montaje rw compartido)               (Persistencia y limpieza de temporales)
```

### Principales Componentes

1. **LRC-CheckEdGen (Frontend & Orquestador):**
   - Escanea la biblioteca musical y genera reportes interactivos HTML/JSON.
   - Mantiene la cola de trabajos persistente en SQLite.
   - Consume la API interna de LRCGen sin exponerla al cliente web.
2. **LRCGen (Backend Whisper):**
   - Procesa la extracciÃ³n de voz (UVR5) y realiza la transcripciÃ³n fonÃ©tica/temporal con OpenAI Whisper.

---

## ??ï¸?Requisitos Previos

1. **Docker Engine** y **Docker Compose** (o Portainer sobre OpenMediaVault / Linux).
2. **Red Docker Externa:** Debe crearse una red compartida llamada `lrc_network` para permitir la comunicaciÃ³n privada DNS entre contenedores:
   ```bash
   docker network create lrc_network
   ```
3. **Directorios en el Host:**
   - Ruta(s) a la biblioteca de mÃºsica (con permisos de lectura y escritura `rw`).
   - Ruta para los reportes y base de datos SQLite.
   - Ruta temporal compartida para cargas.

---

## ?“¦ Despliegue con Docker Compose (Docker Hub)

Ambas imÃ¡genes se obtienen directamente de **Docker Hub**. No se requiere Dockerfile ni compilaciÃ³n local.

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
      # Montajes de Bibliotecas de MÃºsica
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
      # CachÃ© de modelos Whisper y UVR
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

## ?™ï? Variables de Entorno Clave

| Variable | DescripciÃ³n | Valor por Defecto / Ejemplo |
| :--- | :--- | :--- |
| `SERVER_URL` | URL pÃºblica accesible desde el navegador web para los reportes. | `http://IP-DE-TU-SERVIDOR:9080` |
| `LRCGEN_URL` | URL interna privada de red Docker hacia el contenedor de LRCGen. | `http://lrcgen-whisper:8000` |
| `LRCGEN_QUEUE_DB` | Ruta interna de la base SQLite para la cola secuencial. | `/reports/lrcgen_queue.sqlite3` |
| `LRCGEN_UPLOADS_DIR` | Punto de montaje para la limpieza compartida de temporales por UUID. | `/lrcgen-uploads` |
| `WHISPER_MODEL` | Modelo de OpenAI Whisper a utilizar (`tiny`, `base`, `small`, `medium`, `large`). | `medium` |
| `CHECK_ORPHANS` | Activa la verificaciÃ³n de archivos `.lrc` huÃ©rfanos sin audio. | `true` |

---

## ?–¥ï¸?Despliegue en Portainer / OMV (Paso a Paso)

1. **Crear la red Docker externa:**
   En la consola SSH de tu servidor OMV:
   ```bash
   docker network create lrc_network
   ```
2. **Descargar y desplegar las imÃ¡genes desde Docker Hub:**
   Ejecuta mediante Docker Compose:
   ```bash
   docker compose pull
   docker compose up -d --force-recreate
   ```
   *O desde la interfaz web de Portainer:*
   - Ve a **Stacks -> Add stack** (o edita el Stack existente).
   - Copia el bloque Compose con los nombres de imagen de Docker Hub (`tu-usuario/lrc-checkedgen:1.1.0` y `tu-usuario/lrcgen:latest`).
   - Activa la casilla **Re-pull image and update** y presiona **Deploy the stack**.

3. **VerificaciÃ³n de estado de los contenedores:**
   ```bash
   docker ps --format 'table {{.Names}}	{{.Image}}	{{.Status}}	{{.Ports}}'
   ```
   Ambos contenedores (`lrc-checkedgen` y `lrcgen-whisper`) deben aparecer activos (`Up`).

---

## ?? GuÃ­a de Uso

1. Accede desde tu navegador a la direcciÃ³n configurada en `SERVER_URL` (ej. `http://IP-DE-TU-SERVIDOR:9080`).
2. El sistema mostrarÃ¡ el reporte interactivo de tu biblioteca de mÃºsica.
3. En cualquier Ã¡lbum que presente canciones en estado `missing`, haz clic en **GENERAR LRC** (botÃ³n azul).
4. Se mostrarÃ¡ la **tarjeta azul de seguimiento** informando el avance en tiempo real de la cola y la pista actual.
5. Puedes pulsar la tarjeta azul para desplazarte inmediatamente al Ã¡lbum en proceso.
6. Al finalizar, el estado cambiarÃ¡ a **Terminado** y el archivo `.lrc` se escribirÃ¡ de forma segura en tu directorio de mÃºsica.

---

## ?? SoluciÃ³n de Problemas Frecuentes

- **El reporte indica que LRCGen no estÃ¡ disponible:**
  - Verifica que `lrcgen-whisper` estÃ© en ejecuciÃ³n (`docker ps`).
  - Confirma que ambos contenedores compartan la red `lrc_network`.
  - Verifica que la variable `LRCGEN_URL` apunte a `http://lrcgen-whisper:8000`.
- **Error de permisos al guardar los `.lrc`:**
  - AsegÃºrate de que los montajes de tus volÃºmenes de mÃºsica tengan permisos de escritura (`:rw`).
- **Incompatibilidad 503 en la cola SQLite:**
  - Al migrar desde la versiÃ³n antigua 7.5.0, descarga la versiÃ³n `1.1.0` desde Docker Hub y recrea el contenedor conservando el directorio `/reports`.

---

## ?? Licencia y CrÃ©ditos

Desarrollado para la auditorÃ­a, sincronizaciÃ³n y automatizaciÃ³n de bibliotecas musicales.
- **Motor de TranscripciÃ³n:** OpenAI Whisper & UVR5.
- **OrquestaciÃ³n y AuditorÃ­a:** Proyecto LRC-CheckEdGen.
