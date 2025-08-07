================================================================================================
Study Center....: Universidad Técnica Nacional
Campus..........: Pacífico (JRMP)
College career..: Ingeniería en Tecnologías de Información
Period..........: 2C-2025
Course..........: ITI-522 - Computación en la nube
Document........: 10 - Docker compose A.txt
Goals...........: Create an API with virtual variables.
Professor.......: Jorge Ruiz (york)
Student.........: 
================================================================================================


Note:
------------------------------------------------------------
If you are using lab equipment, skip to step #3
------------------------------------------------------------

Step 01 - Create volume to persistent data:
	
	sudo docker volume create data-postgreSQL
	
	sudo docker volume ls	(list all volumes created)


Step 02 - Download postgres image from dockerHub, latest version:

	- Download postgreSQL image:
	
		sudo docker pull postgres
	
	
	- Creates postgreSQL container and run it:
			
		sudo docker run --name postgreSQL -p 5432:5432 -e POSTGRES_PASSWORD=parda99* -v data-postgreSQL:/var/lib/postgresql/data -d postgres


	- Verify the postgreSQL container is running
	
		sudo docker ps  or  docker ps -a
		
			- You might see a message like the following:
			
				CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
				43cc6548afd8   postgres   "docker-entrypoint.s…"   9 minutes ago   Up 9 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgreSQL


	- Yupi time 01
		
		
Step 03 - Create server connection:	

	- Open pgAdmin4 in remote computer (Windows or Mac)
	
	- Register the new server connection
		
		- Right click on Servers
		
			- Register server
			
				Name......: dockDemo
				Host......: 127.0.0.01
				Port......: 5432
				User......: postgres
				Password..: parda99*
				
				
			- Save changes and connect
			
			
	- Yupi time 02
	
	
Step 04 - Create database

	- In open connection right click on Databases
	
		- Select Database / Create
		
			- General:
				- Database..........: testdb
				
			- Definition:
				- Encoding..........: utf8
				- Locale Provider...: libc
				- Collation.........: en_US.utf8
				- Character type....: en_US.utf8
				
		- Press [Save] button


	- Right click on the new database testdb and select Query tool, writes the next code:
	
		create schema if not exists principal authorization postgres;

		create table if not exists principal.tb_imagenes(
			id smallint not null,
			nombre character varying(35) collate pg_catalog."default" not null,
			imagen text collate pg_catalog."default" not null,
			mime character varying(25) collate pg_catalog."default" not null,
			constraint tb_imagenes_pkey primary key (id)
		)

		tablespace pg_default;

		alter table if exists principal.tb_imagenes	owner to postgres;
	
	
		- Press [>] Execute script button
	
	
	- Yupi
	
	
Step 05 - Create express project

	- Create a folder for you new project (you must use PowerShell or cmd terminal)
	
		mkdir apiFoto
		mkdir apiFoto/subirfoto
		
		cd apiFoto

		
	- Init express project
	
		npm init -y
		
		- you can see a message like this:
		
			{
			  "name": "apiFoto",
			  "version": "1.0.0",
			  "description": "",
			  "main": "index.js",
			  "scripts": {
				"test": "echo \"Error: no test specified\" && exit 1"
			  },
			  "keywords": [],
			  "author": "",
			  "license": "ISC"
			}
			
			
	- Install tools required (express, postgreSQL and other required libraries)
	
		npm install express http-errors body-parser pg cors multer dotenv --save
		
		
