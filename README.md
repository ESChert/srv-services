# Services

Repositorio de despliegue local con varios stacks Docker para automatización, media, DNS, IA y juego. La idea es mantener cada servicio aislado en su carpeta, con persistencia local y una configuración suficientemente simple para levantar o parar cada stack de forma independiente.

## Qué hay desplegado

### Stack raíz

Definido en [docker-compose.yml](/home/nobody93/services/docker-compose.yml).

- `postgres`: base de datos para `n8n`.
- `n8n`: automatizaciones y webhooks.
- `caddy`: proxy HTTPS para publicar `n8n` bajo `/n8n`.

### `ia-stack/`

Definido en [ia-stack/docker-compose.yml](/home/nobody93/services/ia-stack/docker-compose.yml).

- `ollama`: inferencia local de modelos.
- Datos persistidos en `ia-stack/ollama/data/`.

### `pihole/`

Definido en [pihole/docker-compose.yml](/home/nobody93/services/pihole/docker-compose.yml).

- `pihole`: DNS con bloqueo de anuncios.
- Persistencia en `pihole/data/` y `pihole/dnsmasq/`.
- Expone DNS en `53/tcp` y `53/udp`, y panel web en `8081`.

### `plex/`

Definido en [plex/docker-compose.yml](/home/nobody93/services/plex/docker-compose.yml).

- `plex`
- `qbittorrent`
- `prowlarr`
- `radarr`
- `sonarr`
- `bazarr`

Persistencia principal:

- `plex/config/`
- `plex/downloads/`
- `plex/media/`

### `minecraft/`

Definido en [minecraft/compose/docker-compose.yml](/home/nobody93/services/minecraft/compose/docker-compose.yml).

- `minecraft`: servidor Java basado en `itzg/minecraft-server`.
- Datos persistidos en `minecraft/data/`.

## Estructura

```text
.
├── .env
├── .env.example
├── docker-compose.yml
├── caddy/
├── ia-stack/
├── minecraft/
├── pihole/
└── plex/
```

## Variables de entorno

El archivo real `.env` no debe versionarse. Usa el ejemplo:

```bash
cp .env.example .env
```

Variables relevantes:

- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB_N8N`: credenciales de PostgreSQL para `n8n`.
- `N8N_ENCRYPTION_KEY`: clave de cifrado interna de `n8n`.
- `N8N_HOST`, `N8N_PROTOCOL`, `N8N_PORT`, `N8N_PATH`: publicación externa de `n8n`.
- `WEBHOOK_URL`, `N8N_EDITOR_BASE_URL`: URLs públicas usadas por `n8n`.
- `PIHOLE_WEBPASSWORD`: contraseña del panel de Pi-hole.
- `TZ`, `PUID`, `PGID`: valores comunes para varios contenedores.
- `MINECRAFT_VERSION`, `MINECRAFT_MEMORY`: ajuste rápido del servidor de Minecraft.

## Arranque

Todos los comandos siguientes están pensados para ejecutarse desde la raíz del repo.

### Stack raíz

```bash
docker compose --env-file .env up -d
```

### Ollama

```bash
docker compose --env-file .env -f ia-stack/docker-compose.yml up -d
```

### Pi-hole

```bash
docker compose --env-file .env -f pihole/docker-compose.yml up -d
```

### Plex stack

```bash
docker compose --env-file .env -f plex/docker-compose.yml up -d
```

### Minecraft

```bash
docker compose --env-file .env -f minecraft/compose/docker-compose.yml up -d
```

## Parada

```bash
docker compose --env-file .env down
docker compose --env-file .env -f ia-stack/docker-compose.yml down
docker compose --env-file .env -f pihole/docker-compose.yml down
docker compose --env-file .env -f plex/docker-compose.yml down
docker compose --env-file .env -f minecraft/compose/docker-compose.yml down
```

## Notas operativas

- [caddy/Caddyfile](/home/nobody93/services/caddy/Caddyfile) está cableado a un host concreto de Tailscale y a certificados montados desde `/certs`. Si cambia el dominio o la ruta de certificados, hay que editar ese archivo.
- `n8n` se publica bajo `/n8n`, así que `N8N_PATH`, `WEBHOOK_URL` y `N8N_EDITOR_BASE_URL` deben mantenerse coherentes entre sí.
- `plex` usa `network_mode: host`, así que comparte red con el host.
- `pihole` requiere `NET_ADMIN`.
- Los directorios de datos están ignorados en git para no mezclar estado runtime con infraestructura.

## Ficheros importantes

- [README.md](/home/nobody93/services/README.md): visión general y comandos básicos.
- [.env.example](/home/nobody93/services/.env.example): plantilla sin secretos reales.
- [AGENT_CONTEXT.md](/home/nobody93/services/AGENT_CONTEXT.md): contexto operativo para siguientes agentes.
- [docker-compose.yml](/home/nobody93/services/docker-compose.yml): stack raíz con `postgres`, `n8n` y `caddy`.
