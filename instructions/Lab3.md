# Kubernetes Hands-on Lab 3: Helm

## Part 1: Prepare lab environment 

### 1. Install Helm
- Windows: `winget install -e --id Helm.Helm` or `choco install kubernetes-helm` 
- Mac: `brew install helm`
- Or read for other possibilities https://helm.sh/docs/intro/install/

### [Optional] 2. Download Kubernetes/Helm plugins 
VS Code: https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools


## Part 2: Parameterize and deploy

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
for labels, images, ports...

<details><summary>solution</summary><p>

Solution can be found in directory [helm-solutions/nginx-chart](..%2Fhelm-solutions%2Fnginx-chart)

```shell
# install the deployment to the cluster
# (cd in the chart directory
helm install nginx-hands-on .
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

