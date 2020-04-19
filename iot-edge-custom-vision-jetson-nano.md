# Create your own vision alerting system with IoT Edge, Azure Custom Vision and a Jetson Nano

In this article I will guide you throught the steps needed to create your own object alertings system running on an edge device. For this we will use a NVidia Jetson Nano, the Azure Custom Vision service and Azure IoT Edge.

The goal is to process the camera frames localy on the Jetson Nano and only send a message to the cloud when the detected object hits a certain confidence threshold.

[Insert graphic]

**Requirements before you starting:**
- You need a [Nvidia Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit)
- An USB camera.
- An Azure Subscription ([Get started for free here](https://azure.microsoft.com/en-us/free/?WT.mc_id=aiapril-blog-heboelma))


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
To get started we need to setup a few resources in Azure. For this we are going to use the [Azure CLI](https://docs.microsoft.com/cli/azure/?WT.mc_id=aiapril-blog-heboelma). If you don't have the Azure CLI installed on your machine you can follow the [tutorial on MS Docs here](https://docs.microsoft.com/cli/azure/install-azure-cli?WT.mc_id=aiapril-blog-heboelma).


### 1.1 Create an Azure IoT Hub
The first resources we need is an IoT Hub. We will use this Hub to communicate with our Edge Device. It will give us the ability to deploy our IoT Edge models to the device and give the Edge device the ability to send messages back to the cloud. 

**Create a Resource group**
```
az group create --name [resource group name] --location westeurope
```

**Create an IoT Hub**
```
az iot hub create --name [iot hub name] --resource-group [resource group name] --sku S1
```


### 1.2 Create an Azure Container Registry
The second resource we need to create is an Azure Container Registry. In this container registry we will store our IoT Edge modules

**Create an Azure Container Registry**    
```
az acr create --resource-group [resource group name] --name [container registry name] --sku Basic
```

**Login to your Azure Container Registry**
First use the CLI to get the credentials from the ACR.
```
az acr update -n [container registry name] --admin-enabled true
az acr credential show -n [container registry name]
```
You will need these credentials in part 3.

**Learn more about these resources on MS Docs**    
- [Create an IoT hub using the CLI on MS Docs](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-using-cli?WT.mc_id=aiapril-blog-heboelma)
- [Create a Container Registry using the CLI on MS Docs](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-azure-cli?WT.mc_id=aiapril-blog-heboelma)


## Part 2 - Setup the Nvidia Jetson Nano

In this part we are going to prepare our Jetson Nano device to run the 3 IoT Edge modules we are going to build later on. 

*The easiest way to follow along with this walk-through is to connect with SSH to your Jetson Nano.*

### 2.1 Install the latest operating system   
Install the latest version of the operating system on the Jetson Nano. The Nvidia learn website has a great tutorial for that.
[Follow the instructions here](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit). When the device boots and the desktop appears on the screen you can continue with the next step.

### 2.2 Install IoT Edge on the Jetson Nano
ARM64 builds of IoT Edge are currently being offered in preview and will eventually go into General Availability. We will make use of the ARM64 builds to ensure that we get the best performance out of our IoT Edge solutions.

Run the code below on the Nvidia Jetson device to install IoT Edge

```
# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
sudo apt-get -y install libssl-dev libffi-dev jq python-pip
pip install iotedgedev
sudo mv ~/.local/bin/iotedgedev /usr/local/bin

# Download and install the standard libiothsm implementation
curl -L https://github.com/Azure/azure-iotedge/releases/download/1.0.8-rc1/libiothsm-std_1.0.8.rc1-1_arm64.deb -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb

# Download and install the IoT Edge Security Daemon
curl -L https://github.com/Azure/azure-iotedge/releases/download/1.0.8-rc1/iotedge_1.0.8.rc1-1_arm64.deb -o iotedge.deb && sudo dpkg -i ./iotedge.deb

# Run apt-get fix
sudo apt-get install -f
```

Now that IoT Edge is installed we need to connect the device to the IoT Hub.
For this we need to setup a device in the IoT hub and retrieve the connection string. We can retrieve this connection string using the Azure CLI with the azure-iot extension.

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

Once you have obtained the connection string, open the configuration file on the Jetson Nano:

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

**More details on setting up Azure IoT Edge on the Jetson Nano**
- [Register an IoT Edge device on MS Docs](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-register-device?WT.mc_id=aiapril-blog-heboelma)
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

**Test your GPU support**      
```
docker run -it jitteam/devicequery ./deviceQuery
```
The response should look like this:
![GPU Support](https://raw.githubusercontent.com/hnky/blog/master/images/ml_iot_cuda_test.jpg)
  
*If the last line doesn't say "Result = PASS"  follow the instructions on screen to enable GPU support in Docker.*
  
Now you are ready to run Docker containers that support Tensorflow with GPU.



## 3. Create the IoT Edge modules

Start with cloning the repository that contains the sample IoT Edge modules and deployment config.

```
git clone https://github.com/hnky/iot-edge-custom-vision-jetson-nano
```

The repo has the following structure:
- modules => The folder that holds 3 IoT Edge modules
- deployment.template.json => Template from which the deployment config is created.
- .env => your secrets

*Building the containers can take a while*


**Login to your ACR**
First we need to login to our Container Registery on the Jetson Nano
```
docker login [container registry name].azurecr.io
```


### 3.1 Custom Vision Module 
This module contains a small webserver that exposes a model created with the Azure Custom Vision service through an API on port 80. This container contains a simple model with 3 classes: Apple, Banana or Negative.

**Build the container and push to ACR**
```
cd modules/CustomVisionModule
docker build . -f Dockerfile.arm64v8 -t [container registry name].azurecr.io/customvisionmodule:latest-arm64v8
docker push [container registry name].azurecr.io/customvisionmodule:latest-arm64v8
```

**Test the container**
```
# Run the container
docker run -p 127.0.0.1:80:80 -d [container registry name].azurecr.io/customvisionmodule:latest-arm64v8

# Send a test image of a banana to the containers
curl -X POST http://127.0.0.1/url -d '{ "url": "https://www.wievultuwbroodtrommel.nl/205-large_default/banaan.jpg"}'
```

**Stop the container**
```
# List all the containers and locate the Container ID
docker container list

# Stop the container
docker container stop [container id]
```

**Learn more**
If you want to learn how to create your own model and run it as a container on the Jetson Nano you can read 
[this blog ](https://medium.com/microsoftazure/running-a-gpu-enabled-azure-custom-vision-docker-container-on-a-nvidia-jetson-nano-db8747b00b4f).


### 3.2 Camera Module 
This module grabs the camera frames and send the images to the Computer vision module and put the result on the local IoT hub.
```
cd modules/CameraModule
docker build . -f Dockerfile.arm64v8 -t [container registry name].azurecr.io/cameramodule:latest-arm64v8
docker push [container registry name].azurecr.io/cameramodule:latest-arm64v8
```


### 3.3 Alert Module 
This model grabs the result of the local IoT hub and send it to the IoT hub in Azure if the treshold of 60% is reached for one of the classes.
```
cd modules/AlertModule
docker build . -f Dockerfile.arm64v8 -t [container registry name].azurecr.io/alertmodule:latest-arm64v8
docker push [container registry name].azurecr.io/alertmodule:latest-arm64v8
```




## 4. Deploy the modules to the IoT Edge

**Create a deployment**

Update the .env file with your credentials.
```
nano .env
```

Generate the deployment manifest from the deployment.template.json.
```
iotedgedev genconfig -f deployment.template.json -P arm64v8
```

Copy the generated manifest file 'config/deployment.config' to your computer running the Azure CLI.

```
az iot edge set-modules --device-id [device id] --hub-name [hub name] --content deployment.config
```

Check on the Jetson Nano if the modules are up and running.
```
iotedge list
```

To see the messages that the alert module is sending you can use the command below 
```
iotedge logs alert-module
```

To view the output of the camera-module you can open a webbrowser and enter the ip of the Jetson Nano on port 5012 (http://x.x.x.x:5012/
). You will see a small preview of the captured frame.


**Learn more about module composition and deployment**
- [Learn how to deploy modules and establish routes in IoT Edge](https://docs.microsoft.com/azure/iot-edge/module-composition?WT.mc_id=aiapril-blog-heboelma)
- [Develop your own IoT Edge modules](https://docs.microsoft.com/azure/iot-edge/module-development?WT.mc_id=aiapril-blog-heboelma)
- [Deploy Azure IoT Edge modules with Azure CLI](https://docs.microsoft.com/azure/iot-edge/how-to-deploy-modules-cli?WT.mc_id=aiapril-blog-heboelma)



## 5. Create a Logic app that send an Alert

### 5.1 Create a logic app resource






```
[
  {
    "id": "fc52510d-e9b7-da86-c7af-c05b7b820554",
    "topic": "/SUBSCRIPTIONS/431DBAE5-40CA-438A-8DAF-77D7D5580B41/RESOURCEGROUPS/AIAPRIL_IOT/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/AIAPRILIOTHUB",
    "subject": "devices/AIAprilDevice/alert-module",
    "eventType": "Microsoft.Devices.DeviceTelemetry",
    "eventTime": "2020-04-19T18:52:03.247Z",
    "data": {
      "properties": {},
      "systemProperties": {
        "correlation-id": "test-1234",
        "message-id": "d2b805e5-cef5-4eea-a9fc-536516408bfd",
        "iothub-content-type": "application/json",
        "iothub-content-encoding": "utf-8",
        "iothub-connection-device-id": "AIAprilDevice",
        "iothub-connection-module-id": "alert-module",
        "iothub-connection-auth-method": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "iothub-connection-auth-generation-id": "637228229021652362",
        "iothub-enqueuedtime": "2020-04-19T18:52:03.247Z",
        "iothub-message-source": "Telemetry"
      },
      "body": {
        "tag": "Apple"
      }
    },
    "dataVersion": "",
    "metadataVersion": "1"
  }
]
```





## Continue learning

- [Tutorial: Send email notifications about Azure IoT Hub events using Event Grid and Logic Apps](https://docs.microsoft.com/en-us/azure/event-grid/publish-iot-hub-events-to-logic-apps?WT.mc_id=aiapril-blog-heboelma)


**Further reading**
- https://dev.to/azure/getting-started-with-devops-ci-cd-pipelines-on-nvidia-arm64-devices-4668

** Work in progress **

- https://github.com/janza/docker-python3-opencv/blob/master/Dockerfile
- https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-edge-scenarios/send_message_to_output.py
- https://github.com/toolboc/IntelligentEdgeHOL
- https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl
- https://www.youtube.com/watch?v=_K5fqGLO8us
- https://www.youtube.com/watch?v=gMJgsQ13SKs&feature=youtu.be
[Extended information on running GPU enabled Custom Vision on the Jetson Nano](https://medium.com/microsoftazure/running-a-gpu-enabled-azure-custom-vision-docker-container-on-a-nvidia-jetson-nano-db8747b00b4f)
