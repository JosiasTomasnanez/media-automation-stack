# TRABAJO FINAL

**Materia:** Aspectos de Sistemas Operativos, Bases de Datos y Microservicios  
**Formato:** Grupal  


---

### Para Opción A — Media Server

Antes de escribir una sola línea de código, **lee esto:**

#### ¿Qué es un Media Server?
Un media server es una aplicación que:
1. **Centraliza** tu contenido multimedia (películas, series, documentales)
2. **Organiza** automáticamente según metadatos (título, año, actor, género)
3. **Sirve** contenido a cualquier dispositivo (PC, smartphone, TV)
4. **Transcodes** en tiempo real (ajusta calidad según conexión/dispositivo)

**Ejemplo real:** Netflix/Plex funcionan así, pero en tu servidor.

#### ¿Qué es Jellyfin?
- **Servidor de streaming** open-source
- Reemplaza a Plex (pero gratis y sin cuentas)
- Interfaz web + apps nativas (Windows, Android, iOS, Smart TV)
- Organiza películas, series, música, fotos
- Transcode automático
- **Requisitos:** 2GB RAM mínimo, 10GB disco

**Problema:** Jellyfin solo SIRVE contenido. No lo BUSCA ni lo DESCARGA automáticamente.

#### ¿Qué es Radarr?
- **Automatizador de descargas** de películas
- Monitorea listas de películas que quieres ver
- Automáticamente busca torrents de buena calidad
- Descarga via qBittorrent
- Una vez descargado, lo mueve a la carpeta que Jellyfin vigila
- Actualiza metadatos

**Ejemplo:** 
```
Agregas "Dune" a tu lista en Radarr
    ↓
Radarr busca "Dune 2024 720p" en trackers
    ↓
Encuentra torrent de calidad
    ↓
Descarga via qBittorrent
    ↓
Mueve a /movies/Dune
    ↓
Jellyfin lo detecta automáticamente
    ↓
Aparece en tu biblioteca
```

#### ¿Qué es qBittorrent?
- **Cliente torrent** con interfaz web
- Descarga archivos desde torrents
- Controla velocidad, ratios, seeders
- Importante: Radarr se comunica CON qBittorrent para descargar

#### ¿Qué es Prowlarr?
- **Agregador de trackers/índices**
- Radarr NO busca directamente en trackers
- Radarr pregunta a Prowlarr: "¿Dónde busco Dune?"
- Prowlarr responde con lista de trackers donde existe
- Radarr busca en esos trackers

**Flujo completo:**
```
┌─────────────────────────────────────────────────────────┐
│                     USUARIO                             │
│        "Quiero ver Dune en mi TV"                       │
└──────────────┬──────────────────────────────────────────┘
               │
       ┌───────▼─────────┐
       │  Jellyfin Web   │  ← Interfaz amigable
       │  (busca en BD)  │
       └─────────────────┘
               ▲
               │
       ┌───────┴──────────────────────────┐
       │                                  │
    Archivos detectados             Base de datos
    en /movies                       (metadatos)
       ▲                                  ▲
       │                                  │
       │   ┌────────────────────────────┤
       │   │                            │
   ┌───┴───▼──┐  ┌────────┐  ┌─────────▼────┐ 
   │ qBittor  │  │Prowlarr│  │   Radarr     │  
   │  rent    │  │        │  │              │  
   └──────────┘  └────────┘  └──────────────┘ 
       ▲              ▲            ▲ │
       └──────────────┴────────────┘ │
                  Radarr coordina todo
```

---

### Para Opción B — Photo Server 

#### ¿Qué es un Photo Server?
Un servidor personal de fotos que:
1. **Sincroniza** fotos desde móvil automáticamente
2. **Organiza** con IA (detecta caras, objetos, lugares)
3. **Busca** inteligentemente ("mostrame fotos con perro")
4. **Respalda** sin ir a Google Photos/iCloud

#### ¿Qué es Immich?
- **Servidor de fotos** open-source (alternativa a Google Photos)
- Backend API + interfaz web + app mobile
- Sincroniza fotos/videos desde tu teléfono
- Comparte álbumes con familia
- **El problema:** Sin IA no puede buscar inteligentemente

