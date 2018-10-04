# Intelligent IoT Edge-DGT001:
Let's Take it to the Edge! (IoT Edge, AI, Machine Learning and Cognitive Services )

The Azure IoT Edge service is meant for customers who want to analyze data on devices, a.k.a. "at the edge", instead of in the cloud. By moving parts of your workload to the edge, your devices can spend less time sending messages to the cloud and react more quickly to changes in status.
Azure IoT Edge moves cloud analytics and custom business logic to devices so that your organization can focus on business insights instead of data management. Enable your solution to truly scale by configuring your IoT software, deploying it to devices via standard containers, and monitoring it all from the cloud.  

## Pre-work
There is prework that needs to be done in advance as this deployment phase of Kubernetes takes a while about 15 mins.  The deployment of Kubernetes is required during the milestone 3 stage. It is recommended that you initialize this and let it run in the background. 

## Deploy AKS cluster
1. Open https://shell.azure.com in a new browser tab, and enter the following commands in the Bash prompt.

## Create resource group 
_az group create --name ${USER}DGT01AKSCluster --location eastus_ 

## Create AKS cluster (this operation can take several minutes) 
_az aks create --resource-group ${USER}DGT01AKSCluster --name ${USER}AKSCluster --node-count 1 --generate-ssh-keys_ 

Cluster creation takes some time so it will be wise to start this process and let it run in the background

If you run into errors with duplicate resource group or cluster name replace ${USER} in the commands above with your Microsoft alias.


# Milestone 1: Standup Edge enabled VM on Azure

IoT Edge Runtime requires a host to run on. This can either be on-prem gateway or device. For this hands-on meetup we will deploy a Data Science Ubuntu VM as the host or gateway and this VM will be the IoT Edge Device.

Go to the azure portal and login with your own subscription. In the portal click on Create a resource and look for Ubuntu Linux Server. 

