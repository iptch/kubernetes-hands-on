# Kubernetes Hands-on Lab 1: Setup and the Basics

## Part 1: Setup Lab Environment

### 1. Download kubectl
- Windows: `winget install kubectl`
- Mac: `brew install kubectl`

### 2. Download the config file into the .kube folder in your home directory
- `~/.kube/config`
- Hint: The config file configures your namespace as the default namespace. If you don't specify another namespace, your commands will be run in your own namespace and you can omit the "-n" flag to select a namespace. 

### 3. Run a command to test if it works:
- `kubectl get namespaces`

### 4. [Optional] Configure: Alias, autocomplete, etc.
https://kubernetes.io/docs/reference/kubectl/quick-reference/

### 5. [Optional] Get familiar with kubectl doc and try out commands
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

### 5. [Optional] Install k9s 
K9s is a terminal based UI to interact with your Kubernetes clusters: https://k9scli.io/
- Windows: `winget install k9s`
- Mac: `brew install derailed/k9s/k9s`

## Part 2: The Basics

### 1. Get the yaml definition of your namespace via kubectl 
Hint: Use the "get" command and the output flag as described here: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
<details><summary>solution</summary><p>

```bash
kubectl get ns <your short name> -o yaml
```
</p></details>

### 2. Run and interactively connect to a busybox container, view its environment variables, then exit the container
Hint: Use the "run" command as described here: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run
<details><summary>solution</summary><p>

```bash
kubectl run busybox --image=busybox -it
env
exit
```
</p></details>

### 3. Show all the pods in your namespace
Hint: Usually we'd need to select our namespace with the `-n <your short name>` flag. Since we already configured our namespace in the `~/.kube/config` file, the flag is optional for these tasks.
<details><summary>solution</summary><p>

```bash
kubectl get pods
```
</p></details>

### 4. Show all the pods in all namespaces
<details><summary>solution</summary><p>

```bash
kubectl get pods --all-namespaces
```
</p></details>

### 5. Delete your busybox pod from before
<details><summary>solution</summary><p>

```bash
kubectl delete pod busybox
```
</p></details>

### 6. Generate a basic yaml config for an nginx pod that exposes port 80
Store the yaml as a file `nginx.yaml` in your current directory.

Hint: Use the dry-run flag and the output flag
<details><summary>solution</summary><p>

```bash
kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml > nginx.yaml
```
</p></details>

### 7. Rewrite the yaml file from a pod configuration to a deployment named "nginx-deployment" with 2 replicas
Hint: Use the pod as the template for the deployment. See how a deployment is configured here: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Hint 2: You can also use kubectl to create a deployment in dry-run mode and with output as yaml to see how a deployment looks like.
<details><summary>solution</summary><p>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
      dnsPolicy: ClusterFirst
      restartPolicy: Always
```
</p></details>

### 8. Deploy the yaml file
<details><summary>solution</summary><p>

```bash
kubectl create -f nginx.yaml
```
</p></details>

### 9. Scale your nginx deployment to 3 replicas
View the results by getting the deployment.
<details><summary>solution</summary><p>

```bash
kubectl scale --replicas=3 deployment/nginx-deployment
kubectl get deployment
```
</p></details>



## Part 3: Services

### 1. Create a service that points to the nginx deployment and serves on port 8080
<details><summary>solution</summary><p>

```bash
kubectl expose deployment nginx-deployment --port=8080 --target-port=80
```
</p></details>

### 2. View all the resources in your namespaces.
You should see your new service, the deployment and its pods.
<details><summary>solution</summary><p>

```bash
kubectl get all
```
</p></details>

### 3. Connect to the service via port-forward command of kubectl and view the nginx start page
Forward form the service port 8080 to your own port 8080: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

<details><summary>solution</summary><p>

```bash
kubectl port-forward service/nginx-deployment 8080:8080
# Open http://localhost:8080/ in a browser 
```
</p></details>


## [OPTIONAL] Part 4: kubectl explain
You can use `kubectl explain` to find out more about a Kubernetes resource and its fields:

### 1. Find out more about services using kubectl explain
<details><summary>solution</summary><p>

```bash
kubectl explain services
```
</p></details>

 ### 2. Find out more about pods using kubectl explain
<details><summary>solution</summary><p>

```bash
kubectl explain pods
```
</p></details>

 ### 3. Find out more about the containers within a pod using kubectl explain
<details><summary>solution</summary><p>

```bash
kubectl explain pods.spec.containers
```
</p></details>

