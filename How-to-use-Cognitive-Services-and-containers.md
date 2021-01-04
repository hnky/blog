# How to use Cognitive Services and containers


## What are cognitive services
*Intro to cognitive services*

## Cognitive Services in containers
*Intro to cognitive services in containers*

### Billing
*copied from docs* The container needs the billing argument values to run. These values allow the container to connect to the billing endpoint. The container reports usage about every 10 to 15 minutes. If the container doesn't connect to Azure within the allowed time window, the container continues to run but doesn't serve queries until the billing endpoint is restored. The connection is attempted 10 times at the same time interval of 10 to 15 minutes. If it can't connect to the billing endpoint within the 10 tries, the container stops serving requests.


### Which services are availabe
*List of available services*

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

## Services on docs
- [Text Analytics](https://docs.microsoft.com/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-install-containers)
- [x]()
- [x]()

-------
## Internal notes

Henk: Demos use Azure CLI


## Todo:
- Add tracking




