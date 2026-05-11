# BONUS: Secciones Técnicas Avanzadas

**Estas secciones son OPCIONALES.** El trabajo base está 100% completo sin ellas.

Pero si eres curioso/a y quisiste meterte más profundo, acá hay 4 temas que escalalan la complejidad: **monitoreo**, **seguridad**, **backups** 

Cada sección es independiente — puedes hacer una, dos, o todas.

---

## ¿Por Dónde Empezar?

| Si te interesa... | Lee primero |
|-------------------|-------------|
| **Saber si el sistema está sano** | Bonus 1: Monitoreo |
| **Que nadie vea tus passwords** | Bonus 2: Seguridad |
| **No perder datos jamás** | Bonus 3: Backups |
| **Encontrar errores rápido** | Bonus 4: Logging |
| **Todo lo anterior** | En orden 1→2→3→4 |

---

---

## BONUS 1: Monitoreo con Prometheus + Grafana

### ¿Por qué?

Ahora tienes 4+ servicios corriendo. Preguntas que no puedes responder:
- ¿Cuánta CPU está usando Jellyfin?
- ¿Se está quedando sin disco?
- ¿Hay memory leaks en algún servicio?
- ¿Los requests están lentos?

**Prometheus** = recopila métricas
**Grafana** = muestra esas métricas en dashboards bonitos
**Alertas** = te notifican cuando algo está mal

### Conceptos a Investigar

1. **¿Qué es una métrica?**
   - CPU, RAM, disk usage, network I/O
   - ¿Cómo se miden? (porcentaje, bytes, requests/segundo?)

2. **¿Cómo se recopilan en Docker?**
   - cAdvisor, Prometheus scraper
   - ¿Exposición de métricas? (puertos, endpoints)

3. **¿Cómo funcionan las alertas?**
   - Umbrales (cuando CPU > 80%, alertar)
   - Destinations (email, Slack, webhook)

### Tareas Guiadas

#### Tarea 1.1: Entender Prometheus

1. Lee documentación de Prometheus:
   - ¿Cuáles son los componentes principales?
   - ¿Cómo se configura? (archivo YAML)

2. Investiga: ¿cómo Docker expone sus métricas?
   - Búsqueda: "Docker metrics endpoint"
   - ¿Necesito un exportador? (ej: cAdvisor)

#### Tarea 1.2: Diseñar docker-compose

Agrega Prometheus y Grafana a tu docker-compose actual:

**Preguntas guiadas:**
- ¿Qué imagen de Prometheus usas?
- ¿Qué puertos necesita Prometheus? 
- ¿Qué puertos necesita Grafana? 
- ¿Dónde se guardan los datos de Prometheus?
- ¿Cómo le dices a Prometheus dónde están tus servicios?

**Investigación:**
- ¿Qué es `prometheus.yml`? (archivo de configuración)
- ¿Cómo se monta en el contenedor?
- Ejemplo en documentación de Prometheus

#### Tarea 1.3: Configurar Scraping

Prometheus necesita saber de dónde sacar métricas.

**Investigación:**
- ¿Qué es "scraping"?
- ¿Cómo se configura qué servicios monitorear?
  ¿Cómo adaptarías esto a tus servicios?

#### Tarea 1.4: Dashboards en Grafana

1. **Conecta Grafana a Prometheus:**
   - ¿Cuál es la URL de Prometheus desde Grafana?
   - Variables de entorno en Grafana

2. **Crea un dashboard que muestre:**
   - CPU de cada contenedor (% utilización)
   - RAM de cada contenedor (bytes usados/disponibles)
   - Disk space del volumen de datos
   - Network I/O (bytes enviados/recibidos)

**Investigación:**
- ¿Qué métricas hay disponibles de Docker?
- Documentación de Grafana: ¿cómo crear dashboards?
- ¿Puedes importar dashboards pre-hechos?

#### Tarea 1.5: Alertas

Configura 3 alertas simples:

1. **CPU alta:** Cuando CPU de cualquier servicio > 80% por 5 minutos
   - ¿Dónde se configura? (Prometheus rules o Grafana?)
   - ¿Cómo lo notificas? (email, Slack?)

2. **Memoria alta:** Cuando RAM > 85%

3. **Disk casi lleno:** Cuando disk > 90%

**Investigación:**
- Prometheus alerting: ¿cómo se escriben rules?
- Grafana alerts: ¿cuál es la diferencia?

### Validación

Haz esto para verificar que funciona:

