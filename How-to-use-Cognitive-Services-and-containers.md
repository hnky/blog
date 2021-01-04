# How to use Cognitive Services and containers


## What are cognitive services
*Intro to cognitive services*

## Cognitive Services in containers
*Intro to cognitive services in containers*
*copied from docs*  Container support in Azure Cognitive Services allows developers to use the same rich APIs that are available in Azure, and enables flexibility in where to deploy and host the services that come with Docker containers.

### Which services are availabe
| Group | Service | Documentation | Preview |
| -- | -- | -- | -- |
| Anomaly Detector | Anomaly Detector | [Documentation](https://docs.microsoft.com/azure/cognitive-services/anomaly-detector/anomaly-detector-container-howto) |
| Computer Vision | Read OCR (Optical Character Recognition) | [Documentation](https://docs.microsoft.com/azure/cognitive-services/computer-vision/computer-vision-how-to-install-containers)
| Form Recognizer | Form Recognizer | [Documentation](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/form-recognizer-container-howto) |
| Language Understanding | Language Understanding | [Documentation](https://docs.microsoft.com/azure/cognitive-services/luis/luis-container-howto)|  
| Speech | Custom Speech-to-text | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=cstt) |
| | Custom Text-to-speech | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=ctts) |
| | Speech-to-text |  [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=stt) |
| | Text-to-speech | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=tts) |
| | Neural Text-to-speech | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=ntts) |
| | Speech language detection | [Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-container-howto?tabs=lid) |
| Text Analytics | Key Phrase Extraction | 
| | Text language detection | [Documentation](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers?tabs=language) |
| | Sentiment analysis | [Documentation](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers?tabs=sentiment) |


### Billing
*copied from docs* The container needs the billing argument values to run. These values allow the container to connect to the billing endpoint. The container reports usage about every 10 to 15 minutes. If the container doesn't connect to Azure within the allowed time window, the container continues to run but doesn't serve queries until the billing endpoint is restored. The connection is attempted 10 times at the same time interval of 10 to 15 minutes. If it can't connect to the billing endpoint within the 10 tries, the container stops serving requests.


### Use cases
*List of some use cases why you want to use these sevices in containers vs api*


## How to use Cognitive Services in containers
*Here we zoom in how to use the containers*

### The different options
*Over view of the different options*

### Run on you local Machine
*Tutorial for local machine*

### Azure Container Instance
*Tutorial for ACI machine*

### Azure Kubernetes Services
*Tutorial for AKS machine*

## Further reading
- [Getting started with Azure Cognitive Services in containers](https://azure.microsoft.com/blog/getting-started-with-azure-cognitive-services-in-containers/)
- [Bringing AI to the edge](https://azure.microsoft.com/blog/bringing-ai-to-the-edge/)
- [Running Cognitive Service containers](https://azure.microsoft.com/blog/running-cognitive-service-containers/)

## Learn more on Microsoft Docs

### Services
- [Text Analytics](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers)

### General
- [x](https://docs.microsoft.com/azure/cognitive-services/containers?WT.mc_id=aiml-12167-heboelma)
- [x](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support?WT.mc_id=aiml-12167-heboelma)
- [x](https://docs.microsoft.com/azure/cognitive-services/containers/container-reuse-recipe?WT.mc_id=aiml-12167-heboelma)
- [x](https://docs.microsoft.com/azure/cognitive-services/containers/azure-container-instance-recipe?WT.mc_id=aiml-12167-heboelma)
- [x](https://docs.microsoft.com/azure/cognitive-services/containers/azure-kubernetes-recipe?WT.mc_id=aiml-12167-heboelma)
- [x](https://docs.microsoft.com/azure/cognitive-services/containers/docker-compose-recipe?WT.mc_id=aiml-12167-heboelma)
- [x](https://docs.microsoft.com/azure/cognitive-services/containers/container-faq?WT.mc_id=aiml-12167-heboelma)

### Usefull services






-------
## Internal notes

Henk: Demos use Azure CLI


## Todo:
- Add tracking




