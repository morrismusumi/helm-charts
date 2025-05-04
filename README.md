# Helm Charts

Welcome to the **Helm Charts** repository!  
This repository contains a collection of Helm charts for commonly used developer tools and infrastructure components, helping teams deploy services efficiently on Kubernetes.

## 📦 Charts Included

| Chart Name       | Description                             |
|------------------|-----------------------------------------|
| `pi-hole`        | A Pi-Hole Helm chart                    |
| `...`            | More charts coming soon!                |

> Each chart is self-contained in its own directory under `/charts`.

---

## 🚀 Getting Started

To use a chart from this repository:

1. Add the repo to Helm:
   ```bash
   helm repo add morrismusumi-helm-charts https://morrismusumi.github.io/helm-charts
   helm repo update

   helm search repo morrismusumi-helm-charts
   NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
   morrismusumi-helm-charts/pi-hole        0.1.2           2025.04.0       A Pi-Hole Helm chart