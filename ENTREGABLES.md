# Entregables: Qué Entregar y Cómo Evaluar

**Fecha de entrega:** [Definir con profesor]  
**Formato:** Repositorio Git con esta estructura

---

## Estructura de Carpetas Esperada

```
SO/TP_Final/
├── ENUNCIADO.md                          # Este archivo
├── BONUS_AVANZADO.md                     # Bonus opcional
├── ENTREGABLES.md                        # Este archivo
├── README_IMMICH.md / README_MEDIA_SERVER.md  # Introducción elegida
│
├── docker-compose.yml                    # TU implementación 
├── .env.example                          # Template de variables (si lo hiciste)
├── .gitignore                            # Git ignore config (importante!)
│
└── docs/
    ├── ANALISIS_CONTENEDOR_PRINCIPAL.md  # Checkpoint 1 ⭐
    ├── PERSISTENCIA.md                   # Checkpoint 2 ⭐
    ├── OTRO.md                           # PUEDEN AGREGAR MAS DOCUS, como prefieran
    │
    └── BONUS/  (OPCIONAL)
        ├── 01_MONITOREO.md
        ├── 02_SEGURIDAD.md
        ├── 03_DIAGRAMAS.md
        ├── 04_LOGGING.md
        └── 05_BACKUPS.md
        
```

**⭐ = OBLIGATORIO**

---

## Checklist Básico (Obligatorio)

Antes de entregar, verifica:

### Código

- [ ] `docker-compose.yml` está limpio y no tiene hardcoded passwords
- [ ] `.env.example` existe si usaste `.env`
- [ ] `.gitignore` contiene `.env`, `*.log`, caché (si aplica)
- [ ] No hay credenciales en git history (`git log --grep=password` no da resultados)

### Documentación

- [ ] `docs/ANALISIS_CONTENEDOR_PRINCIPAL.md` existe y está completo
  - [ ] Qué funciona y qué no
  - [ ] Logs de errores reales
  - [ ] Hipótesis de por qué pasa
  
- [ ] `docs/PERSISTENCIA.md` existe y documenta:
  - [ ] Volúmenes utilizados
  - [ ] Dónde van en el host
  - [ ] Validación que persisten después de down+up

### Funcionalidad

- [ ] `docker-compose up -d` levanta sin errores
- [ ] Todos los servicios corren: `docker ps` muestra X servicios
- [ ] Interface web es accesible
- [ ] Datos persisten después de `docker-compose down && docker-compose up -d`

### Testing

- [ ] Documentaste qué funcionó y qué no

---

## Checklist Bonus (Opcional, pero +Puntos)

Si hiciste bonus, verifica:

### Bonus 1: Monitoreo

- [ ] `docs/BONUS/01_MONITOREO.md` explica setup
- [ ] Prometheus + Grafana funcionan
- [ ] Dashboard muestra CPU/RAM/Disk
- [ ] Screenshot del dashboard

### Bonus 2: Seguridad

- [ ] `docs/BONUS/02_SEGURIDAD.md` explica cambios
- [ ] Passwords movidos a `.env`
- [ ] (Opcional) Docker secrets implementados

### Bonus 3: Backups

- [ ] `docs/BONUS/03_BACKUPS.md` documenta plan
- [ ] Backup automático configurado

### Bonus 4: Diagrama de RED y Arquitectura

## Preguntas Frecuentes

**P: ¿Tengo que hacer los bonus?**  
R: No, son opcionales. El trabajo base (100 puntos) es completo sin ellos.


**P: ¿Tengo que usar Traefik?**  
R: Es altamente recomendado (es parte del ENUNCIADO). Si no lo usas, usa otro proxy que cumpla con lo mismo.
