# Kubernetes Hands-on Lab 3: Helm

Helm Commands are documented here: https://helm.sh/docs/helm/helm/#see-also

## Part 1: Prepare lab environment 

### 1. Install Helm
- Windows: `winget install -e --id Helm.Helm` or `choco install kubernetes-helm` 
- Mac: `brew install helm`
- Or read for other possibilities https://helm.sh/docs/intro/install/


### 2. Delete all resources in your namespace

### [Optional] 2. Download Kubernetes/Helm plugins 
VS Code: https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools


## Part 2: Getting started

### 1. List helm deployments

There are already some helm deployments in the cluster. List deployments from all namespaces.

<details><summary>solution</summary><p>

```bash
helm list -A
```
</p></details>

### 2. Metadata

There is already a kubernetes dashboard deployed via helm. Get the metadata information for this helm deployment. (pay attention to the correct namespace)


<details><summary>solution</summary><p>

```bash
helm get metadata kubernetes-dashboard -n kubernetes-dashboard
```

</p></details>




## Part 3: Parameterize and deploy

### 1. Folder structure

Create a minimal folder structure like the following:

```text
+-Chart.yaml
+-templates/
| +-deployment.yaml
| +-service.yaml
+-values.yaml
```

Content for the Chart.yaml:
```yaml
apiVersion: v2
name: hands-on-<your ipt Shortcut>
description: A Helm chart for Kubernetes
version: 0.1.0
appVersion: "0.1"
```

### 2. First deployment

Create a simple deployment with 2 nginx pods and point a service to them. Use variables from the values.yaml 
for labels, images, ports and everything that seems to make sense

You can use the yaml files and commands form the last exercises. 

<details><summary>solution</summary><p>

Solution can be found in directory [helm-solutions/nginx-chart](..%2Fhelm-solutions%2Fnginx-chart)

```shell
# install the deployment to the cluster
# cd in the correct directory of the Chart.yaml
helm install nginx-<ipt acronym> .
```

</p></details>





```skizze
Projekt aufbauen und viele ressourcen miteinander verbinden in dem variablen verwendet werden für Labels & Co.
ein vorhandenes helm chart deployen
```


<details><summary>solution</summary><p>

```bash
$ <command>
```
</p></details>

