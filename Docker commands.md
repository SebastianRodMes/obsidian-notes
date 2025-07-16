# Guía de Comandos Docker Esenciales

## 📋 Índice

1. [Estructura de Comandos Docker](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#estructura-de-comandos-docker)
2. [Comandos de Imágenes](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#comandos-de-im%C3%A1genes)
3. [Comandos de Contenedores](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#comandos-de-contenedores)
4. [Comandos de Interacción](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#comandos-de-interacci%C3%B3n)
5. [Comandos de Gestión de Datos](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#comandos-de-gesti%C3%B3n-de-datos)
6. [Comandos de Red](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#comandos-de-red)
7. [Comandos de Limpieza](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#comandos-de-limpieza)
8. [Docker Compose](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#docker-compose)
9. [Ejemplos Prácticos](https://claude.ai/chat/c57327de-b07c-4c18-bf65-8adeef330a56#ejemplos-pr%C3%A1cticos)

---

## 📖 Estructura de Comandos Docker

### Anatomía de un comando Docker

```bash
docker [COMANDO] [OPCIONES] [ARGUMENTOS]
```

**Ejemplo desglosado:**

```bash
docker run -d -p 80:80 --name mi-web nginx:latest
```

- `docker` = Comando base
- `run` = Subcomando (crear y ejecutar contenedor)
- `-d` = Opción (detached/segundo plano)
- `-p 80:80` = Opción con valor (mapeo de puertos)
- `--name mi-web` = Opción con valor (nombre del contenedor)
- `nginx:latest` = Argumento (imagen con tag)

### Tipos de opciones comunes:

- **Opciones de una letra:** `-d`, `-p`, `-v`, `-e`, `-i`, `-t`
- **Opciones largas:** `--name`, `--volume`, `--env`, `--network`
- **Opciones combinadas:** `-it` (equivale a `-i -t`)

### Convenciones importantes:

- Las **opciones** van antes del **argumento principal**
- Los **argumentos** pueden ser nombres, IDs, o rutas
- Usar `--help` con cualquier comando para ver opciones disponibles

---

## 🖼️ Comandos de Imágenes

### `docker pull`

**Estructura:** `docker pull [OPCIONES] IMAGEN[:TAG]`

```bash
docker pull nginx:latest
docker pull ubuntu:20.04
docker pull --quiet mysql:8.0  # Descarga silenciosa
```

### `docker images`

**Estructura:** `docker images [OPCIONES] [REPOSITORIO[:TAG]]`

```bash
docker images
docker images -a        # Incluye imágenes intermedias
docker images nginx     # Solo imágenes de nginx
docker images --quiet   # Solo muestra IDs
```

### `docker build`

**Estructura:** `docker build [OPCIONES] CONTEXTO`

```bash
docker build -t mi-app:1.0 .
docker build -t mi-app:latest --no-cache .
docker build -f Dockerfile.dev -t mi-app:dev .
```

### `docker rmi`

**Estructura:** `docker rmi [OPCIONES] IMAGEN [IMAGEN...]`

```bash
docker rmi nginx:latest
docker rmi -f mi-app:1.0        # Fuerza eliminación
docker rmi $(docker images -q)  # Elimina todas las imágenes
```

### `docker tag`

**Estructura:** `docker tag IMAGEN[:TAG] NUEVA_IMAGEN[:TAG]`

```bash
docker tag mi-app:1.0 mi-app:produccion
docker tag mi-app:latest registro.com/mi-app:latest
```

---

## 🐳 Comandos de Contenedores

### `docker run`

**Estructura:** `docker run [OPCIONES] IMAGEN [COMANDO] [ARGUMENTOS...]`

**Ejemplos con explicación:**

```bash
# Ejemplo 1: Nginx en segundo plano
docker run -d -p 80:80 nginx
# -d = detached (segundo plano)
# -p 80:80 = mapeo de puertos (host:contenedor)
# nginx = nombre de la imagen

# Ejemplo 2: Ubuntu interactivo
docker run -it ubuntu:20.04 bash
# -it = interactive + terminal
# ubuntu:20.04 = imagen con tag específico
# bash = comando a ejecutar dentro del contenedor

# Ejemplo 3: Contenedor con nombre personalizado
docker run --name mi-contenedor -d nginx
# --name mi-contenedor = asigna nombre al contenedor
# -d = detached (segundo plano)
# nginx = imagen
```

**Diferencia entre `-d` y sin `-d`:**

```bash
# CON -d (segundo plano)
docker run -d nginx
# Te devuelve el ID del contenedor y puedes seguir usando la terminal

# SIN -d (primer plano)
docker run nginx
# La terminal se "queda" mostrando los logs del contenedor
# Para salir necesitas Ctrl+C (y esto detiene el contenedor)
```

**Opciones más utilizadas:**

- `-d` : Ejecuta en segundo plano (detached)
- `-p` : Mapea puertos (host:contenedor)
- `-it` : Modo interactivo con terminal
- `--name` : Asigna un nombre al contenedor
- `-v` : Monta volúmenes
- `-e` : Variables de entorno
- `--rm` : Elimina el contenedor al salir
- `--network` : Conecta a una red específica

### `docker ps`

**Estructura:** `docker ps [OPCIONES]`

```bash
docker ps
docker ps -a        # Incluye contenedores detenidos
docker ps -q        # Solo muestra IDs
docker ps --filter "status=running"  # Filtra por estado
```

### `docker stop`

**Estructura:** `docker stop [OPCIONES] CONTENEDOR [CONTENEDOR...]`

```bash
docker stop mi-contenedor
docker stop $(docker ps -q)    # Detiene todos los contenedores
docker stop -t 30 mi-contenedor  # Espera 30 segundos antes de forzar
```

### `docker start`

**Estructura:** `docker start [OPCIONES] CONTENEDOR [CONTENEDOR...]`

```bash
docker start mi-contenedor
docker start -a mi-contenedor   # Adjunta stdout/stderr
docker start -i mi-contenedor   # Modo interactivo
```

### `docker restart`

**Estructura:** `docker restart [OPCIONES] CONTENEDOR [CONTENEDOR...]`

```bash
docker restart mi-contenedor
docker restart -t 10 mi-contenedor  # Tiempo de espera de 10 segundos
```

### `docker rm`

**Estructura:** `docker rm [OPCIONES] CONTENEDOR [CONTENEDOR...]`

```bash
docker rm mi-contenedor
docker rm -f mi-contenedor      # Fuerza la eliminación
docker rm $(docker ps -aq)     # Elimina todos los contenedores
docker rm -v mi-contenedor      # Elimina también volúmenes asociados
```

---

## 🔧 Comandos de Interacción

### `docker exec`

**Estructura:** `docker exec [OPCIONES] CONTENEDOR COMANDO [ARGUMENTOS...]`

```bash
docker exec -it mi-contenedor bash
docker exec -it mi-contenedor sh
docker exec mi-contenedor ls -la /app
docker exec -u root mi-contenedor whoami  # Ejecuta como root
```

### `docker logs`

**Estructura:** `docker logs [OPCIONES] CONTENEDOR`

```bash
docker logs mi-contenedor
docker logs -f mi-contenedor          # Sigue los logs en tiempo real
docker logs --tail 100 mi-contenedor  # Últimas 100 líneas
docker logs --since 2h mi-contenedor  # Logs de las últimas 2 horas
```

### `docker inspect`

**Estructura:** `docker inspect [OPCIONES] OBJETO [OBJETO...]`

```bash
docker inspect mi-contenedor
docker inspect nginx:latest
docker inspect --format='{{.NetworkSettings.IPAddress}}' mi-contenedor
```

### `docker stats`

**Estructura:** `docker stats [OPCIONES] [CONTENEDOR...]`

```bash
docker stats
docker stats mi-contenedor
docker stats --no-stream     # Solo una lectura
```

---

## 💾 Comandos de Gestión de Datos

### `docker volume ls`

**Estructura:** `docker volume ls [OPCIONES]`

```bash
docker volume ls
docker volume ls -f dangling=true  # Volúmenes huérfanos
docker volume ls --quiet           # Solo nombres
```

### `docker volume create`

**Estructura:** `docker volume create [OPCIONES] [VOLUMEN]`

```bash
docker volume create mi-volumen
docker volume create --driver local --opt type=tmpfs mi-temp
```

### `docker volume rm`

**Estructura:** `docker volume rm [OPCIONES] VOLUMEN [VOLUMEN...]`

```bash
docker volume rm mi-volumen
docker volume rm $(docker volume ls -q)  # Elimina todos los volúmenes
```

### `docker cp`

**Estructura:** `docker cp [OPCIONES] CONTENEDOR:RUTA_ORIGEN DESTINO` **Estructura:** `docker cp [OPCIONES] ORIGEN CONTENEDOR:RUTA_DESTINO`

```bash
docker cp mi-contenedor:/app/config.txt ./config.txt
docker cp ./archivo.txt mi-contenedor:/app/
docker cp -a mi-contenedor:/app/. ./backup/  # Copia preservando permisos
```

---

## 🌐 Comandos de Red

### `docker network ls`

**Estructura:** `docker network ls [OPCIONES]`

```bash
docker network ls
docker network ls --quiet      # Solo IDs
docker network ls --filter driver=bridge
```

### `docker network create`

**Estructura:** `docker network create [OPCIONES] RED`

```bash
docker network create mi-red
docker network create --driver bridge mi-red-bridge
docker network create --subnet=172.20.0.0/16 mi-red-custom
```

### `docker network rm`

**Estructura:** `docker network rm RED [RED...]`

```bash
docker network rm mi-red
docker network rm $(docker network ls -q)  # Elimina todas las redes personalizadas
```

### `docker network connect`

**Estructura:** `docker network connect [OPCIONES] RED CONTENEDOR`

```bash
docker network connect mi-red mi-contenedor
docker network connect --ip 172.20.0.5 mi-red mi-contenedor
```

### `docker network disconnect`

**Estructura:** `docker network disconnect [OPCIONES] RED CONTENEDOR`

```bash
docker network disconnect mi-red mi-contenedor
docker network disconnect -f mi-red mi-contenedor  # Fuerza desconexión
```

---

## 🧹 Comandos de Limpieza

### `docker system prune`

**Estructura:** `docker system prune [OPCIONES]`

```bash
docker system prune
docker system prune -a          # Incluye todas las imágenes no utilizadas
docker system prune --volumes   # Incluye volúmenes
docker system prune -f          # No pide confirmación
```

### `docker container prune`

**Estructura:** `docker container prune [OPCIONES]`

```bash
docker container prune
docker container prune -f       # No pide confirmación
docker container prune --filter "until=24h"  # Contenedores de más de 24h
```

### `docker image prune`

**Estructura:** `docker image prune [OPCIONES]`

```bash
docker image prune
docker image prune -a           # Elimina todas las imágenes no utilizadas
docker image prune --filter "until=48h"
```

### `docker volume prune`

**Estructura:** `docker volume prune [OPCIONES]`

```bash
docker volume prune
docker volume prune -f          # No pide confirmación
```

---

## 🚀 Docker Compose

### `docker-compose up`

**Estructura:** `docker-compose up [OPCIONES] [SERVICIO...]`

```bash
docker-compose up
docker-compose up -d            # En segundo plano
docker-compose up --build       # Reconstruye las imágenes
docker-compose up web db        # Solo servicios específicos
```

### `docker-compose down`

**Estructura:** `docker-compose down [OPCIONES]`

```bash
docker-compose down
docker-compose down -v          # Incluye volúmenes
docker-compose down --rmi all   # Elimina también las imágenes
```

### `docker-compose ps`

**Estructura:** `docker-compose ps [OPCIONES] [SERVICIO...]`

```bash
docker-compose ps
docker-compose ps web           # Solo el servicio web
docker-compose ps -q            # Solo IDs
```

### `docker-compose logs`

**Estructura:** `docker-compose logs [OPCIONES] [SERVICIO...]`

```bash
docker-compose logs
docker-compose logs -f          # Sigue los logs
docker-compose logs web         # Logs de un servicio específico
docker-compose logs --tail=100 web  # Últimas 100 líneas
```

### `docker-compose exec`

**Estructura:** `docker-compose exec [OPCIONES] SERVICIO COMANDO`

```bash
docker-compose exec web bash
docker-compose exec db mysql -u root -p
docker-compose exec -T web ls  # Sin TTY
```

### `docker-compose build`

**Estructura:** `docker-compose build [OPCIONES] [SERVICIO...]`

```bash
docker-compose build
docker-compose build --no-cache web    # Reconstruye sin caché
docker-compose build --parallel        # Construcción paralela
```

---

## 💡 Ejemplos Prácticos

### Ejemplo 1: Ejecutar un servidor web Nginx

```bash
# Descargar la imagen
docker pull nginx:latest

# Ejecutar contenedor con mapeo de puertos
docker run -d -p 8080:80 --name mi-nginx nginx

# Verificar que está corriendo
docker ps

# Ver logs
docker logs mi-nginx

# Detener y eliminar
docker stop mi-nginx
docker rm mi-nginx
```

### Ejemplo 2: Desarrollo con volúmenes

```bash
# Crear un volumen para datos persistentes
docker volume create mi-datos

# Ejecutar contenedor con volumen montado
docker run -d -v mi-datos:/data --name mi-app ubuntu:20.04

# Copiar archivos al contenedor
docker cp ./mi-archivo.txt mi-app:/data/

# Acceder al contenedor
docker exec -it mi-app bash
```

### Ejemplo 3: Construir una aplicación personalizada

```bash
# Crear Dockerfile
echo "FROM nginx:alpine
COPY . /usr/share/nginx/html" > Dockerfile

# Construir imagen
docker build -t mi-web:1.0 .

# Ejecutar contenedor
docker run -d -p 3000:80 --name mi-web-app mi-web:1.0
```

---

## 📝 Notas Importantes

- Usa `docker --help` o `docker <comando> --help` para obtener ayuda específica
- Los contenedores son efímeros por defecto, usa volúmenes para persistir datos
- Siempre especifica tags en producción (evita `:latest`)
- Usa `.dockerignore` para excluir archivos innecesarios del build context
- Ejecuta `docker system prune` regularmente para mantener limpio tu sistema

---

## 🔍 Comandos de Diagnóstico

### `docker info`

**Función:** Muestra información del sistema Docker.

```bash
docker info
```

### `docker version`

**Función:** Muestra la versión de Docker.

```bash
docker version
```

### `docker system df`

**Función:** Muestra el uso de espacio en disco.

```bash
docker system df
docker system df -v  # Información detallada
```