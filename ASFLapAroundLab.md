# Azure Service Fabric Lap Around Lab

[![RealTalk w/ RealTech](https://rtwrt.blob.core.windows.net/post1/rtwrtlogo.png)](https://rtwrt.io)

This lab is designed to give you a best practiced introduction to Microsoft Azure Service Fabric. Here I assume you know the fundamentals of microservices and bisics of Azure Service Fabric and other Azure Resources. I aggregated resources from across multiple documentation spaces to provide the best onboarding method to deploy a microservice application in an ASF cluster.

Azure Service Fabric requires an understanding of VMSS, Key Vault, and ARM Templates. An app is partitioned on separate nodes in a cluster. These nodes exist on different machines inside the Azure Data Center and continuously move from machine to machine as Azure optimizes app performance, updates and mediates machine failure. The size of the machine your node can exist on can be configured using Virtual Machine Scale Sets. Key Vault handles the permissions and certificates to access your cluster. Don’t create unsecure cluster as they allow anonymous users to connect and the engineering team will limit capabilities of your cluster. Azure Resource Manager (ARM) is what you’ll use to configure the additional resources (i.e. Network Interface, NSG, Key Vault, V-Net, VMSS, Public IP, DNS, Storage Account)  required to deploy the ASF cluster to Azure. All resources must be deployed in one Resource group. For build, release and deployment management, we will be using Visual Studio Team Services to package, version our SF service.


## Prerequisites
  - Windows 10
  > With the Windows 10 anniversay edition you may need to turn off Secure Boot in the BIOS of your machine in order for the SF cluster to run locally
  - [Visual Studio 2015 with Update 2 or Later](http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015)
  - [Service Fabric Runtime](https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015) & [SDK](http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015)
  - Windows Powershell
  - Visual Studio Team Services Account

## Setup you KeyVault

Open Powershell and Login to Azure on Powershell

```Powershell
Login-AzureRmAccount
Get-AzureRmSubscription
```
> If you have multiple supscrioptions attached to your Microsoft Account, select the one you want to work with using this command

```Powershell
Set-AzureRmContext -SubscriptionId <guid>
```

Create a Resource Group in the Azure portal where you would like to deploy all the resources in this lab. (Best Practices: You should place your KeyVault & Secrets in a separate RG then your cluster resources for Production)

Create a KeyVault

```Powershell
New-AzureRmKeyVault -VaultName 'myvault' -ResourceGroupName 'mycluster-keyvault' -Location 'West US' -EnabledForDeployment
```

Use the [ServiceFapricRPHelpers scripts](https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers) (included in this repo) The ServiceFabricRPHelpers.psm1 module provides helper methods for adding certs to the keyvault for use in the Service Fabric Cluster
 - Goto module directory
 - Import module  
 ```Powershell
Import-Module .\ServiceFabricRPHelpers.psm1
```
- Use this command to generate a certificate to secure you SF cluster. Be sure to choose your own Password, DNS Name, and Output directory

```Powershell
Invoke-AddCertToKeyVault -SubscriptionId <string> -ResourceGroupName <string> -Location <string> -VaultName <string> -CertificateName <string> -Password <string> -UseExistingCertificate -ExistingPfxFilePath <string>  [<CommonParameters>]
```
You should see something similar to the output below in your Powershell window. Take note of your Certificate Thumbprint

![img1](https://rtwrt.blob.core.windows.net/post1/img1_Ink_LI.jpg)

Check the Azure Portal and view your KeyVault and Certificate you just genereated in Powershell

<img src="https://rtwrt.blob.core.windows.net/post1/img2.png" width="600">

## Create your Cluster
  > There are two ways to create a Service Fabric cluster in Azure: The Azure Portal or using the Azure Resource Manager. In this Lab we used the Azure Portal and the instruction from the official documentation. Follow the instructions here at this [link](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-creation-via-portal#create-cluster-in-the-azure-portal).
  
  <img src="https://rtwrt.blob.core.windows.net/post1/img3.png" width="600">
  
## Remote Desktop into your VMSS Cluster node

You may want to remote into the VMs that your cluster nodes are running on. In this part we will be breifly remoting into a Cluster node VM through Remote Desktop.

In the Azure portal, Grab the IP address from the Load Balancer that your cluster exists as a backend pool. It should be the only load balancer in your resource group with the name LB-<your cluster name>-<your node-type>

  <img src="https://rtwrt.blob.core.windows.net/post1/img4.png" width="700">

On your machine, search & open Remote Desktop Connection and enter thePublic IP address for you Cluster Load Balancer

 <img src="https://rtwrt.blob.core.windows.net/post1/img5.png" width="400">

 Enter the Username and Password you used in the Create your Cluster section to connect to the VMSS

 > If you used an A series VM for your Cluster node-type, expect some latency and lag for connecting to the VM and mouse input

 <img src="https://rtwrt.blob.core.windows.net/post1/img6.png" width="600">

## Run ASF application on your local machine

 Here we'll test to verify that one of the ASF sample applications runs on a local cluster.
> Grab a sample project from the [Service Fabric Getting Started Samples](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/master/Actors/VisualObjects) or use the VisualObject solution included in this lab repo.

Open Visual Studio 2015 as an Administrator - This is required to run the powershell script that deploys the cluster on your local machine

Open the sample project solution in Visual studio and press ctrl+F5 to **Run**

You will get a output log similar to the one below. 

```Powershell
	Started executing script 'Publish-NewServiceFabricApplication'.
	[void](Connect-ServiceFabricCluster); Import-Module 'C:\Program Files\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1'; Publish-NewServiceFabricApplication -ApplicationPackagePath 'C:\Users\naros\Source\Repos\ServiceFabricHackFest\Actors\VisualObjects\VisualObjects\pkg\Debug' -ApplicationParameterFilePath 'C:\Users\naros\Source\Repos\ServiceFabricHackFest\Actors\VisualObjects\VisualObjects\PublishProfiles\..\ApplicationParameters\Local.5Node.xml' -ApplicationParameter @{_WFDebugParams_='[{"""ServiceManifestName""":"""VisualObjects.WebServicePkg""","""CodePackageName""":"""Code""","""EntryPointType""":"""Main""","""DebugExePath""":"""C:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe""","""DebugArguments""":""" {a5a5b5c4-2b1b-47d6-9b73-660445a87abd} -p [ProcessId] -tid [ThreadId]""","""EnvironmentBlock""":"""_NO_DEBUG_HEAP=1\u0000"""},{"""ServiceManifestName""":"""VisualObjects.ActorServicePkg""","""CodePackageName""":"""Code""","""EntryPointType""":"""Main""","""DebugExePath""":"""C:\\Program Files (x86)\\Microsoft Visual Studio 14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe""","""DebugArguments""":""" {877fc87f-7b99-4f92-ae94-5a7078a09586} -p [ProcessId] -tid [ThreadId]""","""EnvironmentBlock""":"""_NO_DEBUG_HEAP=1\u0000"""}]'} -Action Create -SkipPackageValidation:$true -ErrorAction Stop
	Creating application...
	
	
	ApplicationName        : fabric:/VisualObjects
	ApplicationTypeName    : VisualObjectsApplicationType
	ApplicationTypeVersion : 1.0.0
	ApplicationParameters  : { "VisualObjects.ActorService_PartitionCount" = "5";
	                         "VisualObjects.WebService_InstanceCount" = "1";
	                         "_WFDebugParams_" = "[{"ServiceManifestName":"VisualObjects.WebServicePkg","CodePackageName":"
	                         Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual 
	                         Studio 14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" 
	                         {a5a5b5c4-2b1b-47d6-9b73-660445a87abd} -p [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO
	                         _DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"VisualObjects.ActorServicePkg","CodePackageName"
	                         :"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual 
	                         Studio 14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" 
	                         {877fc87f-7b99-4f92-ae94-5a7078a09586} -p [ProcessId] -tid 
	                         [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }
	
	Create application succeeded.
	
	
	Finished executing script 'Publish-NewServiceFabricApplication'.
	Time elapsed: 00:00:03.8471278
	Started executing script 'Get-ServiceFabricApplicationStatus'.
	[void](Connect-ServiceFabricCluster); Import-Module 'C:\Program Files\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1'; Get-ServiceFabricApplicationStatus -ApplicationName 'fabric:/VisualObjects' -ErrorAction Stop
	The application has started.
	Service Status:
	fabric:/VisualObjects/VisualObjects.ActorService is not ready, 5 partitions remaining.
	fabric:/VisualObjects/VisualObjects.WebService is ready.
	
	Service Status:
	fabric:/VisualObjects/VisualObjects.ActorService is not ready, 5 partitions remaining.
	fabric:/VisualObjects/VisualObjects.WebService is ready.
```

Once the output states your Application Service is ready, your App has successfully been deployed to you local Service Fabric Cluster

Open a browser and navigate to (http://localhost:19080)

>This is the Service Fabric Explorer (SFX). SFX is a web-based tool for inspecting and managing applications and nodes in an Azure Service Fabric cluster. Service Fabric Explorer is hosted directly within the cluster, so it is always available, regardless of where your cluster is running. 

One healthy application should be displayed in SFX.

 <img src="https://rtwrt.blob.core.windows.net/post1/img7.png" width="700">

>If you chose to use the VisualObject solution, navigate to http://localhost:8081 in 2 different browsers. Here you can see the sample project renders a set of objects in the browser. Each object is represented by an Actor, where the location and trajectory of each is calculated on the server side by the Actor representing the object. The two browsers show that the objects remain in identical location regardless of the session.

 <img src="https://rtwrt.blob.core.windows.net/post1/img8.png" width="700">

Stop the application in Visual Studio. Once the application has ended, you will see 0 apps running in SFX.

## Deploying to your Service Fabric Cluster in Azure

- Follow the instructions on how to create a [Visual Studio Team Service Team Project](https://www.visualstudio.com/en-us/docs/setup-admin/create-team-project).
- Follow the instruction on how to [Publish your Code](https://www.visualstudio.com/da-dk/docs/git/share-your-code-in-git-vs#publish-your-code) for the sample solution to the VSTS Team Project Repository.

Once you get that done, we will be configuring Build and Release settings for deploying your App code to your cloud ASF cluster. 

> Overview of build and Release definitions

Navigate to the Build & Release tab at the top of the VSTS window and click **+ New definition**

 <img src="https://rtwrt.blob.core.windows.net/post1/img9.PNG" width="700">



   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [@thomasfuchs]: <http://twitter.com/thomasfuchs>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [keymaster.js]: <https://github.com/madrobby/keymaster>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]:  <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
