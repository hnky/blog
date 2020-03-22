# Online MeetUp using OBS and Skype

In a previous blog I talked about how you can stream your MeetUp from a phisical location. But now that we live in a time where meetups kan not take place phisical we need to move our meetups online. 

There are lots of ways of doing it. Definately the easiest ways are create a live teams meeting or a public available zoom meeting. But if you want to have the same control over your layout as you do when streaming your meetup live this blog is for you.

Let's first talk about hwo to run a regular meetup, with a regular meetup I mean you have one host, talk introduces the speakers, handles questions and two or three speakers. 

We dive in runnign a conference online a bit later, when the Global AI Community On virtual tour is done.

## Streaming location
This is the first thing you have to think about. Can you stream from your home in full HD 30 frames per second? Does your internet provider likes that, do you have wired in your house or wireless. Trust me, lots of things can go break when streaming from your home. To avoid all this hassle setup a virtual machine in the cloud. Yes a VM with enough specs can be expensive if you look at the price per month, but remember you only stream for a few hours. A decent VM costs around 1 / 2 euro a hour, just don't forget to shut it down when your are done.

### Step 1
Create a Virtual Machine in the cloud, running Windows 10. 
Use this template to create VM in Azure.

## Streaming software
Now that we have a machine to run our streaming software on, we need to to install the software that will enable us to stream to an online platform. My pick is OBS (Open Broadcaster Software). There are multiple versions available. There is a version from StreamLabs and there is an opensource verion OBS Studio. At this time of writing you have to pick the opensource OBS Studio, because later we are using a few plugins that only work correctly in this version.

### Step 2
Install OBS Studio in your VM.
[Download OBS Studio for Windows 10](https://obsproject.com/download)

## Connection with your host and speakers
To connect with our host and speakers we are going to use Skype for Windows. Not Skype for Business, not Skype for Windows 10. No Skype for Windows. In this version of skype you can enable NDI (....).

### Step 3
Download and install Skype
Enable NDI support in Skype.

## Enable NDI Support in OBS
To get access to all the individual video feeds from Skype in OBS you have to install the NDI Plugin for OBS.

### Step 4
Download and install NDI support for OBS
- 1. [Download and install the NDI 4.0+ Runtime ](http://new.tk/NDIRedistV4) 
- 2. [Download and install the latest: obs-ndi-X.X.X-Windows-Installer.exe](https://github.com/Palakis/obs-ndi/releases)
- 3. Reboot your VM







