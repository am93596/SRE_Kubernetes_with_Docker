# Kubernetes With Docker

## What is Kubernetes (K8s)?
Kubernetes, also known as K8s (`K` + 8 letters + `s`) is an orchestration tool for microservices. It can be used to manage any container, such as Docker containers or CRI-O containers. Kubernetes was built by Google, and they developed and maintained it for over 15 years, before gifting it to the Linux foundation in 2014.  

## Kubernetes Deployment
*unfinished*

## Benefits of Kubernetes
*unfinished*
- self healing
- load balancing
- automated rollbacks
- autoscaling

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
  
