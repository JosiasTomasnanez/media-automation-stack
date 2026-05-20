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
- no hay nadie que avise a Jellyfin cuando llega contenido nuevo (puede ser Radarr)

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
Intenta agregar contenido (sin Radarr / sin app mobile aún)

Jellyfin - Prueba manual:
¿Dónde espera Jellyfin las películas?
Jellyfin es un servidor que SIRVE archivos que ya existen en el filesystem
Pero ¿en cuál carpeta los busca?
Busquen en la documentación de Jellyfin: ¿cuál es la carpeta por defecto?
¿Cómo se configura dónde buscar? -- al crear la biblioteca dentro de la UI
Jellyfin busca los archivos en la carpeta que nosotros configuremos, por defecto, busca los archivos en la carpeta /media y dentro de la interfaz de usuario al crear una biblioteca podemos seleccionar en que directorio buscar los archivos.
Crear contenido de prueba:
```bash
   # Opción A: Crear video dummy pequeño (si tienen ffmpeg)
   ffmpeg -f lavfi -i color=c=blue:s=320x240:d=5 -f lavfi -i sine=f=440:d=5 -pix\_fmt yuv420p test.mp4
   
   # Opción B: Descargar algo pequeño de internet
   # Opción C: Usar cualquier .mp4 que tengan
   ```
Coloquen el archivo en la carpeta esperada:
¿Dónde copiaron? (responda la ruta exacta)
¿Cómo verifican que el archivo está ahí? (comando para listar)
¿Qué permisos tiene? (`ls -la`)
El archivo de prueba fue creado en la carpeta ./media del repositorio del proyecto verificando con 'ls ./media'. Los permisos del archivo fueron diferentes en cuestiones del sistema operativo. En windows el archivo tuvo todos los permisos, en cambio, en linux fue creado por el usuario root, el grupo tenia permisos de lectura y escritura, y en cuanto a usuarios "otros" tenia permisos solo de lectura.
Abran Jellyfin y observen:
¿Apareció inmediatamente?
¿Tardó segundos? ¿Minutos?
¿Mostró metadata (carátula, sinopsis, año)?
Si NO apareció, ¿qué revisar?
¿El archivo está en la carpeta correcta?
¿Los permisos permiten que el contenedor lo lea?
¿Jellyfin necesita una acción manual para "buscar"? (hay un botón en la interfaz?)
En cuanto a windows el archivo aparecio inmediatamente y en linux tardo un par de minutos. En ambos sistemas operativos mostro sus metadatos correctamente.
Investiguen en los logs:
```bash
   docker-compose logs jellyfin | grep -i error
   # ¿Hay errores de permisos? -- no
   # ¿Hay errores de lectura de archivo? -- no
   ```
No hubo errores de permisos ni errores de lecturas en el archivo
Documenten:
Archivo que pusieron: `\[nombre]`
Carpeta donde lo colocaron: `\[ruta completa]`
¿Apareció en Jellyfin? Sí/No
Si sí, ¿cuánto tardó?
¿Qué metadata mostró?
Si no, ¿cuál fue el problema?
• Nombre del archivo : "test.mp4" almacenado en la carpeta './media/test.mp4' el cual aparecio en jellyfin sin problemas
• Titulos del archivo : Sometimes it's just a matter of chance
• Descripcion y año : San Francisco, 1985. Two opposites attract at a modern dance company. Together, their courage and resilience are tested as they navigate a world full of risks and promise, against the backdrop of a disease no one seems to know anything about.
