# TD1 - Baby steps in observability

In this exercise, you will gain hands-on experience with the basic concepts of observability.  
You'll deploy a simple application called Squash-TM and monitor its metrics through the Minikube dashboard.  
This practical exercise is designed to help you understand the fundamentals of observability and how to use tools to monitor and troubleshoot applications.

## 1 - Start minikube

For this exercise, we'll use minikube, it is easy to grasp and we don't need the complexity of a real cluster.

1. Start the minikube

  ```Bash
  minikube start
  ```

> Note: If you wan't to eliminate the sudo rights, you'll need to add docker to sudoers:
>
> ```Bash
> sudo groupadd docker
> sudo usermod -aG docker $USER
> newgrp docker
> docker run hello-world
>  ```

2. In another shell, activate the minikube nginx ingress plugin and add the ip to your host:

  ```Bash
  # Enable nginx
  minikube addons enable ingress

  # Get the minikube ip
  minikube ip

  # Add IP to your /etc/host
  sudo vi /etc/hosts 
  # add <minikube IP>   squashtm.local.com 
  ```

3. (Optional) start you minikube tunnel for loadbalencer services:

```Bash
minikube tunnel
```

## 2 - Deploy squash

### Deploy DB

First thing, first!  
Squash-tm needs a database, there is an embedded one but today will do the formal installation.  

  ```Bash
  # 1 create a namespace for the squash-tm app:
  kubectl create ns squash

  # 2 Deploy DB
  kubectl apply -f 01-postgresql-deploy.yaml -n squash

  # 3 Check that postgresql started correctly : 
  kubectl get pods -n squash

  kubectl logs -n squash <postgresql-pod>
  ```

### Deploy Squash TM

We will apply the Config maps associated to TM first.  
Inside, there are simple configurations to add the metrics endpoint and the TM plugins.

  ```Bash
  # 1 configmaps
  kubectl apply -f 02-squash-tm-cm.yaml -n squash

  # 2 TM deployment
  kubectl apply -f 03-squash-tm-deploy.yaml -n squash
  ```

### Deploy the orchestrator

The orchestrator is a standalone application.

  ```Bash
  # 1 Start the orchestrator
  kubectl apply -f 04-squash-orchestrator-deploy.yaml -n squash

  # 2 Collect the token 
  kubectl get pods  -n squash 
  kubectl logs <orchestrator pod> -n squash
  ```

## 3 - minikube dashboard

Minikube embed the Kubernetes Dashboard, and we will use this bypass to get a few example on the collected metrics.

  ```Bash
  # 1 Apply the metric server
  minikube addons enable metrics-server

  # 2 Use the minikube dashboard
  minikube dashboard

  # 3 Wait a bit and wander in the dashboards
  ```

What is the CPU value for squash-orchestrator ?
How much memory used by the mariadb ?
