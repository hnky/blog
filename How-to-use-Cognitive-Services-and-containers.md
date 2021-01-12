# How to use Cognitive Services and containers

*intro*

## What are Cognitive Services
Azure Cognitive Services are cloud-based services that expose AI models through a REST API. These services enable you to add cognitive features, like object detection and speech recognition to your applications without having data science skills. By using the provided SDKs in the programming language of your choice you can create application that can see (Computer Vision), hear (Speech), speak (Speech), understand (Language), and even make decisions (Decision).

## Cognitive Services in containers
Azure Cognitive Service in containers gives developers the flexibility in where to deploy and host the services that come with Docker containers and keeping the same API experience as when they where hosted in the Azure.

Using these containers gives you the flexibility to bring Cognitive Services closer to your data for compliance, security or other operational reasons.

> **What are containers**   
> Containerization is an approach to software distribution in which an application or service, including its dependencies & configuration, is packaged together as a container image. With little or no modification, a container image can be deployed on a container host. Containers are isolated from each other and the underlying operating system, with a smaller footprint than a virtual machine. Containers can be instantiated from container images for short-term tasks, and removed when no longer needed.  

[Embed Video](https://www.youtube.com/watch?v=hdfbn4Q8jbo)


## When to use Cognitive Services in containers?
Running Cognitive Services in containers can be the solution for you if you have specific requirements or constraints making that make it impossible to run Cognitive services in Azure. The most common scenarios include connectivity and control over the data. If you are running Cognitive Services in Azure all the infrastructure is taken care of, running them in containers moves the infrastructure responsibility, like performance and updating the container, to you.

A case where you choose for container could be, if your connection to Azure is not stable enought. For instance if you have 1000's of document on-prem and you want to run OCR. If you use the Computer Vision OCR endpoint in the cloud you would need to send all the documents to the end point in azure, while if you run the container localy you only need to send the billing information every 15 minutes to Azure.

### Features and benefits

**Immutable infrastructure:** Enable DevOps teams' to leverage a consistent and reliable set of known system parameters, while being able to adapt to change. Containers provide the flexibility to pivot within a predictable ecosystem and avoid configuration drift.

**Control over data:** Choose where your data gets processed by Cognitive Services. This can be essential if you can't send data to the cloud but need access to Cognitive Services APIs. Support consistency in hybrid environments – across data, management, identity, and security.

**Control over model updates:** Flexibility in versioning and updating of models deployed in their solutions.
Portable architecture: Enables the creation of a portable application architecture that can be deployed on Azure, on-premises and the edge. Containers can be deployed directly to Azure Kubernetes Service, Azure Container Instances, or to a Kubernetes cluster deployed to Azure Stack. For more information, see Deploy Kubernetes to Azure Stack.

**High throughput / low latency:** 
Provide customers the ability to scale for high throughput and low latency requirements by enabling Cognitive Services to run physically close to their application logic and data. Containers do not cap transactions per second (TPS) and can be made to scale both up and out to handle demand if you provide the necessary hardware resources.

**Scalability:** With the ever growing popularity of containerization and container orchestration software, such as Kubernetes; scalability is at the forefront of technological advancements. Building on a scalable cluster foundation, application development caters to high availability.


### Which services are availabe
Container support is currently available for a subset of Azure Cognitive Services, including parts of:

| Group | Service | Documentation | Preview |
| -- | -- | -- | -- |
| Anomaly Detector | Anomaly Detector | [Documentation](https://docs.microsoft.com/azure/cognitive-services/anomaly-detector/anomaly-detector-container-howto?WT.mc_id=aiml-12167-heboelma) |
| Computer Vision | Read OCR (Optical Character Recognition) | [Documentation](https://docs.microsoft.com/azure/cognitive-services/computer-vision/computer-vision-how-to-install-containers?WT.mc_id=aiml-12167-heboelma)
| | Spatial Analysis | [Documentation](https://docs.microsoft.com/azure/cognitive-services/computer-vision/spatial-analysis-container?tabs=azure-stack-edge&WT.mc_id=aiml-12167-heboelma) 
| Form Recognizer | Form Recognizer | [Documentation](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/form-recognizer-container-howto?) |
| Language Understanding | Language Understanding | [Documentation](https://docs.microsoft.com/azure/cognitive-services/luis/luis-container-howto?WT.mc_id=aiml-12167-heboelma)| 
| Speech | Custom Speech-to-text | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=cstt&WT.mc_id=aiml-12167-heboelma) |
| | Custom Text-to-speech | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=ctts&WT.mc_id=aiml-12167-heboelma) |
| | Speech-to-text |  [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=stt&WT.mc_id=aiml-12167-heboelma) |
| | Text-to-speech | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=tts&WT.mc_id=aiml-12167-heboelma) |
| | Neural Text-to-speech | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=ntts&WT.mc_id=aiml-12167-heboelma) |
| | Speech language detection | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=lid&WT.mc_id=aiml-12167-heboelma) |
| Text Analytics | Key Phrase Extraction | [Documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers?tabs=keyphrase&WT.mc_id=aiml-12167-heboelma)
| | Text language detection | [Documentation](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers?tabs=language&WT.mc_id=aiml-12167-heboelma) |
| | Sentiment analysis | [Documentation](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers?tabs=sentiment&WT.mc_id=aiml-12167-heboelma) |
| Face | Face |  [Documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/face/face-how-to-install-containers?&WT.mc_id=aiml-12167-heboelma)