#### ¿Qué es Machine Learning en Immich?
- **Servicio separado** que procesa fotos con IA
- Detecta caras (facial recognition)
- Etiqueta objetos ("perro", "coche", "playa")
- Detecta ubicación aproximada
- Genera thumbnails
- **Requisitos:** 4GB+ RAM, GPU opcional pero recomendada

#### ¿Qué es PostgreSQL?
- **Base de datos relacional**
- Immich Server almacena aquí:
  - Usuarios y credenciales
  - Metadatos de fotos (fecha, ubicación, etc.)
  - Información de caras detectadas
  - Álbumes y comparticiones

#### ¿Qué es Redis?
- **Cache + cola de mensajes**
- Funciones en Immich:
  - **Cache:** Acelera búsquedas frecuentes
  - **Cola:** Cuando subes 100 fotos, Redis las encola para procesamiento IA
  - **Sessions:** Mantiene logins de usuarios

**Flujo:**
```
Subes foto desde móvil
    ↓
Immich Server la recibe y guarda en BD (PostgreSQL)
    ↓
Immich Server la encola en Redis
    ↓
ML Service toma foto de la cola
    ↓
IA procesa: detecta caras, objetos
    ↓
Resultados van a PostgreSQL
    ↓
Búsqueda funciona: "mostrame fotos con Juan"
```

---

## ANÁLISIS DEL SERVICIO PRINCIPAL

### Objetivo
**Ejecutar el servicio principal SOLO en Docker (Immich-server/Jellyfin) y descubrir por qué no es suficiente.**

Este es un ejercicio de **descubrimiento guiado**. No te decimos qué va a fallar, lo descubrirás.

### Antes de empezar: ¿Qué deben observar?

**Cuando levanten el servicio principal:**

```bash
docker-compose up -d
```

**Hagan esto inmediatamente:**

1. **¿El contenedor inició?**
   - `docker ps` — ¿ven el servicio?
   - `docker-compose logs` — ¿hay errores o warnings?
   - ¿El servicio dice "ready" o sigue en startup?

2. **¿La interfaz web funciona?**
   - Abran `localhost:puerto` en navegador
   - ¿Qué ven? (interfaz normal, error, página en blanco?)
   - Si error, ¿cuál es exactamente? (copien el mensaje)

3. **¿Se puede crear usuario/loguear?**
   - ¿Pide credenciales?
   - ¿Puedo crear un usuario nuevo?
   - ¿Se guarda la credencial? (reinician, ¿sigue?)

4. **¿Pueden subir contenido?**
   - Para Jellyfin: ¿Hay opción para agregar carpeta de media?
   - Para Immich: ¿Hay opción para subir foto?
   - ¿Funciona sin errores?

5. **Ahora lo importante: ¿Qué NO funciona?**
   - Intenten hacer algo que esperarían que funcione
   - Documenten el error exacto
   - No adivinen, **PRUEBEN**

---

#### Analiza y DOCUMENTA qué funciona y qué NO

Crea `docs/ANALISIS_CONTENEDOR_PRINCIPAL.md` con estructura similar a esta:

