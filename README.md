# Lacework AKS K8s Demo
This repo with provision an AKS Cluster using Terraform and add the Lacework Agent on Microsoft Azure Kubernetes Service.
  
[![CIS](https://app.soluble.cloud/api/v1/public/badges/733ddd27-6da8-4c31-868b-bec81a68ef73.svg)](https://app.soluble.cloud/repos/details/github.com/anthonygrees/lw_aks_k8s_demo) [![IaC](https://app.soluble.cloud/api/v1/public/badges/0b471ba2-e283-4357-8067-d80961252c51.svg)](https://app.soluble.cloud/repos/details/github.com/anthonygrees/lw_aks_k8s_demo)
  
### 0. Lacework
Once deployed, you will see the AKS cluster in the Lacework Polygraph.  
  
![Lacework Poly](/images/aks_poly.png)

### 1. Get Started
Install the Azure CLI
```bash
brew install azure-cli
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
  
To get a list of supported K8s versions, run the following:
```bash
az aks get-versions --location <region>
```
*Note*: The terraform uses `kubernetes_version  = "1.22.2"` which is hardcoded in the `main.tf`.
  
### 2. Provision AKS via Terraform
Initialise your terraform.  
```bash
terraform init
```
  
Execute the terraform.  
```bash
terraform apply
```
  
Your output will look like:  
```bash
azurerm_kubernetes_cluster.cluster: Creation complete after 5m29s [id=/subscriptions/XXX-XXXXXX-XXXXXX-XXXXX-XXXXX/resourcegroups/reesy-aks-cluster/providers/Microsoft.ContainerService/managedClusters/reesy-aks]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

kube_config = apiVersion: v1
clusters:
```

Once provisioned, you can export the kubeconfig with:  
```bash
echo "$(terraform output kube_config)" > azurek8s
```
  
You can add the credentials to your kubeconfig (`~/.kube/config`) with:  
```bash
KUBECONFIG=~/.kube/config:/path/to/another/config.yml kubectl config view --flatten > ~/.kube/config-new.yaml

cp ~/.kube/config-new.yaml ~/.kube/config
```
*Note*:Do not pipe directly to the config file! Otherwise, it will delete all your old K8s content!  
  
You can verify that you can connect to the cluster with:  
```bash
kubectl get nodes
```
  
### 3. Installing the Lacework Agent
In the Lacework Console, download the two Kubernetes YAML files. Navigate to `Settings > Agents`. Either use an existing agent access token or create a new agent token by clicking + Create New. Click Install Options. Download Kubernetes Config and Kubernetes Orchestration.  
  
Using the kubectl command line interface, add the Lacework configuration file into the cluster.  
  
`kubectl create -f lacework-cfg-k8s.yaml`  
  
Instruct the Kubernetes orchestrator to deploy an agent using a DaemonSet on all nodes in the cluster, including the master.  
  
To change the CPU and memory limits, see Change Agent Resource Installation Limits on K8s Environments.  
  
`kubectl create -f lacework-k8s.yaml`   
  
Repeat the above steps for each Kubernetes cluster. The `config.json` file is embedded in the `lacework-cfg-k8s.yaml` file. To customize FIM or add tags in a Kubernetes environment, edit the configuration section of the YAML file and push the revised `lacework-cfg-k8s.yaml` file to the cluster using the following command.  
  
`kubectl replace -f lacework-cfg-k8s.yaml`   
  
Make sure you have the correct tags for the cluster name in your `lacework-cfg-k8s.yaml`.  It MUST be `KubernetesCluster` or the Lacework UI will not recognise it.   
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: lacework-config
data:
  config.json: |
    {"tokens":{"AccessToken":"add_your_token_here"}, "tags":{"KubernetesCluster":"reesy.aks.local"}, "serverurl":"https://api.lacework.net"}
```
  
  
### 4. Check that LW Agent is running on K8s
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
  
### 5. Lacework UI
In the Lacework UI, navigate to `Resources -> Kubernetes` and filter on your AKS cluster.  It may take up to 1 hour for the polygraph to show your K8s cluster.  
  
![Lacework Poly](/images/aks_polygraph.png)
  
