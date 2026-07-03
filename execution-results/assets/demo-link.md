# Live Deployment & Demo

This repository (`Nscrape_Series`) contains the **tools documentation** — what each tool does, how to set it up, and how to use it: Docker images, usage docs, and Compose files for running LetsNScrape, ytNscrape, and SnapNScrape locally.

To see the **deployment of these tools** in a real production environment — running on a self-managed Kubernetes cluster on AWS, with Ansible automation, Helm packaging, and Prometheus/Grafana monitoring — see:

**[NSeries Platform Engineering →](https://github.com/pratham1kruk/NSeries-Platform-Engineering)**

That repository covers:
- AWS EC2 + kubeadm cluster setup (no EKS/GKE)
- Ansible-driven node provisioning
- Helm-packaged deployments of LetsNScrape and SnapNScrape
- Docker Compose deployment of ytNscrape's 9-service pipeline
- Prometheus + Grafana monitoring and alerting
- Full demo videos and deployment screenshots (hosted externally on Drive, linked from that repo)

**In short:**
| This repo (Nscrape_Series) | Platform Engineering repo (referred) |
|---|---|
| Tools documentation — what each tool does, how to set it up, and how to use it | Deployment of these tools — how they're run in production |
| Docker images, usage docs, Compose setup | AWS, Kubernetes, Ansible, Helm, monitoring |
| "What these tools do and how to run them" | "How they're deployed and operated in production" |

Screenshots and video walkthroughs of the tools actually running — including cluster state, Grafana dashboards, and live UI captures — live in that repo's `assets/` folder and linked Drive demo videos, not here.