```markdown
# Análisis: Contenedor Principal SOLO

**Fecha:** [YYYY-MM-DD]
**Servicio:** [Jellyfin / Immich]
**Versión imagen:** [ej: jellyfin:10.8]

## Qué FUNCIONA

- [x/  ] La interfaz web se abre en localhost:PUERTO
  - URL exacta: [ej: http://localhost:8096]
  - ¿Qué vieron? (formulario login, interfaz, error, etc)

- [ ] El servicio inicia sin errores en logs
  - ¿Cuánto tardó en "ready"? [segundos]
  - ¿Hay warnings? (aunque inicie, ¿hay mensajes amarillos?)

- [ ] Puedo crear usuario/loguear
  - ¿Aceptó credenciales?
  - ¿Mostró interfaz después de login?

- [ ] Puedo ver/agregar contenido
  - ¿Qué intentaron? (ej: subir foto, crear carpeta movies)
  - ¿Funcionó?

## Qué NO funciona / Problemas encontrados

### Problema 1: Almacenamiento/Base de Datos

**¿Qué intentaron?**
- (describir exactamente qué hicieron)

**¿Qué pasó?**
- (describir el resultado, error exacto, comportamiento inesperado)

**Evidencia (copien de logs):**
\`\`\`
[Error log exacto, no parafraseen]
\`\`\`

**Pregunta de análisis:**
- ¿Dónde debería guardarse este dato?
- ¿Está en el contenedor o en el host?
- ¿Desapareció después de docker-compose down?

**Mi hipótesis de por qué pasa:**
- (¿Por qué creen que sucede?)
- (¿Qué componente falta?)

**Lo que necesitaría para funcionar:**
- (¿Qué agregarían al docker-compose.yml?)
- (¿Qué servicio/volumen/variable de entorno?)

---

### Problema 2: [Describe otro que encontraste]

Repitan la estructura arriba para cada problema.

---

### Problema 3: [Describe otro que encontraste]

...

## Conclusión

**¿Qué servicios ADICIONALES crees que necesitas?**
- Listar por orden de prioridad
- Una frase de por qué cada uno

**¿Por qué el servicio principal SOLO no es suficiente?**
- (síntesis de los problemas encontrados)

**¿Qué aprendieron de este análisis?**
- (1-3 insights principales)

**Comandos útiles que descubrieron:**
- ¿Cómo leyeron los logs?
- ¿Cómo verificaron que el contenedor estaba corriendo?
- ¿Algún comando de Docker que fue clave?
```

---

### Cómo validar el análisis

Antes de continuar, el análisis debe:
- [ ] Tener al menos 2 problemas identificados
- [ ] Cada problema tiene: qué hizo, qué pasó, log exacto
- [ ] Cada problema tiene una hipótesis de "por qué"


Si falta algo, **vuelvan e investiguen más** antes de pasar a la siguiente sección.

#### Intenta agregar contenido (sin Radarr / sin app mobile aún)

**Jellyfin - Prueba manual:**

1. **¿Dónde espera Jellyfin las películas?**
   - Jellyfin es un servidor que SIRVE archivos que ya existen en el filesystem
   - Pero ¿en cuál carpeta los busca?
   - Busquen en la documentación de Jellyfin: ¿cuál es la carpeta por defecto?
   - ¿Cómo se configura dónde buscar?

2. **Crear contenido de prueba:**
   ```bash
   # Opción A: Crear video dummy pequeño (si tienen ffmpeg)
   ffmpeg -f lavfi -i color=c=blue:s=320x240:d=5 -f lavfi -i sine=f=440:d=5 -pix_fmt yuv420p test.mp4
   
   # Opción B: Descargar algo pequeño de internet
   # Opción C: Usar cualquier .mp4 que tengan
   ```

3. **Coloquen el archivo en la carpeta esperada:**
   - ¿Dónde copiaron? (responda la ruta exacta)
   - ¿Cómo verifican que el archivo está ahí? (comando para listar)
   - ¿Qué permisos tiene? (`ls -la`)

4. **Abran Jellyfin y observen:**
   - ¿Apareció inmediatamente?
   - ¿Tardó segundos? ¿Minutos?
   - ¿Mostró metadata (carátula, sinopsis, año)?
   - Si NO apareció, ¿qué revisar?
     - ¿El archivo está en la carpeta correcta?
     - ¿Los permisos permiten que el contenedor lo lea?
     - ¿Jellyfin necesita una acción manual para "buscar"? (hay un botón en la interfaz?)

5. **Investiguen en los logs:**
   ```bash
   docker-compose logs jellyfin | grep -i error
   # ¿Hay errores de permisos?
   # ¿Hay errores de lectura de archivo?
   ```

6. **Documenten:**
   - Archivo que pusieron: `[nombre]`
   - Carpeta donde lo colocaron: `[ruta completa]`
   - ¿Apareció en Jellyfin? Sí/No
   - Si sí, ¿cuánto tardó?
   - ¿Qué metadata mostró?
   - Si no, ¿cuál fue el problema?

---

**Immich - Prueba manual:**

1. **¿Cómo suben fotos SIN la app mobile?**
   - Immich tiene una API REST
   - Todos los clientes (app, web, etc) usan esa API
   - Busquen: ¿cuál es el endpoint para subir fotos en Immich?
   - ¿Qué parámetros necesita?
   - ¿Qué autenticación necesita?

