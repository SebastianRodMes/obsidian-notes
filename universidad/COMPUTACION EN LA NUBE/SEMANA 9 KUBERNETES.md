======================================================================================
Centro....: Universidad Técnica Nacional
Sede......: Pacífico
Carrera...: Tecnologías de Información
Curso.....: ITI-522 - Computación en la Nube
Periodo...: 2-2025
Documento.: Tratando de crear Kubernetes
Profesor..: Jorge Ruiz (york)
Estudiante:  
======================================================================================

Step 01 - Create 3 local virtual machine

	- OS........: Ubuntu 22.04 LTS
	- vCPUs.....: 2 cores
	- Memory....: 8 GB per node
	- Storage...: 35 GB (dynamic growth)
	- Network...: Bridged (allow all)
	- Mirror....: http://archive.ubuntu.com/ubuntu/
	- Software..: Only console
	              SSH Server
				  
	The nodes name is:
[[Pasted image 20250717135941.png]]

		k8s-master			10.236.2.133/24
		k8s-worker-1		10.236.2.138/24
		k8s-worker-2		10.236.2.137/24


Step 02 - Update system for all

	- Connect usign SSH connection and execute the next commands:

	sudo apt update
	
	sudo apte upgrade (only if required)	
	
	
Step 03 - Configure name of your machines

	- Master node:
	
		sudo hostnamectl set-hostname "k8s-master.sk.local"

		e.g. sudo hostnamectl set-hostname "k8s-master.demoyork.local"
		
		
	- Worker nodes

		sudo hostnamectl set-hostname "k8s-worker1.<sk>.local"
		sudo hostnamectl set-hostname "k8s-worker2.<sk>.local"
		
		e.g. sudo hostnamectl set-hostname "k8s-worker1.demoyork.local"
		
		
	- sudo nano /etc/hosts and writes at the end of file:  (all machines)
		
		# K8s cluster nodes in home
		#10.236.2.133 k8s-master.sk.local k8s-master
		#10.236.2.138 k8s-worker-1.sk.local k8s-worker1
		#10.236.2.137 k8s-worker-2.sk.local k8s-worker2
				
		# K8s cluster nodes in classroom
		10.90.28.150 k8s-master.sk.local k8s-master
		10.90.28.134 k8s-worker-1.sk.local k8s-worker1
		10.90.28.156 k8s-worker-2.sk.local k8s-worker2
			
		
Step 04 - Disable swap configuration (all machines)

		sudo swapoff -a
		free -h
		
		sudo nano /etc/fstab, and comment the next line:
		
			/swap.img	none	swap	sw	0	0
			
			
		sudo mount -a	
		

Step 05 - Load kernel modules (all machines)

echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF



	
	sudo sysctl --system

	
Step 06 - Install Docker module	(all machines)
	
	sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

	sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

	sudo apt update

	sudo apt install -y containerd.io

	containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

	sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

	sudo systemctl restart containerd

	sudo systemctl enable containerd


