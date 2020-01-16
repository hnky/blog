# Object alert with Custom Vision and IoT Edge on the Jetson Nano

In this article I will guide throught the steps needed to create your own edge device that can send alerts when an object is detect in a camera feed on the Jetson Nano. The goal is to do the processing of the frames localy and only send a message to the cloud when the detected object hits a certain confidence tresshold.

[Insert graphic]


## What do we need to build?
To get this working we need to:
- Setup IoT Edge in Azure
- Configure the Jetson Nano
  - Install IoT Edge
  - Configure the device to run our custom vision modules.
- Create 3 IoT Edge module
  - A module 1 that runs the our computer vision model
  - A module 2 that grabs camera frames, send the images to the computer vision module and put the result on the local IoT hub.
  - A module 3 that grabs the results of the local IoT hub and send it to the IoT hub in Azure
- Deploy the modules to the IoT Edge
- Setup an Event Grid and Logic app to handle the alerts.


## Setup IoT Edge in Azure


## 2. Configure the Jetson Nano

In this part we are going to configure our Jetson Nano device to run the 3 IoT Edge modules we are going to build later on. 

To continue you need an [Nvidia Jetson Nano](https://developer.nvidia.com/embedded/jetson-nano-developer-kit) and an USB camera.

### 2.1 Install the latest operating system
Install the latest version of the operating system on the Jetson Nano. The Nvidia learn website has a great tutorial for that.
[Follow the instructions here](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit). When the device boots and the desktop appears on the screen you can continue with the next step.


### 2.2 Install IoT Edge on the Jetson Nano
ARM64 builds of IoT Edge are currently being offered in preview and will eventually go into General Availability. We will make use of the ARM64 builds to ensure that we get the best performance out of our IoT Edge solutions.

These builds are provided starting in the 1.0.8-rc1 release tag. To install the 1.0.8-rc1 release of IoT Edge, run the following from a terminal on your Nvidia Jetson device:

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

[You can find an extended tutorial here](https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl)

## Create 3 IoT Edge module


## Deploy the modules to the IoT Edge


** Work in progress **

- https://github.com/janza/docker-python3-opencv/blob/master/Dockerfile
- https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-edge-scenarios/send_message_to_output.py
- https://github.com/toolboc/IntelligentEdgeHOL
- https://dev.to/azure/getting-started-with-iot-edge-development-on-nvidia-jetson-devices-2dfl
- https://www.youtube.com/watch?v=_K5fqGLO8us
- https://www.youtube.com/watch?v=gMJgsQ13SKs&feature=youtu.be
