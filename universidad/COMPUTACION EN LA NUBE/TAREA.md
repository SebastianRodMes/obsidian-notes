Usando api_data, se creo una imagen de docker, pero posteriormente ser usada.


# CONTENIDO DEL DOCKERFILE-------------
FROM python:3.11-slim

  

LABEL author="Sebas"

  

SHELL ["/bin/bash", "-c"]

  

WORKDIR /apps/api_data

  

# Crear el entorno virtual

RUN python3 -m venv apiEnv

  

# Activar el entorno virtual y actualizar pip

RUN source apiEnv/bin/activate && pip install --upgrade pip

  

# Copiar requirements.txt e instalar dependencias

COPY ./requirements.txt ./

RUN source apiEnv/bin/activate && pip install --no-cache-dir -r requirements.txt

  

# Copiar el código fuente

COPY ./src ./src

  

# Exponer el puerto

EXPOSE 5001

  

# Comando para ejecutar la aplicación con Gunicorn

CMD ["bash", "-c", "source apiEnv/bin/activate && gunicorn -w 4 -b 0.0.0.0:5001 src.api_data:app"]

  




# REQUIREMENTS.TXT--------------------
Flask==2.3.3

flask-cors==4.0.0

pymongo==4.5.0

Werkzeug==2.3.7

gunicorn==21.2.0