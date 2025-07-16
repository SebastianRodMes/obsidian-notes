===============================================================================================
Institute....: Universidad Técnica Nacional
Campus.......: Pacífico
Career.......: Tecnologías de Información
Course.......: ITI-522 - Computación en la Nube
Period.......: 2-2025
Document.....: Remote_development.txt
Goals........: Create developmente environment
                - Create virtual machine with Ubuntu Server 22.04 lts version 4
				- Install Docker environment
				- Install NodeJs tools
				- Create a single React application (only for testing)
				- Update Visual Studio Code
				- Connect the developer tool with remote React project
Professor....: Jorge Ruiz (york)
Student......:
===============================================================================================

Using the local servers and  

Step 01 - Create a virtual machine

	- Name...................: Develop-York
	- CPU....................: 3 vCPUs	
	- Memory.................: 16384 Mbs
	- Disk...................: 65Gbs
	- Audio..................: Disable
	- Network................: External and conneted		
	- USB 3.1	
	
	
Step 02 - Start installation 	
	
	- Try or install Ubuntu Server
	
	- Select language
	
	- Installer update available (take a choice)
	
	- Keyboard layout configuration (take a choice)
	
	- Network connections
	
		- enp0s3	eth
		- DHCPv4	10.236.2.115/24


	- Configure proxy, only is required
	
	- Configure Ubuntu archive mirror
	
		- Mirror address: http://archive.ubuntu.com/ubuntu/


	- Guided storage configuration

		- Use an entire	disk.................: check in
		- Set up this disk as an LVM group...: check out
		
		
	- Profile setup (remember customize all variables)
	
		- Your name...........: merlina addams
		- Your servers name...: develyork
		- Pick a username.....: maddams
		- Choose a password...: caramia2K23*
	
	
	- Upgrade to Ubuntu Pro...: Skip for now
	
	- SSH Setup:
	
		- Install OpenSSH Server...: Check in
		- Import SSH identity......: No
		
		
	- Featured Server Snaps:

		- Don't select anythings
		
		
	Note:
		This installation take awhile, please take a coffee cup
		
		
Step 03 - Update installation (using sudo user)
	lo primero: su -

	apt update
	apt upgrade (si es requerido)
	
	apt install net-tools
	apt install build-essential
	apt install curl



Step 04 - Connect with your new remote server		
		
	- Open PowerShell on Windows

	- Write the next sentence:	(remember change the username and ip address)
	
		ssh maddams@10.236.2.115
		
		- Acceppt the fingerprint
		
		
	- Changes to power user 	
	
		sudo su
		
		
Step 05 - Install tools, services and required files:

	- Update system
	
		apt update
		
		apt upgrade	
		
		
	-apt update
apt install ca-certificates curl gnupg

install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

chmod a+r /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null



				
		apt update
	
	
	- Install Docker:
			     
		apt update
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


		
		
	- Validate installation

		systemctl status docker
		
		docker --version
		
		
	- Configure Docker to start on boot with systemd:

		systemctl enable docker
		
		
Step 06 - Manage Docker as a non-root user

	- Create the docker group: (maybe the group exist)
	
		groupadd docker
		newgrp docker
		
		
	- Add your user to the docker group: (remember change the username)

		usermod -aG docker maddams
		
	- Reboot your virtual machine

		reboot
		
		
Step 07 - Verify that you can run docker commands without sudo:

	- Execute the next command:
	
		docker run hello-world
		
		You can see the next message:
		
			Unable to find image 'hello-world:latest' locally
			latest: Pulling from library/hello-world
			719385e32844: Pull complete
			Digest: sha256:c79d06dfdfd3d3eb04cafd0dc2bacab0992ebc243e083cabe208bac4dd7759e0
			Status: Downloaded newer image for hello-world:latest

			Hello from Docker!
			This message shows that your installation appears to be working correctly.

			To generate this message, Docker took the following steps:
			
			 1. The Docker client contacted the Docker daemon.
			
			 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
				(amd64)
			 
			 3. The Docker daemon created a new container from that image which runs the
				executable that produces the output you are currently reading.
			 
			 4. The Docker daemon streamed that output to the Docker client, which sent it
				to your terminal.

			
			To try something more ambitious, you can run an Ubuntu container with:
			
				$ docker run -it ubuntu bash

			
			Share images, automate workflows, and more with a free Docker ID:
			
				https://hub.docker.com/

			
			For more examples and ideas, visit:
			
				https://docs.docker.com/get-started/
			 
	
	- Show the all containers and images
	
		docker ps -a
		
		docker images
	
	- Be happy


