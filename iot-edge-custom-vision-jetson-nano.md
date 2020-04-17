# Object alert with Custom Vision and IoT Edge on the Jetson Nano

In this article I will guide throught the steps needed to create your own object alertings system running on an edge device. For this we will use a NVidia Jetson Nano, custom vision and Azure IoT Edge.

The goal is to process the camera frames localy and only send a message to the cloud when the detected object hits a certain confidence threshold.

[Insert graphic]

Requirements before you start:
- To continue you need an [Nvidia Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) and an USB camera.
- A Azure Subscription, if you don't have one you [create a free one here]().


## What do we need to build?
To get this working we need to:
- Setup Azure
  - Azure IoT Edge
  - Azure Container Registery
- Configure the Jetson Nano
  - Install IoT Edge
  - Configure the device to run our custom vision modules.
- Create 3 IoT Edge module
  - A module that runs the our computer vision model
  - A module that grabs camera frames, send the images to the computer vision module and put the result on the local IoT hub.
  - A module that grabs the results of the local IoT hub and send it to the IoT hub in Azure
- Deploy the modules to the IoT Edge
- Setup an Event Grid and Logic app to handle the alerts.


## Part 1 - Setup resources in Azure
To get started we need to setup a few things in Azure. For this we are going to use the Azure CLI. If you don't have the Azure CLI installed on your machine you can follow the [tutorial on MS Docs here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).


### 1.1 Create an IoT Hub
The first resources we need is an IoT Hub. We will use this Hub to communicate with our Edge Device. It will give us the ability to deploy to deploy our IoT Edge models to the device and give the Edge device the ability to send data back to the cloud. 

**Create a Resource group**
```
az group create --name {your resource group name} --location westeurope
```

**Create an IoT Hub**
```
az iot hub create --name {your iot hub name} --resource-group {your resource group name} --sku S1
```


### 1.2 Create an Azure Container Registry
The second resources we need to create is an Azure Container Registry. In this container registry we will store our IoT Edge modules

**Create an Azure Container Registry**    
```
az acr create --resource-group {your resource group name} --name {your container registry name} --sku Basic
```

**Learn more about these resources on MS Docs**    
- [Create an IoT hub using the CLI on MS Docs](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-using-cli)
- [Create a Container Registry using the CLI on MS Docs](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-azure-cli)


## Part 2 - Setup the Nvidia Jetson Nano

In this part we are going to configure our Jetson Nano device to run the 3 IoT Edge modules we are going to build later on. 



The easiest way to follow along with this walk-through is to connect with SSH to your Jetson Nano.

### 2.1 Install the latest operating system   
Install the latest version of the operating system on the Jetson Nano. The Nvidia learn website has a great tutorial for that.
[Follow the instructions here](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit). When the device boots and the desktop appears on the screen you can continue with the next step.


### 2.2 Install IoT Edge on the Jetson Nano
ARM64 builds of IoT Edge are currently being offered in preview and will eventually go into General Availability. We will make use of the ARM64 builds to ensure that we get the best performance out of our IoT Edge solutions.

Run the following from a terminal on your Nvidia Jetson device:

```
# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

# Download and install the standard libiothsm implementation
curl -L https://github.com/Azure/azure-iotedge/releases/download/1.0.8-rc1/libiothsm-std_1.0.8.rc1-1_arm64.deb -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb

# Download and install the IoT Edge Security Daemon
curl -L https://github.com/Azure/azure-iotedge/releases/download/1.0.8-rc1/iotedge_1.0.8.rc1-1_arm64.deb -o iotedge.deb && sudo dpkg -i ./iotedge.deb

# Run apt-get fix
sudo apt-get install -f
```

Get your connection string from IoT Edge in Azure. You can 

```
# First add the azure-iot extension.
az extension add --name azure-iot

# Create a new IoT Edge device
az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled

# List all IoT Edge devices
az iot hub device-identity list --hub-name [hub name]

# Get the Connection string
az iot hub device-identity show-connection-string --device-id [device id] --hub-name [hub name]
```


Once you have obtained a connection string, open the configuration file:

```
sudo nano /etc/iotedge/config.yaml
```

Find the provisioning section of the file and uncomment the manual provisioning mode. Update the value of device_connection_string with the connection string from your IoT Edge device.

```
provisioning:
  source: "manual"
  device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"

# provisioning: 
#   source: "dps"
#   global_endpoint: "https://global.azure-devices-provisioning.net"
#   scope_id: "{scope_id}"
#   registration_id: "{registration_id}"
```

