# � Deployment of Microservices Application using Ingress Controller

### by Kastro Kiran V

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Jenkins](https://img.shields.io/badge/jenkins-%232C5263.svg?style=for-the-badge&logo=jenkins&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

This project contains a multi-page web application deployed on **AWS EKS** with **Ingress Controller** using **Jenkins CI/CD** pipeline.

## 🛠️ Tools Used

| Category        | Tools                                                                                      |
|-----------------|-------------------------------------------------------------------------------------------|
| Version Control | ![Git](https://img.shields.io/badge/git-%23F05033.svg?style=flat&logo=git&logoColor=white) ![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=flat&logo=github&logoColor=white) |
| CI/CD           | ![Jenkins](https://img.shields.io/badge/jenkins-%232C5263.svg?style=flat&logo=jenkins&logoColor=white) |
| Containers      | ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white) |
| Orchestration   | ![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=flat&logo=kubernetes&logoColor=white) ![AWS EKS](https://img.shields.io/badge/AWS_EKS-%23FF9900.svg?style=flat&logo=amazon-aws&logoColor=white) |
| Monitoring      | ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=Prometheus&logoColor=white) ![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=flat&logo=grafana&logoColor=white) |
| GitOps          | ![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat&logo=argo&logoColor=white) |
| Package Manager | ![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat&logo=helm&logoColor=white) |

## 🌐 Application Paths

The application provides the following endpoints:

| Path          | Description                          |
|---------------|--------------------------------------|
| `/`           | 🏠 Landing page with overview        |
| `/about`      | ℹ️ Company information and team      |
| `/services`   | 🛠️ Detailed service offerings        |
| `/contact`    | 📧 Contact information and form      |

## 🔄 Pipeline Stages

```mermaid
graph LR
    A[✅ Checkout Code] --> B[✅ Build Docker Image]
    B --> C[✅ Push to DockerHub]
    C --> D[✅ Configure AWS/kubectl]
    D --> E[✅ Deploy to Kubernetes]
    E --> F[✅ Deploy Ingress]
    F --> G[✅ Get Ingress URL]

## ⚖️ Scaling the Application

```mermaid
# Scale up replicas
kubectl scale deployment techsolutions-deployment --replicas=5

# Scale down replicas
kubectl scale deployment techsolutions-deployment --replicas=2
