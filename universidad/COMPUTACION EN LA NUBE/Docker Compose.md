================================================================================================
Study Center....: Universidad Técnica Nacional
Campus..........: Pacífico (JRMP)
College career..: Ingeniería en Tecnologías de Información
Period..........: 2C-2025
Course..........: ITI-522 - Computación en la nube
Document........: 10 - Docker compose B.txt
Goals...........: Create Docker Compose with:
                     - virtual variables.
					 - postgreSQL engine
					 - local network
Professor.......: Jorge Ruiz (york)
Student.........: 
================================================================================================

Run this codes in your windows machine 

Step 01 - Create a Docker compose file

	- Locate in your work directory create file

	- Create a new work project (only if required)
	
	- Create a file called docker-compose.yml and writes the next sentences:		

		services:
		  db:
			image: postgres:13
			volumes:
			  - postgres_data:/var/lib/postgresql/data
			environment:
			  POSTGRES_USER: postgres
			  POSTGRES_PASSWORD: parda99*
			  POSTGRES_DB: testdb
			ports:
			  - "5432:5432"	
			restart: always

		volumes:
		  postgres_data:
		  
		  
	- Save changes	  
	
	- Testing composer file:
	
		docker-compose up -d
		
		
	- Open the Docker Desktop and locate the running service into the docker container called as same work project name.
	
	- Open the pgAdmin to connect with the postgreSQL service
	  
	  
Step 02 - Delete all created components from the Docker compose file

	- Delete container
	- Delete image
	- Delete volunme
	
	
Step 03	- Update the Docker Compose file to create the database

	- Create init.sql file in <root_directory> and writes:
	
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

	- Save changes
	
	- Update docker-compose file:
	
		services:
		   db:
			  image: postgres:13
			  container_name: postgres_db      
			  environment:
				 POSTGRES_USER: postgres
				 POSTGRES_PASSWORD: parda99*
				 POSTGRES_DB: testdb
			  ports:
				 - "5432:5432"
			  volumes:
				 - ./init.sql:/docker-entrypoint-initdb.d/init.sql
				 - postgre_data:/var/lib/postgresql/data
			  command: ["postgres", "-c", "log_statement=all"]		 
			  restart: always      

		volumes:
		   postgre_data:		
	
	
	- Save changes
	
	- Testing composer file:
	
		docker-compose up -d
		
		
	- Open the Docker Desktop and locate the running service into the docker container called as same work project name.
	
	- Open the pgAdmin to connect with the postgreSQL service.
	
	- You can see now the our database created with their required tables.
	

Step 04 - Repeat the step 02.


Step 05	- Update the Docker Compose file to add the API-Rest:

	- Update docker-compose file:

		services:
		   db:
			  image: postgres:13
			  container_name: postgres_db      
			  environment:
				 POSTGRES_USER: postgres
				 POSTGRES_PASSWORD: parda99*
				 POSTGRES_DB: testdb
			  ports:
				 - "5432:5432"
			  volumes:
				 - ./init.sql:/docker-entrypoint-initdb.d/init.sql
				 - postgre_data:/var/lib/postgresql/data
			  command: ["postgres", "-c", "log_statement=all"]
			  networks:
				 - micompose_network
			  restart: always      

		   api:
			  image: dangeliza/api-image:latest
			  container_name: apiFoto
			  environment:
				 HOST: db
				 DATABASE: testdb
				 USER: postgres
				 PASS: parda99*
			  ports:
				 - "5000:5000"
			  depends_on:
				 - db	  
			  networks:
				 - micompose_network
			  restart: always      

		networks:
		   micompose_network:
			  driver: bridge

		volumes:
		   postgre_data:

	- Save changes
	
	- Testing composer file:
	
		docker-compose up -d