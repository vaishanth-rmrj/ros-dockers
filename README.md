# ros-dockers
A repository containing docker files for building ROS containers.

### Credits
- turlucode/ros-docker-gui

### Available versions
- ROS Noetic (GPU)

### Installation
1. Install Nvidia drivers for host PC
  - Linux
  ```
  sudo apt install nvidia-driver-515 nvidia-dkms-515
  
  # to check installation
  nvidia-smi
  ```
  
  For more info: (https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/)
  
2. Install Docker

In a nutshell, docker is an open source platform for building, deploying and managing containerized applications.

  - Ubuntu (Linux)
```
# dependencies
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```
```
# add docker official GPG key
sudo mkdir -p /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg-
```
```
# then add their repo
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```
# finally install it
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# create the docker group
sudo groupadd docker

# add your user to the docker group
sudo usermod -aG docker $USER
```
- You can test your docker installation by running the `hello-world` container:

```
docker run --rm hello-world
```

2. Install Nvidia Docker for GPU-Accelerated Containers
- Super! Unfortunately, Docker has no idea how to use your GPU(s), we need the NVIDIA Container Toolkit. It allows users to build and run GPU-accelerated containers.

- To install it
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

```

```
sudo apt-get update
sudo apt-get install -y nvidia-docker2
# to restart docker
sudo systemctl restart docker
```

- Testing the installation:
```
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

- And you should see the correct output from `nvidia-smi` inside the container. In my case:
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.57       Driver Version: 515.57       CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:1D:00.0 Off |                  N/A |
|  0%   38C    P8    15W / 350W |   2761MiB / 24576MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+

```
  
2. Build docker file
  - Select the ROS version
  ```
  cd <ros-ver-folder>/
  ```
  - Build the image
  ```
  docker build -t <docker-image-name> .
  ```
  
3. Run docker container
  - As root user
  
```
docker run --rm -it --runtime=nvidia \
--privileged --net=host --ipc=host \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-e DISPLAY=$DISPLAY \
-v $HOME/.Xauthority:/root/.Xauthority \
-e XAUTHORITY=/root/.Xauthority \
-v <PATH_TO_YOUR_WORKSPACE>:/home/<username>/ \
-e ROS_IP=<HOST_IP or HOSTNAME> \
<created-image-name>
```

  A terminator window will pop-up and the rest you know it! :)
  
  Important Remark: This will launch the container as root. 
  This might have unwanted effects! If you want to run it as  the current user, see next section.

  - As current user
```
docker run --rm -it --runtime=nvidia \
--privileged --net=host --ipc=host \
-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY \
-v $HOME/.Xauthority:/home/$(id -un)/.Xauthority \
-e XAUTHORITY=/home/$(id -un)/.Xauthority \
-e DOCKER_USER_NAME=$(id -un) \
-e DOCKER_USER_ID=$(id -u) \
-e DOCKER_USER_GROUP_NAME=$(id -gn) \
-e DOCKER_USER_GROUP_ID=$(id -g) \
-e ROS_IP=127.0.0.1 \
<created-image-name>
```
#### Note:
  - --rm -> remove container when exited
  - -it -> run interactive terminal
  - --runtime -> allows the docker to utilize nvidia GPU
  - -v -> mount local volume/directories
  - -e -> set environment variables
  
  Important Remark:

  - Please note that you need to pass the `Xauthority` to the correct user's home directory.
  - You may need to run `xhost si:localuser:$USER` or worst case `xhost local:root` if get errors like Error: cannot open display



