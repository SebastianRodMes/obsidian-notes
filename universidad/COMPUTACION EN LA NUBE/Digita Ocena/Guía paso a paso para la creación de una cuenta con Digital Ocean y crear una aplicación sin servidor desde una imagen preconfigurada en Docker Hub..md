
===============================================================================================
Institute....: Universidad Técnica Nacional
Campus.......: Pacífico
Career.......: Tecnologías de Información
Course.......: ITI-522 - Computación en la Nube
Period.......: 2-2025
Document.....: Docker and Digital Ocean.txt
Goals........: Create Digital Ocean environment
                - Create Droplets using the Linux operating system.
					- Install Docker environment
					- Deploy and launch containers
					
				- 	
				- Create a single React application (only for testing)
				- Update Visual Studio Code
				- Connect the developer tool with remote React project
Professor....: Jorge Ruiz (york)
Student......:
===============================================================================================

Step 01 - Use Digital Ocean Platform

	- Login in (in case you haves an account)
	
		https://cloud.digitalocean.com/login
		
	
	- Or maybe you need to register an account, press Sign_Up option in right-upper corner
	
		- Try DigitalOcean with a $200 credit. You will receive a $200 credit 
		  (good for 60 days) when you create a new account on Digital Ocean
		
			https://www.digitalocean.com/try/developer-brand?utm_content=
			
			https://cloud.digitalocean.com/registrations/new?refcode=f6fcd01aaffb
		
		
		- Create account using email & password
		
			email......: ??????@est.utn.ac.cr
			password...: ***********
			
			press [Create free account] button
			
			
		- Validate that you're not a bot (resolves the image)
		
		- Confirm your email address, you must to go at your inbox email
		
			- Press link to redirect you to Digital Ocean to finish the account sign up
						
				- How do you plan to use Digital Ocean?.....: I'm not sure yet
				
				- What is your role or bussiness type?......: Hobbyist or student
				
				- What do you spend on cloud infrastructure?: $0 - $50
				
					Press [Submit] button
				
				
		- Verify your identity

			- Select payment method type:
			
				- Add a Card:
				
					- Card Details
				
						- Card number			MM / YY  CVC:
					
						- Cardholder name:		Merlina Addams
					
					
					- Billing Address
					
						- Country:
						
						- Address Line 1:
						
						- Address Line 2:
						
						- City:
						
						- State / Province / Region:

						- Zip / Postal Code:				
				
				
				- Connect Paypal:
				
			- Save payment method	
			
	- Be happy
			
		
Step 02 - In left pane, select App Platform


Step 03 - Select Source provider 

		- Select Docker Hub and press [Create app] button
		
		- Create Resource From Source Code
		
			- Docker Hub (is selected)
		
			- Repository...: dangeliza/apidemo01-image
			
			- Tag..........: latest
			
			- Credentials..: (empty)
			
			
		- Click Next button	
	
			- Customize your resource, press Edit button
			
				- Name..................: apidemo-01
				
				- Resource Type.........: Web Service (not changes)
				
				- Resource Size:
				
					- Select Shared (Best for prototypes)
					
					- Select Size combo box
						
						- Select choice $5.00/mo (512 MB RAM - 1 vCPU - 50GB bandwith)
						
						
				- Run Command...........: (empty)		
								
				- HTTP-Port.............: 5001
				
				- HTTP Request Routes...: only root /
				
				press Back button

		
		- Click Next button	(Environment variables)
		
		- Choose a region (optional) 
		
		- Change the application name (optional)
		
		- Click on Create Resources button
		
			be patient please, this process take awhile
			

Step 04 - Testing application

	- In the top of page you can see the app url, in my case:

		https://whale-app-43k3r.ondigitalocean.app/
		
		
	- Open the Postman application in Windows desktop
	
		- Create a new collection
		
			- Blank collection
			
			- Rename collection
			
		- Create a variable
			
			- Name...........: url
			
			- Initial value..: https://whale-app-43k3r.ondigitalocean.app/
			
			- Current value..: https://whale-app-43k3r.ondigitalocean.app/
			
		- Use the next guide:

			https://documenter.getpostman.com/view/141107/2sA3QtfBrw
			
			
Step 05 - Be happy