# Kubernetes With Docker

![image](https://user-images.githubusercontent.com/88166874/135991408-1ead1d76-9ef2-4bad-bcff-60bba22aed62.png)

## What is Kubernetes (K8s)?
Kubernetes, also known as K8s (`K` + 8 letters + `s`) is an open source container orchestration tool. Kubernetes is used for automated deployment, scaling, and management of containerised applications. It can be used to manage any container, such as Docker containers or CRI-O containers. Kubernetes was built by Google, and they developed and maintained it for over 15 years, before gifting it to the Linux foundation in 2014.  

## Kubernetes Architecture

![image](https://user-images.githubusercontent.com/88166874/135991823-2811e8bd-d374-48d8-a8b9-83677e571e1a.png)

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

- To get cluster IP: `cubectl get service`

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
