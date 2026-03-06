# Series-Free 🎬

**Tu Netflix personal con descargas automatizadas**

Sistema de automatización de medios locales (películas y series) basado en Docker. Selecciona una película y el sistema se encargará de buscarla, descargarla y agregarla a tu biblioteca automáticamente.

## Servicios Incluidos

| Servicio | Puerto | Función |
|----------|--------|---------|
| qbittorrent | 8011 | Descargador de torrents |
| prowlarr | 9696 | Buscador de torrents (indexadores) |
| radarr | 7878 | Gestor de películas |
| sonarr | 8989 | Gestor de series |
| jellyfin | 8096 | Servidor de streaming |
| jellyseerr | 5055 | Solicitar películas/series desde cualquier dispositivo |

## Requisitos

- Docker
- Docker Compose
- 10GB+ disco duro
- Conexión a internet

## Instalación Rápida

```bash
# 1. Clonar o descargar el proyecto
cd series-free

# 2. Iniciar servicios
docker-compose up -d

# 3. Verificar que estén corriendo
docker-compose ps
```

## Configuración Manual

Esta es la parte más importante. Sigue cada paso en orden:

---

### Paso 1: Configurar qbittorrent

**URL:** http://localhost:8011

1. Inicia sesión (admin / OW&i#a3^g26nhm3M*Z92)
2. Ve a **Settings > Downloads**
3. Configura:
   - **Default Save Path:** `/downloads/`
   - **Temp Path:** `/downloads/incomplete/`
4. (Opcional) Ve a **Categories** y agrega:
   - Category: `radarr` → Save path: `/downloads/radarr`
   - Category: `sonarr` → Save path: `/downloads/sonarr`
5. **Save Settings**

---

### Paso 2: Configurar Prowlarr (Indexadores)

**URL:** http://localhost:9696

1. Ve a **Settings > Indexers**
2. Aquí ves una lista de indexadores. Los que están deshabilitados necesitan configuración adicional.
3. Para habilitar uno:
   - Haz clic en el pencil/editar
   - Ingresa las credenciales/API key necesarias
   - **Save**
4. Indexadores públicos que funcionan sin cuenta:
   - **YTS** (películas)
   - **Internet Archive** (dominio público)
   - **The Pirate Bay** (verificar que funcione)

**NOTA:** Muchos indexadores requieren cuenta/API key. Investiga cuáles funcionan en tu región.

---

### Paso 3: Configurar Radarr (Películas)

**URL:** http://localhost:7878
**Usuario:** admin
**Contraseña:** @=5}zet=BsVZw$T

#### 3.1 Agregar qBittorrent como cliente de descarga

1. Ve a **Settings > Download Clients**
2. Click en el **+** (Add)
3. Selecciona **qBittorrent**
4. Configura:
   - **Host:** `qbittorrent`
   - **Port:** `8011`
   - **Username:** `admin`
   - **Password:** `OW&i#a3^g26nhm3M*Z92`
   - **Category:** (déjalo vacío para aceptar cualquier descarga)
5. **Test** → **Save**

#### 3.2 Configurar Remote Path Mapping (CRÍTICO)

1. Ve a **Settings > Download Client > Remote Path Mappings**
2. Click en **+** (Add)
3. Configura:
   - **Host:** `qbittorrent`
   - **Remote Path:** `/downloads/`
   - **Local Path:** `/downloads/`
4. **Save**

> ⚠️ **SIN ESTO, RADARR NO ENCONTRARÁ LAS DESCARGAS**

#### 3.3 Agregar Indexadores

1. Ve a **Settings > Indexers**
2. Habilita los indexadores que conectaste en Prowlarr
3. Cada indexador debe mostrar "Connected" en el test

---

### Paso 4: Configurar Sonarr (Series)

**URL:** http://localhost:8989
**Usuario:** admin
**Contraseña:** @=5}zet=BsVZw$T

Repite los mismos pasos que Radarr:
1. Agregar qBittorrent (igual que en Radarr)
2. Agregar Remote Path Mapping (igual que en Radarr)
3. Habilitar indexadores

---

### Paso 5: Configurar Jellyseerr (Solicitudes)

**URL:** http://localhost:5055

1. Primera vez: configurar cuenta admin
2. Conectar con Radarr y Sonarr:
   - **Radarr URL:** `http://radarr:7878`
   - **Radarr API Key:** `c490c5168a9b4ba086bf73052cef5f60`
   - **Sonarr URL:** `http://sonarr:8989`
   - **Sonarr API Key:** (ver en Settings > General > API Key)

