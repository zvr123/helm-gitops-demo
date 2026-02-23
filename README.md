PART 4 — Prepare GitHub Helm Repository

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
