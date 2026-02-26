ArgoCD - Excercise
=====================

Step 1 — Start Kubernetes (Minikube)
------------------------------------

$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   36d   v1.34.0

PART 2 — Install ArgoCD
-------------------------

Step 2 — Create Namespace
--------------------------

$ kubectl create namespace argocd
namespace/argocd created

Step 3 — Install ArgoCD
------------------------

$ kubectl get pods -n argocd
NAME                                                READY   STATUS                       RESTARTS         AGE
argocd-application-controller-0                     1/1     Running                      0                3d10h
argocd-applicationset-controller-6d87796c6c-d6jn5   1/1     Running                      636 (9m9s ago)   3d10h
argocd-dex-server-5544864d7-25jvl                   1/1     Running                      0                3d10h
argocd-notifications-controller-5dc9c8c9dd-rjkjb    1/1     Running                      29 (35m ago)     3d10h
argocd-redis-86d798767-lrcj5                        1/1     Running                      0                3d10h
argocd-repo-server-7bbc85547f-kjt56                 0/1     Running                      30 (50s ago)     3d10h
argocd-server-777ffdbd95-nch96                      0/1     CreateContainerConfigError   31 (49s ago)     3d10h

PART 3 — Access ArgoCD UI
------------------------------

Option A — Port Forward (Simple)
-------------------------------------

$ kubectl port-forward svc/argocd-server -n argocd 8080:443
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080

Step 4 — Login
---------------

$ argocd login localhost:8080 --username admin --password \<argoCD password\> --insecure
'admin:login' logged in successfully
Context 'localhost:8080' updated

PART 4 — Prepare GitHub Helm Repository
------------------------------------------

Create GitHub repository: helm-gitops-demo

helm-gitops-demo/
 ├── charts/
 │    └── myapp/
 │         ├── Chart.yaml
 │         ├── values.yaml
 │         └── templates/
 │              ├── deployment.yaml
 │              └── service.yaml
 ├── env/
 │    └── dev/
 │         ├── values-dev.yaml
 │         └── dev-app.yaml
 │    └── prod/
 │         ├── values-prod.yaml
 │         └── prod-app.yaml

Step 5 — Create Helm Chart
---------------------------

$ helm create myapp

Push to GitHub
---------------

$ git init
$ git add .
$ git commit -m "initial helm chart"
$ git branch -M main
$ git remote add origin https://github.com/zvr123/helm-gitops-demo.git
$ git push -u origin main

Create dev branch
---------------------

$ git checkout -b dev
$ git push origin dev

PART 5 — How ArgoCD Connects to GitHub
---------------------------------------

Public Repository
No credentials needed.
$ This error means Argo CD already knows this repo, but with different credentials than what you're trying to use now.
Repository '<https://github.com/zvr123/helm-gitops-demo.git>' added

Private Repository
Add using CLI:
$ argocd repo add <https://github.com/>\<your-user\>/helm-gitops-demo.git --username \<github-username\> --password \<github-token\>

PART 6 — Deploy Dev Application
--------------------------------

Step 6 — Create dev Application
-------------------------------

Apply:
$ kubectl apply -f envs/dev/dev-app.yaml
application.argoproj.io/myapp-dev configured

Verify:
$ kubectl get applications -n argocd
NAME         SYNC STATUS   HEALTH STATUS
hello-app    Synced        Healthy
myapp-dev    Unknown       Degraded
myapp-prod   Synced        Degraded

$ kubectl get pods -n dev
NAME                              READY   STATUS    RESTARTS      AGE
app-deployment-77cf88dc74-bvt9g   1/1     Running   4 (20m ago)   21d
app-deployment-77cf88dc74-dbbwg   1/1     Running   4 (20m ago)   21d
app-deployment-77cf88dc74-hb4v9   1/1     Running   4 (20m ago)   21d
app-deployment-77cf88dc74-nl5wv   1/1     Running   4 (20m ago)   21d
app-deployment-77cf88dc74-sss8w   1/1     Running   4 (20m ago)   21d


PART 7 — Deploy Prod Application
----------------------------------

Apply:
$ kubectl apply -f envs/prod/prod-app.yaml
application.argoproj.io/myapp-prod configured

Verify:
$ kubectl get pods -n prod


PART 8 — GitOps Reconciliation Demo
--------------------------------------

$ git add .
$ git commit -m "scale dev to 2 replicas"
$ git push origin dev

$ kubectl get pods -n dev
NAME                              READY   STATUS    RESTARTS      AGE
app-deployment-77cf88dc74-bvt9g   1/1     Running   4 (89m ago)   22d
app-deployment-77cf88dc74-dbbwg   1/1     Running   4 (89m ago)   22d
app-deployment-77cf88dc74-hb4v9   1/1     Running   4 (89m ago)   22d
app-deployment-77cf88dc74-nl5wv   1/1     Running   4 (89m ago)   22d
app-deployment-77cf88dc74-sss8w   1/1     Running   4 (89m ago)   22d

