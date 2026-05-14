# 🔍 Análisis: Contenedor Principal SOLO

![Servicio](https://img.shields.io/badge/Servicio-Jellyfin-blue)
![Versión](https://img.shields.io/badge/Versión-10.11.8-green)
![Fecha](https://img.shields.io/badge/Fecha-2025--05--14-lightgrey)

---

## Tabla de contenidos

- [Información general](#información-general)
- [Qué FUNCIONA](#qué-funciona)
- [Qué NO funciona](#qué-no-funciona--problemas-encontrados)
  - [Problema 1: /media inaccesible](#problema-1-carpeta-media-inaccesible-o-vacía)
  - [Problema 2: /playlists inaccesible](#problema-2-carpeta-configdataplaylists-inaccesible)
  - [Problema 3: Sin descarga automática](#problema-3-sin-descarga-automática-de-contenido)
- [Conclusión](#conclusión)

---

## Información general

| Campo | Valor |
|---|---|
| Fecha | 2025-05-14 |
| Servicio analizado | Jellyfin |
| Versión de imagen | `jellyfin:10.11.8` |
| URL de acceso | http://localhost:8096 |
| Sistema operativo | WSL (Windows) |

---

## Qué FUNCIONA

- [x] **La interfaz web se abre en localhost:8096**
  - URL exacta: `http://localhost:8096`
  - Se vió: wizard de configuración inicial, luego formulario de login
  - Tiempo de carga: inmediato

- [x] **El servicio inicia sin errores críticos en logs**
  - Tardó ~20 segundos en "ready"
  - Línea exacta: `Startup complete 0:00:20.4875881`
  - Hay warnings (ver sección de problemas)

- [x] **Puedo crear usuario y loguear**
  - Aceptó credenciales correctamente
  - Mostró la interfaz principal después del login

- [x] **Puedo agregar contenido manualmente**
  - Se creó un video dummy con ffmpeg (pantalla azul con pitido)
  - Se colocó en `./media/` → dentro del contenedor: `/media`
  - Permisos: `-rwxrwxrwx` (lectura permitida para todos)
  - El archivo apareció en Jellyfin **solo después de escanear manualmente**

---

## Qué NO funciona / Problemas encontrados

### Problema 1: Carpeta `/media` inaccesible o vacía

**¿Qué intentamos?**
- Agregar `/media` como biblioteca en Jellyfin
- Colocar un archivo `WhatsApp Video.mp4` en `./media/` del host
- Esperar a que Jellyfin escaneara automáticamente

**¿Qué pasó?**
- Jellyfin reportó la carpeta como inaccesible o vacía
- El contenido **no apareció automáticamente** en la biblioteca
- Fue necesario disparar un escaneo manual desde la interfaz

**Evidencia — logs exactos:**

```log
[19:27:45] [WRN] [52] MediaBrowser.Controller.Entities.BaseItem: Library folder /media is inaccessible or empty, skipping
[19:27:46] [WRN] [52] MediaBrowser.Controller.Entities.BaseItem: Library folder /media is inaccessible or empty, skipping
[19:27:46] [WRN] [54] MediaBrowser.Controller.Entities.BaseItem: Library folder /media is inaccessible or empty, skipping
```

**Preguntas de análisis:**

| Pregunta | Respuesta |
|---|---|
| ¿Dónde debería guardarse el contenido? | En el host, en `./media/`, mapeado al contenedor como `/media` |
| ¿Está en el contenedor o en el host? | En el host (volumen montado), pero el contenedor no lo detecta automáticamente |
| ¿Desapareció después de `docker compose down`? | No, el volumen persiste. Pero Jellyfin necesita re-escanear cada vez |

**Mi hipótesis de por qué pasa:**
- Jellyfin no tiene un watcher en tiempo real sobre `/media` — escanea en intervalos o cuando se lo pedís manualmente
- Sin Radarr, no hay nadie que avise a Jellyfin cuando llega contenido nuevo
- El escaneo automático ocurre cada cierto tiempo (configurable), no al instante

**Lo que necesitaría para funcionar bien:**
- **Radarr** → avisa a Jellyfin via API cuando termina de mover una película a `/media`
- Configurar el intervalo de escaneo en Jellyfin (`Settings → Scheduled Tasks`)

---

### Problema 2: Carpeta `/config/data/playlists` inaccesible

**¿Qué intentamos?**
- Iniciar Jellyfin y navegar la interfaz normalmente
- Intentar crear una playlist

**¿Qué pasó?**
- Jellyfin reporta que la carpeta de playlists no existe o no tiene permisos
- Aparece el warning desde el primer arranque, antes de cualquier acción del usuario

**Evidencia — logs exactos:**

```log
[19:02:16] [WRN] [26] MediaBrowser.Controller.Entities.BaseItem: Library folder /config/data/playlists is inaccessible or empty, skipping
[19:27:45] [WRN] [52] MediaBrowser.Controller.Entities.BaseItem: Library folder /config/data/playlists is inaccessible or empty, skipping
```

**Preguntas de análisis:**

| Pregunta | Respuesta |
|---|---|
| ¿Dónde debería guardarse? | En `/config/data/playlists` dentro del contenedor |
| ¿Está en el contenedor o en el host? | Debería estar en el volumen `./config/jellyfin`, pero la subcarpeta no se creó sola |
| ¿Desapareció después de `docker compose down`? | La carpeta persiste si el volumen está montado, pero si nunca existió Jellyfin no la crea |

**Mi hipótesis de por qué pasa:**
- Docker monta el volumen `./config/jellyfin` pero no crea la estructura interna de subcarpetas
- Es un problema de inicialización: Jellyfin espera que `playlists/` ya exista

**Lo que necesitaría para funcionar:**

```bash
# Crear la carpeta antes de levantar el contenedor
mkdir -p ./config/jellyfin/data/playlists
```

---

### Problema 3: Sin descarga automática de contenido

**¿Qué intentamos?**
- Buscar una película directamente desde la interfaz de Jellyfin
- Esperar que aparezca contenido sin agregarlo manualmente

**¿Qué pasó?**
- Jellyfin solo muestra lo que ya existe en `/media`
- No tiene forma de buscar, pedir ni descargar contenido nuevo
- El usuario tiene que conseguir el archivo por su cuenta y copiarlo manualmente

**Evidencia:**
> No hay error en los logs — simplemente la funcionalidad no existe en Jellyfin por diseño.

**Preguntas de análisis:**

| Pregunta | Respuesta |
|---|---|
| ¿Dónde debería gestionarse la descarga? | En servicios externos: Radarr + Prowlarr + qBittorrent |
| ¿Jellyfin puede descargar? | No, por diseño. Solo sirve contenido existente |

**Mi hipótesis de por qué pasa:**
- Jellyfin es un servidor de streaming, no un gestor de descargas
- En arquitectura de microservicios, cada servicio tiene una sola responsabilidad
- La descarga y organización es responsabilidad de otros servicios del stack

**Lo que necesitaría para funcionar:**
- **Radarr** → automatiza búsqueda y descarga de películas
- **Prowlarr** → provee fuentes (trackers) donde buscar los torrents
- **qBittorrent** → ejecuta la descarga física del torrent

---

## Conclusión

### ¿Qué servicios ADICIONALES necesito?

| Prioridad | Servicio | ¿Por qué? |
|---|---|---|
| 1 | **Radarr** | Sin él, nadie avisa a Jellyfin cuando hay contenido nuevo ni hay descarga automatizada |
| 2 | **qBittorrent** | Radarr necesita un cliente torrent para ejecutar las descargas |
| 3 | **Prowlarr** | Sin fuentes de búsqueda, Radarr no sabe dónde encontrar los torrents |
| 4 | **Traefik** | No estrictamente necesario, pero simplifica el acceso con dominios locales en vez de puertos |

### ¿Por qué Jellyfin solo NO es suficiente?

- No detecta contenido nuevo en tiempo real sin un servicio que le avise (Radarr)
- No puede buscar ni descargar contenido por sí solo
- La carpeta de playlists requiere inicialización manual de carpetas
- El usuario tiene que gestionar archivos manualmente, rompiendo la experiencia de "Netflix casero"

### ¿Qué aprendimos?

1. **Un contenedor aislado expone sus limitaciones rápidamente** — Jellyfin necesita que otros servicios "alimenten" su carpeta `/media`
2. **Los volúmenes Docker persisten datos entre reinicios**, pero la estructura interna de carpetas debe existir previamente
3. **En arquitecturas de microservicios cada servicio hace una sola cosa** — Jellyfin sirve, Radarr gestiona, qBittorrent descarga

### Comandos útiles descubiertos

```bash
# Ver logs completos
docker compose logs jellyfin

# Filtrar solo errores
docker compose logs jellyfin | grep -i error

# Filtrar warnings
docker compose logs jellyfin | grep -i wrn

# Ver contenedores corriendo
docker ps

# Ver permisos de archivos
ls -la ./media/

# Parar servicios específicos sin bajar todo el stack
docker compose stop radarr prowlarr qbittorrent traefik

# Volver a levantar todo
docker compose start radarr prowlarr qbittorrent traefik
```
