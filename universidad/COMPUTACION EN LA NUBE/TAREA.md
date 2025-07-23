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


# ============================================
# CONFIGURACIÓN COMPLETA DEL NODO MASTER
# (Ubuntu 22.04 LTS desde cero)
# ============================================

# ============================================
# Step 01 - Actualizar el sistema
# ============================================
sudo apt update
sudo apt upgrade -y

# ============================================
# Step 02 - Configurar hostname del master
# ============================================
sudo hostnamectl set-hostname "k8s-master.sk.local"

Acá quedé
# Configurar /etc/hosts (ajusta las IPs según tu red)
sudo nano /etc/hosts
# Agregar al final del archivo:
# 10.90.28.78 k8s-master.sk.local k8s-master
# 10.90.28.87 k8s-worker-1.sk.local k8s-worker1  
# 10.90.28.96 k8s-worker-2.sk.local k8s-worker2

# ============================================
# Step 03 - Deshabilitar swap
# ============================================
sudo swapoff -a
free -h

# Comentar la línea de swap en fstab
sudo nano /etc/fstab
# Comentar la línea: /swap.img	none	swap	sw	0	0

sudo mount -a

# ============================================
# Step 04 - Cargar módulos del kernel
![[Pasted image 20250722160226.png]]# ============================================
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

# ============================================
# Step 05 - Instalar Docker/Containerd
# ============================================
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

# ============================================
# Step 06 - Instalar Kubernetes
# ============================================

# Remover repositorio anterior (si existe)
sudo add-apt-repository --remove "deb http://apt.kubernetes.io/ kubernetes-xenial main" 2>/dev/null || true

# Agregar nuevo repositorio de Kubernetes
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# ============================================
# Step 07 - Inicializar el cluster Kubernetes
# ============================================

# Inicializar el master (ajusta la IP según tu red)
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --control-plane-endpoint=k8s-master.sk.local


kubeadm join k8s-master.sk.local:6443 --token elkahn.y76fe2h4omhifal5 \
        --discovery-token-ca-cert-hash sha256:1db870711d7f91c6109cf0d2aecd12b24c4c3a4d255b923bdac3e3e9bde6990a

# Configurar kubectl para el usuario actual
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verificar el cluster
kubectl cluster-info

# ============================================
# Step 08 - Instalar red de pods (Calico)
# ============================================

# Descargar Calico
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

# Editar para que coincida con nuestra red de pods
nano calico.yaml
# Buscar: CALICO_IPV4POOL_CIDR
# Cambiar a: value: "10.10.0.0/16"

# Aplicar Calico
kubectl apply -f calico.yaml
![[Pasted image 20250722162306.png]]
# Verificar que todos los pods del sistema estén corriendo
kubectl get pods -n kube-system
![[Pasted image 20250722162333.png]]

# Verificar nodos (debe mostrar master como Ready)
kubectl get nodes

ACA QUEDE# ============================================
# Step 09 - Desplegar MongoDB (base de datos)
# ============================================

# Crear deployment de MongoDB
kubectl create deployment mongodb --image=docker.io/mongo:5.0

# Exponer MongoDB internamente
kubectl expose deployment mongodb --port=27017 --target-port=27017 --name=mongodb-service

# Verificar MongoDB
kubectl get pods
kubectl get services

# ============================================
# Step 10 - Desplegar tu API desde Docker Hub
# ============================================

# Crear deployment de tu API
kubectl create deployment api-data --image=docker.io/seb4s/api_data:latest

# Verificar que se creó
kubectl get deployments
kubectl get pods

# ============================================
# Step 11 - Escalar a 4 réplicas
# ============================================

# Esperar a que se unan los workers, luego escalar
kubectl scale deployment/api-data --replicas=4

# Verificar distribución de réplicas
kubectl get pods -o wide

# ============================================
# Step 12 - Exponer el servicio
# ============================================

# Exponer con LoadBalancer (ajustar IP del master)
kubectl expose deployment/api-data --type="LoadBalancer" --port=5001 --external-ip=10.90.28.150

# Verificar servicios
kubectl get services

# ============================================
# COMANDOS DE VERIFICACIÓN FINAL
# ============================================

# Ver estado general del cluster
kubectl get nodes
kubectl get deployments
kubectl get pods -o wide
kubectl get services

# Ver detalles del deployment
kubectl describe deployment/api-data

# Ver logs de un pod específico (opcional)
kubectl logs <nombre-del-pod>

# ============================================
# NOTAS IMPORTANTES:
# ============================================
# 1. Guarda el comando kubeadm join para los workers
# 2. Ajusta las IPs en /etc/hosts según tu red
# 3. Los workers se configuran después con el comando join
# 4. La API se conectará a MongoDB usando: mongodb-service:27017





# ============================================
# CONFIGURACIÓN COMPLETA DE LOS NODOS WORKERS
# (Ubuntu 22.04 LTS desde cero)
# ============================================

# ============================================
# WORKER 1 - Configuración inicial
# ============================================

# Step 01 - Actualizar el sistema
sudo apt update
sudo apt upgrade -y

