

ssh -i C:\Users\sr331\.ssh\examen01_key.pem sebas@20.115.92.237

# Guía : Configurar API-PDF como Servicio Linux

## ¿Qué vamos a hacer?

Vamos a tomar la API de FastAPI que procesa boletas de matrícula en PDF y convertirla en un servicio normal de Linux que:

- Se ejecute automáticamente al encender el servidor
- Se comunique con NGINX a través de un socket UNIX
- Sea accesible desde cualquier navegador web

Socket UNIX: Es como un "tubo" directo entre NGINX y la aplicación dentro del mismo servidor, mucho más rápido que usar puertos de red.

---

## Paso 1: Preparar el Sistema

### 1.1 Actualizar e instalar lo básico

```bash
# Actualizamos el sistema (siempre buena práctica)
sudo apt update && sudo apt upgrade -y

# Instalamos Python, NGINX y herramientas básicas
sudo apt install python3 python3-pip python3-venv git nginx -y
```

Qué hace esto? Instala Python (el lenguaje de programación que se usa en la api), NGINX que es el servidor web que manejará las peticiones y Git para descargar código.

---

## Paso 2: Crear Usuario y Carpetas

### 2.1 Usuario dedicado (seguridad)

```bash
# Creamos un usuario especial solo para nuestra aplicación
sudo useradd -r -s /bin/false apipdf
```

Por qué?** Por seguridad. Si alguien intenta "hackear" aplicación, solo tendrá acceso a este usuario limitado, no a todo el sistema.

### 2.2 Estructura de carpetas

```bash
# Creamos las carpetas donde vivirá nuestra aplicación
sudo mkdir -p /opt/apipdf/{app,logs,temp,uploads}
sudo mkdir -p /var/run/apipdf

# Le damos permisos al usuario apipdf
sudo chown -R apipdf:apipdf /opt/apipdf
sudo chown -R apipdf:apipdf /var/run/apipdf
```

Qué hace esto?**

- `/opt/apipdf/app`: el código va aquí
- `/opt/apipdf/logs`: Los registros de errores y actividad
- `/opt/apipdf/temp`: Archivos temporales
- `/var/run/apipdf`: Aquí se creará el socket UNIX

---

## Paso 3: Instalar tu Aplicación

### 3.1 Descargar el código

```bash
cd /opt/apipdf
# Descargamos la API 
sudo -u apipdf git clone https://github.com/WEB-ll-2025/api_PDF.git app
```

### 3.2 Crear entorno virtual de Python

```bash
# Creamos un "ambiente aislado" para las librerías de Python
sudo -u apipdf python3 -m venv /opt/apipdf/venv

# Instalamos las dependencias que necesita tu API
sudo -u apipdf /opt/apipdf/venv/bin/pip install fastapi uvicorn pypdf2 python-multipart
```

Por qué entorno virtual?** Para evitar conflictos entre las versiones de librerías de diferentes proyectos.

### 3.3 Probar que funciona

```bash
# Verificamos que el main.py se puede importar
cd /opt/apipdf/app
sudo -u apipdf /opt/apipdf/venv/bin/python -c 'from main import app; print("API funciona")'
```

---

## Paso 4: Convertir en Servicio de Linux

### 4.1 Crear archivo de servicio

Creamos el archivo `/etc/systemd/system/apipdf.service`:
sudo nano /etc/systemd/system/apipdf.service

```ini
[Unit]
Description=API-PDF para procesar boletas de matrícula
After=network.target

[Service]
Type=simple
User=apipdf
Group=apipdf
WorkingDirectory=/opt/apipdf/app
Environment=PATH=/opt/apipdf/venv/bin

# Aquí está la magia: usamos socket UNIX en lugar de puerto
ExecStart=/opt/apipdf/venv/bin/uvicorn main:app \
    --uds /var/run/apipdf/apipdf.sock \
    --workers 2 \
    --log-level info

# Si se cae, que se reinicie automáticamente
Restart=always
RestartSec=5

# Configuración de seguridad básica
NoNewPrivileges=yes
ProtectHome=yes

[Install]
WantedBy=multi-user.target
```

**¿Qué significa esto?**

- `--uds /var/run/apipdf/apipdf.sock`: Crea el socket UNIX en esta ubicación
- `--workers 2`: Usa 2 procesos para manejar peticiones
- `Restart=always`: Si se cae, se reinicia solo

### 4.2 Activar el servicio

```bash
# Le decimos a Linux que reconozca nuestro nuevo servicio
sudo systemctl daemon-reload

# Lo habilitamos para que arranque automáticamente
sudo systemctl enable apipdf

# Lo iniciamos ahora
sudo systemctl start apipdf

# Verificamos que esté corriendo
sudo systemctl status apipdf
```

### 4.3 Verificar el socket

```bash
# Debe aparecer un archivo tipo socket (comienza con 's')
ls -la /var/run/apipdf/
# Esperamos ver: srw-rw---- 1 apipdf apipdf 0 fecha apipdf.sock
```

---

## Paso 5: Configurar NGINX como Proxy

### 5.1 Configuración del sitio

Creamos `/etc/nginx/sites-available/apipdf`:
sudo nano /etc/nginx/sites-available/apipdf
```nginx
# Definimos dónde está nuestro backend (la API)
upstream apipdf_backend {
    server unix:/var/run/apipdf/apipdf.sock;
}

server {
    listen 80;
    server_name localhost;
    
    # Configuración para archivos PDF grandes
    client_max_body_size 25M;
    proxy_read_timeout 120s;
    
    # Ruta principal - muestra la documentación de la API
    location / {
        proxy_pass http://apipdf_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    # Endpoint específico para tu boleta de matrícula
    location /boletamatricula {
        proxy_pass http://apipdf_backend/boletamatricula;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Configuración especial para subir archivos
        client_max_body_size 25M;
        proxy_request_buffering off;
    }
    
    # Documentación interactiva de FastAPI
    location /docs {
        proxy_pass http://apipdf_backend/docs;
        proxy_set_header Host $host;
    }
}
```

