# Configuración ML - Debian 12 (BookWorm)

# ================================================================================================ Study Center....: Universidad Técnica Nacional Campus..........: Pacífico (JRMP) College career..: Ingeniería en Tecnologías de Información Period..........: 2C-2025 Course..........: ITI-522 - Computación en la nube Document........: 06 - Configuración ML Goals...........: Implementation of virtual machines with support for data processing Implement Python and R for machine learning Professor.......: Jorge Ruiz (york) Student.........:

## Notes:

This guide has been updated to use Debian 12 (BookWorm) as the base system, replacing the deprecated CentOS RHEL-7 distribution.

## Resources:

- https://canthonyscott.com/deploying-jupyterhub-on-a-mulit-user-centos-server/
- https://jupyterhub.readthedocs.io/en/1.2.0/installation-guide-hard.html
- https://debian.org/doc/

## System Configuration:

- hostname: sebas
- domain: vm
- password: clave123
- username: sebas

## Step 00 - Create virtual machine on local servers

### Resources

- 4 CPU cores
- 24 Gb main memory
- 75 Gb main storage (dynamic growth)

### Installation process

1. Select language
2. Network & Hostname (on)
    - IP: 10.236.2.155
3. Select installation destination
4. Complete Debian 12 installation

## Step 01 - Connect to the remote computer using SSH client

```bash
ssh sebas@10.236.2.155
```

## Step 02 - Update system

```bash
sudo apt update && sudo apt upgrade -y
```

## Step 03 - Utilities installation

```bash
# Install essential packages
sudo apt install -y apt-utils net-tools nmap wget nano curl
sudo apt install -y openssh-server openssh-client
sudo apt install -y build-essential libssl-dev libbz2-dev libffi-dev
sudo apt install -y software-properties-common dirmngr
sudo apt install -y gfortran libreadline-dev libx11-dev libxt-dev
sudo apt install -y libpng-dev libjpeg-dev libcairo2-dev libcurl4-openssl-dev
sudo apt install -y libxml2-dev libssl-dev libfontconfig1-dev
```

## Step 04 - Install dependencies for R

```bash
# Install required packages for adding repositories
sudo apt install -y dirmngr gnupg apt-transport-https ca-certificates software-properties-common

# Method 1: Import GPG key from keyserver (recommended)
sudo gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys B8F25A8A73EACF41
sudo gpg --export B8F25A8A73EACF41 | sudo tee /usr/share/keyrings/r-project.gpg > /dev/null

# Alternative Method 2: If keyserver fails, use direct download
# wget -qO- https://cran.r-project.org/bin/linux/debian/bookworm-cran40/Release.gpg | sudo gpg --dearmor -o /usr/share/keyrings/r-project.gpg

# Add R repository with proper key reference
echo "deb [signed-by=/usr/share/keyrings/r-project.gpg] https://cloud.r-project.org/bin/linux/debian bookworm-cran40/" | sudo tee /etc/apt/sources.list.d/r-project.list

# Update package lists
sudo apt update
```

## Step 05 - Create instaladores directory

```bash
mkdir ~/instaladores
cd ~/instaladores
```

## Step 06 - Specify R version

```bash
export R_VERSION=4.3.2
```

## Step 07 - Install R

```bash
# Install R from repository
sudo apt install -y r-base r-base-dev

# Verify installation
R --version
```

## Step 08 - Install RStudio Server

```bash
# Download RStudio Server for Debian
wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2023.12.1-402-amd64.deb

# Install RStudio Server
sudo dpkg -i rstudio-server-2023.12.1-402-amd64.deb

# Fix any dependency issues
sudo apt install -f

# Verify installation
sudo systemctl status rstudio-server

# Start and enable RStudio Server
sudo systemctl start rstudio-server
sudo systemctl enable rstudio-server
```

## Step 09 - Configure Firewall (UFW)

```bash
# Enable UFW if not already enabled
sudo ufw enable

# Add RStudio Server port (8787)
sudo ufw allow 8787/tcp

# Add JupyterHub port (8000) - for later use
sudo ufw allow 8000/tcp

# Check firewall status
sudo ufw status
```

## Step 10 - Install R libraries (global)

```bash
# Enter R shell as root
sudo R
```

In the R console:

```r
# Select CRAN repository
chooseCRANmirror()  # Display list of mirrors, select appropriate one
# or
chooseCRANmirror(26)  # Costa Rica mirror

# Install essential packages for rmarkdown
install.packages(c(
  "base64enc", "digest", "evaluate", "glue", "highr", 
  "htmltools", "jsonlite", "knitr", "magrittr", "markdown", 
  "mime", "rmarkdown", "stringi", "stringr", "xfun", "yaml"
))

# Install additional useful packages
install.packages(c(
  "lattice", "ggplot2", "dplyr", "tidyr", "readr", 
  "devtools", "shiny", "DT", "plotly"
))

# Quit R
q()
```

## Step 11 - Testing RStudio Server

