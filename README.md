# Renovate Config Presets for TorrentBox

Shareable Renovate presets for TorrentBox-kubernetes container image management.

## Overview

These presets provide a comprehensive Renovate configuration focused exclusively on container image updates while completely ignoring Helm-related dependencies. Designed for GitOps-managed Kubernetes clusters using ArgoCD.

## Available Presets

### default
Base configuration that:
- Extends `config:best-practices`
- Ignores all Helm managers
- Blocks `latest` container tags
- Enables Kubernetes manifest scanning
- Sets up dependency dashboard
- Configures rate limiting (3 concurrent, 1 hourly)
- Enables platformAutomerge for approved updates
- Schedules automerge for low-traffic hours (1-6am weekdays, weekends)

### statefulset-containers
Specialized handling for StatefulSet applications:
- Groups arr-stack services (Sonarr, Radarr, Lidarr, Bazarr)
- Groups download clients with VPN sidecars
- Handles media servers with hardware acceleration
- Auto-merges digest-only updates
- Auto-merges patch updates for all containers (security fixes)
- Auto-merges minor updates for stable services (hotio, homepage, flaresolverr)
- Adds PR checklists for major updates

### container-security
Security-focused policies:
- Prioritizes CVE patches
- Auto-merges security patches for critical components
- Flags images using `latest` tag
- Manages authentication stack updates carefully
- Adds vulnerability alert labels

### registry-groups
Groups updates by container registry:
- GitHub Container Registry (ghcr.io)
- Docker Hub
- Vendor-specific registries
- Separates patch, minor, and major updates

## Usage

In your `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>stravos97/renovate-config-torrentbox",
    "github>stravos97/renovate-config-torrentbox:statefulset-containers",
    "github>stravos97/renovate-config-torrentbox:container-security",
    "github>stravos97/renovate-config-torrentbox:registry-groups"
  ]
}
```

Or use with `local>` prefix for same-platform repositories:

```json
{
  "extends": [
    "local>stravos97/renovate-config-torrentbox",
    "local>stravos97/renovate-config-torrentbox:statefulset-containers"
  ]
}
```

## Features

- **No Helm Updates**: All Helm managers are disabled
- **Container Focus**: Only Kubernetes container images are managed
- **StatefulSet Ready**: Special handling for persistent workloads
- **Security First**: CVE patches are prioritized and auto-merged
- **GitOps Compatible**: Designed for ArgoCD workflows
- **Smart Grouping**: Logical grouping by purpose and registry
- **Automerge Enabled**: Safe updates merge automatically during low-traffic hours
- **Progressive Rollout**: Patch → Minor → Major update strategy

## Automerge Strategy

The presets implement a progressive automerge strategy:

| Update Type | Containers | Schedule | Approval |
|------------|------------|----------|----------|
| Digest | All | Immediate | Auto |
| Patch | All | Immediate | Auto |
| Minor | Stable services¹ | 1-6am weekdays, weekends | Auto |
| Minor | Other services | N/A | Manual |
| Major | All | N/A | Manual |

¹ Stable services: hotio images, homepage, flaresolverr

## Example Configuration

Combine with project-specific rules:

```json
{
  "extends": [
    "github>stravos97/renovate-config-torrentbox",
    "github>stravos97/renovate-config-torrentbox:container-security"
  ],
  "packageRules": [
    {
      "description": "Project-specific GPU compatibility",
      "matchPackageNames": ["intel/intel-gpu-plugin"],
      "allowedVersions": "< 1.0.0"
    }
  ]
}
```

## License

MIT