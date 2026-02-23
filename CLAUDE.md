# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

**This project is being decommissioned** in favour of a simpler Docker Compose setup. Only minimal maintenance work is expected. Do not add new applications or refactor existing ones.

## Known Issues (not worth fixing)

- `bjw-s-charts` HelmRepository returns 404 — the org moved to `bjw-s-labs` but chart upgrades are not planned. Cached `app-template@1.5.1` still works for existing apps.
- Authentik `auth-snippet` annotations were removed from all apps (esphome, zigbee2mqtt, bazarr, lidarr, radarr, sonarr) — those apps are now unauthenticated at the ingress level.
- `allowSnippetAnnotations: true` remains set in ingress-nginx but is harmless.

## Kubeconfig

The `kubeconfig` file is gitignored. Fetch it from the node:

```bash
ssh <user>@<node-ip> sudo cat /etc/rancher/k3s/k3s.yaml > ./kubeconfig
sed -i 's|https://127.0.0.1:6443|https://<node-ip>:6443|' ./kubeconfig
chmod 600 ./kubeconfig
export KUBECONFIG=./kubeconfig   # or use direnv (already configured in .envrc)
```

## Common Commands

```bash
# Check overall cluster health
flux get kustomizations -A
flux get helmreleases -A
flux get helmreleases -A --status-selector ready=false  # show only failures

# Force Flux to sync from git
flux reconcile source git flux-system

# Force reconcile a specific app
flux reconcile helmrelease <app> -n <namespace>

# Decrypt a SOPS secret for inspection
sops -d cluster/base/cluster-secrets.yaml

# Lint YAML (same as CI)
yamllint -c .github/yamllint.config.yaml cluster/
```

## Architecture

### Flux Deployment Order

1. **CRDs** (`cluster/base/crds.yaml`) — no pruning
2. **Core** (`cluster/base/core.yaml`) — cert-manager, MetalLB, namespaces, system-upgrade-controller; **pruning disabled**
3. **Apps** (`cluster/base/apps.yaml`) — all user applications; **pruning enabled**

### Variable Substitution

`${VAR_NAME}` placeholders in any manifest under `cluster/core/` or `cluster/apps/` are replaced at reconciliation time via `postBuild.substituteFrom`:
- Non-sensitive vars: `cluster-settings` ConfigMap (`cluster/base/cluster-settings.yaml`)
- Secrets: `cluster-secrets` Secret (`cluster/base/cluster-secrets.yaml`, SOPS-encrypted)

### Secrets (SOPS)

Files with `data`/`stringData` fields must be encrypted before committing. The pre-commit `forbid-secrets` hook enforces this. Two PGP keys are used (personal + Flux in-cluster key), defined in `.sops.yaml`.

### Application Pattern

```
cluster/apps/<namespace>/<app-name>/
├── kustomization.yaml
├── helmrelease.yaml   # bjw-s/app-template@1.5.1
├── secret.yaml        # SOPS-encrypted (if needed)
└── pvc.yaml           # (if needed)
```

Ingress uses ingress-nginx with TLS via cert-manager (Let's Encrypt + Cloudflare DNS challenge).

### App Namespaces

| Namespace | Contents |
|-----------|----------|
| `home` | Home Assistant, ESPHome, Node-RED, Zigbee2MQTT, VernemQ, Vikunja, Wiki |
| `media` | Jellyfin, qBittorrent, Sonarr, Radarr, Lidarr, Prowlarr, Bazarr, Calibre-Web, Wallabag |
| `monitoring` | kube-prometheus-stack, Grafana, speedtest-exporter |
| `networking` | ingress-nginx, Authentik, wildcard TLS certificate |

## YAML Style

- 2-space indentation, `indent-sequences: consistent`
- No line length limit
- Allowed truthy values: `true`, `false`, `on`, `yes`
