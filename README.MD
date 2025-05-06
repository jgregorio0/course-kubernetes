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

# Kubernetes Course

- [kubernetes-bootcamp-kubernetes-from-zero-to-cloud](https://sngular.udemy.com/course/kubernetes-bootcamp-kubernetes-from-zero-to-cloud)

## Introduction

- Master node: controls and manages the green Worker nodes

- Worker node: Run Pods, execute containers and handle workloads

- Pod: basic execution unit, a logical host for one or more containers

- Labels: key-value pairs metadata attached to objects (like pods) that help organize, filter, and select resources

- Resource requirements: define the minimum CPU and memory needed by a pod to run properly

## Section 1: Pod

- [Pod configuration file](01-section-1-pod/grade-submission-portal-pod.yml)

- Create or update a Pod

```
kubectl apply -f grade-submission-portal-pod.yml

kubectl apply -f grade-submission-api-pod.yml
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

## Section 2: Service

[Service NodePort](02-section-2-service/grade-submission-portal-service.yml)

- Expose static port 32000

```
apiVersion: v1
kind: Service
metadata:
  name: grade-submission-portal
spec:
  type: NodePort
  selector:
    app.kubernetes.io/instance: grade-submission-portal
  ports:
  - port: 5001
    targetPort: 5001
    nodePort: 32000
```

- Run the sample

```
kubectl apply -f grade-submission-portal-pod.yml

kubectl apply -f grade-submission-portal-service.yml

kubectl apply -f grade-submission-api-pod.yml

kubectl apply -f grade-submission-api-service.yml
```

- Access <minikube ip>:32000:

```
minikube service grade-submission-portal --url -n grade-submission
```

## Section 3: Namespace

[03-section-3-namespace](03-section-3-namespace)

- Create specific namespace

```
kubectl create namespace grade-submission
```

- Apply into namespace

```
kubectl apply -f ./ -n grade-submission
```

- Define a namespace in metadata

```
namespace: grade-submission
```

- Get pods and services in namespace

```
kubectl get pods,services -n grade-submission
```

## Section 4: Rolling Update

When you update the pod template in a Deployment resource, the Deployment controller creates a new ReplicaSet for v2 while preserving the old v1.
During a rolling update, the Deployment manages two ReplicaSets: v1 and v2.  
New v2 pods are created while old v1 pods are gradually terminated. This process continues until all pods are running the new version.

- maxUnavailable: The maximum number of pods that can be unavailable during the update.
- maxSurge: The maximum number of pods that can be created over the desired number of pods.
  Benefits of Rolling Updates

Benefits:

- Minimal Downtime
- Version Control
- Traffic Management

## Section 5: Liveness and readiness

```
livenessProbe: # Checks container health at /healthz endpoint
  httpGet:
    path: /healthz
    port: 3000
  initialDelaySeconds: 15 # Waits 15 seconds after container starts before first check
  periodSeconds: 5 # Performs checks every 5 seconds
readinessProbe: # Detects if container needs to be restarted
  httpGet:
    path: /readyz
    port: 3000
  initialDelaySeconds: 30 # Waits 30 seconds before first check
  periodSeconds: 5 # Performs checks every 5 seconds
```

## Section 10: Deployments and Pod Replicas

Self-healing:

- Automatically recreates Pods that fail
- Maintains desired number of replicas

Rolling Updates:

- Updates Pods gradually to prevent downtime
- Allows easy rollback to previous versions
- Configurable update strategy

Scalability:

- Scale deployments using kubectl scale
- Automatic load balancing
- Flexible resource allocation

Limitations:

- Not suitable for stateful applications
- Cannot guarantee Pod termination order
- Limited control over individual Pods
- Rollback requires manual intervention in some cases

[api deployment](10-section-10-deployments/grade-submission-api-deployment.yml) 
[portal deployment](10-section-10-deployments/grade-submission-portal-deployment.yml)

```
kubectl delete services,pods --all -n grade-submission

kubectl apply -f .

kubectl get deployments -n grade-submission

watch -n 5 'kubectl get pod -n grade-submission'

minikube service grade-submission-portal --url -n grade-submission
```



## Section 13: StatefulSet and Persistent Volumes

Network identity:

- Each Pod derives its hostname from the StatefulSet name and ordinal
- Pattern follows $(statefulset-name)-$(ordinal). Example: web-0, web-1, web-2

Storage Management:

- Uses PersistentVolumeClaims for stable storage
- Volumes persist across Pod recreation
- Each Pod receives its own PersistentVolumeClaim

Deployment Guarantees:

- Sequential deployment from ordinal 0 to N-1
- Ordered termination from N-1 to 0
- Waits for predecessors to be Running and Ready before deploying successors

Limitations:

- Requires manual creation of a Headless Service
- Storage volumes aren't deleted when StatefulSet is scaled down
- No guarantees on Pod termination order when deleting StatefulSet
- Can enter broken states during Rolling Updates requiring manual intervention

- [StatefulSet](13-section-13-stateful-and-persistent-volumes/mongodb/mongodb-statefulset.yml)

```
kubectl delete deployments,services,pods,statefulsets,pvc --all -n grade-submission

kubectl apply -f .

kubectl get statefulset -n grade-submission

kubectl get pods -n grade-submission

kubectl logs -f mongodb-0 -n grade-submission

kubectl get pvc -n grade-submission

minikube service grade-submission-portal --url -n grade-submission

watch -n 1 'kubectl get pod -n grade-submission'
```

## Section 14: ConfigMap and Secret

Manage configuration and sensitive data in your cluster


ConfigMap:

ConfigMaps in Kubernetes are used to store non-confidential configuration data in key-value pairs

- data: For UTF-8 strings
- binaryData: For base64-encoded binary data kubernetes.io
- Keys must use alphanumeric characters, -, _, or .
- Names must be valid DNS subdomain names

Secret:

Specifically designed for sensitive information like passwords, OAuth tokens, and SSH keys

- Opaque (default): Arbitrary user-defined data
- kubernetes.io/service-account-token
- kubernetes.io/dockerconfigjson
- kubernetes.io/basic-auth
- kubernetes.io/ssh-auth
- kubernetes.io/tls kubernetes.io


```
kubectl apply -f db

kubectl apply -f api

kubectl apply -f portal

watch -n 1 'kubectl get pod -n grade-submission'

minikube service grade-submission-portal --url -n grade-submission
```

## Section 14: Horizontal autoscaler

Install metrics API
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type="json" -p="[{\"op\": \"add\", \"path\": \"/spec/template/spec/containers/0/args/-\", \"value\":\"--kubelet-insecure-tls\"}]"
```

Deploy hpa
```
kubectl apply -f portal

kubectl get hpa -n grade-submission

kubectl delete hpa grade-submission-portal-hpa

kubectl top pods -n grade-submission
```

