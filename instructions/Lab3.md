# Kubernetes Hands-on Lab 3: Helm

Helm Commands are documented here: https://helm.sh/docs/helm/helm/#see-also

(This course is based on the IBM Helm 101 Workshop https://ibm.github.io/helm101/)

## Part 1: Prepare lab environment 

### 1. Install Helm
- Windows: `winget install -e --id Helm.Helm` or `choco install kubernetes-helm` 
- Mac: `brew install helm`
- Or read for other possibilities https://helm.sh/docs/intro/install/


### 2. Delete all resources in your namespace

```shell
kubectl delete all --all
```

### [Optional] 3. Download Kubernetes/Helm plugins 
VS Code: https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools


## Part 2: Parameterize and deploy

### 1. Folder structure

Create a minimal folder structure like the following:

```text
+-Chart.yaml
+-templates/
+-values.yaml
```

Content for the Chart.yaml:
```yaml
apiVersion: v2
name: guestbook-<your ipt Shortcut>
description: A Helm chart for a guestbook with redis
version: 0.1.0
appVersion: "0.1"
```

### 2. Parametrize a deployment

There are some prepared resources in the [guestbook-resources](..%2Fguestbook-resources) folder.
Copy all resource in your templates folder.

For reference, here is the helm documentation: https://helm.sh/docs/chart_template_guide/

Taks: Parametrize the all the following attributes:

1. replicaCount of the guestbook-deployment
2. image and tag used for guestbook
3. port and type of the guestbook-service
4. port of the redis services


<details><summary>solution</summary><p>

values.yaml:

```yaml
replicaCount: 2

image:
  repository: ibmcom/guestbook
  tag: v1

service:
  type: LoadBalancer
  port: 3000

redis:
  port: 6379
```

</p></details>


### 3. Conditionals

Create a boolean value in the values.yaml whether the redis-slave should be used or not (set it to `true`). 


<details><summary>solution</summary><p>

```yaml
{{- if .Values.redis.slaveEnabled }}
apiVersion: apps/v1
kind: Deployment
...
{{- end }}
```
The same for the redis-slave-service. Full solution can be found here: [guestbook-solution](..%2Fhelm-solutions%2Fguestbook-solution)

</p></details>


### 4. Deploy your chart

Give your helm-deployment a name and put it on the cluster.

Documentation: https://helm.sh/docs/helm/helm_install/

<details><summary>solution</summary><p>

```bash
helm install my-guestbook .
```
</p></details>


### 5. View you deployment

1. Have a look at the resources that are created (pods, services, deployments)
2. Look up the external IP of the guestbook-service
3. View the created guestbook page in the browser (use http and the port configured in the service e.g. http://20.86.218.180:3000/)
4. Find out the external ip address of someone elses guestbook and leave him a nice comment (if you are the first one, you need to help someone else first)

<details><summary>solution</summary><p>

```bash
# get you own external ip:
kubectl get services -o wide

# get all services and find another service with a external IP
kubectl get services -A
```
</p></details>


### 6. Update your helm chart

You currently have a master and a slave redis database. 
Disable the slave via the attribute in the values.yaml (configured in Task 3).

Now update your deployment so that the changes become active.
Also have a look at the resources with kubectl.

<details><summary>solution</summary><p>

```bash
helm upgrade my-guestbook .
kubectl get all
```
</p></details>


### 7. View the history and roll back

First view the history of your helm chart, then rollback to revision 1 and view the history again

<details><summary>solution</summary><p>

```bash
helm history my-guestbook
helm rollback my-guestbook 1
helm history my-guestbook
```
</p></details>
