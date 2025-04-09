# Minikube

## Reference

- [minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download)

## Requirements

- Docker 18.09 or higher (20.10 or higher is recommended)
- amd64 or arm64 system.
- Don’t forget to follow this step to manage Docker as a non-root user

## Install Minikube

1. Download

```
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
```

2. Install

```
sudo install minikube-linux-amd64 /usr/local/bin/minikube 
```

3. Remove install file

```
rm minikube-linux-amd64
```

## Start

Start a cluster using the docker driver:

```
minikube start --driver=docker
```

# Kubernetes

## References

- [Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux)

## Install Kubectl

1. Download

```
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

2. Download checksum
```
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

3. Validate checksum
```
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

4. Install Kubernetes
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

5. Check version
```
kubectl version --client
```

6. Check current context:

```
kubectl config current-context
```

## Introduction

- Master node: controls and manages the green Worker nodes

- Worker node: Run Pods, execute containers and handle workloads

- Pod: basic execution unit, a logical host for one or more containers

- Labels: key-value pairs metadata attached to objects (like pods) that help organize, filter, and select resources

- Resource requirements: define the minimum CPU and memory needed by a pod to run properly

## Section 1: create a Pod

- [Pod configuration file](section-one/grade-submission-portal-pod.yml)

- Create or update a Pod

```
kubectl apply -f grade-submission-portal-pod.yml
```

- Check Pods

```
kubectl get pods
```

- Describe Pod

```
kubectl describe pod grade-submission-portal
```

- Stream Pod's logs

```
kubectl logs -f lograde-submission-portal

kubectl logs -f grade-submission-portal -c grade-submission-portal-health-checker
```

- Enable access fordwarding port
```
kubectl port-forward grade-submission-portal 8080:5001
```