2. **Preparen una foto de prueba:**
   ```bash
   # Opción A: Hacer screenshot de su pantalla
   # Opción B: Descargar una foto pequeña de internet
   # Opción C: Sacar foto con cámara del celular
   ```

3. **Subanla vía API:**
   - **Investiguen:** ¿Cómo se sube un archivo via curl/REST?
   - **Pista:** Busquen "multipart form data"
   - Primero necesitan un token de autenticación
     - ¿Dónde obtienen el token en Immich?
     - ¿Cómo lo pasan en la solicitud?
   
4. **Ejecuten el upload:**
   - Escriban el comando exacto (curl, Python, lo que sea)
   - ¿Funcionó? (¿status 200?)
   - ¿Aparece en la interfaz web?
   - ¿En qué carpeta/álbum la vieron?

5. **Ahora investiguen: ¿Dónde está el archivo?**
   - Dentro del contenedor, ¿en qué carpeta guarda Immich las fotos?
   - Si hacen `docker exec immich ls -la`, ¿la ven?
   - ¿Es el mismo archivo que subieron o una copia/procesada?

6. **Documenten:**
   - Endpoint usado: `[POST /...]`
   - Token: `[obtenido de dónde?]`
   - Comando curl/script exacto:
     ```
     [incluyan el comando sin datos sensibles]
     ```
   - ¿Funcionó? Sí/No
   - Si no: ¿cuál fue el error exacto?

---

**Pregunta puente para ambos:**

Ahora tienen archivos guardados en los contenedores. Respondan:
- ¿Qué pasa si hacen `docker-compose down`?
- ¿Siguen ahí los archivos después?
- ¿Dónde está "ahí"? (en el contenedor? en el host?)

---

## Agregar Servicios Faltantes + Persistencia

**Objetivo:** Basándote en el análisis anterior, agregar los servicios necesarios Y asegurar que los datos NO se pierdan cuando reinician.

### Pre-requisito: Entiendan el Problema de Persistencia

Después de la sección anterior, respondieron: "¿Qué pasa si hacen `docker-compose down`?"

**La respuesta es crítica:**
- Si los datos DESAPARECEN → no hay persistencia
- Si los datos PERMANECEN → están guardados en el host (fuera del contenedor)

**Investiguen ahora:**
1. ¿Qué es un `volumen` en Docker?
   - ¿Diferencia entre volumen y bind mount?
   - ¿Dónde en el filesystem del HOST quedan los datos?
   - ¿Cuál es mejor para bases de datos? ¿Y para media/fotos?



3. **Pregunta de diseño:**
   - Si Jellyfin y Radarr necesitan acceder a la misma carpeta `/movies`, ¿pueden compartir el MISMO volumen?
   - ¿Qué pasa si PostgreSQL y otro servicio comparten volumen?
   - ¿Es buena idea?

---

### Ahora completen el docker-compose.yml

En el `docker-compose.yml` ya tienen el servicio principal. Necesitan AGREGAR:

**Para Media Server (Jellyfin):**
- ¿Qué servicios faltan? (del análisis anterior)
- Cada servicio necesita:
  - Imagen correcta (¿dónde la buscan? Docker Hub?)
  - Variables de entorno específicas (¿qué necesita cada uno?)
  - Puertos expuestos (¿cuáles son?)
  - Volúmenes (¿qué datos persisten?)

**Para Immich:**
- ¿Necesita PostgreSQL? Sí/No - ¿por qué?
- ¿Necesita Redis? Sí/No - ¿por qué?
- ¿Necesita ML Service? Sí/No - ¿en qué paso?
- Cada uno necesita:
  - Imagen correcta
  - Variables de entorno (contraseña de BD, etc.)
  - Volúmenes para persistencia
  - Puertos para acceso

---

### Checklist de Validación de Persistencia

Antes de decir "terminé", PRUEBEN:

1. **Suban contenido** (película/foto como en la sección anterior)
2. **Hagan:** `docker-compose down`
3. **Hagan:** `docker-compose up -d`
4. **Verifiquen:** ¿El contenido sigue ahí?
5. Si NO: ¿Qué volumen falta? ¿Dónde está mal configurado?