Setp 06 - Open the project folder with your favorite IDE
	
	- Create <root project>/index.js file
	
	- Open <root project>/index.js file and writes:
	
		// call external libraries
		const express = require('express');
		const bodyParser = require('body-parser');
		var cors = require('cors')
		const fs = require('node:fs');
		const { Pool } = require('pg');
		require('dotenv').config();

		// create a middleware to upload files
		const multer = require("multer");
		const subir = multer({ dest: "subirfoto/" });

		// create an instance of express
		const app = express();
		const port = 5000;

		// create a pool of connections to the database
		const pool = new Pool({
			host: 'localhost',
			database: 'testdb',
			user: 'postgres',
			password: 'parda99*',
			port: 5432,
		});

		// Use CORS
		app.use(cors());

		// parse application/x-www-form-urlencoded
		app.use(bodyParser.urlencoded({ extended: true }));

		// parse application/json
		app.use(bodyParser.json());

		// create listener for GET requests to the root URL
		app.get('/', (req, res) => {
			salida = {
				status_code: 200,
				status_message: 'OK',
				content: {
					mensaje: 'Hola Mundo',
					autor: 'Jorge Ruiz (york)',
					fecha: new Date(),
					entorno: process.env
				}
			}
			res.status(200).json(salida);
		});

		// function to convert image to Base64
		function getBase64Image(filePath) {
			const image = fs.readFileSync(filePath);
			return Buffer.from(image).toString('base64');
		}

		app.get('/imagenes/:id', async function obtenerImagenes(req, res) {
			// create the query
			const query = 'SELECT * FROM principal.tb_imagenes where id = $1';
			const id  = req.params.id;
			const values = [id];

			// execute the query
			try {
				const result = await pool.query(query, values);
				salida = {
					status_code: 200,
					status_message: 'OK',
					content: {
						resultado: result.rows
					}
				}
				res.status(200).json(salida);
			} catch (error) {
				salida = {
					status_code: 500,
					status_message: 'Internal Server Error',
					content: {
						error: error.toString()
					}
				}
				res.status(500).json(salida);
			}
		});

		app.get('/foto/:id', async function obtenerImagenes(req, res) {
			// create the query
			const query = 'SELECT imagen, mime FROM principal.tb_imagenes where id = $1';
			const id  = req.params.id;
			const values = [id];

			// execute the query
			try {
				encontro = false;
				const result = await pool.query(query, values);
				result.rows.forEach(row => {
					const image = Buffer.from(row.imagen, 'base64');
					res.writeHead(200, {
						'Content-Type': row.mime,
						'Content-Length': image.length
					});
					encontro = true;
					res.end(image);
				});
			} catch (error) {
				salida = {
					status_code: 500,
					status_message: 'Internal Server Error',
					content: {
						error: error.toString()
					}
				}
				res.status(500).json(salida);
			}
		});

		// create listener for POST requests to the /imagenes URL
		app.post('/imagenes', subir.array("imagen", 1), async function subirFoto(req, res) {
			try {
				// get the values from the request body
				const { id, nombre } = req.body;
				var imagen = getBase64Image(req.files[0].path);
				var mime = req.files[0].mimetype;

				// create the query and values
				const query = 'INSERT INTO principal.tb_imagenes (id, nombre, imagen, mime) VALUES ($1, $2, $3, $4) RETURNING *';
				const values = [id, nombre, imagen, mime];
				const result = await pool.query(query, values);

				salida = {
					status_code: 200,
					status_message: 'OK',
					content: {
						resultado: result
					}
				}
				// delete the file
				fs.unlinkSync(req.files[0].path);

				// return the result
				res.status(200).json(salida);
			} catch (error) {
				salida = {
					status_code: 500,
					status_message: 'Internal Server Error',
					content: {
						error: error.toString()
					}
				}
				// delete the file
				fs.unlinkSync(req.files[0].path);

				// return the error
				res.status(500).json(salida);
			}
		});

		// run server
		app.listen(port, () => {
			console.log(`Servidor escuchando en http://localhost:${port}`);
		});
	
	
	- Save changes


Step 07 - Testing the API

	- In the IDE terminal writes and exectues:
	
		node index.js
		
	- Open the Postman application and create a new collection:	
	
		- Datos generales...:	GET		http://<ip_address>/
		
		- Subir imagen......:	POST	http://<ip_address>/imagenes

			- Body -> form-data
			
				id		(text)
				nombre	(text)
				imagen	(file)
				
		- Datos de la imagen:	GET		http://<ip_address>/imagenes/#

		- Obtener imagen....:	GET		http://<ip_address>/foto/#	
	
	
	- Yupi
	

Step 08 - Add environment variables

	- Create .env file into the <root_project> and writes:
	
		HOST=127.0.0.1
		DATABASE=testdb
		USER=postgres
		PASS=parda99*
	
	- Save changes

	- Update the index.js in the connection section.
	
		- Replace the current connection configuration with the next code:
		
			const pool = new Pool({
				host: process.env.HOST,
				database: process.env.DATABASE,
				user: process.env.USER,
				password: process.env.PASS,
				port: 5432,
			});

	- Save changes

	- Please repeat the step 07
	
	- Yupi


Step 09 - Create Dockerfile

	- Change the name of file .env with .env.old

	- Create Dockerfile file into the <root_project> and writes:
	
		FROM node:latest
		LABEL authors="jruiz"

		RUN mkdir -p /apps/api
		RUN mkdir -p /apps/api/subirfoto

		WORKDIR /apps/api

		COPY package.json /apps/api/
		RUN npm install express http-errors body-parser pg cors multer dotenv --save
		COPY . /apps/api/

		ENV HOST=127.0.0.1
		ENV DATABASE=testdb
		ENV USER=postgres
		ENV PASS=parda99*

		EXPOSE 5000
		CMD ["node", "index.js"]


	- Save changes


Step 10 - Create a Docker Container image	

	- Build image
	
		docker build -t dangeliza/api-image:v1.0.7 .
		
		
	- Create image tag
	
		docker tag dangeliza/api-image:v1.0.7 dangeliza/api-image
		
		
	- Publish the new image in Docker Hub

		docker push dangeliza/api-image
		
		
	- Delete the builder cache 

		docker builder prune -a


Step 11 - Install API-Rest service

	docker pull dangeliza/api-image		(only if required)
	
	docker run --name api-mage -p 5000:5000 -d dangeliza/api-image 
	
	
	- Testing from your work machine using Postman or another 
	  API-Rest client tool
		
		
	Notes:	
	
		-e HOST=127.0.0.1		(changes the postgreSQL ip_address)
		-e DATABASE=testdb		(changes the name of database)
		-e USER=merlina			(changes the postgreSQL user name)
		-e PASS=parda99*		(changes the postgreSQL user password)
	
	
	- Yupi 

