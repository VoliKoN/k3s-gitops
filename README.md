# My k3s cluster

This repository contains files and instructions for deploying my home cluster on [k3s](https://k3s.io/) using [k3sup](https://github.com/alexellis/k3sup) backed by [Flux](https://toolkit.fluxcd.io/) and [SOPS](https://toolkit.fluxcd.io/guides/mozilla-sops/).

I'm currently using a single node running in a vm on Proxmox.

For docs please see [Setup](docs/setup.md).
## :wave:&nbsp; Introduction

The cluster is using those tools:

- [flannel](https://github.com/flannel-io/flannel)
- [flux](https://toolkit.fluxcd.io/)
- [metallb](https://metallb.universe.tf/)
- [cert-manager](https://cert-manager.io/) with Cloudflare DNS challenge
- [traefik](https://doc.traefik.io/traefik/providers/kubernetes-crd/)
- [authentik](https://github.com/goauthentik/authentik)
- [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller)


### :robot:&nbsp; Automation

- [Renovate](https://www.whitesourcesoftware.com/free-developer-tools/renovate) is a very useful tool that when configured will start to create PRs in your Github repository when Docker images, Helm charts or anything else that can be tracked has a newer version. The configuration for renovate is located [here](./.github/renovate.json5).

- [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) will watch for new k3s releases and upgrade your nodes when new releases are found.

There's also a couple Github workflows included in this repository that will help automate some processes.

- [Flux upgrade schedule](./.github/workflows/flux-schedule.yaml) - workflow to upgrade Flux.

## :handshake:&nbsp; Thanks

Big shout out to all the authors and contributors to the original template and also those repositories that influenced this project heavily:

- [onedr0p/home-cluster](https://github.com/onedr0p/home-cluster)
- [bjw-s/k8s-gitops](https://github.com/bjw-s/k8s-gitops)