### Billing & Costs
Running a Cognitive Service in a container doesn't mean that you don't need an Azure Resource or the containers needs connectivity to the endpoint in Azure. 

The container needs to reports it usage about every 10 to 15 minutes. [The costs](https://azure.microsoft.com/pricing/details/cognitive-services/?&WT.mc_id=aiml-12167-heboelma) for using the service in a container is the same as using the endpoint in Azure. 

If the container doesn't connect to Azure within the allowed time window, the container continues to run but doesn't serve queries until the billing endpoint is restored. The connection is attempted 10 times at the same time interval of 10 to 15 minutes. If it can't connect to the billing endpoint within the 10 tries, the container stops serving requests.

## How to use Cognitive Services in containers
The use of the services in containers is exactly the same as if you would use them in Azure. The deployment of the container is the part tahat takes a bit of planning and research. The services are shipped in Docker Containers. This means that the containers can be deployed to any Docker compatible platform. This can be your local machine running Docker Desktop or a fully scalable Kubernetes installation in your on premise data center.


### Generenic workflow
- Create the resource in Azure
- Get the endpoint 
- Retrieve the API Key
- Find the container for the service
- Deploy the container
- Use the container endpoint as you would use the API resource

Optional you can mount your own storage and connect [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview?WT.mc_id=aiml-12167-heboelma).


## Tutorial: Run a Text to Speech container in an Azure Container Instance.
In this tutotial we are going to run a Cognitive Service Speech container in an Azure Container Instance and use the REST api to convert text into speech.

To run the code below you need an Azure Subscription. if you don’t have an [Azure subscription](https://azure.microsoft.com/free/?WT.mc_id=aiml-12167-heboelma) you can get $200 credit for the first month. And have the [Azure command-line interface](https://docs.microsoft.com/cli/azure/what-is-azure-cli?WT.mc_id=aiml-12167-heboelma) installed. If you don't have the Azure CLI installed [follow this tutorial](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?WT.mc_id=aiml-12167-heboelma).


### 1. Create a resource group
Everything in Azure always start with creating a Resource Group. A resource group is a resource that holds related resources for an Azure solution.

To create a resource group using the CLI you have to specify 2 parameters, the name of the group and the location where this group is deployed.
```
az group create --name demo_rg --location westeurope
```

### 2. Create Cognitive Service resource
The next resource that needs to be created is a Cognitive Services. To create this resource we need to specify a few parameters. Besides the name and resource group, you need to specify the kind of cognitive service you want to create. For our tutorial we are creating a 'SpeechServices' service.

```
az cognitiveservices account create \
    --name speech-resource \
    --resource-group demo_rg \
    --kind SpeechServices \
    --sku F0 \
    --location westeurope \
    --yes
```

### 3. Get the endpoint & API Key
If step 1 and 2 are successfully deployed we can extract the properties we need for when we are going to run the container in the next step. The 2 properties we need are the endpoint URL and the API key. The speech service in the container is using these properties to connect to Azure every 15 minutes to send the billing information.

To retieve endpoint:
```
az cognitiveservices account show --name speech-resource --resource-group demo_rg  --query properties.endpoint -o json
```

To retieve the API keys:
```
az cognitiveservices account keys list --name speech-resource --resource-group demo_rg
```

### 3. Deploy the container in an ACI
One of the easiest ways to run a container is to use [Azure Container Instances](https://docs.microsoft.com/azure/container-instances/?WT.mc_id=aiml-12167-heboelma). With one command in the Azure CLI you can deploy a container and make it accessible for the everyone. 

To create an ACI it take a few parameters. If you want your ACI to be accessible from the internet you need to specify the parameter: '--dns-name-label'. The url for the ACI will look like this: http://{dns-name-label}.{region}.azurecontainer. The dns-name-label property needs to be unique.

```
az container create 
    --resource-group demo_rg \
    --name speechcontainer \
    --dns-name-label <insert unique name> \
    --memory 2 --cpu 1 \
    --ports 5000 \
    --image mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest \
    --environment-variables \
        Eula=accept 
        Billing=<insert endpoint> 
        ApiKey=<insert apikey>
```

The deployment of the container takes a few minutes.


### 4. Validate that a container is running

The easiest way to validate if the container is running, is to use a browser and open the container homepage. 
To do this you first need to retrieve the url for the container. This can be done using the Azure CLI with the following command.
```
az container show --name speechcontainer --resource-group demo_rg --query ipAddress.fqdn -o json
```
Navigate to the URL on port 5000. The url should look like this: *http://{dns-name-label}.{region}.azurecontainer.io:5000/*

If everything went well you should see a screen like this:
![Homepage of a Cognitive Service Speech Container](https://raw.githubusercontent.com/hnky/blog/master/images/cog_con/container_is_running.png)
   


### 5. Submit your first task
The Text to Speech service in the container is a REST endpoint. To use it we would need to create a POST request. There are many ways to do a POST request. For our tutorial we are going to use Visual Studio Code to do this. 

**Requirements**:
- [Download](https://code.visualstudio.com/?WT.mc_id=aiml-12167-heboelma) and Install Visual Studio Code
- Install a plugin called [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client&WT.mc_id=aiml-12167-heboelma)

If you have the Visual Studio Code with th REST Client installed create a file call: rest.http and copy past the code below in the file.

```
POST http://<dns-name-label>.<region>.azurecontainer.io:5000/speech/synthesize/cognitiveservices/v1  HTTP/1.1
Content-Type: application/ssml+xml
X-Microsoft-OutputFormat: riff-24khz-16bit-mono-pcm
Accept: audio/*

<speak version="1.0" xml:lang="en-US">
    <voice name="en-US-AriaRUS">
        The future we invent is a choice we make. 
        Not something that just happens.
    </voice>
</speak>
```
- Change the name of the URL to the URL of your ACI.
- Next click on the Send Request link (just above the URL)

On the right side of VS Code you should see the response of the API. In the top right corner you see "Save Response Body" click on the button and save the response as a .wav file. Now you can use any media player to play the response.

![Visual Studio Code REST Call](https://raw.githubusercontent.com/hnky/blog/master/images/cog_con/vscode_api_response.png)

## On Learn more

### Microsoft Learn
Microsoft Learn is a free, online training platform that provides interactive learning for Microsoft products and more. 

For this blog we have created a custom [Collection of Learn Modules](https://aka.ms/ai/learn/cognitive-containers) covering all the topics in depth.

### Blogs and articles
- [Getting started with Azure Cognitive Services in containers](https://azure.microsoft.com/blog/getting-started-with-azure-cognitive-services-in-containers/?WT.mc_id=aiml-12167-heboelma)
- [Bringing AI to the edge](https://azure.microsoft.com/blog/bringing-ai-to-the-edge/?WT.mc_id=aiml-12167-heboelma)
- [Running Cognitive Service containers](https://azure.microsoft.com/blog/running-cognitive-service-containers/?WT.mc_id=aiml-12167-heboelma)

### On Microsoft Docs

- [Azure Cognitive Services containers](https://docs.microsoft.com/azure/cognitive-services/containers?WT.mc_id=aiml-12167-heboelma)
- [Azure Cognitive Services containers Support](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support?WT.mc_id=aiml-12167-heboelma)
- [Create containers for reuse](https://docs.microsoft.com/azure/cognitive-services/containers/container-reuse-recipe?WT.mc_id=aiml-12167-heboelma)
- [Deploy and run container on Azure Container Instance](https://docs.microsoft.com/azure/cognitive-services/containers/azure-container-instance-recipe?WT.mc_id=aiml-12167-heboelma)
- [Deploy the Text Analytics language detection container to Azure Kubernetes Service](https://docs.microsoft.com/azure/cognitive-services/containers/azure-kubernetes-recipe?WT.mc_id=aiml-12167-heboelma)
- [Use Docker Compose to deploy multiple containers](https://docs.microsoft.com/azure/cognitive-services/containers/docker-compose-recipe?WT.mc_id=aiml-12167-heboelma)
- [Azure Cognitive Services containers frequently asked questions (FAQ)](https://docs.microsoft.com/azure/cognitive-services/containers/container-faq?WT.mc_id=aiml-12167-heboelma)