Step 07 - Install Kubernetes (all machines) (

	review:
	============================================================================================================
		https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/		
		https://lzreload.com/2022/12/09/Setup-Kubernetes-cluster/		

		curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo tee /etc/apt/trusted.gpg.d/google-cloud.gpg > /dev/null

				(* no aplicar) sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
	
		sudo nano /etc/apt/sources.list.d/kubernetes.list and writes:
		
			deb http://apt.kubernetes.io/ kubernetes-xenial main	
			
	============================================================================================================	

	- Remove bad repository reference:
	
		sudo add-apt-repository --remove "deb http://apt.kubernetes.io/ kubernetes-xenial main"

		sudo apt update
		
		sudo apt upgrade (only if required)
		
	
	- Add new repository reference:
	
		echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

		curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	
	
	sudo apt update
	
	sudo apt upgrade (only if required)

	sudo apt install -y kubelet kubeadm kubectl

	sudo apt-mark hold kubelet kubeadm kubectl


Step 08 - Init kubernet service (only in master configuration)

	sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --control-plane-endpoint=k8s-master.sk.local
	
	-- W A R N I N G --
	
		After executed the last instruction, you can see a message like this:
			
			Then you can join any number of worker nodes by running the following on each as root:

			kubeadm join k8s-master.demoyork.local:6443 --token lwhfbh.vnono9op2echay13 \
				--discovery-token-ca-cert-hash sha256:0adc8cae0bfca108c9980eaebe502b7267fa953d2aef6a991e01e279c60d2bbc

		Copy this code to bind each node with the server, step 9
	
	-- E N D   O F   W A R N I N G --
		
		
	mkdir -p $HOME/.kube

	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

	sudo chown $(id -u):$(id -g) $HOME/.kube/config

	kubectl cluster-info
	
	- Execute step 09 first

	kubectl get nodes
	
	curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
	
	nano calico.yaml
	
		ctrl-W : CALICO_IPV4POOL_CDIR
		
		- name: CALICO_IPV4POOL_CDIR
		  value: "10.10.0.0/16"


	Save changes
mv calico.yaml calico-backup.yam
l # Descarga la versión oficial más reciente 
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
	kubectl apply -f calico.yaml
	
	kubectl get pods -n kube-system
	
	kubectl get nodes
	

Step 09 - Init kubernet nodes (only in node configuration)

	- Remember use the token and discovery-token-ca-cert-hash generated in step 08
	
	sudo kubeadm join k8s-master.demoyork.local:6443 --token lwhfbh.vnono9op2echay13 --discovery-token-ca-cert-hash sha256:0adc8cae0bfca108c9980eaebe502b7267fa953d2aef6a991e01e279c60d2bbc


Step 10 - Deploy application

	-- Download images from Docker Hub
	
		kubectl create deployment apidemo01 --image=docker.io/dangeliza/apidemo01-image:latest 


	-- Show list of deployments
	
		kubectl get deployments
	
	
	- Show list of pods 

		kubectl get pods


	- Show list of running services
	
		kubectl get services

	
	- Start service from deployment expose port and assign external ip-address
	
		kubectl expose deployment/apidemo01 --type="NodePort" --port 5001 --external-ip  10.90.28.150
		
		
	- Delete running service		
	
		kubectl delete service -l app=apidemo01
		
		
Step 11 - Deploy application in multiple Instances

	- First delete apidemo1 service and run
	
	kubectl expose deployment/apidemo01 --type="LoadBalancer" --port 5001 --external-ip 10.90.28.150
	
	kubectl get services
	
		- you can see:
	
			NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
			apidemo01    LoadBalancer   10.102.242.138   10.236.2.133   5001:30283/TCP   2s
			kubernetes   ClusterIP      10.96.0.1        <none>         443/TCP          5d11h
	

	kubectl scale deployment/apidemo01 --replicas=3

	kubectl get rs
	
		- you can see 2 columns:
		
			NAME                   DESIRED   CURRENT   READY   AGE
			apidemo01-579dbc76c8   3         3         1       5d

			- DESIRED displays the desired number of replicas of the application, 
			  which you define when you create the Deployment.

			- CURRENT displays how many replicas are currently running.

	 kubectl get deployments
	 
		- you can see:
		
			NAME        READY   UP-TO-DATE   AVAILABLE   AGE
			apidemo01   3/3     3            3           5d


	kubectl get pods -o wide
	
		- you can see the distribution of the service:
		
			NAME                         READY   STATUS    RESTARTS      AGE     IP              NODE                         NOMINATED NODE   READINESS GATES
			apidemo01-579dbc76c8-7j5hj   1/1     Running   2 (63m ago)   5d      10.10.167.131   k8s-worker2.demoyork.local   <none>           <none>
			apidemo01-579dbc76c8-d4vk2   1/1     Running   0             4m36s   10.10.167.133   k8s-worker2.demoyork.local   <none>           <none>
			apidemo01-579dbc76c8-v2q67   1/1     Running   0             4m36s   10.10.167.132   k8s-worker2.demoyork.local   <none>           <none>
	
	
	kubectl describe deployment/apidemo01
	



-------------------------------------------------------
Configuration miniKube in the cloud
-------------------------------------------------------


Step 01 - Upgrade your virtual computer in Microsoft Azure

	(Required minimun of 2 vCores)
	
	this process take awhile


Step 02 - Connect to the virtual machine on Microsoft Azure

	ssh -i C:\users\<user_name>\.ssh\<ssh_key_name>.pem <remote_user_name>@<remote_ip>

		e.g. ssh -i C:\users\jruiz\.ssh\azureSSHkey.pem jruiz69@20.83.36.27
	
	
Step 03 - Update system

	sudo apt update
	
	sudo apt upgrade (only if required)
	
	
Step 04 - Install Docker

	sudo apt-get install ca-certificates curl
	
	sudo install -m 0755 -d /etc/apt/keyrings
	
	sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

	sudo chmod a+r /etc/apt/keyrings/docker.asc
	
	
	-- The next sentence is only one --
	
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
	-- End the sentence --
	
	sudo apt-get update
	
	sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
	
	systemctl status docker
	

Step 05 - Install mini-Kube

	curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
	
		ls (you can see minikube_latest_amd64.deb file into your work directory)
	
	
	sudo dpkg -i minikube_latest_amd64.deb
	
	sudo apt-get -y install podman
		
	minikube start 
	
	
	- To see minikube status 
	
		minikube kubectl -- get po -A
		
		
	- To create alias of last command
	
		alias kubectl="minikube kubectl --"


Step 06 - Deploy Example application

	kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
	
	kubectl expose deployment hello-minikube --type=NodePort --port=8080 --external-ip
	
	kubectl get services hello-minikube

	minikube service hello-minikube
	
		- you can see:
		
			 minikube service hello-minikube
			|-----------|----------------|-------------|---------------------------|
			| NAMESPACE |      NAME      | TARGET PORT |            URL            |
			|-----------|----------------|-------------|---------------------------|
			| default   | hello-minikube |        8080 | http://192.168.49.2:31376 |
			|-----------|----------------|-------------|---------------------------|
			🎉  Opening service default/hello-minikube in default browser...
			👉  http://192.168.49.2:31376



	curl http://192.168.49.2:31376

		- you can see:
		
			Request served by hello-minikube-5c898d8489-j6zdb

			HTTP/1.1 GET /

			Host: 192.168.49.2:31376
			Accept: */*
			User-Agent: curl/8.5.0
	

Step 07 - Install NgInx

	sudo apt update

	sudo apt update (if required)
	
	sudo apt install nginx

	sudo ufw allow 80
	
	
Step 08 - Configuring Inbound rule of your virtual machine

	- In the left pane of Microsft Azure Portal
	
		- Select Net / Net Configuration
		
			- Open rules box
			
			- Press [Create port ACL] button
			
				- Select inbound option
				
					- Origin......: Any
					
					- Intervals...: *
					
					- Target......: Any
					
					- Service.....: HTTP
					
					- Action......: Allow
					
				- Press [Add] button	
				
	- This process take awhile (a twice minutes)			
			
	- Testing your ip_address in web browser

	
Step 09 - Configuring NgInx as Reverse Proxy

	nano /etc/nginx/conf.d/hello_minikube.conf
	
	
	stream {
		server {
			listen 20.83.36.27:80;
      
			#TCP traffic will be forwarded to the specified server
			proxy_pass 192.168.39.131:8443;       
		}
	}
	
		