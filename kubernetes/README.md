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
Tag your docker image
```bash
docker tag pyml canvass.azurecr.io/pyml:<tagName>
```
Push to the registry
```bash
docker push canvass.azurecr.io/pyml:<tagName>
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
