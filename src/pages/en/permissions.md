---
title: Kubeshark's Security Context & RBAC
description: This document outlines the necessary Kubernetes permissions for the efficient operation of Kubeshark, a network monitoring tool.
layout: ../../layouts/MainLayout.astro
---

## The Worker DaemonSet

Kubeshark's Worker DaemonSet is a critical component designed to monitor network traffic within the Kubernetes cluster. To function effectively, it requires specific capabilities that go beyond the standard set. These capabilities are essential for enabling network sniffing and detailed traffic analysis. The security context for the Worker DaemonSet is defined as follows:

```yaml
securityContext:
  capabilities:
    add:
      - NET_RAW        # Essential for raw socket handling, used in network sniffing
      - NET_ADMIN      # Needed for network administration tasks
      - SYS_ADMIN      # Grants various system administration permissions
      - SYS_PTRACE     # Allows tracing system calls and processes
      - DAC_OVERRIDE   # Bypasses file read, write, and execute permission checks
      - SYS_RESOURCE   # Permits resource configuration and management
      - CHECKPOINT_RESTORE # Enables checkpoint and restore capabilities of processes
    drop:
      - ALL            # Drops all capabilities not explicitly added

```

## Service Account

Kubeshark utilizes a dedicated Service Account named `kubeshark-service-account` for all its components. This account is specifically configured to provide the necessary access permissions for Kubeshark's operations within the Kubernetes environment, ensuring secure and efficient performance.

## Cluster Role

The Cluster Role in Kubeshark is designed to grant broad permissions across the entire Kubernetes cluster. This role is crucial for Kubeshark to access and monitor various Kubernetes resources at a cluster-wide level. The Cluster Role Binding, detailed below, outlines these permissions:

```yaml
rules:
  - apiGroups:
      - ""
      - extensions
      - apps
    resources:
      - pods
      - services
      - endpoints
      - persistentvolumeclaims
    verbs:
      - list
      - get
      - watch
```

## Namespace Specific Role

Within the specific namespace where Kubeshark is deployed, a Role Binding is used to grant targeted permissions for namespace-level resources. This ensures Kubeshark's access to essential configurations and secrets within its operational namespace:

```yaml
rules:
  - apiGroups:
      - ""           # Core API group
      - v1           # Version 1 of the core API group
    resourceNames:
      - kubeshark-secret      # Specific secret for Kubeshark
      - kubeshark-config-map  # Specific config map for Kubeshark
    resources:
      - secrets       # Access to secrets resource
      - configmaps    # Access to configmaps resource
    verbs:
      - get           # Permission to get resource details
      - watch         # Permission to watch for changes in resources
      - update        # Permission to update resources
```

These permissions are integral for Kubeshark's self-configuration and adaptive operation within the Kubernetes environment.