# Step 02 - Configurar hostname del worker 1
sudo hostnamectl set-hostname "k8s-worker1.sk.local"

# Configurar /etc/hosts (MISMAS IPs que en el master)
sudo nano /etc/hosts
# Agregar al final del archivo:
# 10.90.28.150 k8s-master.sk.local k8s-master
# 10.90.28.134 k8s-worker-1.sk.local k8s-worker1  
# 10.90.28.156 k8s-worker-2.sk.local k8s-worker2

# Step 03 - Deshabilitar swap
sudo swapoff -a
free -h

# Comentar la línea de swap en fstab
sudo nano /etc/fstab
# Comentar la línea: /swap.img	none	swap	sw	0	0

sudo mount -a

# Step 04 - Cargar módulos del kernel
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

# Step 05 - Instalar Docker/Containerd
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

# Step 06 - Instalar Kubernetes
# Remover repositorio anterior (si existe)
sudo add-apt-repository --remove "deb http://apt.kubernetes.io/ kubernetes-xenial main" 2>/dev/null || true

# Agregar nuevo repositorio de Kubernetes
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Step 07 - Unir al cluster (USAR EL TOKEN DEL MASTER)
# IMPORTANTE: Reemplazar con el comando que te dio el master
sudo kubeadm join k8s-master.sk.local:6443 --token xxxxxx.xxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ============================================
# WORKER 2 - Configuración inicial
# ============================================

# Step 01 - Actualizar el sistema
sudo apt update
sudo apt upgrade -y

# Step 02 - Configurar hostname del worker 2
sudo hostnamectl set-hostname "k8s-worker2.sk.local"

# Configurar /etc/hosts (MISMAS IPs que en el master)
sudo nano /etc/hosts
# Agregar al final del archivo:
# 10.90.28.150 k8s-master.sk.local k8s-master
# 10.90.28.134 k8s-worker-1.sk.local k8s-worker1  
# 10.90.28.156 k8s-worker-2.sk.local k8s-worker2

# Step 03 - Deshabilitar swap
sudo swapoff -a
free -h

# Comentar la línea de swap en fstab
sudo nano /etc/fstab
# Comentar la línea: /swap.img	none	swap	sw	0	0

sudo mount -a

# Step 04 - Cargar módulos del kernel
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

# Step 05 - Instalar Docker/Containerd
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

# Step 06 - Instalar Kubernetes
# Remover repositorio anterior (si existe)
sudo add-apt-repository --remove "deb http://apt.kubernetes.io/ kubernetes-xenial main" 2>/dev/null || true

# Agregar nuevo repositorio de Kubernetes
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Step 07 - Unir al cluster (USAR EL MISMO TOKEN DEL MASTER)
# IMPORTANTE: Reemplazar con el comando que te dio el master
sudo kubeadm join k8s-master.sk.local:6443 --token xxxxxx.xxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ============================================
# VERIFICACIÓN DESDE EL MASTER
# ============================================

# Una vez configurados ambos workers, desde el MASTER ejecutar:

# Ver todos los nodos
kubectl get nodes

# Deberías ver algo como:
# NAME                     STATUS   ROLES           AGE   VERSION
# k8s-master.sk.local      Ready    control-plane   Xm    v1.30.x
# k8s-worker1.sk.local     Ready    <none>          Xm    v1.30.x
# k8s-worker2.sk.local     Ready    <none>          Xm    v1.30.x

# Verificar distribución de los pods de la API
kubectl get pods -o wide

# Deberías ver las 4 réplicas distribuidas entre los 2 workers:
# NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE
# api-data-xxxxxxxxx-xxxxx    1/1     Running   0          Xs    10.10.x.x     k8s-worker1.sk.local
# api-data-xxxxxxxxx-xxxxx    1/1     Running   0          Xs    10.10.x.x     k8s-worker1.sk.local  
# api-data-xxxxxxxxx-xxxxx    1/1     Running   0          Xs    10.10.x.x     k8s-worker2.sk.local
# api-data-xxxxxxxxx-xxxxx    1/1     Running   0          Xs    10.10.x.x     k8s-worker2.sk.local

# ============================================
# COMANDOS ÚTILES PARA TROUBLESHOOTING
# ============================================

# Si un worker no se puede unir, verificar:
# 1. Conectividad de red
ping k8s-master.sk.local

# 2. Estado de containerd
sudo systemctl status containerd

# 3. Estado de kubelet
sudo systemctl status kubelet

# 4. Ver logs de kubelet
sudo journalctl -u kubelet -f

# 5. Si el token expiró, generar uno nuevo desde el master:
# kubectl token create --print-join-command

# ============================================
# NOTAS IMPORTANTES
# ============================================

# 1. Los workers NO necesitan kubectl configurado para el usuario
# 2. Solo el master maneja las configuraciones del cluster
# 3. Las IPs en /etc/hosts DEBEN ser exactamente las mismas en todos los nodos
# 4. El comando kubeadm join DEBE ejecutarse con sudo
# 5. Si un worker falla al unirse, puedes hacer reset y volver a intentar:
#    sudo kubeadm reset
#    sudo rm -rf /etc/cni/net.d
#    sudo systemctl restart kubelet