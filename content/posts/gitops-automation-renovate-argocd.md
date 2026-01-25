---
title: "Automated Dependency Updates: Mastering Renovate for ArgoCD & Helm"
description: "How to configure Renovate to handle complex ArgoCD architectures, including Helm values and custom Kubernetes manifests."
date: "2026-01-25"
author: "Nicolas Boulard"
keywords: "renovate, argocd, kubernetes, helm, gitops, automation, devops"
---

In a GitOps world, keeping your dependencies up to date is a marathon, not a sprint. When managing multiple applications via **ArgoCD**, the challenge isn't just updating the Helm chart version, but also the container images hidden inside your `values.yaml` and the static manifests stored alongside them.

As a **Cloud Infrastructure Specialist**, I aim for "zero-touch" operations. Here is how I configure **Renovate** to track everything in a specific directory-per-app architecture.

## The Architecture Challenge

My repository follows a clean, decoupled structure for each application:

```text
my-repo/
├── 1password/
│   └── values.yaml
├── argocd/
│   ├── manifests/
│   │   ├── app-cert-manager.yaml  # Renovate's argocd manager updates the targetRevision here
│   │   ├── app-immich.yaml
│   │   └── ... (many application files)
│   └── root-app.yaml
├── cert-manager/
│   ├── values.yaml                # Helm overrides (including image tags)
│   └── manifests/
│       └── configmap.yaml         # Extra K8s resources (Sidecars, ConfigMaps, etc.)
├── immich/
│   └── values.yaml                # Helm overrides (including image tags)
└── ... (other application folders)
```

Standard Renovate configurations often miss images defined in custom paths or specific Helm values. We need to tell Renovate exactly where to look.

## 🛠 The Configuration: Precision & Autopilot

My setup relies on three main pillars: precise file matching, custom regex managers for edge cases, and aggressive **automerging** for non-breaking updates.

Your architecture uses a dedicated directory for each application, simplifying the tracking of configurations.

{{<mermaid>}}
graph TD
    subgraph Git Repository Source of Truth
        A[IaC Configs: Helm Charts, Argo CD Apps]
    end

    subgraph Automation
        B[Renovate Bot: Creates PRs]
    end
    
    subgraph Deployment
        D[Argo CD: Monitors Git]
        E[Kubernetes Cluster]
    end

    B --> A;
    A -- Change Detected --> D;
    D --> E;

    style A fill:#DCEFFB,stroke:#006BB9
    style D fill:#FAF2DF,stroke:#E9A300
{{</mermaid>}}


```json5
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
      "config:recommended",
      ":preserveSemverRanges",
      ":disableDependencyDashboard",
      ":prImmediately"
    ],
  "kubernetes": {
    "fileMatch": ["manifests/.*\\.ya?ml$"]
  },
  "helm-values": {
    "fileMatch": [".+values.*\\.ya?ml$"]
  },
  "argocd": {
    "fileMatch": ["argocd/manifests/.+\\.ya?ml$"]
  },
  "branchConcurrentLimit": 0,
  "prConcurrentLimit": 0,
  "packageRules": [
    {
      "matchManagers": [
          "argocd",
          "regex"
      ],
      "matchUpdateTypes": [
          "minor",
          "patch"
      ],
      "groupName": null,
      "enabled": true,
      "automerge": true,
      "automergeType": "branch",
      "rebaseWhen": "behind-base-branch",
      "ignoreTests": true
    }
  ],
  "customManagers": [
    {
      "customType": "regex",
      "description": "Update Docker image tags in Helm values files",
      "managerFilePatterns": [
        "/(^|/)values.*\\.ya?ml$/"
      ],
      "matchStrings": [
        "image:\\s*\\n\\s*tag:\\s*[\"']?(?<currentValue>v?\\d+\\.\\d+\\.\\d+)[\"']?"
      ],
      "depNameTemplate": "ghcr.io/immich-app/immich-server",
      "datasourceTemplate": "docker",
      "versioningTemplate": "docker"
    },
  ]
}

```

## 🔍 Key Implementation Insights

### 1. Multi-Layered Detection
Standard Renovate settings can be too broad or too narrow. My configuration targets specific patterns to align with a structured GitOps repository:

* **`kubernetes`**: Specifically scans the `manifests/` directory for standard YAML resources (Deployments, StatefulSets, etc.).
* **`helm-values`**: Dynamically identifies any file with `values` in the name, supporting multi-environment setups (e.g., `values-prod.yaml`).
* **`argocd`**: Tracks ArgoCD Application manifests to ensure the `targetRevision` of Helm sources is always current.

### 2. Custom Regex Managers (The "Immich" Case)
Sometimes, YAML parsers fail to catch image tags when they are structured uniquely or nested deeply. I use a **Regex Manager** to handle specific cases where the `tag` is on a new line following the `image:` key. 

In this example, it ensures that my **Immich** stack stays up-to-date by explicitly mapping the regex match to its Docker registry source, bypassing the limitations of standard auto-detection.

### 3. Automerge: Reducing Decision Fatigue
To keep the maintenance overhead low and focus on high-value tasks, I've configured **Automerge** for:

* **Minor and Patch updates**: Applied automatically for both ArgoCD and Regex-managed dependencies.
* **Branch Automerge**: By using `automergeType: "branch"`, Renovate pushes stable updates directly to the repository. This prevents the UI from being flooded with Pull Requests for routine maintenance, provided the criteria are met.

---

## 🚀 Conclusion

This setup transforms infrastructure maintenance from a manual chore into a passive, secure background process. While **Renovate** monitors and pushes stable versions, **ArgoCD** ensures the state is instantly reflected in the Kubernetes cluster.


In my experience, this automation provides significant **added value, particularly on development clusters**. It allows teams to:
* **Anticipate breaking changes:** Catching version issues early in a non-production environment.
* **Continuous Integration of Apps:** Seamlessly integrating new application versions as soon as they are released.
* **Focus on Code:** Reducing the cognitive load of manual dependency tracking.
