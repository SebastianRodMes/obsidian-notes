========================================================================================
Centro....: Universidad Técnica Nacional
Sede......: Pacífico
Carrera...: Tecnologías de Información
Curso.....: ITI-522 - Computación en la Nube
Periodo...: 2-2025
Documento.: Implementación de API-Restful creado con JavaScript sobre el entorno de 
			Digital Ocean con acceso a base de datos con MongoDB
Profesor..: Jorge Ruiz (york)
========================================================================================

01 - Use Digital Ocean

	https://cloud.digitalocean.com/login
	
	Login in or maybe you need to register an account
	

02 - In left pane, select Databases

		- Create Database (MongoDB)
		
			- Choose a datacenter region	
			
				- New York - Datacenter 3 - NYC3	
					
					
			- Choose a database engine	
			
				- MongoDB v8.0
				
						
			- Select Basic plan ($15/mo / 1 vCPU / 1GB RAM / 15BG SSD)
			
			
		- Press Create Database Clusters
		
		
03 - Select de new database instance

		- In the connection details block
		
			- Choose Public network option
			- Choose Connection string option from comboBox
			
			- Copy the connection string, keep password secure.
			
				e.g.
				
				mongodb+srv://doadmin:<your password>@db-mongodb-nyc1-22786-4e032f46.mongo.ondigitalocean.com/
			    -------------+-------+---------------+------------------------------+------------------------+
                Driver       | User  | Password      | Database instance            | Engine on platform 
				
				
			The password is hidden and in short time is visible
			

04 - Download Article Express application from Campus Virtual and unzip


05 - Open your favorite development IDE (Visual Code)

	- Open terminal and execute

		npm install
		
		npm install express http-errors body-parser mongoose cors --save


04 - Change Article Express connection 

		- The current connection sentence is:
		
			mongoose.connect('mongodb://admin:parda99*@10.236.2.142:27017/',{dbName:'dbArticles'});
			
			
		- Replace with:

			async function connect() {
				await mongoose.connect('mongodb+srv://doadmin:<your password>@<your mongoDB instance>.mongo.ondigitalocean.com/',
									    {dbName: 'dbArticles', 
										 useNewUrlParser: true, 
										 useUnifiedTopology: true},
									    error => error ? console.log(error) : console.log('Connected to MongoDB'));
			}
			
			connect().catch(error => console.log(error));
		
		
05 - Run local application and testing with Postman	

		https://documenter.getpostman.com/view/141107/UVXkoFU4
		
		
06 - Connect with Mongo Compass, using the string connection

		- Maybe need to add trusted sources into MongoDB instance in Digital Ocean Platform
		
			- Select Settings tab
			
				- Edit sources
				
					- Add IPs or Apps service
				
		- Try the open connection with Mongo Compass
		

07 - All run without problems

		You are happy and living in the Y U P I time
	
	
08 - Create docker image (remember apply the required changes)

	- Docker Hub connection
		docker login
			- Username:
			- Password:

		
	- Create image	
		docker build -t dangeliza/articlexpress-image:v2 .
		
		
	- Create proyect tag to use in Docker Hub
		docker tag dangeliza/articlexpress-image:v2 dangeliza/articlexpress-image
		
	
	- Upload proyect to Docker Hub
		docker push dangeliza/articlexpress-image
		
	
	- Clear the Docker Cache used to build images 
		docker builder prune -a
	
		
09 - In left pane, select Apps


10 - Focus on Code, not servers

		- Click on Create App
	
	
11 - Select resources 

		- Select Docker Hub
		
			Repository...: dangeliza/articlexpress-image
			Tag..........: latest
	
	
		- Click Next button	
	
			- Customize your resource, press Edit button
			
				- Name..................: articlexpress
				- HTTP-Port.............: 5005	
				- HTTP Request Routes...: only root /
				
				press Back button
				
				
			- Customize your plan, press Edit Plan button	
			
				- Select Basic (Best for prototypes)
				- Select Size combo box
					- Select choice $5.00/mo-Basic (512 MB RAM - 1 vCPU)
					
				press Back button

		
		- Click Next button	(Environment variables)
			
			
		- Click Next button (App info)
		
		
		- Click Next button (Review)
		
		
		- Click on Create Resources button
		
		
		- After the applicaticon was created
		
			- Click on Create button
			
				- Select Create/Attach Database
				
					- Select Previously Created Digital Ocean Database
					
					- Attach Database
					
				
12 - In left pane, select Databases	

		- Select your mongoDB instance
		
		- Select Settings tab
			
			- Edit Trusted sources
			
				- Add your application (Article Express)
		
		
13 - In left pane, select Apps

		- Select the Articlexpress application

		- Click Actions button
		
			- Select Force Rebuild and Deploy
			
				- Select Clear Build Cache option
				- Press Deploy Button
				
				
		- Testing your application with Postman, remeber your address.		


14 - All run without problems

		You are happy and living in the Y U P I time again
