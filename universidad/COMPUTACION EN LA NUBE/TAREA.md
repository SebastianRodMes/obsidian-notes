Usando api_data, se creo una imagen de docker, pero posteriormente ser usada.
![[Pasted image 20250728195242.png]]
# GUÍA  DE KUBERNETES 

_Desplegando API Flask + MongoDB en cluster Kubernetes_

## CONTENIDO DEL DOCKERFILE

```dockerfile
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
```

## REQUIREMENTS.TXT

```txt
Flask==2.3.3
flask-cors==4.0.0
pymongo==4.5.0
Werkzeug==2.3.7
gunicorn==21.2.0
```

---

# CONFIGURACIÓN COMPLETA DEL NODO MASTER

_

## Step 01 - Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

## Step 02 - Configurar hostname del master

```bash
sudo hostnamectl set-hostname "k8s-master.sk.local"

# Configurar /etc/hosts (ajustar IPs según CORRESPONDA XD)
sudo nano /etc/hosts

# 192.168.0.7 k8s-master.sk.local k8s-master
# 192.168.0.8 k8s-worker1.sk.local k8s-worker1
```

## Step 03 - Deshabilitar swap

```bash
sudo swapoff -a
free -h

# Comentar la línea de swap en fstab
sudo nano /etc/fstab
# Comentar la línea: /swap.img	none	swap	sw	0	0

sudo mount -a
```

## Step 04 - Cargar módulos del kernel

```bash
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

## Step 05 - Instalar Docker/Containerd

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

## Step 06 - Instalar Kubernetes

```bash
# Remover repositorio anterior (si existe)
sudo add-apt-repository --remove "deb http://apt.kubernetes.io/ kubernetes-xenial main" 2>/dev/null || true

# Agregar nuevo repositorio de Kubernetes
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Step 07 - Inicializar el cluster Kubernetes

```bash
# Inicializar el master 
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --control-plane-endpoint=k8s-master.sk.local


SUDO kubeadm join k8s-master.sk.local:6443 --token 1rculf.gs7o7wh1dzsvbx1p \
--discovery-token-ca-cert-hash sha256:baadebc19d80dbc01cc000a051315a3ef1543427b88eb477c581f18a8aa62026

# Configurar kubectl para el usuario actual
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verificar el cluster
kubectl cluster-info
```

## Step 08 - Instalar red de pods (Calico)

```bash
# Descargar Calico
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

# Editar para que coincida con nuestra red de pods
nano calico.yaml
# Buscar: CALICO_IPV4POOL_CIDR
# Cambiar a: value: "10.10.0.0/16"

# Aplicar Calico
kubectl apply -f calico.yaml

# Verificar que todos los pods del sistema estén corriendo
kubectl get pods -n kube-system

# Verificar nodos (debe mostrar master como Ready)
kubectl get nodes
```

## Step 09 - Desplegar MongoDB 

```bash
# ⚠️ IMPORTANTE: Usé MongoDB 4.4 (no 5.0) para evitar problemas con AVX
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:4.4
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_DATABASE
          value: "mydb"
        volumeMounts:
        - name: mongodb-storage
          mountPath: /data/db
      volumes:
      - name: mongodb-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
EOF

# Verificar MongoDB
kubectl get pods | grep mongodb
kubectl get services | grep mongodb
```

## Step 10 - Desplegar  API desde Docker Hub (VERSIÓN CORREGIDA)

```bash
# ⚠️ IMPORTANTE: Usar el tag correcto de la imagen
# Verificar en Docker Hub qué tag se tiene (v1.0, latest, etc.)

# Crear deployment con el tag correcto
kubectl create deployment api-data --image=seb4s/api_data:v1.0

# Verificar que se creó
kubectl get deployments
kubectl get pods
```

## Step 11 - Escalar a 4 réplicas

```bash
# Esperar a que el pod inicial esté Running, luego escalar
kubectl scale deployment/api-data --replicas=4

# Verificar distribución de réplicas
kubectl get pods -o wide
```

## Step 12 - Exponer el servicio 

```bash
# ⚠️ IMPORTANTE: Usar la IP REAL del master, no una inventada
# Primero verificar la IP real:
ip addr show

# Exponer con LoadBalancer usando tu IP real
kubectl expose deployment/api-data --type="LoadBalancer" --port=5001 --external-ip=ip real

# Ejemplo si la es 192.168.0.7:
# kubectl expose deployment/api-data --type="LoadBalancer" --port=5001 --external-ip=192.168.0.7

# Verificar servicios
kubectl get services
```

---

# CONFIGURACIÓN COMPLETA DEL NODO WORKER



## WORKER 1 - Configuración inicial

### Step 01 - Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 02 - Configurar hostname del worker

```bash
sudo hostnamectl set-hostname "k8s-worker1.sk.local"

# Configurar /etc/hosts (MISMAS IPs que en el master)
sudo nano /etc/hosts
# Agregar al final del archivo (USAR LAS MISMAS IPs DEL MASTER):
# 192.168.0.7 k8s-master.sk.local k8s-master
# 192.168.0.8 k8s-worker1.sk.local k8s-worker1
```

### Step 03 - Deshabilitar swap

```bash
sudo swapoff -a
free -h

# Comentar la línea de swap en fstab
sudo nano /etc/fstab
# Comentar la línea: /swap.img	none	swap	sw	0	0

sudo mount -a
```

### Step 04 - Cargar módulos del kernel

```bash
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### Step 05 - Instalar Docker/Containerd

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Step 06 - Instalar Kubernetes

```bash
# Remover repositorio anterior (si existe)
sudo add-apt-repository --remove "deb http://apt.kubernetes.io/ kubernetes-xenial main" 2>/dev/null || true

# Agregar nuevo repositorio de Kubernetes
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Step 07 - Unir al cluster

```bash
# ⚠️ IMPORTANTE: Usar el comando join EXACTO que dió el master
# Ejemplo:
 sudo kubeadm join k8s-master.sk.local:6443 --token 1rculf.gs7o7wh1dzsvbx1p \
--discovery-token-ca-cert-hash sha256:baadebc19d80dbc01cc000a051315a3ef1543427b88eb477c581f18a8aa62026
```

---

# COMANDOS DE VERIFICACIÓN FINAL

## Desde el MASTER, verificar todo el cluster:

```bash
# Ver estado general del cluster
echo "=== NODOS ==="
kubectl get nodes

echo -e "\n=== DEPLOYMENTS ==="
kubectl get deployments

echo -e "\n=== PODS DISTRIBUIDOS ==="
kubectl get pods -o wide

echo -e "\n=== SERVICIOS ==="
kubectl get services

echo -e "\n=== PRUEBA DE CONECTIVIDAD ==="
curl http://192.168.0.7:5001/
```

## Resultado esperado:
![[Pasted image 20250728194436.png]]

```

```

---

#EVIDENCIA


![[Pasted image 20250728184506.png]]
# 
# Step 04 - Cargar módulos del kernel
![[Pasted image 20250722160226.png]]# 

![[Pasted image 20250728185413.png]]
![[Pasted image 20250722162306.png]]
# Verificar que todos los pods del sistema estén corriendo
kubectl get pods -n kube-system
![[Pasted image 20250722162333.png]]


![[Pasted image 20250724124639.png]]
 
![[Pasted image 20250724124732.png]]
