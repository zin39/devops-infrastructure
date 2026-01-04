# Production DevOps Infrastructure

> Self-hosted DevOps platform running 12+ production services with GitOps, automated CI/CD, and full observability.

A complete infrastructure stack built for reliability: Kubernetes orchestration, automated pipelines, centralized secrets, and comprehensive monitoring.

## Highlights

| Metric | Value |
|--------|-------|
| Uptime | 99%+ over 6 months |
| Deployment Time | ~5 minutes (commit to production) |
| Monitoring Coverage | 21 Prometheus scrape targets |
| Storage | ~230GB distributed Ceph cluster |
| Services | 12+ production workloads |

## Architecture

```mermaid
flowchart TB
    subgraph Internet
        USER[Users/Developers]
        CF[Cloudflare DNS]
    end

    subgraph AWS["AWS EC2 Gateway"]
        HAP[HAProxy<br/>Load Balancer]
        HS[Headscale<br/>Mesh VPN]
    end

    subgraph K3S["Kubernetes Cluster (K3s)"]
        subgraph Master["k3s-master"]
            PROM[Prometheus]
            GRAF[Grafana]
            LOKI[Loki]
            VAULT[Vault]
            ARGO[ArgoCD]
        end

        subgraph Worker1["k3s-worker-01"]
            GL[GitLab]
            PG[PostgreSQL]
            RD[Redis]
        end

        subgraph Worker2["k3s-worker-02"]
            JK[Jenkins]
            HB[Harbor]
            GT[Gitaly]
        end
    end

    subgraph Storage["Ceph Storage Cluster"]
        CEPH[(3 nodes<br/>~230GB)]
    end

    USER --> CF
    CF --> HAP
    HAP --> HS
    HS --> Master
    HS --> Worker1
    HS --> Worker2
    Master --> CEPH
    Worker1 --> CEPH
    Worker2 --> CEPH
```

## CI/CD Pipeline

```mermaid
flowchart LR
    A[Git Push] --> B[GitLab]
    B --> C[Jenkins]
    C --> D[Kaniko Build]
    D --> E[Harbor Registry]
    E --> F[ArgoCD]
    F --> G[K3s Cluster]
```

| Stage | Tool | Purpose |
|-------|------|---------|
| Source | GitLab CE | Git repository hosting |
| Build | Jenkins + Kaniko | Rootless container builds |
| Registry | Harbor | Image storage + vulnerability scanning (Trivy) |
| Deploy | ArgoCD | GitOps-based continuous deployment |

## How Deployments Work

1. **Developer pushes code** to GitLab repository
2. **Webhook triggers Jenkins** pipeline automatically
3. **Kaniko builds container** image (rootless, no privileged containers)
4. **Image pushed to Harbor** with Trivy vulnerability scan
5. **Jenkins updates image tag** in GitOps repository
6. **ArgoCD detects change** and syncs to cluster (within 3 minutes)
7. **Kubernetes performs rolling update** with zero downtime

## Components

### Kubernetes (K3s)

| Aspect | Details |
|--------|---------|
| Cluster | 1 master + 2 workers |
| Workloads | 12+ production services |
| Ingress | Nginx Ingress Controller |
| Certificates | cert-manager with Let's Encrypt |
| Load Balancer | MetalLB |

### GitOps (ArgoCD)

- Automated sync every 3 minutes
- Self-healing enabled (auto-corrects drift)
- Drift detection and alerts
- Git as single source of truth

### Monitoring Stack

| Tool | Purpose |
|------|---------|
| Prometheus | Metrics collection (21 scrape targets) |
| Grafana | Visualization and dashboards |
| Loki | Log aggregation |
| Alertmanager | Alert routing and notifications |

### Secrets Management (HashiCorp Vault)

- Kubernetes authentication
- Agent Injector for automatic secret injection
- Path-based access policies per service
- Centralized secret storage with audit logging

### Storage (Ceph)

- 3-node distributed cluster
- RBD block storage via CSI driver
- ~230GB total allocation
- Automatic PVC provisioning

### Networking

| Layer | Tool | Purpose |
|-------|------|---------|
| External | HAProxy | L7 load balancing, TLS termination |
| Internal | Headscale | WireGuard-based mesh VPN |
| Security | Network Policies | Namespace isolation |

## Key Design Decisions

| Decision | Reasoning |
|----------|-----------|
| K3s over K8s | Lightweight, suitable for homelab scale |
| Kaniko over Docker-in-Docker | Security: no privileged containers needed |
| ArgoCD over kubectl in CI | Pull-based GitOps, better security, drift detection |
| Vault over K8s Secrets | Centralized management, audit logging, dynamic secrets |
| Ceph over local storage | Data survives node failures, shared across cluster |
| Headscale over public exposure | Zero-trust networking, private mesh |

## Services

| Service | Purpose |
|---------|---------|
| GitLab | Source control & repository hosting |
| Jenkins | CI/CD automation engine |
| Harbor | Container registry with Trivy scanning |
| ArgoCD | GitOps continuous deployment |
| Grafana | Monitoring dashboards & alerting |
| Vault | Secrets management |

## Tech Stack

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat&logo=jenkins&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat&logo=argo&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![Vault](https://img.shields.io/badge/Vault-000000?style=flat&logo=vault&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)

## Backup Strategy

| Aspect | Details |
|--------|---------|
| Tool | Velero |
| Schedule | Daily automated backups |
| Retention | 30 days |
| Storage | Ceph RGW (S3-compatible) |

## Lessons Learned

1. **GitOps simplifies operations** - Rollbacks are just git reverts
2. **Observability is essential** - Can't fix what you can't see
3. **Security requires layers** - Vault, NetworkPolicies, rootless builds
4. **Distributed storage is complex** - But necessary for high availability

## Roadmap

- [ ] Terraform for AWS infrastructure
- [ ] Multi-environment GitOps (dev/staging/prod)
- [ ] Service mesh implementation (Linkerd)
- [ ] Automated integration testing in pipeline

---

*This infrastructure is actively running and maintained.*
