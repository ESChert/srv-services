# Agent Context

## Resumen

Este repositorio agrupa varios despliegues Docker domésticos/autohospedados. No es una aplicación única: son varios stacks independientes que comparten repo y, en algunos casos, el mismo `.env` raíz.

## Stacks presentes

### Raíz

Archivo: [docker-compose.yml](/home/nobody93/services/docker-compose.yml)

Servicios:

- `postgres`
- `n8n`
- `caddy`

Objetivo:

- `postgres` almacena estado de `n8n`.
- `caddy` publica `n8n` por HTTPS bajo `/n8n`.

Detalles importantes:

- El proxy está definido en [caddy/Caddyfile](/home/nobody93/services/caddy/Caddyfile).
- El host actual está ligado a Tailscale.
- Los certificados se montan desde `/certs`.

### IA

Archivo: [ia-stack/docker-compose.yml](/home/nobody93/services/ia-stack/docker-compose.yml)

Servicio:

- `ollama`

Notas:

- Persiste en `ia-stack/ollama/data/`.
- Expone `11434` en loopback y en una IP concreta adicional.

### Pi-hole

Archivo: [pihole/docker-compose.yml](/home/nobody93/services/pihole/docker-compose.yml)

Servicio:

- `pihole`

Notas:

- Usa `NET_ADMIN`.
- Expone DNS en `53/tcp` y `53/udp`.
- Expone UI en `8081`.
- La contraseña web debe venir por variable `PIHOLE_WEBPASSWORD`.

### Media

Archivo: [plex/docker-compose.yml](/home/nobody93/services/plex/docker-compose.yml)

Servicios:

- `plex`
- `qbittorrent`
- `prowlarr`
- `radarr`
- `sonarr`
- `bazarr`

Notas:

- `plex` usa `network_mode: host`.
- La configuración persistente vive bajo `plex/config/`.
- Descargas y librerías viven bajo `plex/downloads/` y `plex/media/`.

### Minecraft

Archivo: [minecraft/compose/docker-compose.yml](/home/nobody93/services/minecraft/compose/docker-compose.yml)

Servicio:

- `minecraft`

Notas:

- Usa imagen `itzg/minecraft-server:java25`.
- Está configurado para `PAPER`.
- Datos persistentes en `minecraft/data/`.

## Convenciones del repo

- `.env` está ignorado por git y contiene secretos reales locales.
- `.env.example` es la plantilla saneada.
- Los directorios de datos runtime están ignorados en `.gitignore`.
- Cada stack se puede arrancar por separado con `docker compose -f ...`.
- Para evitar ambigüedad con variables, conviene usar siempre `docker compose --env-file .env -f ...`.

## Riesgos / deuda técnica detectada

- [caddy/Caddyfile](/home/nobody93/services/caddy/Caddyfile) tiene dominio y rutas de certificados hardcodeados; no está parametrizado.
- `ia-stack/docker-compose.yml` publica una IP concreta además de loopback; si cambia la red del host, habrá que revisarlo.
- Varios stacks usan valores locales implícitos como `Europe/Madrid` o `PUID=1000`; ahora están documentados, pero no todos estaban externalizados originalmente.
- Este repo parece recién inicializado en git: hay varios archivos en estado `A` y `??`.

## Qué se ha dejado preparado

- `README.md` ampliado con mapa del repo, variables y comandos de arranque.
- `.env.example` sin credenciales reales.
- `pihole/docker-compose.yml` preparado para leer la contraseña desde variable.

## Cómo orientarse rápido

1. Leer [README.md](/home/nobody93/services/README.md).
2. Revisar [docker-compose.yml](/home/nobody93/services/docker-compose.yml) si el problema afecta a `n8n` o `caddy`.
3. Revisar el `docker-compose.yml` de la carpeta correspondiente si el problema es específico de un stack.
4. No tocar `.env` real ni directorios de datos salvo que la tarea lo exija explícitamente.