You will also want to configure the default IoT Edge agent configuration to pull the 1.0.8-rc1 version of the agent. While in the configuration file, scroll down to the agent section and update the image value to the following:

```
agent:
  name: "edgeAgent"
  type: "docker"
  env: {}
  config:
    image: "mcr.microsoft.com/azureiotedge-agent:1.0.8-rc1"
    auth: {}
```

Restart the iot-edge service
```
service iotedge restart
```

See if your device is connected to the IoT Hub
```
az iot hub device-identity show --device-id [device id] --hub-name [hub name]
```

Look in the response for:
```
"connectionState": "Connected"
```

**More details on setting up Azue IoT Edge on the Jetson Nano**
- [Register an IoT Edge device on MS Docs](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-register-device)
- [You can find an extended tutorial here](https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl)


### 2.3 Enable GPU support in Docker
AI models run faster if they can use the power of a GPU. To get the support for the GPU in Docker on the Jetson Nano you have to adjust a few settings.

**Disable the UI**    
By default the Nano runs a visual interface. This takes up resources, which are needed to run the AI models.
```
sudo systemctl set-default multi-user.target
```

**Get more power**      
Set the Nano in high-power (10W) mode:
```
sudo nvpmodel -m 0
```

**NVidia runtime**    
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

**Run your updates**    
Update your Nano OS and packages to the latest versions
```
sudo apt-get update
sudo apt-get dist-upgrade
```

**Add current user to docker group**   
Add current user to docker group to use docker command without sudo, 
following this guide: https://docs.docker.com/install/linux/linux-postinstall/. 
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

**Reboot your device**       
```
sudo reboot
``` 

**Test you GPU support**      
```
docker run -it jitteam/devicequery ./deviceQuery
```
The response should look like this:
![GPU Support](https://raw.githubusercontent.com/hnky/blog/master/images/ml_iot_cuda_test.jpg)
  
*If the last line doesn't say "Result = PASS"  follow the instructions on screen to enable GPU support in Docker.*
  
Now you are ready to run Docker containers that support Tensorflow with GPU.

## 3. Create the IoT Edge modules

Start with cloning the repository that contains the IoT Edge modules and deployment config.
```
git clone https://github.com/hnky/iot-edge-custom-vision-jetson-nano

```

The repo contains the follow structure:
- Modules => The folder that holds 3 IoT Edge modules
- deployment.template.json => Template from which the deployment config is created.
- .env => your secrets


### 3.1 Custom Vision Module 
This module container an application that exposes a model created with the Custom Vision Service through an API on port 80. This container contains a simple model with 3 classes: Apple, Banana or Negative.

**Build the container and push to ACR**
```
cd modules\customvisionmodule
docker build . -f Dockerfile.arm64v8 -t henkboelman.azurecr.io/customvisionmodule:latest-arm64v8
docker push henkboelman.azurecr.io/customvisionmodule:latest-arm64v8
```

**Test the container**
```
docker run XXXXXX
```

If you want to create your own model, I have written an extended article 
[Read more](https://medium.com/microsoftazure/running-a-gpu-enabled-azure-custom-vision-docker-container-on-a-nvidia-jetson-nano-db8747b00b4f)


### 3.2 Camera Module 
A module 2 that grabs camera frames, send the images to the computer vision module and put the result on the local IoT hub.
```
docker build . -f Dockerfile.arm64v8 -t henkboelman.azurecr.io/cameramodule:latest-arm64v8
docker push henkboelman.azurecr.io/cameramodule:latest-arm64v8
```

### 3.3 The Alert Module 
that grabs the results of the local IoT hub and send it to the IoT hub in Azure
```
docker build . -f Dockerfile.arm64v8 -t henkboelman.azurecr.io/alertmodule:latest-arm64v8
docker push henkboelman.azurecr.io/alertmodule:latest-arm64v8
```


## Deploy the modules to the IoT Edge


** Work in progress **

- https://github.com/janza/docker-python3-opencv/blob/master/Dockerfile
- https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-edge-scenarios/send_message_to_output.py
- https://github.com/toolboc/IntelligentEdgeHOL
- https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl
- https://www.youtube.com/watch?v=_K5fqGLO8us
- https://www.youtube.com/watch?v=gMJgsQ13SKs&feature=youtu.be
[Extended information on running GPU enabled Custom Vision on the Jetson Nano](https://medium.com/microsoftazure/running-a-gpu-enabled-azure-custom-vision-docker-container-on-a-nvidia-jetson-nano-db8747b00b4f)
