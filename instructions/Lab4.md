# Kubernetes Hands-on Lab 4: Monitoring and Debugging

## Part 1: Monitoring

Go to Grafana: https://kubernetes-hands-on-dpf8fda4gbdqe4bs.weu.grafana.azure.com/

### 1. Let’s try to query some metrics
1. Go to "Explore"
2. Choose our Prometheus as the datasource
3. Switch between "builder" and "code" to create the queries below
    - You can also use the "Metrics browser" to help creating the queries

#### Query 1: Show the number of pods running in your namespace (use kube_pod_info and filter by your namespace).

<details><summary>solution</summary><p>

```
count(kube_pod_info{namespace="<your namespace>"})
```
</p></details>

#### Query 2: Show the number of pods running in each namespace.

<details><summary>solution</summary><p>

```
count(kube_pod_info) by (namespace)
```
</p></details>

#### Query 3: Show the cpu usage of each pod in your namespace
For the query, use the metric `node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate`. Then create some load:

1. Create a pod that generates some cpu load: `kubectl run high-cpu-usage --image busybox -- sh -c "while true; do echo 'hello world'; done;"`
2. Create a second pod that doesn't generate load for comparison: `kubectl run low-cpu-usage --image busybox -- sh -c "while true; do echo 'hello world'; sleep 3; done;"`
3. Delete the pods again after you're done: `kubectl delete pod high-cpu-usage; kubectl delete pod low-cpu-usage`
<details><summary>solution</summary><p>

```
sum(node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate{namespace="bbr"}) by (pod)
```
</p></details>

### 2. Dashboards
1. Go to "Dashboards"
2. Have a look at the existing dashboards in the "managed-prometheus" folder and answer the following questions:
    1. What namespace is using the most CPU resources? (Use the "Kubernetes / Compute Resources / Cluster" dashboard)
    2. Did the CPU usage in your namespace increase after the previous task? (Use the "Kubernetes / Compute Resources / Cluster" dashboard) 
    3. How many pods and containers are currently running? (Use the "Kubernetes / Kubelet" dashboard)
3. Create a folder for your own dashboards (use your ipt short name as the folder name)
4. Create a dashboard for your own apps in your namespace (show CPU and memory use etc.)
    - If there are currently no pods running in your namespace, create a few nginx or busybox pods

## Part 2: Debugging

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


### 4. Get CPU/memory utilization for nodes

<details><summary>solution</summary><p>

```bash
kubectl top nodes
```
</p></details>
