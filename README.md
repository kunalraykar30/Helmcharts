# Helmcharts

Repo provides the scrpts that do these jobs automactially. 
* Script deploy_gke_helm.sh smoothly deploys the GKE cluster and Helm, kubectx and kubens. 

## Creation of GKE 
Here, I have created GKE Cluster for the installation of the Helm. 

```
# PROJECT_ID=your_gke_project_id
# gcloud config set project ${PROJECT_ID}
# CLUSTER_NAME=gke

# gcloud container clusters create $CLUSTER_NAME \
    --num-nodes=3 --enable-autoupgrade --no-enable-basic-auth \
    --no-issue-client-certificate --enable-ip-alias --zone   us-central1-c  --metadata \
    disable-legacy-endpoints=true
    
# gcloud container clusters list
# kubectl cluster-info   
```	
GKE create a PersistentVolumeClaim ( Dynamically provisioning PersistentVolumes)and Kubernetes automatically provisions a persistent disk to the dpeloyment.

## Installation fo the Helm 

Installation of Helm on cloud shell. 
The latest and stable verison of helm can be downloaded from the below link https://github.com/helm/helm/releases
```
# wget https://get.helm.sh/helm-v3.4.0-linux-amd64.tar.gz
# tar -xvzf helm-v2.13.0-linux-amd64.tar.gz
# sudo cp linux-amd64/helm /usr/local/bin/
```
#### Getting kubectx and kubens on cloud shell, thus allowing easy switch between the context and namespace.
```
# git clone https://github.com/ahmetb/kubectx kubectx/
# export PATH=$PATH:/home/kunal_raykar/

# gcloud container clusters get-credentials $CLUSTER_NAME --zone us-central1-c 
# kubectx gke=gke_${PROJECT_ID}_us-central1-c_gke
```
Seperate service account will be required for the helm in order to create the resouce/objects of the charts from template on Cluster
```
# kubectl create serviceaccount tiller -n kube-system 
# kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller 
```

#### Initialize the tiller as server side component in kube-system 
Once the tiller component is initialzed deployment - tiller-deploy wil be deployed in kube-system namespace of the cluster. 

```
# helm init --service-account tiller 
# kubectl get deploy -n kube-system | grep -i tiller 
# kubectl get po -n kube-system | grep -i tiller 
# helm version
```
## Installation of Jenkins using Helm 

Here before installation the jenkins I have modified the values of jenkins charts as per my requirements. 
Below parameters has been modified by me in the jenkins.values before installation of the jenkins 
```
  adminPassword: "admin"
  serviceType: NodePort
  nodePort: 32222
```
The password for admin is "admin" and NodePort server is used instead of the ClusterIP service.
Once the jenkins chart is installed many K8s resouces will be created by the tiller. It will take sometime for objects to come up. 
```
# helm inspect values stable/jenkins > /tmp/jenkins.values
# vi /tmp/jenkins.values
# helm install stable/jenkins --values /tmp/jenkins.values --name myjenkins 

# helm ls 
# kubectl get po -o wide | grep -i jenkins 
# kubectl get no -o wide
# helm status myjenkins 
```
Jenkins will be accessible on the NodePort 32222 of the worker node. 
