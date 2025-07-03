***ANTES DE SUBIRLO***
DEBEMOS ENTENDER QUE VA DENTRO DE REQUIREMENTS.TXT Y EN DOCKER FILE: 

Cuando se sube un proyecto como una **Docker image** a **Docker Hub**, los _requirements_ o dependencias que debes incluir dependen del tipo de aplicación que se estan conteniendo dentro de la imagen. Generalmente, esos requisitos son los mismos que mi aplicación necesita para ejecutarse correctamente en cualquier entorno.

 Guía clara y rápida para saber **qué requirements incluir y cómo hacerlo bien**:

---

### ✅ 1. **Si el proyecto es en Python**

Debe incluir un archivo `requirements.txt` con todas las dependencias.

**Ejemplo:**

txt



`fastapi==0.110.0 uvicorn==0.29.0 requests`

Y en el `Dockerfile`, lo referimos así:

dockerfile



`COPY requirements.txt . RUN pip install --no-cache-dir -r requirements.txt`

>  Si no se tiene un `requirements.txt`, se puede generarlo con:





`pip freeze > requirements.txt`

---

### ✅ 2. **Si el proyecto es en Node.js (JavaScript)**

  tener un `package.json` (y opcionalmente `package-lock.json`) con las dependencias listadas.

**Dockerfile:**



`COPY package*.json ./ RUN npm install`

CREATE NEW DOCKER IMAGE AND DOCKER CONTAINER
------------------------------------------------------------

Step 12 - Create new application with PyCharm or Visual Code
		- venv	(don't need it to create image)
		- create folder src


Step 13 - Donwload python code into src folder 
	
		- http://demoyork.com/main.py and store into the work directory
	
	
Step 14 - Create file requirements.txt into the root project

	- write the next sentences

		flask
		flask_cors
		wheel
		uwsgi
		
		
Step 15 - Create Dockerfile into the root project

	- write the next commands
	
		FROM python
		LABEL author="Jorge Ruiz (york)"

		SHELL [ "/bin/bash", "-c" ]

		WORKDIR /apps/api_Demo_01
		RUN python3 -m venv apiEnv
		RUN source apiEnv/bin/activate

		COPY ./requirements.txt ./
		RUN pip install --no-cache-dir -r requirements.txt

		COPY ./src ./src
		EXPOSE 5001
		CMD ["uwsgi","-w","src.main:app","--socket","0.0.0.0:5001", "--protocol","http"]


Step 16 - Create the local testing

	-- Create local image 
		docker build -t apidemo01-image .

	-- Create container with image
		docker run --name api_demo_01 -p 5001:5001 -d apidemo01-image
		
	- Use webBrowser for to test your new container
	
		http://localhost:5001
		
		http://localhost:5001/estudiante


	- Using Docker desktop, stop container api_demo_01
	
	- Delete container api_demo_01 and 
	
	- Delete image dangeliza/apidemo01-image

Create docker image to publish (remember apply the required changes)

	-- Docker Hub connection	
		docker login
			- Username:
			- Password:
			
		(or usign Docker desktop)	

		
	-- Create image 
		docker build -t dangeliza/apidemo01-image:v1.0.0 .	


	-- Create image tag to use in Docker Hub
		docker tag dangeliza/apidemo01-image:v1.0.0 dangeliza/apidemo01-image


	-- Upload image to Docker Hub
		docker push dangeliza/apidemo01-image

	 
	-- Create container with image
		docker run --name api_demo_01 -p 5001:5001 -d dangeliza/apidemo01-image


	-- Drop container
		docker container rm api_demo_01


	-- Drop image
		docker image rm dangeliza/apidemo01-image}