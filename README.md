# Introducción al experiment tracking con MLflow – JAIIO 2025

Este repositorio contiene el código y entorno utilizado en la charla **"Introducción al experiment tracking con MLflow"**, presentada en las **Jornadas Argentinas de Informática (JAIIO) 2025**.

El objetivo es mostrar cómo aplicar MLflow para registrar experimentos, métricas, parámetros, artefactos y modelos durante el entrenamiento de una red neuronal simple en PyTorch, usando el dataset MNIST.

Se utilizó para el mismo un Ubuntu como host, que dentro corre el entorno en un contenedor Docker, el cual incluye:

- MLflow 3.1.4
- PyTorch + torchvision
- Jupyter Notebook
- Servidor de MLflow configurado para servir artefactos y registrar modelos

## Cómo usar este repositorio con GPU Nvidia

### Instalar Docker, driver de Nvidia, y nvidia-container-toolkit

1. Instalar Docker

```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo docker run hello-world
```

Optional: ejecutar los pasos de post-instalación para no necesitar sudo.

```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

2. Instalar Nvidia driver 570

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
sudo apt-get install nvidia-driver-570
sudo reboot
```

3. Instalar nvidia-container-toolkit

```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
sudo apt-get install -y \
    nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
sudo docker run --rm --gpus all nvidia/cuda:12.8.1-base-ubuntu24.04 nvidia-smi
```

### Build de la imagen Docker

```bash
cd gpu
docker build -t jaiio2025-gpu .
cd ..
```

La imagen tiene todas las dependencias de Python que están en `requirements.txt`, y levanta automáticamente el servidor de MLflow al ejecutar el container.

### Ejecutar el contenedor

```bash
docker run  -p 5000:5000 --rm -it --gpus all \
    -v ./workspace:/root/workspace \
    -v ./mlflow:/root/mlflow \
    --name jaiio2025 \
    jaiio2025-gpu:latest
```

Esto iniciará el MLflow Tracking Server en el puerto 5000.

### Usar desde VSCode con Dev Containers (recomendado)

1. Abrí VSCode, y asegurate de tener la extensión de Dev Containers.
2. Presione ctrl+shift+P, y seleccione la opción `Dev Containers: Attach to running container...`
3. Seleccione el container `jaiio2025`.

## Cómo usar este repositorio sin GPU

### Instalar Docker

1. Instalar Docker

```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo docker run hello-world
```

Optional: ejecutar los pasos de post-instalación para no necesitar sudo.

```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

### Build de la imagen Docker

```bash
cd cpu
docker build -t jaiio2025-cpu .
cd ..
```

La imagen tiene todas las dependencias de Python que están en `requirements.txt`, y levanta automáticamente el servidor de MLflow al ejecutar el container.

### Ejecutar el contenedor

```bash
docker run  -p 5000:5000 --rm -it --gpus all \
    -v ./workspace:/root/workspace \
    -v ./mlflow:/root/mlflow \
    --name jaiio2025 \
    jaiio2025-cpu:latest
```

Esto iniciará el MLflow Tracking Server en el puerto 5000.

### Usar desde VSCode con Dev Containers (recomendado)

1. Abrí VSCode, y asegurate de tener la extensión de Dev Containers.
2. Presione ctrl+shift+P, y seleccione la opción `Dev Containers: Attach to running container...`
3. Seleccione el container `jaiio2025`.
4. Una vez dentro del container, seleccione el kernel /opt/env/bin/python (en caso de que vscode no lo haya detectado automáticamente).