**Documenten en `docs/PERSISTENCIA.md`:**
- Qué volúmenes declararon
- Por qué cada uno es necesario
- Dónde en el HOST quedan los datos
- Comando para "limpiar todo" (si necesitaran borrar datos)

---

### Comunicación Inter-Servicios

**Objetivo:** Configurar que los servicios se comuniquen correctamente entre sí.

En Docker, cuando los servicios están en la MISMA red (`docker-compose` crea una por defecto), pueden comunicarse. Pero necesitan saber dónde están.

---

#### Problema 1: Service Discovery (¿Cómo un servicio encuentra a otro?)

**Escenario Media Server:**
- Radarr necesita conectarse a qBittorrent
- En tu máquina local, ¿cómo lo harías? → `localhost:8080`
- Pero en Docker, **cada contenedor tiene su propio "localhost"**

**Investiguen:**
1. ¿Cómo se comunican dos servicios en docker-compose?
   - ¿Por IP del contenedor? (¿dónde la ven?)
   - ¿Por nombre del servicio? (¿es posible?)
   - ¿Por hostname especial?

2. En el docker-compose.yml, cada servicio tiene un nombre:
   ```yaml
   services:
     qbittorrent:
       image: ...
     radarr:
       image: ...
   ```
   - ¿Puede Radarr usar `qbittorrent` como hostname?
   - Investiguen: "Docker service name DNS resolution"

3. **Pregunta de diseño:**
   - Si qBittorrent escucha en puerto 8080 DENTRO del contenedor
   - ¿Es el puerto DENTRO del contenedor o el EXPUESTO al host?

---

#### Problema 2: Variables de Entorno Cross-Service

**Escenario Immich:**
- Immich Server necesita conectarse a PostgreSQL
- PostgreSQL está en otro contenedor
- ¿Cómo le dices a Immich dónde está PostgreSQL?

**Investiguen:**
1. Busquen la documentación de Immich:
   - ¿Qué variable de entorno configura la conexión a BD?
   - ¿Cómo se ve? (ejemplo: `DATABASE_URL=...`)

2. En esa variable, ¿qué van a poner?
   - ¿La IP del contenedor PostgreSQL?
   - ¿El nombre del servicio?
   - ¿El hostname?
   - Pista: Si usan nombre del servicio, Docker lo resuelve automáticamente

3. **Pregunta de seguridad:**
   - La contraseña de PostgreSQL va en una variable de entorno
   - ¿Es seguro verla en `docker-compose.yml`?
   - ¿Cómo la protegerían? (investiguen `.env` files)

---

#### Problema 3: Validar Que Funciona

**Para Media Server:**
- Radarr debe conectarse a qBittorrent
- ¿Cómo verifican que la conexión es exitosa?
  - En la interfaz web de Radarr, ¿hay status de conexión?
  - En los logs de Radarr, ¿hay mensajes de éxito/error?
  - Investiguen: `docker-compose logs radarr` → busquen keywords

**Para Immich:**
- Immich Server debe conectarse a PostgreSQL
- Si NO está conectado:
  - La aplicación no inicia
  - Los logs dicen "connection refused"
  - Investiguen: `docker-compose logs immich-server`

---

#### Pregunta Final (para entender por qué es importante)

En el próximo trabajo, deberán:
- Agregar más servicios
- Algunos deberán hablar con muchos otros
- Si no entienden networking ahora, será un caos

**Entonces:** Cuando hayan terminado esta sección, **intenten hacer que un servicio se conecte a DOS servicios diferentes.** Ej:
- Radarr → qBittorrent + Prowlarr
- Immich Server → PostgreSQL + Redis

---

## Proxy Inverso (Traefik)

**Problema que resuelve:**

Ahora tienen:
- Jellyfin en `localhost:8096`
- Radarr en `localhost:7878`
- Prowlarr en `localhost:9696`
- etc.

Si tienen 10 servicios, ¿memorizan 10 puertos? Incómodo.

**Solución:** Un proxy inverso que enruta por NOMBRE en lugar de puerto.

