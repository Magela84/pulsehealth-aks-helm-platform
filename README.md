# AKS + Helm Platform — PulseHealth Systems

> **Project 4 · Magela84 Cloud Engineering Portfolio**
> Industry: Healthcare Technology

## Client Overview

PulseHealth Systems runs four patient-facing applications on individual
VMs, where scaling and patching are painful and inconsistent. This
engagement consolidates them onto a single **managed Azure Kubernetes
Service (AKS)** platform with tenant isolation, standardized Helm
releases, and automatic scaling for peak clinic hours.

Everything here is **Infrastructure as Code — zero manual portal clicks.**

## Problem Statement

Four apps on four hand-patched VMs meant inconsistent updates, no tenant
isolation, ad-hoc releases, and painful scaling when clinics get busy.
PulseHealth needed one managed platform with namespace-level isolation, a
repeatable packaging/release method, and autoscaling — with no passwords
stored anywhere.

## What I Built

- **AKS cluster** via Terraform — separate **system** and **user** node pools
- **One reusable Helm chart**, deployed 4× via per-app values files
- **NGINX Ingress + TLS** (cert-manager / Let's Encrypt), one shared LB
- **Namespace-scoped RBAC** + ServiceAccount per team + default-deny NetworkPolicy
- **ACR via Managed Identity** (`AcrPull`) — passwordless image pulls
- **HorizontalPodAutoscaler** per workload + cluster autoscaler on the user pool
- **GitHub Actions** pipeline (OIDC, no stored secret) for Helm deploys

## Architecture

```
                    Internet (HTTPS)
                          │
              ┌───────────▼────────────┐
              │  NGINX Ingress + TLS   │  (cert-manager / Let's Encrypt)
              └───────────┬────────────┘
        ┌─────────────┬───┴────────┬──────────────┐
        ▼             ▼            ▼              ▼
 ┌────────────┐┌────────────┐┌────────────┐┌────────────┐
 │patient-    ││appointments││ telehealth ││  billing   │   namespaces
 │  portal    ││            ││            ││            │   (tenants)
 │ Deploy+HPA ││ Deploy+HPA ││ Deploy+HPA ││ Deploy+HPA │
 │ SA + RBAC  ││ SA + RBAC  ││ SA + RBAC  ││ SA + RBAC  │
 └────────────┘└────────────┘└────────────┘└────────────┘
        │  user node pool (autoscaling 1→4 nodes)         │
        └──────────────────────┬──────────────────────────┘
                               │ pulls images (Managed Identity, AcrPull)
                        ┌──────▼──────┐
                        │     ACR     │  acrpulsehealthdev001
                        └─────────────┘
   System node pool: cluster add-ons only (tainted)
   Observability: OMS agent → Log Analytics (Container Insights)
```

Full design in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md); step-by-step
commands in [`docs/RUNBOOK.md`](docs/RUNBOOK.md).

## Resources Deployed

| Resource | Type | Purpose |
|---|---|---|
| `rg-pulsehealth-dev` | Resource Group | Container for all resources |
| `aks-pulsehealth-dev` | AKS Cluster | Managed Kubernetes (system + user pools) |
| `acrpulsehealthdev001` | Azure Container Registry | Private image store for 4 apps |
| kubelet Managed Identity | Managed Identity (`AcrPull`) | Passwordless image pulls |
| `law-pulsehealth-dev` | Log Analytics Workspace | Container Insights logs + metrics |
| 4 namespaces + RBAC | Namespace / Role / RoleBinding / SA | Tenant isolation per app |
| 4 Helm releases | Deployment / Service / Ingress / HPA | One per patient app |

## Technologies Used

Terraform · AKS · Helm · kubectl · Azure Container Registry · Managed
Identity · NGINX Ingress · cert-manager · GitHub Actions (OIDC) · Docker

## How to Deploy

```bash
git clone https://github.com/Magela84/pulsehealth-aks-helm-platform.git
cd pulsehealth-aks-helm-platform

# 1. Fill in your IDs (file is git-ignored)
cp terraform/terraform.tfvars.example terraform/terraform.tfvars
#    edit subscription_id + tenant_id

# 2. Provision the platform
./scripts/01-deploy-infra.sh
./scripts/02-connect-cluster.sh
./scripts/03-install-ingress.sh

# 3. Deploy all four apps (pass your ACR login server)
./scripts/04-deploy-apps.sh acrpulsehealthdev001.azurecr.io
```

> **Cost note (dev sizing):** AKS control plane **Free** tier + 1 system
> node + 1–4 user nodes (`Standard_D2s_v3`) + Basic ACR. Run
> `az aks stop` between demos to minimise spend.

## Engagement Outcome

Four VM-hosted apps consolidated onto one AKS platform: each isolated in
its own namespace with scoped RBAC and default-deny networking, packaged
by a single reusable Helm chart, autoscaling at both pod and node level
for clinic peaks, and released through an OIDC-authenticated pipeline with
**no secrets stored anywhere**. Terraform passes `validate`; the platform
is reproducible from code.

## Author

**Magela84** — Cloud Engineer
[github.com/Magela84](https://github.com/Magela84)
