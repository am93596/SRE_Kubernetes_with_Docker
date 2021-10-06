# Kubernetes With Docker

![image](https://user-images.githubusercontent.com/88166874/135991408-1ead1d76-9ef2-4bad-bcff-60bba22aed62.png)

## What is Kubernetes (K8s)?
Kubernetes, also known as K8s (`K` + 8 letters + `s`) is an open source container orchestration tool. Kubernetes is used for automated deployment, scaling, and management of containerised applications. It can be used to manage any container, such as Docker containers or CRI-O containers. Kubernetes was built by Google, and they developed and maintained it for over 15 years, before gifting it to the Linux foundation in 2014.  

## Kubernetes Architecture
### Types of Kubernetes Architecture

![image](https://user-images.githubusercontent.com/88166874/135992346-789e7280-8e61-4837-92cb-ca58657ae409.png)

## Kubernetes Components
A Kubernetes node is a physical server or virtual machine that holds the contents of the application. These contain pods, which are running environments layered over containers. This allows Kubernetes to abstract away the technical aspects of running a container to them simpler to run, as well as making it easy to change container type. Our application container may have one pod, and that may use a database pod with its own container. A pod is usually meant to run just one container, but you can run multiple containers with a single pod (usually used for running an app with a connected helper container, for example a database)

## Benefits of Kubernetes
Kubernetes helps you manage containerised applications in different environments, e.g. physical, virtual, cloud, and hybrid. Kubernetes offers high availability by auto scaling the number of pods as required, so that your application is always available to your users. Another benefit of Kubernetes is load balancing - your users will be automatically distributed between your pods so that pods are not overloaded and your application is always responsive. Kubernetes is also self healing, which means that if anything goes wrong with one of your pods, Kubernetes will pass the users of that pod to another one, then restore pod and add it back to the load balancing pool once it is working again

## Enabling Kubernetes in Docker
- Open Docker Desktop as admin, and make sure docker is running
- Navigate to Settings -> Kubernetes
- Tick the box labelled `Enable Kubernetes`, then click `Apply & Restart`
- Should display a little green tab at the bottom of the screen when it is successful:

![k8s started](https://user-images.githubusercontent.com/88166874/135875076-83489e74-1fdd-45e7-aa3c-80735eb221d8.PNG)

- Then open a new Git Bash as admin, and run the following commands to make sure that your Kubernetes is working:
- `kubectl` - lists all of the commands that can be used with `kubectl`

![kubectl-command](https://user-images.githubusercontent.com/88166874/135875325-fd42a9a7-9e9b-4ca5-b2e8-a0c348b3d469.PNG)

- `kubectl version` - gives details about your version of Kubernetes

![kubectl-version-command](https://user-images.githubusercontent.com/88166874/135875359-356b427f-fbc4-4af4-b793-d4e295b4fc1a.PNG)

- To get cluster IP: `kubectl get service`

- Make a folder called `nginx-deploy`, then make a `nginx-deploy.yml` file with the following contents
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment  # naming the deployment

spec:
  selector:
    matchLabels:
      app: nginx  # look for this label to match with k8s service
  
  # Let's create a replica set of this with 2 instances/pods
  replicas: 2
  
  # Template to use its label for k8s service to launch in the browser
  template:
    metadata:
      labels:
        app: nginx  # this label onnects to the service or any other
    spec:
      containers:
      - name: nginx
        image: am93596/sre_customised_nginx:latest
        ports:
        - containerPort: 80
```
- `kubectl create -f nginx-deploy.yml` - creates the pods from the yaml file

![kubectl-create-yaml](https://user-images.githubusercontent.com/88166874/135881101-2a1e74c9-8e09-4263-ab33-7f14555a4109.PNG)

- `kubectl get deploy`

![kubectl-get-deploy](https://user-images.githubusercontent.com/88166874/135881153-96cec1f1-fd2b-4e84-a5bf-05708008a264.PNG)

- `kubectl get pods`

![kubectl-get-pods](https://user-images.githubusercontent.com/88166874/135881202-6d907b1f-a7a9-47c2-b995-ff9b5d91031a.PNG)

- `kubectl describe pod name_of_pod` -> name comes from `kubectl get pods`

- new business requirement: traffic expected; we need to increase number of pods

- get name of deployment from `kubectl get deploy` command
- `kubectl describe deploy nginx-deployment`
- scale up while the deployment is running
- `kubectl edit deploy nginx-deployment` -> opens notepad
- change `replicas:` from `2` to `3`
- save and close
- `kubectl get pods` to see the new pod starting
- new file: `nginx-service.yml` with contents:
```yaml
---
apiVersion: v1
kind: Service
# Metadata for name
metadata:
  name: nginx-deployment
  namespace: default
spec:
  ports:
  - nodePort: 30442
    port: 80
    protocol: TCP
    targetPort: 80

  selector:
    app: nginx
  
  type: LoadBalancer
```
- run the file with `kubectl create -f nginx-service.yml`
- check it works with `kubectl get service`
- open your browser and type in `localhost` to see the nginx page
- Kubernetes self healing in action:
  - `kubectl get pods`
  - `kubectl delete pod pod_name`
  - Kubernetes automatically makes a new pod, and it doesn't affect the page the user has open in their browser

### Tues 5th Oct
- `kubectl get all`
- `kubectl delete svc nginx-deployment`
- Then open your browser and type in `localhost` - this will not work as you no longer have a service
- The pods are invisible from outside the deployment - so we need a service to make them accessible from the internet
- Works by matching labels with our selector - in our example, we labelled our app as `nginx`
- Within the cluster, an API call is made to search the labels for the selector
- `kubectl edit deploy nginx-deployment` - can edit the version to roll back to in here

### Labels and Selectors
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

### Namespaces
https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

- `kubectl get namespace`
- `kubectl create namespace sparta`
- `kubectl delete namespace sparta`

### Diagram of NodeJS app deployment with Kubernetes

![image](https://user-images.githubusercontent.com/88166874/136041757-53af35bd-6a49-402b-819b-a05e4622f52a.png)

- `kubectl delete deploy nginx-deployment`
- `kubectl delete svc nginx-svc`
- make the following yaml files in a new folder called `node-mongo-deployment`:
#### node-deploy.yml
```yaml
# Create a diagram for node deployment and service
# use the example of nginx deployment and service
# nodeapp deployment to have 3 pods min
# service within the same cluster and namespace to connect to node deploy
# type LoadBalancer
# End goal is to see our node app running port localhost:3000

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node
spec:
  selector:
    matchLabels:
      app: node
  replicas: 3
  template:
    metadata:
      labels:
        app: node
    spec:
      containers:
      - name: node
        image: am93596/sre_node_app:v1
 
        ports:
        - containerPort: 3000
        
        env:
        - name: DB_HOST
          value: mongodb://mongo:27017/posts
```
#### node-svc.yml
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: node
spec:
  selector:
    app: node
  ports:
  - port: 3000  # put 80 to get it to work with localhost (instead of localhost:3000)
    targetPort: 3000
  type: LoadBalancer
```
#### node-hpa.yml
```yaml
# horizontal pod autoscaler - auto scaling group for kubernetes pods
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler # (hpa)

metadata:
  name: sparta-node-app-deploy
  namespace: default

spec:
  maxReplicas: 9  # (max number of instances/pods)
  minReplicas: 3  # (min number of instances/pods)
  scaleTargetRef: # Targets the node deployment
    apiVersion: apps/v1
    kind: Deployment
    name: node
  targetCPUUtilizationPercentage: 50  # 50% of CPU usage
  ```
  - `kubectl create -f name_of_file`
  - `kubectl get hpa`

#### task
- Create deploy and service for mongo
- Create PV and PVC to claim storage
- delete app deploy, uncomment env lines, then recreate both db and app
- create a diagram for mongo deploy and svc

##### mongodb diagram
![image](https://user-images.githubusercontent.com/88166874/136045116-45ccc8bd-82bf-4025-bb08-1a9b9f97d6ff.png)

##### Node app with mongodb diagram
![image](https://user-images.githubusercontent.com/88166874/136045682-cfdb1d54-a8fe-42cb-8b32-07d2318163dc.png)

### Connecting NodeJS app to MongoDB

#### mongodb-deploy.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:latest

        ports:
        - containerPort: 27017
```
#### mongodb-svc.yml
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  selector:
    app: mongo
  ports:
  - port: 27017
    targetPort: 27017
  type: LoadBalancer
```
- `kubectl delete deploy node`
- `kubectl delete svc node`
- `kubectl delete hpa sparta-node-app-deploy`
- `kubectl create -f mongodb-deploy.yml`
- `kubectl create -f mongodb-svc.yml`
- `kubectl create -f node-deploy.yml`
- `kubectl create -f node-svc.yml`
- `kubectl create -f node-hpa.yml`
- `kubectl exec name_of_node_pod env node seeds/seed.js`

### Creating Persistent Volume for MongoDB
#### mongodb-pv.yml
- Makes a persistent volume (PV) for the mongoDB database, to store the data permanently
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-db-pv
spec:
  capacity:
    storage: 256Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  hostPath:
    path: /data/db
    type: Directory
```
#### mongodb-pvc.yml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```
#### Paste the following into the bottom of the mongodb-deploy.yml file
```yaml
        volumeMounts:
        - name: storage
          mountPath: /data/db

      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: mongo-db-pvc
```  
#### Then run the following commands:  
- `kubectl delete deploy node`
- `kubectl delete svc node`
- `kubectl delete hpa sparta-node-app-deploy`
- `kubectl delete deploy mongo`
- `kubectl delete svc mongo`
- `kubectl create -f mongodb-pv.yml`
- `kubectl create -f mongodb-pvc.yml`
- `kubectl create -f mongodb-deploy.yml`
- `kubectl create -f mongodb-svc.yml`
- `kubectl create -f node-deploy.yml`
- `kubectl create -f node-svc.yml`
- `kubectl create -f node-hpa.yml`
- `kubectl exec name_of_node_pod env node seeds/seed.js`  

Open `http://localhost:3000/` in your browser - the Sparta Test Page should appear.  

![image](https://user-images.githubusercontent.com/88166874/136064276-ea9ddb62-f28c-4d6f-ac04-67f8cc10873e.png)

Then open `http://localhost:3000/posts` - the posts should display.  

![image](https://user-images.githubusercontent.com/88166874/136064374-760c4226-9bd3-4520-9ebb-0290ff34d201.png)

*only one of the mongodb pods works; may be to do with 256Mi restriction on the PV and PVC memory*
