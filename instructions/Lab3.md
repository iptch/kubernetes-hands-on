# Kubernetes Hands-on Lab 3: High Availability

## Part 1: Liveness and readiness probes
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

### 1. Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.
<details><summary>solution</summary><p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # our probe
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml

# Check the probe:
# On Linux or Mac, run:
kubectl describe pod nginx | grep -i liveness
# On Windows, run:
kubectl describe pod nginx | findstr -I liveness

kubectl delete -f pod.yaml
```
</p></details>

### 2. Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.
<details><summary>solution</summary><p>

```bash
kubectl explain pod.spec.containers.livenessProbe # get the exact names
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 5 # add this line
      periodSeconds: 5 # add this line as well
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml

# Check the probe:
# On Linux or Mac, run:
kubectl describe pod nginx | grep -i liveness
# On Windows, run:
kubectl describe pod nginx | findstr -I liveness

kubectl delete -f pod.yaml
```
</p></details>

### 3. Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.
<details><summary>solution</summary><p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
      - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml

# Check the probe:
# On Linux or Mac, run:
kubectl describe pod nginx | grep -i readiness
# On Windows, run:
kubectl describe pod nginx | findstr -I readiness

kubectl delete -f pod.yaml
```
</p></details>

## Part 2: Anti-Affinities
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

### 1. Use [the nginx deployment from Lab 1](Lab1.md#rewrite-deployment): Extend it so that the replicas are scheduled on different nodes.
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
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname"
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
      dnsPolicy: ClusterFirst
      restartPolicy: Always
```
</p></details>

### 2. Check if the pods from the previous task are indeed scheduled on different nodes.
<details><summary>solution</summary><p>
Get the pods by their label and use the "wide" output format to display the nodes as well. Then check if the nodes in the node column are different or exactly the same.

```bash
kubectl get pods -l "app=nginx" -o wide
```
</p></details>

### 3. To how many replicas do you have to scale the deployment until they can't be scheduled on different nodes anymore?
<details><summary>solution</summary><p>
Check how many nodes there are using the following command:

```bash
kubectl get nodes
```
</p></details>