Step 08 - Install NodeJS
esto no en root.

	- Download and install fnm:
	
		curl -o- https://fnm.vercel.app/install | bash
		luego de esto, abrimos otra sesion

	- Download and install Node.js:

		fnm install 22


	- Verify the Node.js version:
	
		node -v # Should print "v22.16.0".
		
		npm -v # Should print "10.9.2".

		
		
	- Validates version installed

		node -- version		(v18.19.1)
		
		npm --version		(9.2.0)
		
		npx --version		(9.2.0)
		
	
	- Restart your virtual machine

		reboot
		

Step 09 - Create a React Project (noly for testing)

	- In /home/maddams 			(remember change username)
	
		mkdir react
		
		cd react
	
		npx create-react-app testing_01
		
			Ok to proceed?	(y)

			
		cd testing_01
		
		npm start
		
		- you can see the next urls
		
			Local:            http://localhost:3000
			On Your Network:  http://10.236.2.115:3000
		
		
		- Testing in your favorite web browse in Windows host
		
		- ctrl + c to stop react application


Step 10 - Configure Microsoft Visual Code to remote access

	- Install extensions
	
		- Remote - SSH			v0.107.0
		- Remote Development	v0.24.0
		- Remote Explorer		v0.4.1
		
		
	- Restart Visual Code	
	

-----------------------------------------------------------
Please call your professor for technical support
-----------------------------------------------------------
	
Step 11 - Open Visual Code

	- Select Remote Explorer Icon in left pane
	
		- Select Remotes (Tunnels/SSH) 
		
			- Select SSH and press + button (at right)
			
				Enter SSH Connection Command
				
					Eg. ssh jruiz@10.236.2.115		(you must changes user and ip address)
					
					- Press Enter button
					
					
			- Select SSH configuration file to update
			
				Eg. C:\User\madamms\.ssh\config


			- Open config file
			
				you can see:
				
					Host 10.236.2.115
					HostName 10.236.2.115
					User madamms				(you can change the username)
					
				
			- Close this file	
			
		- On Remotes (Tunnels/SHH) press Refresh Button
		
			- You must see the new ssh connection


		- Open remote connection 
		
			Please enter password.
			
			
Step 12 - Update the React Project

	- Delete files unnecesaries
	
		- In the src directory, delete the next files:
		
			- App.test.js
			
			- reportWebVitals.js
			
			- setupTests.js
			
			
	- Open src/Index.js

		- Delete the next code:
		
			import reportWebVitals from './reportWebVitals';
			
			
			// If you want to start measuring performance in your app, pass a function
			// to log results (for example: reportWebVitals(console.log))
			// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
			reportWebVitals();
			
			
	- Save changes		
			
			
	- Add first component:

		- In the src directory, create new file called Component.js
		
		- Open Component.js and write:
		
			import React, {useState} from "react";

			export default function Component() {
				const [texto, setTexto] = useState("Hola Mundo")
				const [texto2, setTexto2] = useState()

				const buttonClick = () => {
					setTexto2(texto)
				}

				return (
					<div>
						<h5>Mi primer Componente</h5>
						<div>
							<input type="text" value={texto} onChange={(e) => setTexto(e.target.value)}/>
							<button onClick={buttonClick}>Actualizar</button>
							<p>{texto}</p>
							<p>{texto2}</p>
						</div>
					</div>
				)
			}
			
		- Save changes
		

	- Open src/App.js
	
		- At the end of header section writes:
		
			<Component />
			
			
		- Save changes	
			
			
	- Open the window terminal into Visual Code

		- You can see the command line with a path equal next:

			maddmas@usdocker:~/react/testing_01$
			
		
		- Then execute the next command

			npm start		
			
			
-----------------------------------------------------------
Create Docker image and testing into Docker environment
-----------------------------------------------------------

