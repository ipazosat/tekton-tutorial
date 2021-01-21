# Build from Dockerfile and push to registry 

This use case of Tekton creates the following objetcs:

 - Pipeline resources. Source (git repository) and destination registry server.
 - Tasks. This is where the action really happens.
 - Pipeline. Ties it all together, resources and tasks.

Tasks and pipeline are generic and you can reuse them in any project. You just need to define new pipeline resources and pass the correct parameters to tkn pipeline shown in step 8
--resource='appSource=source' and --resource='appImage=destination' 

# Steps

**1. Create a new project**

```
oc new-project tektontutorial
```


**2. Create a service account**

```
oc create sa -n tektontutorial build-bot
```

**3. Create a secret for registry in case needed and add it to the service account**

```
export CONTAINER_REGISTRY_SERVER='<your registry server main url.Example quay.io>'
export CONTAINER_REGISTRY_USER='<your registry user>'
export CONTAINER_REGISTRY_PASSWORD='<your registry password>'


oc create secret -n tektontutorial docker-registry container-registry-secret \
  --docker-server=$CONTAINER_REGISTRY_SERVER \
  --docker-username=$CONTAINER_REGISTRY_USER \
  --docker-password=$CONTAINER_REGISTRY_PASSWORD

oc patch serviceaccount build-bot \
   -p '{"secrets": [{"name": "container-registry-secret"}]}'
```

**4. Create a secret for github in case needed and add it to the service account**

```
export GITHUB_USERNAME='<your github.com username>'
export TEKTON_TUTORIAL_GITHUB_PAT='<your github.com username personal accesstoken>'


oc create secret -n tektontutorial generic github-pat-secret --dry-run=client -o yaml \
  | yq w - type kubernetes.io/basic-auth \
  | yq w - stringData.username $GITHUB_USERNAME \
  | yq w - stringData.password $TEKTON_TUTORIAL_GITHUB_PAT \
  | oc apply -f -

oc annotate -n tektontutorial secret github-pat-secret \
  "tekton.dev/git-0=https://github.com"

oc patch serviceaccount build-bot \
   -p '{"secrets": [{"name": "github-pat-secret"}]}'

```

**5. Create the Pipeline Resources**

```
oc apply -f ./httpd/resources/resources.yaml -n tektontutorial
```

**6. Create the Pipeline**

```
oc apply -f ./httpd/pipelines/pipeline.yaml -n tektontutorial
```

**7. Create the Tasks**

```
oc apply -f ./httpd/tasks/build-app-task.yaml -n tektontutorial
```

**8. Start Pipeline**

```
tkn pipeline start dockerfile-to-image \
  --resource='appSource=git-source' \
  --resource='appImage=httpd-image' \
  --param contextDir='httpd-hello' \
  --param tag='latest' \
  --serviceaccount='build-bot' \
  --showlog
```    
