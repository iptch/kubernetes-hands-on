# Kubernetes Hands-on Lab 4: Monitoring and Debugging

## Part 1: Monitoring

Go to Grafana: https://kubernetes-hands-on-dpf8fda4gbdqe4bs.weu.grafana.azure.com/

### 1. Let’s try to query some metrics
1. Go to "Explore"
2. Choose our Prometheus as the datasource
3. Switch between "builder" and "code" to create the query

### 2. Create a dashboard
1. Go to "Dashboards"
2. Have a look at the existing dashboards
3. Create a folder for your dashboards
4. Create a dashboard for your own apps in your namespace (show CPU and memory use etc.)

## Part 2: Debugging

### 1. Create a busybox pod that runs the command below. Check its logs.
`i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done`

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
kubectl get events | grep -i error # you'll see the error here as well
kubectl delete po busybox --force --grace-period=0
```
</p></details>


### 4. Get CPU/memory utilization for nodes

<details><summary>solution</summary><p>

```bash
kubectl top nodes
```
</p></details>