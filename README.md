# k8s-dep
Here is the instrudction of how to do configuration and deployment with github and dockerhub. Also some steps to run the kubernetes clusters in local minikube.

## 1. Build git project

In case of any update on project source file, build it and test it. Once it is done, commit the update and push to github. If there is link between github and dockerhub, the auto build could be triggered and sync to dockerhub automatically, which is not the case to be mentioned here.

## 2. Docker image
Start docker desktop if it is not running. Before you update the docker image, check the existing version by
```
docker image ls -a
```

### New version
After commit the change, you could tag the new version. Say the old version number is 1.0.0, then the new version could be 1.0.1. Then create the docker image by
```
docker build -t jumkid/<project_name>:<new_tag> . --build-arg <param_name>=<param_value>
```
When build is done, use check the new image 

### Replace the existing version
In the case of replacing the existing version(tag) of docker image, you should remove the existing image from docker first
```
docker image rm <image_id>
```
Then create the dokcer image by
```
docker build -t jumkid/<project_name>:<existing_tag> . --build-arg <param_name>=<param_value>
```

## 3. Push image to dockerhub
In the case of replacing existing docker image, firstly, remove the existing tag on dockerhub. You could open dockerhub and navigate to repositories>jumkid>project>tags, and remove the tag there.

Then push the new image to dockerhub
```
docker push jumkid/<project_name>:<tag>
```
Once it is done, go to the dockerhub to check the update tag there. As an option, you could shutdown the docker desktop to save memory of your machine.

## 4. Create yaml file
Kubernetes uses YAML file to configure infurstraturcture. And you should set the right image from dockerhub in the deployment configuration.
```
spec:
      containers:
      - name: content-vault
        image: docker.io/jumkid/<project_name>:<tag>
        resources:
```


## 5. Start minikube
If it is the first time you run minikube, you could start the minikube with custom config
```
minikube start --driver hyperv --cpus 4 --memory 6144 --kubernetes-version <k8s_version>
```
Wait for a while as it may take some time, then you should see the cluster is up and running. Next time when you start minikube, you could simple do
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

### Force the image reload
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

### Web UI
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
