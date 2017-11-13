# Deploying to a New K8 Cluster
Before deploying the services, you must first deploy secrets


## Creating Secrets
1. 
2. 

## Deploying Services

Services need to be deployed in the following order

```bash
kubectl create -f internal_services.yml
kubectl create -f pyml.yml
kubectl create -f web.yml 
```
## Deleting a Service/Deployment
```bash
kubectl delete -f <file_name>.yml
```
