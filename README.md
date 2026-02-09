# Homelab Infrastructure

**Enterprise-grade Kubernetes homelab deployed via GitOps on Talos Linux.**

## Executive Summary

This repository contains my complete infrastructure-as-code setup for a production-grade Kubernetes homelab. I am running Talos Linux and managing everything through GitOps with FluxCD. The project shows how to apply real cloud-native patterns at home, including multi-tier deployment automation, encrypted secrets, distributed storage, and full observability.

The entire cluster follows GitOps principles. All infrastructure and application state lives in YAML, and FluxCD keeps everything in sync automatically. When I push changes to GitHub, Flux reconciles the cluster within minutes. This means safe, trackable infrastructure changes with built-in drift detection and rollback capability.

I chose Talos Linux for its immutable, minimal design. The setup runs distributed block storage with replication (Longhorn), handles TLS certificates automatically through DNS-01 challenges (cert-manager), load-balances ingress with MetalLB and Traefik, and monitors everything with kube-prometheus-stack. All secrets are encrypted at rest using SOPS and age.

## System Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#665c54', 'edgeLabelBackground': '#ebdbb2'}}}%%

graph TD
    %% ── External Sources ──
    renovate["Renovate"]:::gitops -->|"PRs"| github["GitHub"]:::external
    letsencrypt["Let's Encrypt"]:::external
    headscale["Headscale VPN"]:::external

    %% ── Git Sync ──
    github -->|"git sync"| flux

    %% ── Cluster ──
    subgraph cluster["Talos Linux Cluster · 3 Nodes · 8 CPU · 20GB RAM"]
        flux["FluxCD"]:::gitops -->|"decrypt"| sops["SOPS + age"]:::gitops
        certmanager["cert-manager"]:::network
        metallb["MetalLB"]:::network
        traefik["Traefik"]:::network
        apps["Applications\nVaultwarden · Karakeep · Vikunja\nMealie · Excalidraw · Glance"]:::application
        monitoring["Monitoring\nPrometheus · Grafana · Alertmanager"]:::monitor
        longhorn["Longhorn\nReplicated Storage"]:::store
    end

    %% ── Flux Reconciliation ──
    flux -.-> certmanager
    flux -.-> apps
    flux -.-> longhorn

    %% ── TLS Flow ──
    letsencrypt -->|"certs"| certmanager
    certmanager -->|"TLS"| traefik

    %% ── Traffic Flow ──
    headscale -->|"VPN"| traefik
    metallb --> traefik
    traefik --> apps
    traefik --> monitoring

    %% ── Persistence ──
    apps --> longhorn
    monitoring --> longhorn

    %% ── Gruvbox Palette ──
    classDef external fill:#458588,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef gitops fill:#98971a,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef network fill:#d65d0e,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef monitor fill:#b16286,stroke:#282828,stroke-width:2px,color:#ebdbb2
    classDef application fill:#d79921,stroke:#282828,stroke-width:2px,color:#282828
    classDef store fill:#689d6a,stroke:#282828,stroke-width:2px,color:#ebdbb2

    style cluster fill:#fbf1c7,stroke:#928374,stroke-width:2px,color:#282828
```

## Writing

I write about my homelab architecture and decisions on my blog. These posts provide deeper context on the design choices and implementation details.

- [Installing Talos Linux](https://markonakic.xyz/posts/talos-install/) - Complete walkthrough of setting up Talos Linux from scratch
- [Remote Access Architecture of My Kubernetes Homelab](https://markonakic.xyz/posts/remote-access/) - How I use Headscale for secure remote access to self-hosted services
