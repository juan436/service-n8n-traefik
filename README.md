# Despliegue de n8n con Traefik y MySQL

Este proyecto contiene una pila de Docker para publicar n8n en producción detrás de Traefik con certificados TLS automáticos (ACME) y una base de datos MySQL dedicada.

![License](https://img.shields.io/badge/License-MIT-green.svg)

- Servicios:
  - **MySQL**: Base de datos de n8n.
  - **Traefik**: Proxy inverso y terminación TLS (Let's Encrypt).
  - **n8n**: Plataforma de automatización.

- Redes externas requeridas:
  - **proxy**: Red compartida para Traefik y servicios publicados.
  - **db-net**: Red interna para MySQL y n8n (subred 192.168.100.0/24).

- Email ACME configurado: `test@mail.com`.

## Requisitos
- Docker y Docker Compose instalados.
- Un dominio apuntando a tu servidor (A/AAAA) si vas a publicar vía HTTPS.

## Estructura
```
./mysql/docker-compose.yml     # Servicio MySQL
./traefik/docker-compose.yml   # Servicio Traefik
./traefik/traefik.yml          # Configuración estática de Traefik
./n8n/docker-compose.yml       # Servicio n8n
```

## Preparación inicial
1. Crear redes externas (si no existen):
   - Red proxy (sin subred específica):
     ```bash
     docker network create proxy
     ```
   - Red db-net (con subred 192.168.100.0/24):
     ```bash
     docker network create --subnet=192.168.100.0/24 db-net
     ```

2. Revisar variables importantes:
   - En `mysql/docker-compose.yml` ajusta:
     - `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD` y usuarios/DB según tu seguridad.
   - En `n8n/docker-compose.yml` ajusta:
     - `DB_MYSQLDB_HOST` a la IP del contenedor MySQL (ej. 192.168.100.10) o nombre DNS si lo prefieres.
     - `N8N_HOST`, `N8N_WEBHOOK_URL`, `N8N_EDITOR_BASE_URL`, `N8N_API_BASE_URL` a tu dominio público.
     - Credenciales de `N8N_BASIC_AUTH_USER` y `N8N_BASIC_AUTH_PASSWORD`.
   - En `traefik/traefik.yml` y `traefik/docker-compose.yml`:
     - El email ACME ya está en `test@mail.com`. Cámbialo por el correo real para producción si lo deseas.

3. Archivo ACME para Traefik:
   - Crea el archivo `traefik/acme.json` vacío si no existe:
     ```bash
     echo {} > traefik/acme.json
     ```
   - Otorga permisos adecuados (en Linux/macOS):
     ```bash
     chmod 600 traefik/acme.json
     ```

## Puesta en marcha
1. Levantar MySQL:
   ```bash
   docker compose -f mysql/docker-compose.yml up -d
   ```
2. Levantar Traefik:
   ```bash
   docker compose -f traefik/docker-compose.yml up -d
   ```
3. Levantar n8n:
   ```bash
   docker compose -f n8n/docker-compose.yml up -d
   ```

## Acceso
- n8n en: `https://<tu-dominio>` (según variables `N8N_HOST` y URLs definidas).
- Dashboard Traefik: `http://<tu-servidor>:8080` (inseguro habilitado solo para administración; considera deshabilitarlo en producción).

## Seguridad y buenas prácticas
- Cambia todas las contraseñas por valores seguros antes de publicar.
- Reemplaza el email `test@mail.com` por uno válido si usas Let's Encrypt en producción.
- Limita el dashboard de Traefik o deshabilita `api.insecure=true` en entornos públicos.
- Revisa las IPs fijas en `db-net` (192.168.100.10 para MySQL y 192.168.100.20 para n8n). Asegura que no entren en conflicto con tu red.

## Mantenimiento
- Ver logs:
  ```bash
  docker compose -f traefik/docker-compose.yml logs -f
  docker compose -f mysql/docker-compose.yml logs -f
  docker compose -f n8n/docker-compose.yml logs -f
  ```
- Actualizar imágenes:
  ```bash
  docker compose -f traefik/docker-compose.yml pull && docker compose -f traefik/docker-compose.yml up -d
  docker compose -f mysql/docker-compose.yml pull && docker compose -f mysql/docker-compose.yml up -d
  docker compose -f n8n/docker-compose.yml pull && docker compose -f n8n/docker-compose.yml up -d
  ```

## Notas
- Las variables de entorno actuales usan dominios de ejemplo. Ajusta a tu dominio real antes de exponer.
- Si despliegas en Windows, los comandos mostrados en bloques bash pueden ejecutarse en PowerShell con sintaxis equivalente.

## Autor
- Juan Villegas (`juancvillefer`)

