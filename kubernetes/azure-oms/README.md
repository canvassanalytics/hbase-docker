# Adding Monitoring of the K8 Cluster to Azure OMS

Official Doc from Microsoft can be found here: https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-kubernetes-oms

There are two options for doing this:
1. omsagent.yaml - this embeds the OMS Workspace ID and Primary key in the file
2. Upload secrets - this is the preferred method

## Deploy OMS agent with Secret
Generate the secret file
```bash
bash ./secret.sh
```

Upload the secret to K8
```bash
kubectl create -f omsagentsecret.yaml
```
Verify the secret file was created
```bash
kubectl get secrets
```

Create the omsmanagement daemon-set
```bash
kubectl create -f omsagent-ds-secrets.yaml
```