![image](https://user-images.githubusercontent.com/16392516/46483992-7d9db000-c7be-11e8-98ac-b246c3fc7f9e.png)

Select Ubuntu Server 18.04  
![image](https://user-images.githubusercontent.com/16392516/46484077-a2922300-c7be-11e8-92ac-3557b6b81bc9.png)

Create the VM in it's own resource group
![image](https://user-images.githubusercontent.com/16392516/46484156-cc4b4a00-c7be-11e8-952a-a5809a891b1b.png)

![image](https://user-images.githubusercontent.com/16392516/46484188-e127dd80-c7be-11e8-9ad4-0a3796832391.png)  

Got To the Resource Group created for IoT Edge and select the VM. Click Connect button to get the ssh command to connect to the VM.  
![image](https://user-images.githubusercontent.com/16392516/46484323-28ae6980-c7bf-11e8-82b2-ed0b120e3419.png)  

Once the deployment is done you will need to complete the following operations for you:  
1. Stand up an Data Science VM and IoT Hub.  
2. Install the IoT Edge runtime and az CLI in the VM.  
3. Open ports to allow SSH access to the Data science VM.  
	
Once the deployment is completed, pin it to your dashboard and you will see a VM and IoT Hub resources in the tile.

![image](https://user-images.githubusercontent.com/16392516/46484405-585d7180-c7bf-11e8-9c09-6df18fa16245.png)  

![image](https://user-images.githubusercontent.com/16392516/46484454-71662280-c7bf-11e8-8d41-d90dcedbe4db.png)  

You can also run cloud shell directly on your Azure portal by clicking on the Cloud Shell in the portal  
![image](https://user-images.githubusercontent.com/16392516/46484572-a2deee00-c7bf-11e8-9d02-d72827423e72.png)  

Make sure you select Bash  
![image](https://user-images.githubusercontent.com/16392516/46484640-c43fda00-c7bf-11e8-97a4-1f89efc50d17.png)  

Once logged into the Cloud Shell execute the command:  
_$ ssh iotedgeadmin@137.135.84.26_ (This should be the IP address of your Ubuntu VM Edge Gateway)

You will be prompted to enter the password which you entered when creating the Ubuntu VM. Enter the password and press enter.
You will successfully login to Ubuntu VM    

Change user to get root access by entering the following command  
_sudo su -_

Check for Python version, you should have 2.7.X version.  
_python --version_

Update the apt package index  
_apt install docker.io_  

![image](https://user-images.githubusercontent.com/16392516/46484824-26004400-c7c0-11e8-9cbe-df5f48c4ffbb.png)  


## Install IoT Edge Runtime
Install pip  
_sudo apt-get install python-pip_

Install IoT Edge Runtime  
_sudo pip install -U azure-iot-edge-runtime-ctl_  

![image](https://user-images.githubusercontent.com/16392516/46485005-7bd4ec00-c7c0-11e8-828d-6d229afeca6e.png)

Check IoT Edge Runtime version  
_iotedgectl --version_  
![image](https://user-images.githubusercontent.com/16392516/46485131-b3dc2f00-c7c0-11e8-8872-37e5d38cf14e.png)  

# Manage IoT Edge Devices using IoTHub
Create IoT Edge Device using IoT Hub. Click on IoT Edge from Azure Portal.   

![image](https://user-images.githubusercontent.com/16392516/46485242-ebe37200-c7c0-11e8-98ce-32ba86809d8a.png)  

Device Connection String:
_HostName=FrancisDGTIoTHub1.azure-devices.net;DeviceId=dgt100rc24;SharedAccessKey=v6gagERkYgPD3e3wkRc63MbCM6nnvgJmGDpsePvg3fc=_


Setup Device with the following command. Make sure replace device connection string with the primary connection string you copied.  
_iotedgectl setup --connection-string "{device connection string}" --nopass_   

E.g.  
_iotedgectl setup --connection-string "HostName=FrancisDGTIoTHub1.azure-devices.net;DeviceId=dgt100rc24;SharedAccessKey=v6gagERkYgPD3e3wkRc63MbCM6nnvgJmGDpsePvg3fc=" --nopass_   

![image](https://user-images.githubusercontent.com/16392516/46485393-382eb200-c7c1-11e8-858e-933216975fe6.png)  

Start the runtime and check the IoT Edge Status
_iotedgectl start_
_iotedgectl status_  

![image](https://user-images.githubusercontent.com/16392516/46485486-5eece880-c7c1-11e8-9e77-f07fb4c7deaf.png)  

Check Docker to see that the IoT Edge agent is running as a module  
![image](https://user-images.githubusercontent.com/16392516/46485565-880d7900-c7c1-11e8-9a8f-c7fd1f4c30c2.png)    



# Milestone 2: Deploy a custom module
One of the key capabilities of Azure IoT Edge is being able to deploy modules to your IoT Edge devices from the cloud. An IoT Edge module is an executable package implemented as a container. In this section, you deploy a module that generates telemetry for your simulated device.
	• In the Azure portal, navigate to your IoT hub.
	• Go to IoT Edge (preview) and select your IoT Edge device.
	• Select Set Modules.
	• Select Add IoT Edge Module.
	• In the Name field, enter tempSensor2.
	• In the Image URI field, enter microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview.
	• Leave the other settings unchanged, and select Save.

IoT Edge Modules
Name: tempSensor2
Image URI: microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview  

![image](https://user-images.githubusercontent.com/16392516/46487363-6ca46d00-c7c5-11e8-8ded-54718eb56870.png)  

On the IoT Edge Device Type the following:
_docker ps_  

On the edge device type docker ps to list all the modules running. You should see tempSensor2 module running  

![image](https://user-images.githubusercontent.com/16392516/46487480-ad9c8180-c7c5-11e8-80fe-28ad0fcf98fe.png)  

View logs of tempSensor2 module to see the data being sent to IoT Hub
Verify data is being generated by the tempSensor2 module (CTRL+C will exit)
_docker logs -f tempSensor2_  

![image](https://user-images.githubusercontent.com/16392516/46487548-e89eb500-c7c5-11e8-910f-eeaf3e8894d6.png)  

Verify data is being sent upstream to IoT Hub by checking the IoT Hub Usage tile in the hub's Overview blade.  

![image](https://user-images.githubusercontent.com/16392516/46487592-0b30ce00-c7c6-11e8-8ac5-0f130d87cea1.png)  

Milestone 2 complete!  



# Milestone 3: Deploy IoT Edge AI module using Kubernetes

## Deploy AKS cluster
	1. Open https://shell.azure.com  in a new browser tab, and enter the following commands in the Bash prompt.

## Create resource group 
_az group create --name ${USER}DGT01AKSCluster --location eastus_ 

## Create AKS cluster (this operation can take several minutes) 
_az aks create --resource-group ${USER}DGT01AKSCluster --name ${USER}AKSCluster --node-count 1 --generate-ssh-keys_ 

Cluster creation takes some time typically about 10-15 mins  

If you run into errors with duplicate resource group or cluster name replace ${USER} in the commands above with your Microsoft alias.  

## Verify cluster creation  
First, let's connect to the cluster  
_az aks get-credentials --resource-group ${USER}DGT01AKSCluster --name ${USER}AKSCluster_   

We'll use the kubectl command line tool to interact with the AKS cluster. Check the nodes in the cluster using:    
_kubectl get nodes_  

Output should like this (the nodes names in your cluster will be different):  
![image](https://user-images.githubusercontent.com/16392516/46487876-b5a8f100-c7c6-11e8-83d1-3cf621f0b6b4.png)  

## Install the IoT Edge Kuberentes connector  
1. Clone the IoT Edge connector repo.  
git clone https://github.com/Azure/iot-edge-virtual-kubelet-provider.git  
![image](https://user-images.githubusercontent.com/16392516/46487980-0587b800-c7c7-11e8-8d12-5bcb6d8bdd4d.png)  

2. Initialize Helm, a Kubernetes package manager.  
_helm init_  
![image](https://user-images.githubusercontent.com/16392516/46488054-3a940a80-c7c7-11e8-9e8a-4a6bec008c9f.png)  

3. Get the IoT Hub Owner Connection String for the IoT Hub you created in the previous milestones.  
![image](https://user-images.githubusercontent.com/16392516/46488110-631c0480-c7c7-11e8-8d2d-b0167634a86a.png)  

![image](https://user-images.githubusercontent.com/16392516/46488146-7f1fa600-c7c7-11e8-8ce1-42e4b9a4bd9f.png)  

Connection String - Primary Key
_HostName=FrancisDGTIoTHub1.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=Xu3UQbnJ3b2lZm7nfBAYl8zvmM9xr2N5y6K83tXaglk=_  

5. Save the Owner connection string in key called hub0-cs in a Kubernetes secret store called my-secrets.  
_kubectl create secret generic my-secrets --from-literal=hub0-cs='replace-with-copied-owner-connection-string'_  

Example:  
_kubectl create secret generic my-secrets --from-literal=hub0-cs='HostName=FrancisDGTIoTHub1.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=Xu3UQbnJ3b2lZm7nfBAYl8zvmM9xr2N5y6K83tXaglk='_  

6. Give cluster-admin privileges to default account.  
_kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default_  

7. Install the IoT Edge connector using helm.  
_cd iot-edge-virtual-kubelet-provider/src/_
_helm install -n hub0 --set rbac.install=true charts/iot-edge-connector/_  

![image](https://user-images.githubusercontent.com/16392516/46488437-47652e00-c7c8-11e8-9242-00c94a92164a.png)  

In a minute, kubectl get nodes will show the IoT Edge connector as a virtual node.  
![image](https://user-images.githubusercontent.com/16392516/46488471-6237a280-c7c8-11e8-992a-b61eebf3dddc.png)  


## Create a Kubernetes deployment for IoT Edge Connector
The IoT Edge Kubernetes connector allows expressing IoT Edge module deployment as a first-class Kubernetes deployment. It does so by consuming Kubernetes deployments and translating them to IoT Hub deployments for its backing IoT Hub.
1. Add a tag to the Edge device to make it targetable by the deployment.
From Edge device details, click the Device Twin button from the top bar and add the following json.  
_"tags": {
    "location": "b43"
},_  

![image](https://user-images.githubusercontent.com/16392516/46488582-b2af0000-c7c8-11e8-96af-a2118900d54c.png)  

8. Create the Kubernetes deployment that targets this device.  
_curl -fsSL aka.ms/k8s-fruity-ai-edge | kubectl apply -f -_  
![image](https://user-images.githubusercontent.com/16392516/46488687-fbff4f80-c7c8-11e8-8230-6236cea7d80f.png)  

Here is the full content of the deployment:
<script src="https://gist.github.com/veyalla/2de3e39f4bd811af8f07b2ed81d9032d.js"></script>  

9. In the Azure portal you should see the fruity-ai deployment show in the IoT Edge Deployments list. And, shortly the deployment will be applied to the Edge device since it is tagged with value that the deployment is targetting.  

If you don't see the deployment, hit the Refresh button every 30 secs or so to update the list.  
![image](https://user-images.githubusercontent.com/16392516/46488740-2e10b180-c7c9-11e8-9bb0-3e7a3d97285f.png)  

10. The change in configuration will also be reflected on the VM you created in Milestone 1. List the modules in the SSH session.  
_sudo iotedge list_  

```If the Bash cloud shell times out or becomes unreponsive, click the power button on the top bar of the cloud shell window to reconnect. Progress is saved, but you will need to SSH in to the Edge VM again.```  

11. You can view the results of the realtime object recognition using:  
_watch -n 0.2 sudo iotedge logs --tail 1 cameracapture_  

CTRL+c will exit.  
Milestone 3 complete!  


# Milestone 4: At scale deployments with Kubernetes
The power of Kubernetes becomes clear when doing at-scale deployments. Consider a common scenario - you have multiple IoT Hub all over the world and want the same fruity-ai configuration on all of them. With Kubernetes this is literally a one command operation! Let's see how.
1. Create a new IoT Hub and VM Edge device following steps in Milestone 1.  

2. Add the same tag to edge device as in Milestone 3.   

3. Create a new secrets store my-secrets1 and save the Owner connection string for the new hub in hub1-cs.
```Enter the below commands in the cloud shell window, not in the VM's SSH session.```
_kubectl create secret generic my-secrets1 --from-literal=hub1-cs='replace-with-2nd-hub-owner-connection-string'_  

4. Create a file called values.yaml with the following content.  
_edgeproviderimage:
  repository: microsoft/iot-edge-vk-provider
  tag: latest
  pullPolicy: Always
  port: 5000
  secretsStoreName: my-secrets1
  secretKey: hub1-cs
vkimage:
  repository: microsoft/virtual-kubelet
  tag: 0.3
  pullPolicy: Always
env:
  nodeName: iot-edge-connector-hub1
  nodeTaint: 
#Install Default RBAC roles and bindings
rbac:
  install: false
  serviceAccountName: virtual-kubelet_
  #RBAC api version (currently v1)
  apiVersion: v1
  #Cluster role reference
  roleRef: cluster-admin_   
  
5. Use Helm to install another IoT Edge connector virtual node that is backed by the 2nd IoT Hub by pointing it to the file created in the previous step.  
_helm install -n hub1 --set rbac.install=true /home/${USER}/iot-edge-virtual-kubelet-provider/src/charts/iot-edge-connector/ -f ./values.yaml_ 
	
```In a few seconds, kubectl get nodes will show the new virtual node.```  

6. Scale the deployment to two replicas of the deployment specification. The second replica will be scheduled on the new virtual node and sent to the backing 2nd IoT hub.  
_kubectl scale deployment fruity-ai --replicas=2_  

You should now see the new deployment show up in the 2nd hub, and get applied to the device in it.
Any updates to the deployment from Kubernetes will now reflect in both IoT hubs, allowing you to manage devices at scale.
If you've made it till here, you're awesome! Check out the additional resources for more information.


# Milestone 5: Azure IoT Edge & AI in action:
## Recap On Azure IoT Edge:
- Azure IoT Edge is a fully managed service that delivers cloud intelligence locally (at the device level)  

	· Make hybrid cloud and Internet of Things (IoT) solutions a reality with Azure IoT Edge, a fully managed service that delivers cloud intelligence locally. Seamlessly deploy and run artificial intelligence, Azure services, and custom logic directly on cross-platform IoT devices—from a small Raspberry Pi to large industrial machines. And manage it all centrally in the cloud with the security of Microsoft.  

- Some of the key reasons an organization would want to use Azure IoT Edge are  

	o Mission Critical Scenarios  
		- If network latency or loss in/lack of connectivity that could cause even a 1 second delay in an action (i.e. device shut down after… registering pressure is too high in oil and gas, temperature is too high/low in medical settings, or the RPMs on a manufacturing machine are dangerously high) could mean catastrophic results – you need decisions to be made at the device level, without sending data to the cloud, your rules engines (ASA) and machine learning (AML) models make a decision, and then trigger an action sent back to the device – instead it all happens locally  
    
	o Data Privacy & Compliance/Regulations  
		- There are cases where data must be cleaned or given a special encryption (i.e. HIPPA or GDPR), IoT Edge enables a type of holding bay for this cleaning or encryption to be performed on the data before sent to the cloud.  
    
	o Cost & Practical Considerations  
		- Does all of your data need to be sent to the cloud? Maybe so, or maybe only certain data and messages (i.e. temperature > 80 degrees, inventory count < 5, etc.) need to be sent. Further, maybe the transmission of data doesn’t need to be real-time and instead can be sent to the cloud via batch upload.  
    
- More info on Azure IoT Edge can be found [here](https://azure.microsoft.com/en-us/services/iot-edge/)

# Intelligent Azure IoT Edge demo URL:
https://azureedgedemo.azurewebsites.net  
Credentials: N/A – enter any text into username and password fields 

# Additional resources:  
• [Azure IoT Edge documentation](https://docs.microsoft.com/en-us/azure/iot-edge/about-iot-edge)  
• [IoT Edge Github repo](https://github.com/azure/iotedge)  
• [IoT Edge integration with Kubernetes](https://azure.microsoft.com/en-us/blog/manage-azure-iot-edge-deployments-with-kubernetes/)  
• Kubernetes integration Channel 9 video:  
<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Kubernetes-integration-with-Azure-IoT-Edge/player" width="480" height="270" allowFullScreen frameBorder="0"></iframe>    
• [IoT Edge custom vision sample](https://azure.microsoft.com/en-us/resources/samples/custom-vision-service-iot-edge-raspberry-pi/)  
• [Cognitive services customvision.ai](https://customvision.ai/)  
• [Welcome to the AI Toolkit for Azure IoT Edge](https://github.com/Azure/ai-toolkit-iot-edge)  


