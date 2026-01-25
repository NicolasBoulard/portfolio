---
title: "Homelab: Building a Resilient Cloud-Native Ecosystem"
description: "How I manage a high-availability Homelab using K3s, ArgoCD, and Kubernetes Operators like CNPG and Longhorn."
date: "2025-12-10"
author: "Nicolas Boulard"
keywords: "homelab, kubernetes, argocd, cnpg, longhorn, keycloak, cloud-native, self-hosted"
---

Many engineers see a Homelab as a playground. For someone focused on reliability and modern engineering, it's more than that: it's a production-grade testing ground for **resilience, automation, and security**.

My Homelab isn't just a collection of Docker containers; it's a fully-fledged **Kubernetes ecosystem** managed via GitOps. Here is how I’ve structured my infrastructure to mirror professional cloud environments.

## 🏗 The Architecture: A Unified GitOps Approach

My repository structure follows a strictly decoupled pattern. Every application has its own dedicated directory, separating Helm `values.yaml` from custom Kubernetes `manifests/`. This allows for granular control and clean automation via Renovate.


### Core Components of the Stack:
* **Orchestration:** K3s managed by **ArgoCD** (App-of-Apps pattern).
* **Identity & Secrets:** **Keycloak** for OIDC, integrated with **1Password** and **External Secrets Operator**.
* **Storage:** **Longhorn** for distributed block storage and **CSI Driver NFS** for media.
* **Database:** **CloudNative-PG (CNPG)** for automated PostgreSQL management.

---

## 💾 Mastering Data: Longhorn & CNPG

One of the biggest challenges in a Homelab is **stateful data**. I rely on Kubernetes Operators to handle the heavy lifting:

1. **CloudNative-PG (CNPG):** Instead of running a single Postgres pod, I use the CNPG Operator. It handles backups, streaming replication, and failover automatically for apps like **Immich**, **Nextcloud**, and **Home Assistant**.
2. **Longhorn:** To ensure my data survives a node failure, Longhorn provides replicated, highly available block storage across my cluster nodes.

---

## 🔐 Security & Identity First

Security shouldn't be an afterthought. My stack is protected by an "Identity-Aware" layer:
- **Traefik** acts as the Ingress Controller, using custom Middlewares for headers and security.
- **Keycloak** + **traefik-oidc-auth plugin** : Most of my internal tools (Prometheus, Grafana, Longhorn) are behind OIDC authentication.
- **External Secrets:** I never store plain-text secrets in Git. I use the **1Password** backend to inject secrets into Kubernetes at runtime.


## 🚀 Conclusion

A Homelab is the ultimate proof of a **engineering mindset**. It’s where "Innovation-first" meets "Reliability-first." By using GitOps and Cloud-Native best practices, I’ve turned a simple server into a resilient, automated platform that is constantly evolving.