```
Usuario escribe: https://jellyfin.media.local
    ↓
Traefik (escucha en puerto 443)
    ↓
Traefik lee el hostname "jellyfin.media.local"
    ↓
Traefik busca: "¿A quién debo enrutar esto?"
    ↓
Encuentra Jellyfin en puerto 8096
    ↓
Traefik forwarde el request
```

---

### Conceptos Base - Investiguen

#### 1. ¿Qué es un Proxy Inverso?

**Búsqueda guiada:**
- Diferencia entre proxy normal y proxy inverso
- ¿Por qué se llama "inverso"?
- ¿Dónde está Traefik en la arquitectura?
  - ¿En el host del usuario?
  - ¿En el servidor?
  - ¿Ambos lados?

---

#### 2. ¿Cómo Sabe Traefik Dónde Enrutar?

**Problema:** Traefik necesita saber:
- "jellyfin.media.local" → Jellyfin (puerto 8096)
- "radarr.media.local" → Radarr (puerto 7878)

**¿Cómo se lo dices?**

**Opción A - Archivo de configuración:**
```
# Traefik lee un archivo que dice:
# Si hostname = jellyfin.media.local, envía a http://jellyfin:8096
```

**Opción B - Auto-descubrimiento (labels):**
```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin
    labels:
      - "traefik.http.routers.jellyfin.rule=Host(`jellyfin.media.local`)"
      - "traefik.http.services.jellyfin.loadbalancer.server.port=8096"
```

**Investiguen:**
- ¿Qué es un "label" en Docker?
- ¿Cómo lee Traefik los labels de servicios?
- ¿Cuál opción es mejor para múltiples servicios?

---

#### 3. El Problema de Dominios Locales

**Ahora vienen complicaciones:**

Si escriben `https://jellyfin.media.local` en el navegador:
- El navegador busca dónde está `media.local`
- Va al servidor DNS (Google, ISP, etc)
- Pero `media.local` NO existe en internet global

**¿Cómo lo resuelven?**

**Opción A - `/etc/hosts`:**
```bash
# En /etc/hosts (en el HOST, no en Docker)
127.0.0.1 media.local
127.0.0.1 jellyfin.media.local
127.0.0.1 radarr.media.local
```
- El SO consulta `/etc/hosts` ANTES de ir a DNS
- Así localhost sabe que "jellyfin.media.local" es 127.0.0.1

**Opción B - DNS local:**
- Instalar un servidor DNS local (dnsmasq)
- Configurar que `*.media.local` apunte a 127.0.0.1
- Más complejo, pero automático

**Investiguen:**
- ¿Cómo editar `/etc/hosts` en su SO?
- Permiso de escribir en `/etc/hosts`
- Sintaxis exacta

---

#### 4. El Problema de HTTPS

**Traefik puede hacer HTTPS.** Pero HTTPS necesita certificados.

**El problema:**
- Certificados de Let's Encrypt son para dominios públicos
- `media.local` NO es público
- El navegador va a protestar ("certificate not valid")

**Solución: mkcert**
- Herramienta que crea certificados FALSOS pero válidos localmente
- El navegador confía porque los instalaron localmente
- Solo funciona en tu máquina

**Investiguen:**
1. ¿Qué es mkcert?
2. ¿Cómo genera certificados?
3. ¿Dónde guarda los certificados? (Traefik necesita encontrarlos)
4. ¿Cómo "registra" en tu SO que confíe en esos certificados?

---

### Implementación - Paso a Paso (BONUS si avanzan)

**Punto de partida:** Ya tienen docker-compose corriendo con servicios.

#### Paso 1: Preparar el ambiente

**Investiguen y hagan:**
1. ¿Está instalado mkcert? Si no, instalenlo
2. Generen certificados para dominios locales:
   ```bash
   # Investigar exactamente cómo
   mkcert ...
   ```
3. ¿Dónde quedaron los certificados? (ruta exacta)
4. Agreguen entrada a `/etc/hosts`:
   ```
   127.0.0.1 media.local jellyfin.media.local radarr.media.local prowlarr.media.local
   ```

#### Paso 2: Configurar Traefik en docker-compose

