# Exercise 04: Use an Azure AI Services Container

## Lab scenario

Using Azure AI services hosted in Azure enables application developers to focus on the infrastructure for their own code while benefiting from scalable services that are managed by Microsoft. However, in many scenarios, organizations require more control over their service infrastructure and the data that is passed between services.

Many of the Azure AI services APIs can be packaged and deployed in a container, enabling organizations to host Azure AI services in their own infrastructure; for example in local Docker servers, Azure Container Instances, or Azure Kubernetes Services clusters. Containerized Azure AI services need to communicate with an Azure-based Azure AI services account to support billing; but application data is not passed to the back-end service, and organizations have greater control over the deployment configuration of their containers, enabling custom solutions for authentication, scalability, and other considerations.

## Objectives

In this lab, you will complete the following tasks:

+ Task 1: Deploy and run a Text Analytics container
+ Task 2: Use the container

## Task 1: Deploy and run a Text Analytics container

Many commonly used Azure AI services APIs are available in container images. For a full list, check out the [Azure AI services documentation](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#container-availability-in-azure-cognitive-services). In this exercise, you'll use the container image for the Text Analytics *language detection* API; but the principles are the same for all of the available images.

1. In the Azure portal, on the **Home** page, select the **&#65291; Create a resource** button, search for *container instances*, and create a **Container Instances** resource with the following settings:

    - **Basics**:
        - **Subscription**: *Your Azure subscription*
        
        - **Resource group**: **ai-102-<inject key="DeploymentID" enableCopy="false"/>**
        
        - **Container name**: **aicontainer<inject key="DeploymentID" enableCopy="false"/>**

        - **Region**: *Choose any available region*
        
        - **Image source**: other Registry
        
        - **Image type**: Public
        
        - **Image**: `mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest`
        
        - **OS type**: Linux
        
        - **Size**: 1 vcpu, 12 GB memory
    
    - **Networking**:
        
        - **Networking type**: Public
        
        - **DNS name label**: **aiendpoint<inject key="DeploymentID" enableCopy="false"/>**
        
        - **Ports**: *Change the TCP port from 80 to **5000***
   
    - **Advanced**:
        
        - **Restart policy**: On failure
        
        - **Environment variables**:

            | Mark as secure | Key | Value |
            | -------------- | --- | ----- |
            | Yes | `ApiKey` | *Either key for your Azure AI services resource* |
            | Yes | `Billing` | *The endpoint URI for your Azure AI services resource* |
            | No | `Eula` | `accept` |

        - **Command override**: [ ]
    
    - **Tags**:
        
        - *Don't add any tags*

    - Select **Review + create**, and **Create**.

1. Wait for deployment to complete, and select **Go to resource**.
    
    >**Note:** Please note that deploying an Azure AI container to Azure Container Instances typically takes 5-10 minutes (provisioning) before they are ready to use.

1. Observe the following properties of your container instance resource on its **Overview** page:
    - **Status**: This should be *Running*.
    
    - **IP Address**: This is the public IP address you can use to access your container instances.
    
    - **FQDN**: This is the *fully-qualified domain name* of the container instances resource, you can use this to access the container instances instead of the IP address.

    > **Note**: In this exercise, you've deployed the Azure AI services container image for text translation to an Azure Container Instances (ACI) resource. You can use a similar approach to deploy it to a *[Docker](https://www.docker.com/products/docker-desktop)* host on your own computer or network by running the following command (on a single line) to deploy the language detection container to your local Docker instance, replacing *&lt;yourEndpoint&gt;* and *&lt;yourKey&gt;* with your endpoint URI and either of the keys for your Azure AI services resource.
    > The command will look for the image on your local machine, and if it doesn't find it there it will pull it from the *mcr.microsoft.com* image registry and deploy it to your Docker instance. When deployment is complete, the container will start and listen for incoming requests on port 5000.

    ```
    docker run --rm -it -p 5000:5000 --memory 12g --cpus 1 mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest Eula=accept Billing=<yourEndpoint> ApiKey=<yourKey>
    ```

## Task 2: Use the container

1. In Visual Studio Code, in the **04-containers** folder, open **rest-test.cmd** and edit the **curl** command it contains (shown below), replacing *&lt;your_ACI_IP_address_or_FQDN&gt;* with the IP address or FQDN for your container.

    ```
    curl -X POST "http://<your_ACI_IP_address_or_FQDN>:5000/text/analytics/v3.0/languages?" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'Hello world.'},{'id':2,'text':'Salut tout le monde.'}]}"
    ```

1. Save your changes to the script. Note that you do not need to specify the Azure AI services endpoint or key - the request is processed by the containerized service. The container in turn communicates periodically with the service in Azure to report usage for billing, but does not send request data.


1. Right-click the **04-containers** folder and open an integrated terminal. Then enter the following command to run the script:

    ```
    .\rest-test
    ```

1. Verify that the command returns a JSON document containing information about the language detected in the two input documents (which should be English and French).

### Review
In this lab, you have completed:

+ Deployed and run a Text Analytics container
+ Used the container

## You have successfully completed the lab
