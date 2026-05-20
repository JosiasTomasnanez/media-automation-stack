# Análisis: Contenedor Principal SOLO

**Fecha:** 2025-05-14
**Servicio:** Jellyfin
**Versión imagen:** jellyfin:10.11.8

## Qué FUNCIONA

- [x] La interfaz web se abre en localhost:8096
  - URL exacta: http://localhost:8096
  - ¿Qué vieron? Wizard de configuración inicial, luego formulario de login
  - Tiempo de carga: inmediato

- [x] El servicio inicia sin errores en logs
  - Tardó ~20 segundos en "ready" (`Startup complete 0:00:20.4875881`)
  - Hay warnings (ver sección de problemas)

- [x] Puedo crear usuario/loguear
  - Aceptó credenciales correctamente
  - Mostró la interfaz principal después del login

- [x] Puedo agregar contenido manualmente
  - Se creó un video dummy con ffmpeg: `test.mp4` (pantalla azul con pitido)
  - Se colocó en `./media/` → dentro del contenedor: `/media`
  - El archivo apareció en Jellyfin después de escanear manualmente
  - Permisos: `-rwxrwxrwx` (lectura permitida para todos)

## Qué NO funciona / Problemas encontrados

### Problema 1: Carpeta /media inaccesible o vacía

**¿Qué intentaron?**
- Se agregó `/media` como biblioteca en Jellyfin
- Se colocó un archivo `Video 1.mp4` en `./media/` del host
- Se esperó a que Jellyfin escaneara automáticamente

**¿Qué pasó?**
- Jellyfin reportó la carpeta como inaccesible o vacía
- El contenido no apareció automáticamente en la biblioteca
- Fue necesario disparar un escaneo manual desde la interfaz

**Evidencia (logs):**
```
[19:27:45] [WRN] [52] MediaBrowser.Controller.Entities.BaseItem: Library folder /media is inaccessible or empty, skipping
[19:27:46] [WRN] [52] MediaBrowser.Controller.Entities.BaseItem: Library folder /media is inaccessible or empty, skipping
[19:27:46] [WRN] [54] MediaBrowser.Controller.Entities.BaseItem: Library folder /media is inaccessible or empty, skipping
```

**Pregunta de análisis:**
- ¿Dónde debería guardarse el contenido? En el host, en `./media/`, mapeado al contenedor como `/media`
- ¿Está en el contenedor o en el host? En el host (volumen montado), pero el contenedor no lo detecta automáticamente
- ¿Desapareció después de docker-compose down? No, porque es un volumen del host.

**Mi hipótesis de por qué pasa:**
- Jellyfin no tiene un watcher en tiempo real sobre `/media` — escanea en intervalos o cuando se lo pedís manualmente
- no hay nadie que avise a Jellyfin cuando llega contenido nuevo

**Lo que necesitaría para funcionar bien:**
- Radarr: avisa a Jellyfin via API cuando termina de mover una película a `/media`
  
### Problema 2: Sin descarga automática de contenido

**¿Qué intentaron?**
- Buscar una película directamente en Jellyfin
- Esperar que aparezca contenido sin agregarlo manualmente

**¿Qué pasó?**
- Jellyfin solo muestra lo que ya existe en `/media`
- No tiene forma de buscar, pedir ni descargar contenido nuevo
- El usuario tiene que conseguir el archivo por su cuenta y copiarlo manualmente

**Evidencia (logs):**
- No hay error específico, simplemente no existe la funcionalidad

**Pregunta de análisis:**
- ¿Dónde debería gestionarse la descarga? En servicios externos (Radarr + Prowlarr + qBittorrent)
- ¿Está en el contenedor? No, Jellyfin no tiene esta capacidad por diseño

**Mi hipótesis de por qué pasa:**
- Jellyfin es un servidor de streaming, no un gestor de descargas
- Por diseño solo sirve contenido existente — la descarga y organización es responsabilidad de otros servicios

**Lo que necesitaría para funcionar:**
- Radarr: automatiza búsqueda y descarga de películas
- Prowlarr: provee fuentes (trackers) donde buscar
- qBittorrent: ejecuta la descarga física del torrent

---

## Conclusión

**¿Qué servicios ADICIONALES necesito?**

1. **Radarr** — sin él, nadie avisa a Jellyfin cuando hay contenido nuevo y no hay descarga automatizada
2. **qBittorrent** — Radarr necesita un cliente torrent para ejecutar las descargas
3. **Prowlarr** — sin fuentes de búsqueda, Radarr no sabe dónde encontrar los torrents
4. **Traefik** — no es estrictamente necesario, pero simplifica el acceso a todos los servicios con dominios locales en vez de puertos