1. **Abre Grafana:** `http://localhost:3000` (login: admin/admin por defecto)
2. **Dashboard muestra datos reales** (no gráficos vacíos)
3. **Causa una alerta deliberadamente:**
   - Ej: corre un stress test en CPU
   - ¿La alerta se dispara?
4. **La notificación te llega** (donde la configuraste)

**Documenta:**
- Screenshot del dashboard
- Alertas configuradas
- Cómo verificaste que funciona

---

---

## BONUS 2: Seguridad - Secrets y Environment Variables

### ¿Por qué?

En tu docker-compose actual, probablemente tienes passwords

**Problemas:**
- ¿Estará en git?
- ¿La ven en los logs?
- ¿Los commits históricos la exponen?

**Solución:**
- `.env` files (secretos fuera de git)
- Docker secrets mechanism (para prod)
- TLS/HTTPS automático

### Conceptos a Investigar

1. **¿Por qué no hardcodear passwords?**
   - Riesgos de seguridad
   - Buenas prácticas

2. **.env files:**
   - ¿Qué es un `.env`?
   - ¿Cómo lo usa docker-compose?
   - `.env` vs `.env.example` (uno en git, otro no)

3. **Docker secrets:**
   - ¿Diferencia con variables de entorno?
   - ¿Cuándo usar cada una?
   - Sintaxis en docker-compose

4. **TLS/HTTPS:**
   - Ya hiciste mkcert (local)
   - ¿Cómo automatizar con Let's Encrypt?
   - Diferencias: certificados locales vs públicos

### Tareas Guiadas

#### Tarea 2.1: Auditoría de Secretos

Inspecciona tu docker-compose actual:

1. ¿Dónde están las contraseñas?
   - Grep: `docker-compose.yml | grep -i password`
2. ¿Hay API keys o tokens?
3. ¿Hay credenciales en archivos de configuración?

Haz una lista:
```
- PostgreSQL password: [ENCONTRADA EN ENUNCIADO.MD:123]
- Jellyfin admin password: [NO HARDCODEADA]
- Etc.
```

#### Tarea 2.2: Implementar .env

1. **Crea `.env.example`** (puedes commitear):
   ```
   POSTGRES_PASSWORD=change_me
   POSTGRES_USER=immichuser
   JELLYFIN_ADMIN_PASSWORD=change_me
   ```

2. **Crea `.env`** (NUNCA en git, agregalo a `.gitignore`):
   ```
   POSTGRES_PASSWORD=miPasswordReal123
   POSTGRES_USER=immichuser
   JELLYFIN_ADMIN_PASSWORD=miPassword456
   ```

3. **Actualiza docker-compose:**
   ```yaml
   services:
     postgres:
       environment:
         POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
   ```

**Investigación:**
- ¿Cómo docker-compose interpola variables de `.env`?
- ¿Orden de precedencia? (¿qué gana si está en varios lados?)

#### Tarea 2.3: Git Cleanup (importante!)


#### Tarea 2.3: Docker Secrets (Bonus+)

Si quieres ir más profundo:

1. **¿Qué es Docker secrets?**
   - Almacenamiento seguro de passwords
   - Solo disponible en Docker Swarm o Stack
   - Para producción, es mejor que `.env`

2. **¿Cómo se usa en docker-compose?**
   ```yaml
   secrets:
     db_password:
       file: ./secrets/db_password.txt
   
   services:
     postgres:
       secrets:
         - db_password
   ```

3. **Implementa 1-2 secrets** en tu setup

#### Tarea 2.4: TLS Automático con Let's Encrypt (Bonus++)

Ya hiciste mkcert (certificados locales). Ahora:

1. **Investigación:**
   - ¿Cómo funciona Let's Encrypt?
   - ¿Qué es ACME protocol?
   - ¿Cómo Traefik lo automatiza?

2. **Configuración:**
   - Traefik con Let's Encrypt en docker-compose
   - ¿Dónde guarda los certificados? (volumen)
   - ¿Cómo se renuevan automáticamente?

3. **Testing:**
   - ¿Se renuevan antes de expirar?
   - Verificar en logs de Traefik

### Validación

1. **Corre:** `grep -r "PASSWORD\|SECRET\|API" docker-compose.yml` — ¿hay secretos hardcodeados?
2. **Verifica `.env` funciona:**
   - Borra el `.env`
   - Intenta `docker-compose up` — ¿falla diciendo que falta variable?
   - Restaura `.env` y funciona
3. **Git limpio:**
   - `git log -p | grep -i password` — ¿hay resultados?
