# ArgoCD - Excercise

## Step 1 — Start Kubernetes (Minikube)

$ kubectl get nodes

NAME STATUS ROLES AGE VERSION

minikube Ready control-plane 36d v1.34.0



## PART 2 — Install ArgoCD

## Step 2 — Create Namespace

$ kubectl create namespace argocd

namespace/argocd created

  

## Step 3 — Install ArgoCD

$ kubectl get pods -n argocd

NAME READY STATUS RESTARTS AGE

argocd-application-controller-0 1/1 Running 0 3d10h

argocd-applicationset-controller-6d87796c6c-d6jn5 1/1 Running 636 (9m9s ago) 3d10h

argocd-dex-server-5544864d7-25jvl 1/1 Running 0 3d10h

argocd-notifications-controller-5dc9c8c9dd-rjkjb 1/1 Running 29 (35m ago) 3d10h

argocd-redis-86d798767-lrcj5 1/1 Running 0 3d10h

argocd-repo-server-7bbc85547f-kjt56 0/1 Running 30 (50s ago) 3d10h

argocd-server-777ffdbd95-nch96 0/1 CreateContainerConfigError 31 (49s ago) 3d10h

  

## PART 3 — Access ArgoCD UI

### Option A — Port Forward (Simple)

$ kubectl port-forward svc/argocd-server -n argocd 8080:443

Forwarding from 127.0.0.1:8080 -> 8080

Forwarding from [::1]:8080 -> 8080

Handling connection for 8080

  

## Step 4 — Login

$ argocd login localhost:8080 --username admin --password \<argoCD password\> --insecure

'admin:login' logged in successfully

Context 'localhost:8080' updated

  

## PART 4 — Prepare GitHub Helm Repository

Create GitHub repository: helm-gitops-demo

```
**<u>helm-gitops-demo/</u>**
.
├── charts
│   └── myapp
│       ├── Chart.yaml
│       ├── templates
│       │   ├── deployment.yaml
│       │   ├── _helpers.tpl
│       │   ├── NOTES.txt
│       │   ├── serviceAccount.yaml
│       │   └── service.yaml
│       ├── values-dev.yaml
│       ├── values-prod.yaml
│       └── values.yaml
├── envs
│   ├── dev
│   │   ├── dev-app.yaml
│   │   └── values-dev.yaml
│   └── prod
│       ├── prod-app.yaml
│       └── values-prod.yaml
└── README.md
```


## Step 5 — Create Helm Chart

$ helm create myapp

### Push to GitHub

$ git init
$ git add .
$ git commit -m "initial helm chart"
$ git branch -M main
$ git remote add origin https://github.com/zvr123/helm-gitops-demo.git
$ git push -u origin main
  

### Create dev branch

$ git checkout -b dev
$ git push origin dev

  

## PART 5 — How ArgoCD Connects to GitHub

**Public Repository**
		No credentials needed.

$ This error means Argo CD already knows this repo, but with different credentials than what you're trying to use now.

Repository '<https://github.com/zvr123/helm-gitops-demo.git>' added

  
**Private Repository**

Add using CLI:

$ argocd repo add <https://github.com/>\<your-user\>/helm-gitops-demo.git --username \<github-username\> --password \<github-token\>

  

## PART 6 — Deploy Dev Application

## Step 6 — Create dev Application

**Apply**:

$ kubectl apply -f envs/dev/dev-app.yaml

application.argoproj.io/myapp-dev configured

  
**Verify**:

$ kubectl get applications -n argocd

<mark style="background:#d3f8b6">NAME             SYNC STATUS    HEALTH STATUS</mark>

<mark style="background:#d3f8b6">hello-app        Synced                Healthy</mark>

<mark style="background:#d3f8b6">myapp-dev     Unknown             Degraded</mark>

<mark style="background:#d3f8b6">myapp-prod    Synced                Degraded</mark>

  

$ kubectl get pods -n dev

<mark style="background:#d3f8b6">NAME                                                       READY    STATUS RESTARTS AGE</mark>

<mark style="background:#d3f8b6">app-deployment-77cf88dc74-bvt9g     1/1           Running 4 (20m ago) 21d</mark>

<mark style="background:#d3f8b6">app-deployment-77cf88dc74-dbbwg    1/1          Running 4 (20m ago) 21d</mark>

<mark style="background:#d3f8b6">app-deployment-77cf88dc74-hb4v9     1/1          Running 4 (20m ago) 21d</mark>

<mark style="background:#d3f8b6">app-deployment-77cf88dc74-nl5wv     1/1           Running 4 (20m ago) 21d</mark>

<mark style="background:#d3f8b6">app-deployment-77cf88dc74-sss8w     1/1          Running 4 (20m ago) 21d</mark>

  
  

## PART 7 — Deploy Prod Application

**Apply:**

$ kubectl apply -f envs/prod/prod-app.yaml

application.argoproj.io/myapp-prod configured

  
**Verify:**

$ kubectl get pods -n prod

<mark style="background:#d3f8b6">NAME READY STATUS RESTARTS AGE</mark>

<mark style="background:#d3f8b6">myapp-prod-558f4648f8-lxs57 1/1 Running 0 69s</mark>

<mark style="background:#d3f8b6">myapp-prod-558f4648f8-n9j4w 1/1 Running 0 71s</mark>

<mark style="background:#d3f8b6">myapp-prod-558f4648f8-x6fkn 1/1 Running 0 70s</mark>

  
  

## PART 8 — GitOps Reconciliation Demo

$ git add .

$ git commit -m "scale dev to 2 replicas"

$ git push origin dev

  

$ kubectl get pods -n dev

<mark style="background:#d3f8b6">NAME READY STATUS RESTARTS AGE</mark>

<mark style="background:#d3f8b6">myapp-dev-6f54bbd4b4-2pmx6 1/1 Running 0 2m22s</mark>

<mark style="background:#d3f8b6">myapp-dev-6f54bbd4b4-7jbkr 1/1 Running 0 2m30s</mark>