**¿Por qué Jellyfin solo NO es suficiente?**
- No detecta contenido nuevo en tiempo real sin un servicio que le avise (Radarr)
- No puede buscar ni descargar contenido por sí solo

**¿Qué aprendimos?**
1. Un contenedor aislado expone sus limitaciones rápidamente — Jellyfin necesita que otros servicios "alimenten" su carpeta `/media`
2. Los volúmenes Docker persisten datos entre reinicios, pero la estructura interna de carpetas debe existir previamente
3. En arquitecturas de microservicios cada servicio hace una sola cosa bien — Jellyfin sirve, Radarr gestiona, qBittorrent descarga

**Comandos útiles que usamos:**
```bash
# Ver logs en tiempo real
docker compose logs jellyfin

# Filtrar solo errores
docker compose logs jellyfin | grep -i error

# Filtrar warnings
docker compose logs jellyfin | grep -i wrn

# Ver contenedores corriendo
docker ps

# Ver permisos de archivos
ls -la ./media/

# Parar servicios específicos sin bajar todo
docker compose stop radarr prowlarr qbittorrent traefik

# Volver a levantar todo
docker compose start radarr prowlarr qbittorrent traefik
```
**Jellyfin - Prueba manual:**

 **¿Dónde espera Jellyfin las películas?**
    
    - Jellyfin es un servidor que SIRVE archivos que ya existen en el filesystem
    - Pero ¿en cuál carpeta los busca?
    - Busquen en la documentación de Jellyfin: ¿cuál es la carpeta por defecto?
    - ¿Cómo se configura dónde buscar?

Al crear la biblioteca dentro de la UI de Jellyfin busca los archivos en la carpeta que nosotros configuremos. por defecto, busca los archivos en la carpeta /media y dentro de la interfaz de usuario al crear una biblioteca podemos seleccionar en que directorio buscar los archivos.

 **Coloquen el archivo en la carpeta esperada:**
    
    - ¿Dónde copiaron? (responda la ruta exacta)
    - ¿Cómo verifican que el archivo está ahí? (comando para listar)
    - ¿Qué permisos tiene? (`ls -la`)

El archivo de prueba fue creado en la carpeta ./media del repositorio del proyecto verificando con 'ls ./media'. 
Los permisos del archivo fueron diferentes en cuestiones del sistema operativo: En windows el archivo tuvo todos los permisos. En linux fue creado por el usuario root, el grupo tenia permisos de lectura y escritura, y en cuanto a usuarios "otros" tenia permisos solo de lectura.

- **Archivo** : video1.mp4
- **Carpeta donde lo colocaron:** './media/video1.mp4'
- **¿Apareció en Jellyfin?** Solo al escanear manualmente
- **¿Qué metadata mostró?** Titulo, descripción y año: *San Francisco, 1985. Two opposites attract at a modern dance company. Together, their courage and resilience are tested as they navigate a world full of risks and promise, against the backdrop of a disease no one seems to know anything about*.

##Docker-compose.yml
*Faltan los servicios nombrados anteriormente (radarr, qbitorrent, prowlarr y traefik). Traefik se puede buscar por dockerhub pero los demas no y se encuentran en un linuxserver. No fueron necesarias las variables de entorno*

#### Problema 1: Service Discovery (¿Cómo un servicio encuentra a otro?)

**¿Cómo se comunican dos servicios en docker-compose?**

*Los contenedores reciben una IP dinámica asignada por el motor de Docker dentro del rango de la red virtual bridge creada por Docker Compose. Para ver la ip hay que ejecutar el comando : docker inspect*

**¿Por nombre del servicio? (¿es posible?)**

*Si, es posible por nombre del servicio, es la forma recomendada para conectar los servicios. Se hace dentro del docker-compose.yml*

**¿Por hostname especial?**

*Tambien es posible pero no es necesario ya que docker compose se encarga de inyectar los alias de la red basandose en la clave del servicio*

**En el docker-compose.yml, cada servicio tiene un nombre:**

- **¿Puede Radarr usar `qbittorrent` como hostname?**
- **Investiguen: "Docker service name DNS resolution"**

*Radarr puede utilizar qbittorrent como hostname para ello deben compartir la network de docker.*
*Docker service name DNS resolution hace referencia a que docker permite a los contenedores a comunicarse entre si utilizando el nombre definido en el archivo de docker-compose.yml o mediante la linea de comandos usando las direcciones ip estaticas.*

**Pregunta de diseño:**
**Si qBittorrent escucha en puerto 8080 DENTRO del contenedor**
**¿Es el puerto DENTRO del contenedor o el EXPUESTO al host?**

*Estamos hablando del puerto dentro del contenedor no del expuesto en el host*