Step 13 - Create Dockerfile

	- Into the root project
	
		- Right click and select new file:
		
			- Name.......: Dockerfile
			- Extension..: without extension

	- Open Dockerfile and writes the next code:
	
		FROM node:20-alpine

		LABEL authors="jruiz"

		RUN mkdir -p /apps/testing_01
		WORKDIR /apps/testing_01

		COPY package.json /apps/testing_01/		
		COPY . /apps/testing_01/

		EXPOSE 3000
		CMD ["npm", "start"]
	

	- Save changes


Step 14 - Create Dockerfile image

	- Into the terminal window of the Visual Code, execute the next commands:
	
		docker build -t testing_01-image .		
		
			- the above step take awhile, explicitly the "RUN npm install" instruction
			- be patience please
			
			- you can see the next verbose information
			
				[+] Building 366.8s (11/11) FINISHED                                                                                                                     docker:default
				 => [internal] load build definition from Dockerfile                                                                                                               0.2s
				 => => transferring dockerfile: 251B                                                                                                                               0.0s
				 => [internal] load .dockerignore                                                                                                                                  0.1s
				 => => transferring context: 2B                                                                                                                                    0.0s
				 => [internal] load metadata for docker.io/library/node:20-alpine                                                                                                  2.1s
				 => [1/6] FROM docker.io/library/node:20-alpine@sha256:b1789b7be6aa16afd642eaaaccdeeeb33bd8f08e69b3d27d931aa9665b731f01                                           30.5s
				 => => resolve docker.io/library/node:20-alpine@sha256:b1789b7be6aa16afd642eaaaccdeeeb33bd8f08e69b3d27d931aa9665b731f01                                            0.1s
				 => => sha256:6bff055d05cc0a93fd9a955ade6f25ab4623f3c9823053b4362576d4a024828b 7.13kB / 7.13kB                                                                     0.0s
				 => => sha256:96526aa774ef0126ad0fe9e9a95764c5fc37f409ab9e97021e7b4775d82bf6fa 3.40MB / 3.40MB                                                                     0.9s
				 => => sha256:4d922691d718dabc542bcc05349c0d1fd55d67728111c2797e649b378c682704 41.96MB / 41.96MB                                                                   5.0s
				 => => sha256:b1789b7be6aa16afd642eaaaccdeeeb33bd8f08e69b3d27d931aa9665b731f01 1.43kB / 1.43kB                                                                     0.0s
				 => => sha256:5ffaaf1eed5668f16f2d59130993b6b4e91263ea73d7556e44faa341d7d1c78a 1.16kB / 1.16kB                                                                     0.0s
				 => => sha256:8504be75e4ac94664146fe3c81584e386a44c501790c91dfe599313744d28ae4 2.34MB / 2.34MB                                                                     1.8s
				 => => extracting sha256:96526aa774ef0126ad0fe9e9a95764c5fc37f409ab9e97021e7b4775d82bf6fa                                                                          0.9s
				 => => sha256:8fe61125fbb93e3be3218d688f4a57d547cbdce0a66246671115146c6bd8491f 450B / 450B                                                                         1.3s
				 => => extracting sha256:4d922691d718dabc542bcc05349c0d1fd55d67728111c2797e649b378c682704                                                                         23.2s
				 => => extracting sha256:8504be75e4ac94664146fe3c81584e386a44c501790c91dfe599313744d28ae4                                                                          1.0s
				 => => extracting sha256:8fe61125fbb93e3be3218d688f4a57d547cbdce0a66246671115146c6bd8491f                                                                          0.0s
				 => [internal] load build context                                                                                                                                 61.7s
				 => => transferring context: 304.21MB                                                                                                                             61.4s
				 => [2/6] RUN mkdir -p /apps/testing_01                                                                                                                            4.1s
				 => [3/6] WORKDIR /apps/testing_01                                                                                                                                 0.5s
				 => [4/6] COPY package.json /apps/testing_01/                                                                                                                      0.9s
				 => [5/6] RUN npm install                                                                                                                                        287.6s
				 => [6/6] COPY . /apps/testing_01/                                                                                                                                 7.9s
				 => exporting to image                                                                                                                                             6.3s
				 => => exporting layers                                                                                                                                            6.3s
				 => => writing image sha256:ae6aed47ee210c001f9b8775736deb669941b6e7aff265d782b79f61a4570a10                                                                       0.0s
				 => => naming to docker.io/library/testing_01-image
			
		
		docker run --name testing_01 -p 3000:3000 -d testing_01-image
		
			- you can see a text same this:
			
				e5a8a7dd04ac7644afa3e6519c4004af55e2cc4ff462d2cbfb5eba5d9c0c20ae
		

