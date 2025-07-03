
================================================================================================
Study Center....: Universidad Técnica Nacional
Campus..........: Pacífico (JRMP)
College career..: Ingeniería en Tecnologías de Información
Period..........: 2C-2025
Course..........: ITI-522 - Computación en la nube
Document........: 09 - Azure Container Apps and Docker Registry
Goals...........: Use Azure Container infrastructure
					- Create Container Apps (from Docker.io).
					- Create Container Registry.
					- Publish an own image from local machine 
					- Resolve the practice
Professor.......: Jorge Ruiz (york)
Student.........: 
================================================================================================

Step 01 - Open Microsoft Azure Portal

	Go to: https://portal.azure.com/  and sign in with your account


Step 02 - Create Container Apps (from Docker.io).

	- When your personal pane is open, writes Container Apps 
	  into the search bar.
	  
	- Press [Create] option menu and select Container App
	
		- Basics:

			- Project details:
			
				- Subscription............: Azure for Students
				
				- Resource group..........: use your resources environment (e.g: rscYork)
				
				- Container app name......: apidemo01
				
				- Optimize Az. functions..: [ ]
				
				- Deployment resource.....: Container image
				
				
			- Container Apps environment:	
			
				- Show envs in all regions..: [ ]
				
				- Region....................: select the last used region (e.g: Central US)
				
				- Container Apps environment: use default
				
				
			- Press [Next: Container >] button


		- Container:
		
			- Use quickstart image.......: [ ]
			
			- Container details:
			
				- Name...................: apidemo01
				
				- Image source...........: Docker Hub or other registries
				
				- Image type.............: public
				
				- Registry login server..: docker.io
				
				- Image and tag..........: dangeliza/apidemo01-image  (replace dangeliza for your account)
				
				- Command override.......: [ ] (don't changes)
				
				- Arguments override.....: [ ] (don't changes)
				
				
			- Development stack-specific features:

				- Development stack......: unspecified
				
				- Workload profile.......: Consumption - UP to 4 vCPUs, 8 Gb memory
				
				- CPU and memory.........: 0.5 CPU cores, 1 GB memory
				
				
			- Environment variables:

				- Not required 
				
				
			- Press [Next: Ingress >] button
			
			
		- Application ingress settings

			- Enable ingress for applications that need an HTTP or TCP endpoint.
			
				- Ingress....................: Enabled [X]
				
				- Ingress traffic............: Accepting traffic from anywhere
				
				- Ingress type...............: HTTP
				
				- Transport..................: Auto
				
				- Insecure connections.......: Allowed
				
				- Target port................: 5001
				
				- Session affinity...........: [ ]
				
				- Not required additional ports
			
			
			- Press [Next: Tags >] button
			
				Name           | Value     | Resource
				---------------+-----------+------------------------				
				Virtualization | vmTest-01 | All resources selected 


			- Pres [Next: Review + create >] button
				
				- When passed is done, press [Create] button
				
					- This process take awhile (3 minutes)
					
					
			- Press [Go to resource] button		
					
			- Copy the Application Url and testing in your favorite web browser or 
			  use Postman application
			  			  
			  
	- if you have reached this point without problems congratulations:

		YUPI time

			
Step 03 - Create Microsof Azure Container Registry.
	
	- When your personal pane is open, writes Container Resgistries 
	  into the search bar


	- Press [Create] option menu and select Container Registries
	
		- Basics:

			- Project details:
			
				- Subscription............: Azure for Students
				
				- Resource group..........: use your resources environment (e.g: rscYork)
				
				
			- Instance details	
				
				- Registry name...........: imagesyork
				
				- Location................: select the last used region (e.g: Central US)
				
				- Domain name label scope.: Unsecure
				
				- Registry domain name....: imagesyork.azurecr.io
				
				- Uses availability zonee.: [ ]
				
				- Pricing plan............: Basic
				
				- Role assigment 
				  permissions mode........: RBAC Registry Permissions
		
		
		- Press [Next: Networking >] button	

			- Don't changes required
			
			
		- Press [Next: Encryption >] button	

			- Don't changes required


		- Press [Next: Tags >] button
			
				Name           | Value     | Resource
				---------------+-----------+------------------------				
				Virtualization | vmTest-01 | Container registry


		- Pres [Next: Review + create >] button
				
				- Press [Create] button
				
					- This process take awhile (2 minutes)
					
					
		- Press [Go to resource] button	

		- Into the new Container Registry resource (in my case: imagesyork)
		
			- Copy Login Server: (e.g: imagesyork.azurecr.io)
			
			
Step 04 - Publish your Docker Images in Microsof Azure Container Registry.

	- Select the Settings option in the left pane
	
		- Select Access keys
		
			Enable Admin User...[X] (this process take awhile)
			
			You can see the user and password
			

	- Open the PowerShell in your computer
	
	- Execute the next command:
	
		docker images
		
			try to locate:
			
				REPOSITORY			TAG		IMAGE ID		CREATED			SIZE
				apidemo01-image		latest	137ddb17be84	13 days ago		1.05GB
				
				
		docker tag apidemo01-image imagesyork.azurecr.io/apidemo01-image:latest	
			
		docker images (you can see the changes)
		
		
		- Push the image in azure:			
		
			docker login imagesyork.azurecr.io
			
			Username:   
			
			Password:	(the pass is totally hidden)
			
			if login successfuly:
			
				docker push imagesyork.azurecr.io/apidemo01-image:latest
				
				
	- Go back to Azure platform and select Services / Repositories	
			
			
Step 05 - Practice

	- Create instance "Docker Container" of the MongoDB as Microsoft Docker Apps
	
	- Create a new image of apiData_01 into container registry
	
	- Create apiData_01 as docker container using the apiData_01 imgage from ACR.
