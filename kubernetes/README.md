# Kubernetes Documentation
Some items of note:
* All Docker Images must be in the Azure Registry

## Azure Registry
### Pushing to the Registry
Get the Registry Name   
```bash
az acr list --resource-group devops-canvass --query "[].{acrLoginServer:loginServer}" --output table
AcrLoginServer
------------------
canvass.azurecr.io
```
login to the registry (get the Username and Password from the Azure Portal in the Registry Resource > Access keys)
```bash
docker login canvass.azurecr.io -u myAdminName -p myPassword1
```
Tag your docker image
```bash
docker tag pyml canvass.azurecr.io/pyml:<tagName>
```
Push to the registry
```bash
docker push canvass.azurecr.io/pyml:<tagName>
```
### Creating K8 Cluster
```bash
az acs create \
--orchestrator-type kubernetes \
--name <ProdK8> \
--resource-group <produuctionK8> \
--agent-vm-size <> \
--dns-prefix <productionK8> \
--generate-ssh-keys \
--location <canadacentral> \
--master-vnet-subnet-id <id> \
--master-first-consecutive-static-ip 10.x.x.x \
--agent-vnet-subnet-id <id> \
--ssh-key-value /path/to/public/key
```
To get locations:
```bash
az account list-locations
```
To get vnet-subnets
```bash
az network vnet subnet list --resource-group <rg> --vnet-name <vnet_name>
```

## Deploying to the K8 Cluster
To deploy based on file
```bash
kubectl create -f canvass-all-latests.yml 
```
To delete a deployment
```bash
kubectl delete -f canvass-all-latests.yml 
```

## Secrets
Secrets need not be stored here, but must be loaded into K8 before running file that uses them.
They can be loaded using the following command.

NOTE: Secrets are created in the K8 cluster, so they do not need to be recreated or included in the compose file after they have been created.
```bash
kubectl create -f ./secret/azure-K8-dev01.yml
```
NOTE: all secret information must be base64 encoded, when adding a secret for azure storage account you must get the storage account name from azure then run the following (this must be done for both the account name and the key):
```bash
echo -n "00ouyp73y7hko3sagnt0" | base64
MDBvdXlwNzN5N2hrbzNzYWdudDA=
```
Then place the output in the secret file


You can verify that the secrets are loaded with either of the following commands
```bash
kubectl get secrets
kubectl describe secret/azure-K8-dev01.yml
```

## Performing a Rolling Update on a Deployment
When performing a rolling update, it is going to spin up new pods with the new Docker Image, then terminate the old pods.  Rolling Update Strategy will govern how many of the new/old pods are update at a time. 

Updating the deployment
```bash
kubectl set image deployment/ml ml=canvass.azurecr.io/pyml:latest
```
Description of the command
```bash
kubectl set image deployment/<NameOfDeployment> <NameOfDeployment>=<ImageName>:<tag>
```
You can see deployments by running
```bash
kubectl get deployments
```
You can view the Rolling update status with
```bash
kubectl get pods --selector=app=ml
```

# Accessing a K8 Cluster
```bash
az acs kubernetes get-credentials --resource-group myResourceGroup --name myK8SCluster
```

# Lanuching the K8 UI
```bash
az acs kubernetes browse -g [Resource Group] -n [Container service instance name]
```