Step 15 - Testing the new Docker constainer:

	- Into the virtual server, execute:
	
		- To see the new images:
		
			docker images
			
			- You can see:
			
				REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
				testing_01-image   latest    0eac0f61c658   21 minutes ago   834MB
				hello-world        latest    9c7a54a9a43c   6 months ago     13.3kB
		

		- To see the running docker:
		
			docker ps -a
			
			- You can see:
			
				CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS                  PORTS                                       NAMES
				e5a8a7dd04ac   testing_01-image   "docker-entrypoint.s…"   23 minutes ago   Up 23 minutes           0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   testing_01
				275082746b08   hello-world        "/hello"                 3 days ago       Exited (0) 3 days ago                                               quirky_khorana
			

		- For local testing

			curl http://10.90.28.54:3000		(you must changes ip address)
			
			- You can see:
			
				<!DOCTYPE html>
				<html lang="en">
				  <head>
					<meta charset="utf-8" />
					<link rel="icon" href="/favicon.ico" />
					<meta name="viewport" content="width=device-width, initial-scale=1" />
					<meta name="theme-color" content="#000000" />
					<meta
					  name="description"
					  content="Web site created using create-react-app"
					/>
					<link rel="apple-touch-icon" href="/logo192.png" />
					<!--
					  manifest.json provides metadata used when your web app is installed on a
					  user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
					-->
					<link rel="manifest" href="/manifest.json" />
					<!--
					  Notice the use of  in the tags above.
					  It will be replaced with the URL of the `public` folder during the build.
					  Only files inside the `public` folder can be referenced from the HTML.

					  Unlike "/favicon.ico" or "favicon.ico", "/favicon.ico" will
					  work correctly both with client-side routing and a non-root public URL.
					  Learn how to configure a non-root public URL by running `npm run build`.
					-->
					<title>React App</title>
				  <script defer src="/static/js/bundle.js"></script></head>
				  <body>
					<noscript>You need to enable JavaScript to run this app.</noscript>
					<div id="root"></div>
					<!--
					  This HTML file is a template.
					  If you open it directly in the browser, you will see an empty page.

					  You can add webfonts, meta tags, or analytics to this file.
					  The build step will place the bundled scripts into the <body> tag.

					  To begin the development, run `npm start` or `yarn start`.
					  To create a production bundle, use `npm run build` or `yarn build`.
					-->
				  </body>
				</html>			
			
		
		- For remote testing
		
			- Open your favorite web browser application and use the URL: http://10.236.2.115:3000
			- remember changes the ip address		


Step 16 - Remember be happy

		
------------------------------------------------------------
DOCKER COMMANDS
------------------------------------------------------------

	Command                          | Description
	---------------------------------+----------------------------------------
	docker images ls                 | Show downloaded images
	---------------------------------+----------------------------------------
	docker image rm name             | Delete a selected image
	---------------------------------+----------------------------------------
	docker image prune               | Clear old images
	---------------------------------+----------------------------------------
	docker image prune -a            | Clear all images without use       
	---------------------------------+----------------------------------------
	docker ps                        | Show only active containers
	---------------------------------+----------------------------------------
	docker ps -a                     | Show all containers
	---------------------------------+----------------------------------------
	docker ps -l                     | Show the new containers
	---------------------------------+----------------------------------------
	docker start container_id/name   | Start container by id or by name
	---------------------------------+----------------------------------------
	docker stop container_id/name    | Stop container by id or by name
	---------------------------------+----------------------------------------
	docker rm container_id/name      | Delete container by id or by name
	---------------------------------+----------------------------------------
	docker run -it container-id/name | Execute shell of a running container
	---------------------------------+----------------------------------------
	docker builder prune -a          | Limpia cache de creación de imágenes   
	---------------------------------+----------------------------------------
	