---

### Paso 6: Configurar Jellyfin (Streaming)

**URL:** http://localhost:8096

1. Primera vez: wizard de configuración
2. Agregar bibliotecas:
   - Movies: `/data/movies` (mapeado a tu carpeta de películas)
   - TV Shows: `/data/tvshows`

---

## Cómo Usar el Sistema

### Opción 1: Desde Jellyseerr (Recomendado)

1. Ve a http://localhost:5055
2. Busca una película o serie
3. Click en "Request"
4. Radarr/Sonarr la buscarán, descargarán e importarán automáticamente

### Opción 2: Directamente desde Radarr

1. Ve a http://localhost:7878
2. Click en **+** o **Add Movie**
3. Busca una película
4. Agrega a tu biblioteca con **Search on Add** marcado

### Opción 3: Descarga manual + Importación automática

1. Descarga un torrent desde cualquier lugar
2. Agrégalo a qBittorrent
3. Radarr detectará la descarga y la importará automáticamente

---

## Estructura de Archivos

```
series-free/
├── docker-compose.yml          # Configuración de servicios
├── .env                       # Variables de entorno
├── CLAUDE.md                  # Guía para Claude Code
├── README.md                  # Este archivo
└── config/
    ├── radarr/
    │   └── custom-cont-init.d/  # Scripts de inicio
    ├── prowlarr/
    │   └── custom-cont-init.d/
    ├── sonarr/
    └── qbittorrent/
```

### Carpetas de Datos (en tu host)

```
/media/
├── downloads/           # Torrents en descarga
├── radarr/
│   ├── config/         # Configuración de Radarr
│   └── movies/        # Películas organizadas
├── sonarr/
│   ├── config/
│   └── tvshows/       # Series organizadas
├── jellyfin/
│   ├── config/
│   └── cache/
└── qbittorrent/
    └── config/
```

---

## Comandos Útiles

```bash
# Iniciar servicios
docker-compose up -d

# Detener servicios
docker-compose down

# Ver logs de un servicio
docker-compose logs -f radarr
docker-compose logs -f qbittorrent

# Reiniciar un servicio
docker-compose restart radarr

# Ver estado
docker-compose ps
```

---

## Troubleshooting

### "No results found" al buscar película

1. Verifica que los indexadores estén habilitados en Prowlarr
2. Testea cada indexador en Prowlarr
3. Verifica que Radarr vea los indexadores (Settings > Indexers)

### La descarga no se importa

1. Ve a Radarr > Settings > Remote Path Mappings
2. Verifica que exista el mapeo: qbittorrent, /downloads/ → /downloads/
3. Revisa los logs: `docker-compose logs radarr | grep -i import`

### Error de conexión con qBittorrent

1. Verifica que qBittorrent esté corriendo
2. Revisa el puerto (8011)
3. Verifica credenciales

### DNS o conexión externa no funciona

Los contenedores ya tienen DNS configurado. Si hay problemas:
```bash
docker-compose restart
```

---

## Credenciales por Defecto

| Servicio | Usuario | Contraseña |
|----------|---------|------------|
| qbittorrent | admin | OW&i#a3^g26nhm3M*Z92 |
| radarr | admin | @=5}zet=BsVZw$T |
| sonarr | admin | @=5}zet=BsVZw$T |
| jellyseerr | (tu config) | - |
| jellyfin | (tu config) | - |

**Cambia estas contraseñas en producción!**

---

## API Keys

- **Radarr:** `c490c5168a9b4ba086bf73052cef5f60`
- **Prowlarr:** `a435f8db0d014d3690d5a8ef70f4c4e3`
- **Sonarr:** (ver en Settings > General)

---

## Flujo de Descarga Automática

```
1. Usuario solicita película (Jellyseerr / Radarr)
       ↓
2. Radarr busca en indexadores (Prowlarr)
       ↓
3. Prowlар возвращает resultados
       ↓
4. Radarr selecciona mejor resultado
       ↓
5. Envía torrent a qBittorrent
       ↓
6. qBittorrent descarga a /downloads/
       ↓
7. Radarr detecta descarga completa
       ↓
8. Radarr mueve a /data/movies/
       ↓
9. Jellyfin detecta nueva película
       ↓
10. ¡Lista para ver en streaming!
```

---

## Notas

- Las películas se descargan en `/media/downloads/`
- Radarr las mueve a `/media/radarr/movies/`
- Jellyfin sirve desde `/media/radarr/movies/`
- Todo el proceso es automático una vez configurado
