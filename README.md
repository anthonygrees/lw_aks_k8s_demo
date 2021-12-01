# Lacework AKS K8s Demo
This repo with provision an AKS Cluster using Terraform and add the Lacework Agent on Microsoft Azure Kubernetes Service.
  
### Get Started
Install the Azure CLI
```bash
brew 
```
  
Link your Azure CLI to your account with:  
```bash
az login
```
  
And you can list your accounts with:  
```bash
az account list
```
If you have more than one subscription, you can set your active subscription with `az account set --subscription="SUBSCRIPTION_ID"`.
  
Create the Service Principal with:  
```bash
az ad sp create-for-rbac \
  --role="Contributor" \
  --scopes="/subscriptions/SUBSCRIPTION_ID"
```
  
The previous command should print a JSON payload like this:  
```bash
{
  "appId": "00000000-0000-0000-0000-000000000000",
  "displayName": "azure-cli-YYY-MM-DD-HH-MM-SS",
  "name": "http://azure-cli-YYY-MM-DD-HH-MM-SS",
  "password": "0000-0000-0000-0000-000000000000",
  "tenant": "00000000-0000-0000-0000-000000000000"
}
```
Make a note of the `appId`, `password` and `tenant`.  
  
Export the following environment variables:  
```bash
export ARM_CLIENT_ID=<insert the appId from above>
export ARM_SUBSCRIPTION_ID=<insert your subscription id>
export ARM_TENANT_ID=<insert the tenant from above>
export ARM_CLIENT_SECRET=<insert the password from above>
```
  
### Provision AKS via Terraform
Initialise your terraform.  
```bash
terraform init
```
  
Execute the terraform.  
```bash
terraform apply
```
  
Once provisioned, you can export the kubeconfig with:  
```bash
echo "$(terraform output kube_config)" > azurek8s
```
  
You can add the credentials to your kubeconfig or temporarily use the file as your kubeconfig with:  
```bash
export KUBECONFIG="${PWD}/azurek8s"
```
  
You can verify that you can connect to the cluster with:  
```bash
kubectl get nodes
```
  
### Installing the Lacework Agent
In the Lacework Console, download the two Kubernetes YAML files. Navigate to `Settings > Agents`. Either use an existing agent access token or create a new agent token by clicking + Create New. Click Install Options. Download Kubernetes Config and Kubernetes Orchestration.  
  
Using the kubectl command line interface, add the Lacework configuration file into the cluster.  
  
`kubectl create -f lacework-cfg-k8s.yaml`  
  
Instruct the Kubernetes orchestrator to deploy an agent using a DaemonSet on all nodes in the cluster, including the master.  
  
To change the CPU and memory limits, see Change Agent Resource Installation Limits on K8s Environments.  
  
`kubectl create -f lacework-k8s.yaml`   
  
Repeat the above steps for each Kubernetes cluster. The `config.json` file is embedded in the `lacework-cfg-k8s.yaml` file. To customize FIM or add tags in a Kubernetes environment, edit the configuration section of the YAML file and push the revised `lacework-cfg-k8s.yaml` file to the cluster using the following command.  
  
`kubectl replace -f lacework-cfg-k8s.yaml`   
  
### Check that LW Agent is running on K8s
You can check what is running on the pod with:   
  
`kubectl get pods`
  
The response will be as follows:
```bash
kubectl get pods  
  
NAME                                  READY   STATUS    RESTARTS   AGE  
lacework-agent-9gfwq                  1/1     Running   0          17d  
lacework-agent-v96s2                  1/1     Running   0          17d  
lacework-agent-vds79                  1/1     Running   0          17d  
```