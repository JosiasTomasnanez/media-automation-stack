# 🎬 MEDIA SERVER: Servidor de Películas y Series

**Lee esto ANTES de empezar el trabajo final.**

---

## ¿Qué es un Media Server?

Un servidor que **centraliza, organiza y sirve** películas y series a todos tus dispositivos.

**Lo que hace:**
- Descarga automáticamente películas que quieres ver
- Las organiza automáticamente
- Las sirve a Smart TV, PC, móvil
- Transcode en tiempo real (ajusta calidad)

**Caso de uso:**
```
Ves en redes: "Dune 2 salió"
    ↓
Agregas a tu lista en Radarr
    ↓
Radarr busca automáticamente
    ↓
Descarga con qBittorrent
    ↓
Jellyfin lo organiza
    ↓
Ves en tu TV con carátula, sinopsis, etc.
```

---

## Arquitectura

```
Tú agregas película a Radarr
    ↓
Radarr pregunta a Prowlarr: ¿dónde la busco?
    ↓
Radarr busca en trackers y descarga con qBittorrent
    ↓
Jellyfin organiza y sirve a tus dispositivos
```

---

## Componentes

### Jellyfin
- Servidor de streaming (reemplaza Plex/Netflix)
- Interfaz web bonita
- Apps para Smart TV, Android, iOS
- **Puerto:** 8096
- **Lee:** [jellyfin.org/docs](https://docs.jellyfin.org)

### Radarr
- Busca y descarga películas automáticamente
- Monitorea tu lista de películas
- Integra con qBittorrent para descargar
- **Puerto:** 7878
- **Lee:** [wiki.servarr.com/radarr](https://wiki.servarr.com/radarr)

### Prowlarr
- Agregador de trackers/índices
- Radarr pregunta: "¿dónde busco?"
- Prowlarr responde con lista de trackers
- **Puerto:** 9696
- **Lee:** [wiki.servarr.com/prowlarr](https://wiki.servarr.com/prowlarr)

### qBittorrent
- Cliente torrent con interfaz web
- Descarga archivos desde torrents
- Radarr lo comanda automáticamente
- **Puerto:** 6881 + 8080
- **Lee:** [qbittorrent.org](https://www.qbittorrent.org)

---


## Flujo completo (para entender)

```
1. TÚ: Agrego "Dune 2" a Radarr

2. RADARR: Pregunta a Prowlarr dónde buscar

3. PROWLARR: Responde con lista de trackers

4. RADARR: Busca en esos trackers, encuentra torrent bueno

5. RADARR: Envía a qBittorrent para descargar

6. qBITTORRENT: Descarga archivo

7. RADARR: Verifica integridad, renombra, mueve a /movies

8. JELLYFIN: Detecta nuevo archivo, descarga metadata

9. TÚ: Ves "Dune 2" en tu biblioteca con carátula y sinopsis
```


---

## Para aprender más

- [Jellyfin Docs](https://docs.jellyfin.org)
- [Radarr Docs](https://wiki.servarr.com/radarr)
- [Prowlarr Docs](https://wiki.servarr.com/prowlarr)
- [Docker Networking](https://docs.docker.com/network/)

**Investiga la documentación oficial de cada componente cuando lo necesites.**
