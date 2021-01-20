# Build from Dockerfile and push to Quay

The following example shows how to create resources, tasks and pipelines on Tekton:

# Steps

**1. Create a new project**
```
oc new-project tektontutorial
```
**2. Create a secret for registry**

```
oc create secret -n tektontutorial docker-registry container-registry-secret \
  --docker-server=$CONTAINER_REGISTRY_SERVER \
  --docker-username=$CONTAINER_REGISTRY_USER \
  --docker-password=$CONTAINER_REGISTRY_PASSWORD
```

**3. Create a service account**
```
oc create sa -n tektontutorial build-bot
```
**4. Associate the secret to the service account**
```
oc patch serviceaccount build-bot \
   -p '{"secrets": [{"name": "container-registry-secret"}]}'
```    
**5. Create the Pipeline Resources**
```
oc apply -f httpd/resources/resources.yaml -n tektontutorial
```
**6. Create the Pipeline**
```
oc apply -f httpd/resouces/pipeline.yaml -n tektontutorial
```
**7. Create the Tasks**
```
oc apply -f httpd/tasks/pipeline.yaml -n tektontutorial
```
**8. Start Pipeline**
```
tkn pipeline start dockerfile-to-image \
  --resource="appSource=git-source" \
  --resource='appImage=httpd-image' \
  --param contextDir='httpd-hello' \
  --serviceaccount=build-bot \
  --showlog
```    
