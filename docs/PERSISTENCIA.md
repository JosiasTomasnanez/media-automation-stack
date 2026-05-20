**Volumenes declarados y porque son necesarios:**
- media : Contenido de jellyfin, en este caso, peliculas. Permite que jellyfin acceda a las peliculas almacenadas en dicho volumen.
- cache : Datos temporales. Optimiza el rendimiento.
- config : Almacena la base de datos y la configuracion. Es necesario ya que mantiene la configuracion de los usuarios y los metadatos, y si recreamos el contenedor no perderiamos los ajustes aplicados.
- downloads : Intercambio de archivos entre los servicios. Es la carpeta comun donde qBittorent deja las descargas que luego Radarr las procesa y mueve a media.

**Dónde en el HOST quedan los datos** 
- Si usas docker volumes queda en una carpeta de docker con la path : /var/lib/docker/volumes/
- Con mount queda en la carpeta que se especifica, en este caso, en el repositorio del proyecto estan los volumenes de los datos.

**Comando para "limpiar todo" (si necesitaran borrar datos)**
```
docker system prune -a --volumes -f
```
- docker system prune : Limpia los contenedores detenidos, redes y la cache
- -a : elimina las imagenes que no se usan
- --volumes : los volumenes de dichos contenedores.
- -f : fuerza que se ejecute el comando.
