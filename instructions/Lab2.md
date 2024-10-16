# Kubernetes Hands-on Lab 2: Advanced Topics

## Part 1: Multi-container pods

### 1. Create a deployment with a sidecar
Write a deployment as yaml with the following properties:
- Name the deployment "sidecar-deployment"
- The first container is called "main-container". It is a busybox (image=busybox) that writes "Hello from main-container" to a file every 3 seconds: `while true; do echo "Hello from main-container" >> /mnt/log.txt;sleep 3; done;`
- The second container is called "sidecar-container". It is a busybox that reads the file and logs each line: `tail -f /mnt/log.txt`
- Use an empty volume to write to and read from: https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
Deploy the yaml.
<details><summary>solution</summary><p>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: sidecar-deployment
  name: sidecar-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sidecar-deployment
  template:
    metadata:
      labels:
        app: sidecar-deployment
    spec:
      containers:
      - name: main-container
        image: busybox
        command: ['sh', '-c', 'while true; do echo "Hello from main-container" >> /mnt/log.txt;sleep 3; done;']
        volumeMounts:
        - name: logging-data
          mountPath: /mnt
      - name: sidecar-container
        image: busybox
        command: ['sh', '-c', 'tail -f /mnt/log.txt']
        volumeMounts:
        - name: logging-data
          mountPath: /mnt
      volumes:
      - name: logging-data
        emptyDir: {}
```
```bash
kubectl create -f sidecar-deployment.yaml
```
</p></details>

### 2. Read the logs from each container
Read the logs continuously using the `follow` flag. Use: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs

If everything worked out, you should only see logs from the sidecar-container.
<details><summary>solution</summary><p>

```bash
kubectl logs deployment/sidecar-deployment -c main-container --follow
kubectl logs deployment/sidecar-deployment -c sidecar-container --follow
```
</p></details>

### 3. Create a pod with an initContainer. 
- Create a pod with an nginx container exposed on port 80. 
- Add a busybox initContainer which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". 
- Make a volume of type emptyDir and mount it in both containers. 
- For the nginx container, mount it on "/usr/share/nginx/html" and for the initContainer, mount it on "/work-dir". 
- Do a port-forward to the pod (port 80 to port 80 on your machine) and open localhost:80 in a browser. You should see the downloaded website.
<details><summary>solution</summary><p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  initContainers:
  - image: busybox
    name: box
    command: ['sh', '-c', 'wget -O /work-dir/index.html http://neverssl.com/online']
    volumeMounts:
      - name: vol
        mountPath: /work-dir
  containers:
    - image: nginx
      name: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: vol
          mountPath: /usr/share/nginx/html
  volumes:
    - name: vol
      emptyDir: {}
```
```bash
kubectl port-forward pod/init-container-pod 8080:80
```
</p></details>

## Part 2: Config Maps
https://kubernetes.io/docs/concepts/configuration/configmap/

### 1. Create a configmap named config with values foo=lala,foo2=lolo
<details><summary>solution</summary><p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```
</p></details>

### 2. Display its values
<details><summary>solution</summary><p>

```bash
kubectl get cm config -o yaml
# or
kubectl describe cm config
```
</p></details>

### 3. Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```
<details><summary>solution</summary><p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```
</p></details>

### 4. Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

Hint: Read https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap
<details><summary>solution</summary><p>

```bash
kubectl create cm options --from-literal=var5=val5
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
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- sh -c "env | grep option" # will show 'option=val5'
```
</p></details>

## Part 3: Secrets

- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/

### 1. Create a secret called mysecret with the values password=mypass
<details><summary>solution</summary><p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```
</p></details>

### 2. Get the value of mysecret
<details><summary>solution</summary><p>

```bash
kubectl get secret mysecret -o yaml
# Decode the base64 encoded value:
echo -n bXlwYXNz | base64 -d # on MAC it is -D, which decodes the value and shows 'mypass'. 
# On Windows, just use https://www.base64decode.org/ or store the value in a file and run:
certutil -decode input.txt output.txt
```
</p></details>

### 3. Create an nginx pod that mounts the secret mysecret in a volume on path /etc/foo
<details><summary>solution</summary><p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/bash
ls /etc/foo  # shows password
cat /etc/foo/password # shows mypass
```
</p></details>

### 4. Delete the pod you just created and mount the variable 'password' from secret mysecret onto a new nginx pod in env variable called 'PASSWORD'
Show the environment variable using `kubectl exec -it nginx -- sh -c "env | grep PASSWORD"`
<details><summary>solution</summary><p>

```bash
kubectl delete pod nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
    env: # our env variables
    - name: PASSWORD # asked name
      valueFrom:
        secretKeyRef: # secret reference
          name: mysecret # our secret's name
          key: password # the key of the data in the secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- sh -c "env | grep PASSWORD"  # will show 'PASSWORD=mypass'
```
</p></details>

## Part 4: Debugging

### 1. Create a busybox pod that runs the command below. Check its logs.
The command that should be run is `/bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'`. 

<details><summary>solution</summary><p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # follow the logs
```
</p></details>

### 2. Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>solution</summary><p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```
</p></details>

### 3. Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

<details><summary>solution</summary><p>

```bash
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # will bring nothing! container never started
kubectl describe po busybox # in the events section, you'll see the error
# also...

# On Linux/Mac, run:
kubectl get events | grep -i error # you'll see the error here as well
# On Windows, run:
kubectl get events | findstr -I error

kubectl delete po busybox --force --grace-period=0
```
</p></details>


### 4. Get CPU/memory utilization of the nodes

<details><summary>solution</summary><p>

```bash
kubectl top nodes
```
</p></details>
