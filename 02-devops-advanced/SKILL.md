---
name: devops-advanced
description: Auto-activating advanced DevOps skill — Kubernetes, Terraform, Helm, Ansible, Istio, Prometheus/Grafana, ArgoCD, Vault, and GitOps workflows
origin: jeremylongshore/claude-code-plugins-plus-skills/02-devops-advanced
tools: Read, Write, Edit, Bash, Grep
---

# DevOps Advanced Skill

Automated assistance for advanced infrastructure: Kubernetes orchestration, Infrastructure-as-Code (Terraform/Helm), service mesh, monitoring stacks, and GitOps.

## When to Activate

Activates automatically when you mention: kubernetes, k8s, terraform, helm, ansible, istio, prometheus, grafana, argocd, flux, vault, cert-manager, configmap, deployment, ingress, service mesh, infrastructure as code.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `kubernetes-deployment-creator` | Deployment manifests, rolling updates, resource limits |
| `kubernetes-configmap-handler` | ConfigMap creation, volume mounts, env injection |
| `kubernetes-secrets-manager` | Secret creation, sealed secrets, external secrets operator |
| `kubernetes-service-manager` | ClusterIP, NodePort, LoadBalancer, ExternalName |
| `kubernetes-ingress-config` | NGINX/Traefik ingress, TLS, path routing |
| `terraform-module-creator` | Reusable modules, variables, outputs, versions |
| `terraform-state-manager` | Remote state (S3/GCS), locking, workspaces |
| `helm-chart-generator` | Chart structure, values.yaml, templates, hooks |
| `ansible-playbook-generator` | Tasks, handlers, variables, conditionals |
| `prometheus-config-generator` | Scrape configs, alerting rules, recording rules |
| `grafana-dashboard-creator` | Dashboard JSON, panels, variables, datasources |
| `argocd-app-deployer` | Application CRDs, sync policies, health checks |
| `vault-secrets-integrator` | Dynamic secrets, policies, Kubernetes auth |
| `cert-manager-setup` | ClusterIssuer, Let's Encrypt, certificate CRDs |
| `istio-service-mesh-config` | VirtualService, DestinationRule, PeerAuthentication |

## Kubernetes Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
    version: v1.0.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests: { cpu: 100m, memory: 128Mi }
          limits: { cpu: 500m, memory: 512Mi }
        readinessProbe:
          httpGet: { path: /health, port: 8080 }
          initialDelaySeconds: 10
        livenessProbe:
          httpGet: { path: /health, port: 8080 }
          initialDelaySeconds: 30
        envFrom:
        - configMapRef: { name: my-app-config }
        - secretRef: { name: my-app-secrets }
```

## Terraform Module Structure

```hcl
# modules/my-module/main.tf
variable "environment" { type = string }
variable "instance_count" { type = number, default = 2 }

resource "aws_instance" "app" {
  count         = var.instance_count
  ami           = data.aws_ami.latest.id
  instance_type = var.environment == "prod" ? "t3.large" : "t3.small"
  tags = { Environment = var.environment, ManagedBy = "terraform" }
}

output "instance_ids" { value = aws_instance.app[*].id }
```

## Helm Values Pattern

```yaml
# values.yaml
replicaCount: 3
image:
  repository: my-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths: [{ path: /, pathType: Prefix }]
resources:
  requests: { cpu: 100m, memory: 128Mi }
  limits: { cpu: 500m, memory: 512Mi }
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `02-devops-advanced` (25 skills)
