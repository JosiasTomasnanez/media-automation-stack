# 📸 IMMICH: Servidor de Fotos con IA

**Lee esto ANTES de empezar el trabajo final.**

---

## ¿Qué es Immich?

Immich es un **servidor de fotos open-source** que reemplaza a Google Photos.

**Lo que hace:**
- Sincroniza fotos automáticamente desde tu móvil
- Busca inteligentemente (detecta caras, objetos, lugares)
- Organiza automáticamente
- Comparte álbumes con familia

**Caso de uso:**
```
Tomas 50 fotos en un viaje
    ↓
Immich las sincroniza
    ↓
IA detecta: caras de personas, playa, montaña
    ↓
Puedes buscar: "fotos con Juan en la playa"
    ↓
Aparecen exactamente esas
```

---

## Arquitectura

```
Tu móvil (app Immich)
    ↓ (sube fotos)
Immich Server (recibe, guarda, sirve)
    ↓ (necesita estos 3)
├── PostgreSQL (guarda metadata)
├── Redis (caché + cola de tareas)
└── ML Service (IA detecta caras)
```

---

## Componentes

### Immich Server
- Recibe fotos desde móvil
- API REST para consultas
- Guarda metadatos en BD
- **Puerto:** 3001
- **Lee:** [docs.immich.app](https://docs.immich.app)

### PostgreSQL
- BD para usuarios, fotos, caras detectadas
- Metadatos persistentes
- **Sin esto:** pierdes todo al reiniciar

### Redis
- Caché (búsquedas rápidas)
- Cola de tareas (IA procesa fotos en orden)
- **Sin esto:** lento y sin paralelismo

### Immich ML Service
- Procesa fotos con IA
- Detecta caras, objetos, ubicación
- Corre en segundo plano
- **Sin esto:** no hay búsqueda inteligente

---

## Para aprender más

- [Immich Official Docs](https://docs.immich.app/overview/introduction)
- [Immich Docker Compose Setup](https://docs.immich.app/install/docker-compose/)
- [PostgreSQL Basics](https://www.postgresql.org/docs/current/tutorial.html)
- [Redis Basics](https://redis.io/documentation)
- [Docker Networking](https://docs.docker.com/network/)

