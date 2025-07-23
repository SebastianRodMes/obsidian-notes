Aquí tienes los comandos básicos para descargar y ejecutar una imagen de Docker Hub:

## Comandos esenciales

**1. Descargar una imagen:**

```bash
docker pull nombre_imagen:tag
```

Ejemplo:

```bash
docker pull nginx:latest
```

**2. Ejecutar un contenedor:**

```bash
docker run nombre_imagen:tag
```

**3. Ejecutar en modo interactivo y con terminal:**

```bash
docker run -it nombre_imagen:tag
```

**4. Ejecutar en segundo plano (modo daemon):**

```bash
docker run -d nombre_imagen:tag
```

**5. Mapear puertos (host:contenedor):**

```bash
docker run -p 8080:80 nginx:latest
```

**6. Asignar un nombre al contenedor:**

```bash
docker run --name mi_contenedor nginx:latest
```

## Comandos útiles adicionales

**Ver imágenes descargadas:**

```bash
docker images
```

**Ver contenedores en ejecución:**

```bash
docker ps
```

**Ver todos los contenedores:**

```bash
docker ps -a
```

**Detener un contenedor:**

```bash
docker stop nombre_contenedor
```

**Eliminar un contenedor:**

```bash
docker rm nombre_contenedor
```

## Ejemplo completo

```bash
# Descargar la imagen
docker pull nginx:latest

# Ejecutar el contenedor con puerto mapeado y nombre
docker run -d -p 8080:80 --name mi_nginx nginx:latest
```

¿Hay alguna imagen específica que quieras usar o necesitas ayuda con algún comando en particular?