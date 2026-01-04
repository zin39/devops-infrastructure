# Production DevOps Infrastructure

A complete self-hosted DevOps platform running in production, featuring Kubernetes, CI/CD pipelines, GitOps, and comprehensive monitoring.

## Architecture

INTERNET
|
v
[Cloudflare DNS] --> [AWS EC2 Gateway: HAProxy + Headscale]
|
(Headscale Mesh VPN)
|
+-----------------+-----------------+
| | |
v v v
[k3s-master] [k3s-worker-01] [k3s-worker-02] - Prometheus - GitLab - Jenkins - Grafana - PostgreSQL - Harbor - Loki - Redis - Gitaly - Vault - ArgoCD
| | |
+-----------------+-----------------+
|
[Ceph Storage Cluster]
3 nodes / ~230GB

## Components

### CI/CD Pipeline

| Stage    | Tool             | Purpose                                        |
| -------- | ---------------- | ---------------------------------------------- |
| Source   | GitLab CE        | Git repository hosting                         |
| Build    | Jenkins + Kaniko | Rootless container builds                      |
| Registry | Harbor           | Image storage + vulnerability scanning (Trivy) |
| Deploy   | ArgoCD           | GitOps-based continuous deployment             |

**Flow:** `Git Push → GitLab → Jenkins → Kaniko Build → Harbor → ArgoCD → K3s`

### Kubernetes (K3s)

- **Cluster:** 1 master + 2 workers
- **Workloads:** 12+ production services
- **Ingress:** Nginx Ingress Controller
- **Certificates:** cert-manager with Let's Encrypt
- **Load Balancer:** MetalLB

### GitOps (ArgoCD)

- Automated sync every 3 minutes
- Self-healing enabled
- Drift detection
- Git as single source of truth

### Monitoring Stack

| Tool         | Purpose                                |
| ------------ | -------------------------------------- |
| Prometheus   | Metrics collection (21 scrape targets) |
| Grafana      | Visualization and dashboards           |
| Loki         | Log aggregation                        |
| Alertmanager | Alert routing and notifications        |

### Secrets Management (HashiCorp Vault)

- Kubernetes authentication
- Agent Injector for automatic secret injection
- Path-based access policies
- Centralized secret storage

### Storage (Ceph)

- 3-node distributed cluster
- RBD block storage via CSI
- ~230GB total allocation
- Automatic PVC provisioning

### Networking

- **External:** HAProxy for L7 routing
- **Internal:** Headscale mesh VPN (Tailscale-compatible)
- **Security:** Network Policies for namespace isolation

## Key Design Decisions

| Decision                         | Reasoning                                              |
| -------------------------------- | ------------------------------------------------------ |
| K3s over K8s                     | Lightweight, suitable for homelab scale                |
| Kaniko over Docker-in-Docker     | Security: no privileged containers needed              |
| ArgoCD over kubectl in CI        | Pull-based GitOps, better security, drift detection    |
| Vault over K8s Secrets           | Centralized management, audit logging, dynamic secrets |
| Ceph over local storage          | Data survives node failures, shared across cluster     |
| Headscale over exposing services | Zero-trust networking, private mesh                    |

## Services Hosted

| Service | URL                   | Purpose               |
| ------- | --------------------- | --------------------- |
| GitLab  | git.lumetium.com      | Source control        |
| Jenkins | ci.lumetium.com       | CI/CD engine          |
| Harbor  | registry.lumetium.com | Container registry    |
| ArgoCD  | argocd.lumetium.com   | GitOps deployments    |
| Grafana | grafana.lumetium.com  | Monitoring dashboards |
| Vault   | vault.lumetium.com    | Secrets management    |

## Tech Stack

`Kubernetes` `K3s` `Docker` `Jenkins` `ArgoCD` `Prometheus` `Grafana` `Loki` `HashiCorp Vault` `GitLab` `Harbor` `Ceph` `HAProxy` `Headscale` `AWS EC2` `Terraform` `Linux`

## Backups

- **Tool:** Velero
- **Schedule:** Daily
- **Retention:** 30 days
- **Storage:** Ceph RGW (S3-compatible)

## What I Learned

1. **GitOps simplifies operations** - Rollbacks are just git reverts
2. **Observability is essential** - Can't fix what you can't see
3. **Security requires layers** - Vault, NetworkPolicies, rootless builds
4. **Distributed storage is complex** - But necessary for HA

## Future Improvements

- [ ] Terraform for AWS resources
- [ ] Multi-environment GitOps (dev/staging/prod)
- [ ] Service mesh (Linkerd)
- [ ] Automated integration testing in pipeline

---

_This infrastructure is actively running and maintained._