**Preguntas guiadas:**
1. ¿Qué imagen de Traefik usan? (busquen en Docker Hub)
2. ¿Traefik necesita exponer puertos? ¿Cuáles?
3. ¿Traefik necesita acceso a la lista de servicios?
   - Pista: Necesita leer Docker socket (`/var/run/docker.sock`)
   - ¿Cómo montan eso en el contenedor?

#### Paso 3: Agregar labels a los servicios

**Cada servicio necesita saber:**
- Su dominio Traefik (ej: `jellyfin.media.local`)
- Su puerto interno (ej: 8096)

**Investigen:**
- Sintaxis exacta de labels para Traefik
- Documentación de Traefik Docker provider
- Cuáles labels son obligatorios

#### Paso 4: Configurar HTTPS en Traefik

**Traefik necesita saber:**
- Dónde están los certificados (creados con mkcert)
- Qué certificado usar para cada dominio

**Investiguen:**
- Archivos de configuración de Traefik
- Sección de TLS/HTTPS
- Cómo apuntar a certificados locales (NO Let's Encrypt)

#### Paso 5: Validación

**Prueben:**
1. Escriban en navegador: `https://jellyfin.media.local`
   - ¿Funciona?
   - ¿El certificado es válido? (sin warnings)
   - ¿Llega a Jellyfin?

2. Escriban: `https://radarr.media.local`
   - ¿Funciona igual?

3. Accedan al dashboard de Traefik (¿a qué puerto?)
   - ¿Ven listados todos los servicios?
   - ¿El estado muestra verde (UP)?

4. Escriban una ruta incorrecta: `https://noexiste.media.local`
   - ¿Traefik da error 404?
   - ¿Eso es correcto?

---
### Preguntas Avanzadas (si quieren profundizar)

1. Traefik lee servicios en tiempo real
   - Si levantan UN nuevo servicio, ¿Traefik lo detecta automáticamente?
   - ¿O necesitan reiniciar Traefik?

2. Balanceo de carga
   - Si tienen 2 instancias de Jellyfin, ¿Traefik las balancea?
   - ¿Cómo configura eso?

---

## Validación Final - Todo Junto

Cuando crean que terminaron, **hagan esta prueba de fuego:**

### Para Media Server

```
1. Tengo docker-compose con: Jellyfin + Radarr + Prowlarr + qBittorrent
2. Levanto: docker-compose up -d
3. Todos los servicios están corriendo y sin errores (docker ps)
4. Abro interfaz de Radarr, agrego una película a "mi lista"
5. ¿Radarr la busca? (ven en logs que sale a buscar)
6. ¿Qué pasa cuando se descarga?
   - ¿Se mueve automáticamente a carpeta de Jellyfin?
   - ¿Jellyfin la detecta?
   - ¿Aparece en la web de Jellyfin?
7. Hago docker-compose down
8. Hago docker-compose up -d
9. ¿La película SIGUE en Jellyfin?
10. ¿Radarr SIGUE teniendo registro de que la monitoreó?
```

**Si TODO funciona:** ✅ Completaron el TP

---

### Para Immich

```
1. Tengo docker-compose con: Immich Server + PostgreSQL + Redis + ML Service
2. Levanto: docker-compose up -d
3. Todos los servicios están corriendo (docker ps)
4. Subo una foto vía API
5. ¿Aparece en la interfaz web?
6. ¿El ML Service procesa la foto? (revisen logs)
   - ¿Detecta objetos? ¿Caras?
   - ¿Generó thumbnail?
7. Hago docker-compose down
8. Hago docker-compose up -d
9. ¿La foto SIGUE ahí?
10. ¿Puedo buscar "fotos con [objeto detectado]"?
```

**Si TODO funciona:** ✅ Completaron el TP

---

## REFERENCIAS — Documentación Complementaria

### Para todos
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Networking Guide](https://docs.docker.com/network/)

### Opción A — Media Server
- [Jellyfin Docs](https://docs.jellyfin.org/)
- [Radarr Docs](https://wiki.servarr.com/radarr)
- [Prowlarr Docs](https://wiki.servarr.com/prowlarr)

### Opción B — Photo Server
- [Immich Docs](https://immich.app/docs/overview/introduction)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Redis Docs](https://redis.io/docs/)

### Traefik 
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [mkcert - Create local certificates](https://github.com/FiloSottile/mkcert)

---