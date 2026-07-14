# AKS Platform Migration — Healthcare ISV Engagement

## Engagement Overview
A healthcare ISV running four patient-facing applications
on individual Virtual Machines needed to consolidate onto
a managed Kubernetes platform with tenant isolation and
standardized deployment processes.

## Problem Statement
Scaling was painful requiring manual VM resizing during
peak clinic hours. Patching was inconsistent across four
separate VMs. No standardized deployment process existed
across the four application teams.

## What I Built
- AKS cluster with dedicated system and user node pools
- Separate system pool for cluster internals only
- User pool with autoscaling from 1 to 4 nodes
- One reusable Helm chart for all four patient applications
- Per-application values files for customization
- Namespace-level RBAC with ServiceAccount per team
- Default-deny NetworkPolicy between namespaces
- Horizontal Pod Autoscaler per application
- Azure Container Registry with Managed Identity pull access
- GitHub Actions pipeline with OIDC authentication

## Architecture
Four patient applications each isolated in own namespace:
- patient-portal  (HPA: 3 to 12 pods)
- appointments    (HPA: 2 to 8 pods)
- telehealth      (HPA: 2 to 10 pods)
- billing         (HPA: 1 to 4 pods)

## Technologies Used
Kubernetes, AKS, Helm, Terraform, Azure Container Registry,
Managed Identities, GitHub Actions, RBAC, NetworkPolicy,
Log Analytics, Python, Flask

## Subscription Note
AKS cluster provisioning is restricted on free Azure
subscriptions due to VM quota limits. Terraform code is
production-ready and validated. All other resources
deployed successfully including ACR and Log Analytics.

## Engagement Outcome
Delivered a standardized Kubernetes platform with tenant
isolation automatic scaling and a consistent Helm-based
deployment process across all four patient applications.

## Author
Cloud Engineer - Magela84
