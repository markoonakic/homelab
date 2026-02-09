# Homelab Infrastructure

**Enterprise-grade Kubernetes homelab deployed via GitOps on Talos Linux.**

## Executive Summary

This repository contains my complete infrastructure-as-code setup for a production-grade Kubernetes homelab. I am running Talos Linux and managing everything through GitOps with FluxCD. The project shows how to apply real cloud-native patterns at home, including multi-tier deployment automation, encrypted secrets, distributed storage, and full observability.

The entire cluster follows GitOps principles. All infrastructure and application state lives in YAML, and FluxCD keeps everything in sync automatically. When I push changes to GitHub, Flux reconciles the cluster within minutes. This means safe, trackable infrastructure changes with built-in drift detection and rollback capability.

I chose Talos Linux for its immutable, minimal design. The setup runs distributed block storage with replication (Longhorn), handles TLS certificates automatically through DNS-01 challenges (cert-manager), load-balances ingress with MetalLB and Traefik, and monitors everything with kube-prometheus-stack. All secrets are encrypted at rest using SOPS and age.

## System Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'background': '#ebdbb2', 'lineColor': '#665c54', 'edgeLabelBackground': '#ebdbb2', 'primaryTextColor': '#282828'}}}%%

graph TD
    %% ── External Services ──
    subgraph external["External Services"]
        direction LR
        github["GitHub<br/>Source of Truth"]
        letsencrypt["Let's Encrypt<br/>TLS Certificates"]
        headscale["Headscale VPN<br/>Remote Access"]
    end

    %% ── GitOps Pipeline ──
    subgraph gitops["GitOps Pipeline"]
        direction LR
        flux["FluxCD<br/>Continuous Reconciliation"]
        sops["SOPS + age<br/>Secret Encryption"]
        renovate["Renovate<br/>Dependency Automation"]
    end

    %% ── Networking ──
    subgraph net["Networking"]
        direction LR
        certmanager["cert-manager<br/>DNS-01 Wildcards"]
        traefik["Traefik<br/>Ingress Controller"]
        metallb["MetalLB<br/>Load Balancer"]
    end

    %% ── Applications ──
    subgraph apps["Applications"]
        direction LR
        vaultwarden["Vaultwarden"]
        karakeep["Karakeep"]
        vikunja["Vikunja"]
        mealie["Mealie"]
        excalidraw["Excalidraw"]
        glance["Glance"]
    end

    %% ── Monitoring ──
    subgraph mon["Monitoring & Observability"]
        direction LR
        prometheus["Prometheus"]
        grafana["Grafana"]
        alertmanager["Alertmanager"]
    end

    %% ── Storage ──
    subgraph stor["Distributed Storage"]
        longhorn["Longhorn<br/>Replicated Block Storage"]
    end

    %% ── Foundation ──
    subgraph infra["Talos Linux Cluster"]
        direction LR
        node1["Control Plane + Worker<br/>4 CPU · 8GB RAM"]
        node2["Control Plane<br/>2 CPU · 8GB RAM"]
        node3["Worker<br/>2 CPU · 4GB RAM"]
    end

    %% ── GitOps Flow ──
    renovate -->|PRs| github
    github -->|git sync| flux
    flux -->|decrypt| sops

    %% ── Flux Deploys ──
    flux -.->|deploy| net
    flux -.->|deploy| apps
    flux -.->|deploy| mon
    flux -.->|deploy| stor

    %% ── TLS Flow ──
    letsencrypt -->|certs| certmanager
    certmanager -->|TLS| traefik

    %% ── Traffic Flow ──
    headscale -->|VPN tunnel| traefik
    metallb --> traefik
    traefik --> apps
    traefik --> mon

    %% ── Monitoring ──
    prometheus --- grafana
    prometheus --- alertmanager

    %% ── Persistence ──
    apps --> longhorn
    mon --> longhorn

    %% ── Node Styles (Gruvbox Palette) ──
    classDef ext fill:#458588,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef git fill:#98971a,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef network fill:#d65d0e,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef monitor fill:#b16286,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef app fill:#d79921,stroke:#282828,stroke-width:2px,color:#282828
    classDef store fill:#689d6a,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef foundation fill:#cc241d,stroke:#282828,stroke-width:2px,color:#ebdbb2

    class github,letsencrypt,headscale ext
    class flux,sops,renovate git
    class certmanager,traefik,metallb network
    class prometheus,grafana,alertmanager monitor
    class vaultwarden,karakeep,vikunja,mealie,excalidraw,glance app
    class longhorn store
    class node1,node2,node3 foundation

    %% ── Subgraph Styles ──
    style external fill:#ebdbb2,stroke:#458588,stroke-width:2px,color:#282828
    style gitops fill:#ebdbb2,stroke:#98971a,stroke-width:2px,color:#282828
    style net fill:#ebdbb2,stroke:#d65d0e,stroke-width:2px,color:#282828
    style apps fill:#ebdbb2,stroke:#d79921,stroke-width:2px,color:#282828
    style mon fill:#ebdbb2,stroke:#b16286,stroke-width:2px,color:#282828
    style stor fill:#ebdbb2,stroke:#689d6a,stroke-width:2px,color:#282828
    style infra fill:#ebdbb2,stroke:#cc241d,stroke-width:2px,color:#282828
```

## Writing

I write about my homelab architecture and decisions on my blog. These posts provide deeper context on the design choices and implementation details.

- [Installing Talos Linux](https://markonakic.xyz/posts/talos-install/) - Complete walkthrough of setting up Talos Linux from scratch
- [Remote Access Architecture of My Kubernetes Homelab](https://markonakic.xyz/posts/remote-access/) - How I use Headscale for secure remote access to self-hosted services
