**status: work in progress**

# Running a GPU enabled Azure Custom Vision container on a NVidia Jetson nano
In this article we will go through the steps needed to run computer vision containers created with [Microsoft Azure Custom Vision](https://docs.microsoft.com/en-us/azure/cognitive-services/custom-vision-service/home?WT.mc_id=AI4DEV02-blog-heboelma).
An AI service and end-to-end platform for applying computer vision to your specific scenario. 

At the end of this walk through you are running your own models on a GPU enabled NVidia Jetson nano in a Docker Container.

## 1 - Setup your device
First you have to setup your device with the lastest version and change some settings.

### 1.1 Install the latest operating system.
Install the latest version of the operating system on the Jetson Nano. The NVidia learn website has a great tutorial for that.
[Follow the instructions here](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit). When the device boots and the desktop appears on the screen you can continue with the next step.

### 1.2 Configure the Jetson nano.
Before we can run the Docker containers created by the Custom Vision service we have to chance some settings on the Nano. 

Connect to the Nano through SSH or open a terminal.

- Disable the UI 
```
sudo systemctl set-default multi-user.target
```

- Set the Nano in high-power (10W) mode:
```
sudo nvpmodel -m 0
```

- Set the NVidia runtime as a default runtime in Docker. 
Your /etc/docker/daemon.json file should look like this.
```
{
  “default-runtime”: “nvidia”,
  “runtimes”: {
    “nvidia”: {
      “path”: “nvidia-container-runtime”,
      “runtimeArgs”: []
     }
   }
}
```

- Update your Nano OS and packages to the latest versions
```
sudo apt-get update
sudo apt-get dist-upgrade
```

- Add current user to docker group to use docker command without sudo, following this guide: https://docs.docker.com/install/linux/linux-postinstall/. 
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```
- Reboot your device

- Test you GPU support
```
docker run -it jitteam/devicequery ./deviceQuery
```
When the last line states: Result = PASS you can go to step 2, otherwise try follow the instructions on screen to enable GPU support in Docker.
![GPU Support](https://raw.githubusercontent.com/hnky/blog/master/images/001.jpg)

Now you are ready to run Docker containers that support Tensorflow with GPU.

## 2 - Train your model and download your container
With Azure Custom Vision you can create computer vision models and export this models to run localy on your machine. One of the export options is to get Docker container that exposes the model through an API. If you exporting the container there are 3 types you can coose from, Windows, Linux and Raspberry Pi. The only problem is that none of them use GPU support on an ARM64 device. but with a few tweaks in the the .DockerFile we can enable this and run the container on the Nano.

But first let's create a model and download the docker container. You can create your own model using the tutorials below or download my unmodified container.

#### Download sample container
This zip files containers a simple model that can classify Marge and Homer Simpson lego figures.
[Download](https://github.com/hnky/blog/raw/master/downloads/HomerOrMarge.DockerFile.Linux.zip)

#### Create your model 
Create your classification model using the Microsoft Azure Custom Vision.
- [Use Python to create a classification model](https://www.henkboelman.com/articles/create-your-first-model-with-azure-custom-vision-and-python/)
- [Create classification model through the interface](https://docs.microsoft.com/en-us/azure/cognitive-services/custom-vision-service/getting-started-build-a-classifier?WT.mc_id=AI4DEV02-blog-heboelma)

When you have created your classification model:
- Go to latest interation 
- Click on export (If export is disabled, make sure you have trained using a 'compact' domain)
- Select Docker
- Choose for the Linux download.
- Copy the download link (right click on the download button)

![Export Docker](https://raw.githubusercontent.com/hnky/blog/master/images/003.jpg)

At the end of this step you have the link to a zip file containing the model, code and DockerFile.


## 3 - Modify the Custom Vision container to run on the Jetson nano
Now we have the zip file containing the DockerFile and model we can download and modify it so it can run on the Jetson Nano. It is the easiest to do the steps below through a SSH session.

- Download the zip file
```
wget -O customvision.zip "https://github.com/hnky/blog/raw/master/downloads/HomerOrMarge.DockerFile.Linux.zip"
```
You can replace the link to the zip file with the link to your container.

- Unzip the file downloaded from the Custom Vision Service.
```
unzip customvision.zip -d customvision
```
- Open the Dockerfile
```
cd customvision
nano Dockerfile
```

- The contents of the docker file looks like this.
```
FROM python:3.7-slim

RUN pip install -U pip
RUN pip install numpy==1.17.3 tensorflow==2.0.0 flask pillow

COPY app /app

# By default, we run manual image resizing to maintain parity with CVS webservice prediction results.
# If parity is not required, you can enable faster image resizing by uncommenting the following lines.
# RUN echo "deb http://security.debian.org/debian-security jessie/updates main" >> /etc/apt/sources.list & apt update -y
# RUN apt install -y libglib2.0-bin libsm6 libxext6 libxrender1 libjasper-dev libpng16-16 libopenexr23 libgstreamer1.0-0 libavcodec58 libavformat58 libswscale5 libqtgui4 libqt4-test libqtcore4
# RUN pip install opencv-python

# Expose the port
EXPOSE 80

# Set the working directory
WORKDIR /app

# Run the flask server for the endpoints
CMD python -u app.py
```

- Replace the contents of the Docker file with this.
```
FROM nvcr.io/nvidia/l4t-base:r32.2
RUN apt-get update -y
RUN apt-get install python3-pip -y
RUN pip3 install -U pip

RUN DEBIAN_FRONTEND=noninteractive apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev -y
RUN DEBIAN_FRONTEND=noninteractive apt-get install python3 python3-dev python-dev build-essential libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev -y
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y python3-opencv

RUN pip3 install --pre tensorflow-gpu==2.0.0 --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42
RUN pip3 install flask pillow

COPY app /app

# Expose the port
EXPOSE 80

# Set the working directory
WORKDIR /app

# Run the flask server for the endpoints
CMD python3 -u app.py
```

### Build, run and test the container

- Build the container
```
docker build . -t mycustomvision
```

- Run the container
```
docker run -o 127.0.0.1:80:80 mycustomvision
```

- Test the container
```


```



### Build the container on Windows 10 and run on the device
You can build the container on Windows 10 and push the image to an Azure Container registery. On the device you can pull that container from the registery and run it on your device.

#### Azure Container Registry
- Create an Azure Container Registry (https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal?WT.mc_id=AI4DEV02-blog-heboelma)
- On the Windows Machine login to the registry.
```
docker login myregistry.azurecr.io
```
- On Windows 10 you can build the docker image using the buildx command. To enable this [use this tutorial](https://docs.docker.com/buildx/working-with-buildx/).
```
docker buildx build --platform linux/aarch64 -t myregistry.azurecr.io/mycustomvision --load .
```



### References
- https://medium.com/jit-team/building-a-gpu-enabled-kubernets-cluster-for-machine-learning-with-nvidia-jetson-nano-7b67de74172a
- https://github.com/jit-team/jetson-nano/tree/master/docker/jetson-nano-tf-gpu
- https://github.com/janza/docker-python3-opencv/blob/master/Dockerfile
- https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-edge-scenarios/send_message_to_output.py
- https://github.com/toolboc/IntelligentEdgeHOL
- https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl
- https://www.youtube.com/watch?v=_K5fqGLO8us
- https://www.youtube.com/watch?v=gMJgsQ13SKs&feature=youtu.be
