# k8s-dep
Here is the instrudction of how to setup kubernetes local environment and deploy projects as well as essential softwares. This instruction is used windows 10 as sample, but it should also work on MacOS.

## Prerequisite

### 1. Docker desktop
Go to dockerhub website and download the latest docker desktop installer. Then run it on your local machine. Please also install chocolatey software management (it somes with the docker desktop usually) so that you can install some following essential software with command line. 

### 2. Install WSL and upgrade it to WSL2 On windows 10
For better performance, suggest upgrade WSL to WSL2. You can search the wsl2 upgrade package from microsoft website.
Open a powershell terminal and type the command below
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### 3. Minikube
In order to deploy projects on a local kubernetes cluster, you can install minkube which is a single kubernetes cluster. To install minikube, make sure your laptop has at least 16GB memory with 20GB harddisk space. Run powershell with administrator permission, type the command below to install
```
choco install minikube
```
Just wait for a while. When it is completed, then start a minikube
```
minikube --driver=hyperv --cpus 4 --memory 8192
```
When it is completed, set the default driver
```
minikube config set driver hyperv
```

All set, verify the minikube cluster with kubectl command
```
kubectl get all
```

## Deploy essentials
Before deploy any project, you need to deploy the following essential software first. Please be aware that you just need to deploy them once and make sure the essentials work well every time you deploy/redeploy project.

### 1. MySQL
Open powershell and go to k8s-dep folder
```
kubectl apply -f mysql-pv.yaml
```
when it is completed
```
kubectl apply -f mysql.yaml
```
By default, it enables root account to administrate the database. You could install MySQL Workbench to do further management works.

### 2. Elasticsearch
deploy ES server with the following command
```
kubectl apply -f elasticsearch-pv.yaml
```
when it is completed
```
kubectl apply -f elasticsearch.yaml
```

### 3. Redis
deploy Redis database with the following command
```
kubectl apply -f redis-pv.yaml
```
when it is completed
```
kubectl apply -f redis-master.yaml
```

To verify redis you could logon the redis pod
```
kubectl exec -it service/redis-master -- /bin/bash
```
then type the command to operate the database
```
redis-cli
```

### 4. Keycloak


## Deploy project

### 1. Build git project

In case of any update on project source file, build it and test it. Once it is done, commit the update and push to github. If there is link between github and dockerhub, the auto build could be triggered and sync to dockerhub automatically, which is not the case to be mentioned here.

### 2. Docker image
Start docker desktop if it is not running. Before you update the docker image, check the existing version by
```
docker image ls -a
```

#### New version
After commit the change, you could tag the new version. Go to the project home path where contains a Dockerfile. And let's say, there is an old version 1.0.0, then the new version could be 1.0.1. You could create the docker image by the following command. 
```
docker build -t jumkid/<project_name>:<new_tag> . --build-arg <param_name>=<param_value>
```
When build is done, use check the new image 

#### Replace the existing version
In the case of replacing the existing version(tag) of docker image, you should remove the existing image from docker first
```
docker image rm <image_id>
```
Then create the dokcer image by
```
docker build -t jumkid/<project_name>:<existing_tag> . --build-arg <param_name>=<param_value>
```

### 3. Push image to dockerhub
In the case of replacing existing docker image, firstly, remove the existing tag on dockerhub. You could open dockerhub and navigate to repositories>jumkid>project>tags, and remove the tag there.

Then push the new image to dockerhub
```
docker push jumkid/<project_name>:<tag>
```
Once it is done, go to the dockerhub to check the update tag there. As an option, you could shutdown the docker desktop to save memory of your machine.

### 4. Create yaml file
Kubernetes uses YAML file to configure infurstraturcture. And you should set the right image from dockerhub in the deployment configuration.
```
spec:
      containers:
      - name: content-vault
        image: docker.io/jumkid/<project_name>:<tag>
        resources:
```

### 5. Start minikube
Start minikube with
```
minikube start
```
with the existing cluster.

Check the pods and deployment, see if the old image is running
```
kubectl get pods
```
If it is running, you should delete all the services, deployment, pods, etc before update. 
```
kubectl delete -f <project_name>.yaml
```

#### Force the image reload
Open the <project_name>.yaml and change the config
```
imagePullPolicy: Always
```
Then run the deployment
```
kubectl apply -f <project_name>.yaml
```
And you can watch the deployment running
```
kubectl get pods --watch
```

Once the image reload is done, you could change the yaml config back to
```
imagePullPolicy: IfNotPresent
```

In case of anything wrong, you could access the pod or restart the image
```
kubectl exec -it <pod_name> -- /bin/bash

kubectl -n <network> rollout restart deployment <project_name>
```

#### Web UI
You could use minikube build in dashboard to monitor all resources of the cluster
First, deploy the dashboard with the latest version matches your minikube
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```
Then you could start a proxy
```
kubectl proxy
```
Access the web ui on http://127.0.0.1:8001