**¿Qué hace esto?**

- NGINX recibe peticiones HTTP normales en el puerto 80
- Las redirige a la aplicación a través del socket UNIX
- Configura límites apropiados para archivos PDF
- Expone automáticamente la documentación de FastAPI en `/docs`

### 5.2 Activar la configuración

```bash
# Verificamos que la configuración esté bien escrita
sudo nginx -t

# Habilitamos nuestro sitio
sudo ln -s /etc/nginx/sites-available/apipdf /etc/nginx/sites-enabled/

# Deshabilitamos el sitio por defecto (opcional)
sudo rm -f /etc/nginx/sites-enabled/default

# Reiniciamos NGINX
sudo systemctl reload nginx
```

---

## Paso 6: Configurar Permisos

### 6.1 Permitir que NGINX acceda al socket

```bash
# Agregamos el usuario de NGINX (www-data) al grupo de la aplicación
sudo usermod -a -G apipdf www-data

# Verificamos que se agregó correctamente
groups www-data
```

**¿Por qué?** NGINX necesita permisos para leer/escribir en el socket de la aplicación.

---

## Paso 7: Probar que todo funciona

### 7.1 Verificar servicios

```bash
# Ambos deben mostrar "active (running)"
sudo systemctl status apipdf
sudo systemctl status nginx
```

### 7.2 Probar la API

```bash
# Prueba básica - debe mostrar información de FastAPI
curl http://localhost/

# Acceder a la documentación 
curl http://localhost/docs
```

### 7.3 Probar con archivo PDF

```bash

http://20.115.92.237/docs
```

---

## Comandos Útiles para Administrar

### Gestión del servicio

```bash
# Ver logs en tiempo real (útil para debug)
sudo journalctl -u apipdf -f

# Reiniciar si cambias algo en el código
sudo systemctl restart apipdf

# Ver estado detallado
sudo systemctl status apipdf -l
```

### Gestión de NGINX

```bash
# Verificar configuración antes de aplicar cambios
sudo nginx -t

# Recargar configuración sin interrumpir el servicio
sudo systemctl reload nginx

# Ver logs de acceso
sudo tail -f /var/log/nginx/access.log
```

---



## Qué se logra con esto?

1. ****: La API ahora es un servicio del sistema que se inicia automáticamente
2. ****: Comunicación directa entre NGINX y la app sin overhead de red :D
3. : Usuario dedicado con permisos limitados, fue un tema mas de seguridad por practica, que por otra cosa xd
4. : NGINX puede manejar múltiples conexiones simultáneas
5. ****: Logs integrados con el sistema de Linux



# PARTE 2 
Utilizando la misma aplicación dada por su profesor para este examen, usted deberá de crear una segunda guía que permita identificar los pasos para: 
1. Crear una imagen Docker desde tu entorno local (Computadora personal o equipo de laboratorio) y que a la vez esta sea publicada mediante Microsoft Azure Container Registry. 


![[Pasted image 20250701185812.png]]

Dockerfile: 
FROM python

        LABEL author="Sebastian Rodriguez"

  

        SHELL [ "/bin/bash", "-c" ]

  

        WORKDIR /opt/apipdf/{app,temp}

        RUN python3 -m venv apiEnv

        RUN source apiEnv/bin/activate

  

        COPY ./requirements.txt ./

        RUN pip install --no-cache-dir -r requirements.txt

  

        COPY ./src ./src

        EXPOSE 5025

        CMD ["uvicorn","-w","src.main:app","--socket","0.0.0.0:5025", "--protocol","http"]

requirements.txt:
fastapi
uvicorn
PyPDF2
python-multipart
Luego: 
PS C:\Users\sr331\OneDrive\Escritorio\api_PDF> docker build -t seb4s/apipdf-image:v1.0.0 . 
PS C:\Users\sr331\OneDrive\Escritorio\api_PDF> docker tag seb4s/apipdf-image:v1.0.0 seb4s/apipdf-image
PS C:\Users\sr331\OneDrive\Escritorio\api_PDF> docker push seb4s/apipdf-image 
![[Pasted image 20250701192834.png]]


2. Crear una aplicación mediante la plataforma de Microsoft Azure Container App, donde pueda utilizar la imagen creada en el punto anterior 
3. aca fue donde me meti, para seleccionar la imagen que ya estaba subida al container registry

4. ![[Pasted image 20250702193238.png]]

![[Screenshot 2025-07-02 192826.png]]
![[Pasted image 20250701194105.png]]
PS C:\Users\sr331\OneDrive\Escritorio\api_PDF> docker login  docker tag apipdf-image imagesseb4s.azurecr.io/apipdf-image:latest
docker: 'docker login' requires at most 1 argument

Usage:  docker login [OPTIONS] [SERVER]

SRun 'docker login --help' for more information
PS C:\Users\sr331\OneDrive\Escritorio\api_PDF> docker login imagesseb4s.azurecr.io
Username: imagesseb4s
Password:

Login Succeeded
PS C:\Users\sr331\OneDrive\Escritorio\api_PDF> docker push imagesseb4s.azurecr.io/apipdf-image:latest

https://apipdf.lemondesert-84e4e1d2.westus2.azurecontainerapps.io/docs
![[Pasted image 20250702192946.png]]