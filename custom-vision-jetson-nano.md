

Flash device
Connect internet
https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#next-resources
https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit

install curl
install iot edge
https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl


# Running a GPU enabled Azure Custom Vision container on a NVidia Jetson nano


## 1 - Setup your device

Install OS

Set the Nano in high-power (10W) mode:
```
sudo nvpmodel -m 0
```

Set the NVidia runtime as a default runtime in Docker. 
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

Update your Nano to the latest versions
```
sudo apt-get update
sudo apt-get dist-upgrade
```

Add current user to docker group to use docker command without sudo, following this guide: https://docs.docker.com/install/linux/linux-postinstall/. 
The required commands are:
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```
Reboot your device

Test you GPU support
```
docker run -it jitteam/devicequery ./deviceQuery
```

## 2 - Train your model and download your container


## 3 - Modify the container to run on the nano

- Unzip the file downloaded from the Custom Vision Service.
- Open the Dockerfile




```
FROM nvcr.io/nvidia/l4t-base:r32.2
RUN apt-get update -y 
RUN apt-get install python3-pip -y 
RUN pip3 install -U pip
RUN DEBIAN_FRONTEND=noninteractive apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev -y
RUN DEBIAN_FRONTEND=noninteractive apt-get install python3 python-dev python3-dev build-essential libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev -yq
RUN pip3 install -U numpy grpcio absl-py py-cpuinfo psutil portpicker six mock requests gast h5py astor termcolor protobuf keras-applications keras-preprocessing wrapt google-pasta
RUN pip3 install --pre tensorflow-gpu==1.13.1 --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v42 

#RUN apt update && apt install -y libjpeg62-turbo libopenjp2-7 libtiff5 libatlas-base-dev
#RUN pip install absl-py six protobuf wrapt gast astor termcolor keras_applications keras_preprocessing --no-deps
RUN pip3 install -U pillow
#RUN pip install -U numpy==1.16.1 --no-deps
RUN pip3 install flask 
#RUN pip install tensorflow-gpu==1.13.1 --extra-index-url 'https://developer.download.nvidia.com/compute/redist/jp/v42' --no-deps


# By default, we run manual image resizing to maintain parity with CVS webservice prediction results.
# If parity is not required, you can enable faster image resizing by uncommenting the following lines.
#RUN echo "deb http://security.debian.org/debian-security jessie/updates main" >> /etc/apt/sources.list & apt update -y
#RUN apt install -y  zlib1g-dev libjpeg-dev gcc libglib2.0-bin libsm6 libxext6 libxrender1 libjasper-dev libpng16-16 libopenexr23 libgstreamer1.0-0 libavcodec58 libavformat58 libswscale5 libqtgui4 libqt4-test libqtcore4
#RUN pip3 install python3-opencv
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y python3-opencv

COPY app /app

# Expose the port
EXPOSE 80

# Set the working directory
WORKDIR /app

# Run the flask server for the endpoints
CMD python3 -u app.py

```





### References
- https://medium.com/jit-team/building-a-gpu-enabled-kubernets-cluster-for-machine-learning-with-nvidia-jetson-nano-7b67de74172a