4. **.env en .gitignore:**
   - `git check-ignore .env` — ¿sale "ignored"?

**Documenta:**
- Qué secretos identificaste
- Cómo los migraste a `.env`
- Tests que hiciste para validar

---

---

## BONUS 3: Backups y Recuperación

### ¿Por qué?

Preguntas que dan miedo:
- ¿Y si PostgreSQL explota?
- ¿Y si pierdo el volumen con todas las fotos?
- ¿Cuánto tarda recuperarse?
- ¿Dónde hago backup?

**Strategie:**
- Backups automáticos de base de datos
- Plan de restore (que funciona)
- Disaster recovery documentation

### Conceptos a Investigar

1. **Backup strategies:**
   - Full backup (todo)
   - Incremental (solo cambios)
   - Diferencial (cambios desde último full)
   - ¿Cuál para este TP?

2. **RTO vs RPO:**
   - RTO = Recovery Time Objective (¿cuánto tarda recuperar?)
   - RPO = Recovery Point Objective (¿pierdo cuántos datos?)
   - Ej: "RTO 1 hora, RPO 5 minutos" = recupero en 1 hora, pierdo máximo 5 min de datos

3. **Dónde guardar backups:**
   - Local (mismo servidor)
   - Remoto (cloud, otro disco)
   - Redundancia (copia en 2 lugares)

4. **Disaster recovery plan:**
   - Pasos exactos para recuperar
   - Tiempos estimados
   - Quién lo ejecuta

### Tareas Guiadas

#### Tarea 3.1: Investigar PostgreSQL Backup

1. **¿Cómo se hace backup de PostgreSQL?**
   - Comando: `pg_dump`
   - ¿Formato? (SQL script, binario, tar)
   - Desde dentro o fuera del contenedor?

2. **Investigación:**
   - Documentación de PostgreSQL: pg_dump
   - En Docker: ¿cómo ejecuto comandos en contenedor?
   - Comando: `docker exec postgres pg_dump ...`

3. **¿Qué parámetros?**
   - Base de datos específica o todas?
   - Con datos o solo esquema?
   - Comprimido? (`.sql.gz`)

#### Tarea 3.2: Backup Automático

Configura backup automático (sin código, solo investigación):

1. **¿Cómo automatizar?**
   - Cronjob en host (cada día a las 3 AM)
   - Contenedor dedicado (runs nightly)
   - Systemd timer
   - ¿Cuál es más mantenible?

2. **¿Dónde guardar?**
   - Volumen en host
   - Cloud (S3, GCP, Dropbox)
   - Servidor remoto (SSH, rsync)

3. **Preguntas de diseño:**
   - ¿Cuántos backups guardar? (últimos 7 días?)
   - ¿Compresión? (¿cuánto espacio ahorras?)
   - ¿Notificaciones si falla?

#### Tarea 3.3: Restore Procedure

Documenta exactamente cómo recuperarse:

1. **Escenario:** PostgreSQL está corrupted, necesito restaurar

   **Pasos:**
   1. Detener servicios que usan la BD
   2. Conectarme al contenedor PostgreSQL
   3. Restaurar desde backup (comando exacto)
   4. Verificar que la BD está OK
   5. Reiniciar servicios

   **Tiempo estimado:** [minutos]

2. **Escenario:** Perdí el volumen de fotos

   **Pasos:**
   1. Crear volumen nuevo
   2. Copiar archivos de backup
   3. Verificar permisos
   4. Reiniciar servicios

   **Tiempo estimado:** [minutos]


### Validación

1. **Backup ejecuta sin errores**
   - Corre manual: `docker exec postgres pg_dump ...`
   - ¿Output es válido? (contiene `CREATE TABLE`, datos, etc)

2. **Puedes restaurar desde backup**
   - Corrompe intencionalmente un dato
   - Restaura desde backup
   - Dato vuelve a su estado original

3. **Documenta qué pasó**
   - Comandos que usaste
   - Tiempos de cada paso
   - Lecciones aprendidas

---

---

## BONUS 4: Logging Centralizado (Loki)

### ¿Por qué?

Problemas actuales:
- `docker-compose logs jellyfin` solo ve Jellyfin
- Si hay error en Radarr, ¿cómo correlaciono con logs de qBittorrent?
- ¿Cómo busco "ERROR" en todos los servicios?
- ¿Cómo detecto patterns de error que se repiten?

**Logging centralizado =** todos los logs en un lugar, searchable, con alertas.