1. Open your favorite Web Browser
2. Navigate to: `http://10.236.2.155:8787`
3. Login with your system credentials (sebas/clave123)

## Step 12 - Install Anaconda/Miniconda

```bash
# Install prerequisites
sudo apt install -y libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 
sudo apt install -y libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6

# Download Miniconda (lighter alternative to Anaconda)
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# Verify download
sha256sum Miniconda3-latest-Linux-x86_64.sh

# Install Miniconda
bash Miniconda3-latest-Linux-x86_64.sh

# Follow installation prompts:
# - Accept license
# - Install to /opt/miniconda3 (for system-wide installation)
# - Initialize conda

# Reload bash profile
source ~/.bashrc

# Update conda
conda update conda

# Verify Python installation
python --version
```

## Step 13 - Install JupyterHub

```bash
# Install JupyterHub and related packages
conda install -c conda-forge jupyterhub
conda install -c conda-forge notebook
conda install -c conda-forge jupyterlab

# Create application directory
sudo mkdir -p /apps/jupyterhub
cd /apps/jupyterhub

# Generate JupyterHub configuration
sudo jupyterhub --generate-config

# Create systemd service file
sudo nano /etc/systemd/system/jupyterhub.service
```

Content for jupyterhub.service:

```ini
[Unit]
Description=JupyterHub Service
After=multi-user.target

[Service]
Environment=PATH=/opt/miniconda3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
ExecStart=/opt/miniconda3/bin/jupyterhub --ip 0.0.0.0 --port 8000 --config=/apps/jupyterhub/jupyterhub_config.py
Restart=on-failure
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=jupyterhub
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Start and enable JupyterHub
sudo systemctl start jupyterhub.service
sudo systemctl enable jupyterhub.service

# Check service status
sudo systemctl status jupyterhub.service
```

## Step 14 - Create user management script

```bash
# Create utilities directory
sudo mkdir -p /utiles

# Create adduser script
sudo nano /utiles/adduser.sh
```

Content for adduser.sh:

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <username>"
    exit 1
fi

USERNAME=$1

# Create user with home directory
sudo useradd -m -s /bin/bash "$USERNAME"

# Set permissions for home directory
sudo chmod 755 /home/"$USERNAME"

# Set password
sudo passwd "$USERNAME"

echo "User $USERNAME created successfully"
```

```bash
# Make script executable
sudo chmod +x /utiles/adduser.sh

# Create example user
sudo /utiles/adduser.sh york
```

## Step 15 - Testing JupyterHub

1. Open your web browser
2. Navigate to: `http://10.236.2.155:8000`
3. Login with system credentials

## Additional Configuration Notes

### For Python Data Science Libraries:

```bash
# Install common data science packages
conda install -c conda-forge pandas numpy scipy matplotlib seaborn
conda install -c conda-forge scikit-learn jupyter ipykernel
conda install -c conda-forge plotly bokeh
```

### For R Integration with Jupyter:

```bash
# Install R kernel for Jupyter
conda install -c conda-forge r-irkernel
```

In R console:

```r
# Install R kernel for Jupyter
IRkernel::installspec()
```

### Security Considerations:

- Change default passwords
- Configure SSL/TLS for production use
- Set up proper user authentication
- Configure backup procedures

### Troubleshooting:

#### GPG Key Issues:

If you encounter GPG key errors, try these solutions:

```bash
# Clean up any existing problematic entries
sudo rm -f /etc/apt/sources.list.d/r-project.list
sudo rm -f /usr/share/keyrings/r-project.gpg

# Method 1: Import from keyserver (try different keyservers if one fails)
sudo gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys B8F25A8A73EACF41
sudo gpg --export B8F25A8A73EACF41 | sudo tee /usr/share/keyrings/r-project.gpg > /dev/null

# If the above fails, try alternative keyservers:
# sudo gpg --keyserver hkp://keys.gnupg.net:80 --recv-keys B8F25A8A73EACF41
# sudo gpg --keyserver hkp://pgp.mit.edu:80 --recv-keys B8F25A8A73EACF41

# Method 2: Manual key download and import
curl -fsSL https://cran.r-project.org/bin/linux/debian/bookworm-cran40/Release.gpg | sudo gpg --dearmor -o /usr/share/keyrings/r-project.gpg

# Add the repository
echo "deb [signed-by=/usr/share/keyrings/r-project.gpg] https://cloud.r-project.org/bin/linux/debian bookworm-cran40/" | sudo tee /etc/apt/sources.list.d/r-project.list

# Update and test
sudo apt update
```

#### Alternative: Install R from Debian repositories

If CRAN repository continues to have issues:

```bash
# Install R from default Debian repositories (may be older version)
sudo apt install -y r-base r-base-dev

# Check version
R --version
```

## Final Notes:

This guide provides a complete setup for a machine learning environment on Debian 12, including both R and Python ecosystems. The system is configured for multi-user access through both RStudio Server and JupyterHub.