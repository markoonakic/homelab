# Homelab Infrastructure

**Enterprise-grade Kubernetes homelab deployed via GitOps on Talos Linux.**

## Executive Summary

This repository contains my complete infrastructure-as-code setup for a production-grade Kubernetes homelab. I am running Talos Linux and managing everything through GitOps with FluxCD. The project shows how to apply real cloud-native patterns at home, including multi-tier deployment automation, encrypted secrets, distributed storage, and full observability.

The entire cluster follows GitOps principles. All infrastructure and application state lives in YAML, and FluxCD keeps everything in sync automatically. When I push changes to GitHub, Flux reconciles the cluster within minutes. This means safe, trackable infrastructure changes with built-in drift detection and rollback capability.

I chose Talos Linux for its immutable, minimal design. The setup runs distributed block storage with replication (Longhorn), handles TLS certificates automatically through DNS-01 challenges (cert-manager), load-balances ingress with MetalLB and Traefik, and monitors everything with kube-prometheus-stack. All secrets are encrypted at rest using SOPS and age.

## System Architecture

```mermaid
flowchart TD

    subgraph external["External Services"]
        direction LR
        github["GitHub<br/>Source of Truth"]
        letsencrypt["Let's Encrypt<br/>TLS Certificates"]
        headscale["Headscale VPN<br/>Remote Access"]
    end

    subgraph gitops["GitOps Pipeline"]
        direction LR
        flux["FluxCD"]
        sops["SOPS + age"]
        renovate["Renovate"]
        reflector["Reflector"]
        reloader["Reloader"]
    end

    subgraph net["Networking"]
        direction LR
        certmanager["cert-manager"]
        traefik["Traefik"]
        metallb["MetalLB"]
        tailscale["Tailscale Router"]
    end

    subgraph mon["Monitoring"]
        direction LR
        prometheus["Prometheus"]
        grafana["Grafana"]
        alertmanager["Alertmanager"]
        metrics["Metrics Server"]
    end

    subgraph stor["Storage"]
        longhorn["Longhorn"]
    end

    subgraph apps["Applications"]
        direction LR
        vaultwarden["Vaultwarden"]
        karakeep["Karakeep"]
        vikunja["Vikunja"]
        mealie["Mealie"]
        excalidraw["Excalidraw"]
        glance["Glance"]
        tarnished["Tarnished"]
    end

    subgraph cluster["Talos Linux Cluster"]
        direction LR
        node1["Control Plane + Worker"]
        node2["Control Plane"]
        node3["Worker"]
    end

    renovate -->|PRs| github
    github -->|sync| flux
    flux -->|decrypt| sops

    flux -.->|deploy| net
    flux -.->|deploy| mon
    flux -.->|deploy| stor
    flux -.->|deploy| apps
    flux -.->|configure| reflector
    flux -.->|configure| reloader

    letsencrypt -->|certs| certmanager
    certmanager -->|TLS| traefik

    headscale -->|VPN| traefik
    metallb --> traefik
    traefik --> apps
    traefik --> mon

    prometheus --- grafana
    prometheus --- alertmanager

    apps --> longhorn
    mon --> longhorn

    net --> cluster
    stor --> cluster
    apps --> cluster
```

## Writing

I write about my homelab architecture and decisions on my blog. These posts provide deeper context on the design choices and implementation details.

- [Installing Talos Linux](https://markonakic.xyz/posts/talos-install/) - Complete walkthrough of setting up Talos Linux from scratch
- [Remote Access Architecture of My Kubernetes Homelab](https://markonakic.xyz/posts/remote-access/) - How I use Headscale for secure remote access to self-hosted services
