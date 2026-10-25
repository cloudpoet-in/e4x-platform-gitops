# e4x-platform-gitops

GitOps source for the e4x platform cluster. Currently Argo CD only.

## Layout

```
bootstrap/argo-cd/
  install.yaml              upstream Argo CD v3.5.4 install manifest (vendored)
  kustomization.yaml        argocd-cm / argocd-rbac-cm merges
  argocd-oidc-config.yaml   Keycloak OIDC config (oidc.config)
  argocd-rbac-policy.csv    group -> role mappings
```

## How it gets onto a cluster

[`e4x-platform-bootstrap`](../e4x-platform-bootstrap) (Terraform) applies
`install.yaml` once, then creates the `argo-cd` Application pointing at
`bootstrap/argo-cd` in this repo, so Argo CD manages itself from here on.

Per-cluster values are **not** committed here; Terraform passes them in:

| Value | Where it lands |
|---|---|
| `argocd_fqdn` | `argocd-cm` `url` (kustomize patch on the `argo-cd` Application) |
| `argocd_oidc` | `argocd-oidc-secret` Secret, read by `oidc.config` via `$argocd-oidc-secret:<KEY>` |

## Not included (vs. cloudpoet-in-gitops-main)

- **HTTPRoute** - needs Envoy Gateway. Reach Argo CD via port-forward or add
  your own ingress.
- **ExternalSecret for the OIDC client** - needs External Secrets + Akeyless.
  Terraform creates the Secret directly instead.
- **`server.insecure=true`** - only made sense behind a TLS-terminating
  Gateway. argocd-server serves its own (self-signed) TLS here.
- `root` / `cluster-resources` / `projects/` - Argo CD only, for now.