**Opciones:**
- **Loki** = lightweight, barato, simple

### Conceptos a Investigar

1. **¿Por qué logs centralizados?**
   - Debugging rápido
   - Historial completo
   - Patterns y correlación
   - Alertas automáticas

2. **Structured logging:**
   - ¿Qué es?
   - JSON vs plain text
   - ¿Mejora searching?

3. **Log aggregation:**
   - ¿De dónde recolectar logs?
   - Alloy, Fluentd, Logstash
   - Docker log drivers



#### Tarea 4.1: Diseñar docker-compose

Agrega tu solución elegida a docker-compose:

**Si eleges Loki:**
```yaml
services:
  loki:
    image: grafana/loki
    ports:
      - "3100:3100"
  Alloy:
    image: grafana/alloy
    # Colecta logs de docker
  grafana:
    # Visualiza logs de Loki
```

**Preguntas:**
- ¿Qué puertos necesitan?
- ¿Dónde se guardan los datos? (volumen)
- ¿Cómo se conectan entre sí?

#### Tarea 4.3: Colectar Logs

Configura que tus servicios envíen logs a Loki:

1. **Docker log driver:**
   - `docker-compose` soporta `logging` en cada servicio
   - Ej: `loki` driver
   - Investiga documentación

2. **Para cada servicio, agrega:**
   ```yaml
   services:
     jellyfin:
       logging:
         driver: loki
         options:
           loki-url: http://loki:3100/loki/api/v1/push
   ```

3. **Verifica:**
   - Levanta servicios
   - Generan logs (ej: request)
   - ¿Aparecen en Loki?

#### Tarea 4.4: Queries y Búsqueda

Escribe queries útiles para encontrar información:

**En Loki:**
```
{job="jellyfin"} |= "ERROR"
{job="radarr", environment="prod"} | stats count() by level
```

**Tarea:**
1. Escribe 5 queries útiles:
   - "Todos los errores del último 1h"
   - "Logs de un servicio específico"
   - "Errores que contengan 'connection refused'"
   - "Warnings o errores agrupados por severidad"
   - Etc.

2. Verifica que cada una funciona

#### Tarea 4.5: Alertas

Configura alertas por patterns:

1. **¿Cuándo alertar?**
   - X errores en 5 minutos
   - Pattern "connection refused"
   - "Out of memory" en algún servicio

2. **¿Dónde notificar?**
   - Email
   - Slack
   - Webhook customizado

3. **Implementa 3 alertas:**
   - Error rate alto
   - Patrón específico detectado
   - Servicio down (logs vacíos)

### Validación

1. **Logs de todos los servicios aparecen centralizados**
   - Abre interfaz (Grafana para Loki)
   - ¿Ves logs de Jellyfin, Radarr, PostgreSQL, etc?

2. **Puedes buscar**
   - Queries funcionan
   - Encuentras errores específicos

3. **Alertas funcionan**
   - Generá un error deliberado (ej: auth fallida)
   - ¿Se dispara la alerta?
   - ¿Te llega la notificación?

4. **Correlacionas eventos**
   - Si Radarr falla, ¿ves error en qBittorrent al mismo tiempo?
   - ¿Ves patrón de escalada (primero warning, luego error)?

---

## Entregas

Si haces bonus, entrega en `docs/BONUS/`:

```
docs/
└── BONUS/
    ├── 01_MONITOREO.md
    │   ├── Conceptos aprendidos
    │   ├── Setup docker-compose (cambios agregados)
    │   ├── Screenshots dashboards
    │   └── Alertas configuradas
    ├── 02_SEGURIDAD.md
    │   ├── Auditoría inicial
    │   ├── .env.example
    │   ├── Cambios en docker-compose
    │   └── Validación (git status limpio)
    ├── 03_BACKUPS.md
    │   ├── Estrategia elegida
    │   ├── RTO/RPO targets
    │   ├── Disaster recovery procedures
    │   └── Test de restore (qué pasó)
    └── 04_LOGGING.md
        ├── Solución elegida (Loki)
        ├── Setup docker-compose
        ├── 5 queries útiles
        └── Alertas configuradas
```

---

## Referencias

- [Prometheus Docs](https://prometheus.io/docs/)
- [Grafana Docs](https://grafana.com/docs/)
- [Docker Secrets](https://docs.docker.com/engine/swarm/secrets/)
- [PostgreSQL Backup](https://www.postgresql.org/docs/current/backup.html)
- [Loki Docs](https://grafana.com/docs/loki/)

