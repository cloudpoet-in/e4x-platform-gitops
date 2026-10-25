# e4x-platform-gitops

GitOps source for the e4x platform cluster. Currently Argo CD only.

## Layout

```
bootstrap/argo-cd/
  values.yaml               values for the argo/argo-cd Helm chart
```

Argo CD itself is sourced from the public chart repo
(`https://argoproj.github.io/argo-helm`, chart `argo-cd`), not vendored here.
Only the values live in this repo.

## How it gets onto a cluster

[`e4x-platform-bootstrap`](../e4x-platform-bootstrap) (Terraform) renders the
chart once with `helm template` and applies it, then creates the `argo-cd`
Application (a multi-source Application: the chart as one source, this repo
as a `ref`-only source for `bootstrap/argo-cd/values.yaml`), so Argo CD
manages itself from here on.

Per-cluster values are **not** committed here; Terraform passes them in:

| Value | Where it lands |
|---|---|
| `argocd_fqdn` | `configs.cm.url` (Application `source.helm.valuesObject` merge, on top of this repo's `values.yaml`) |
| `argocd_oidc` | `argocd-oidc-secret` Secret, read by `configs.cm."oidc.config"` via `$argocd-oidc-secret:<KEY>` |

## Not included (vs. cloudpoet-in-gitops-main)

- **HTTPRoute** - needs Envoy Gateway. Reach Argo CD via port-forward or add
  your own ingress.
- **ExternalSecret for the OIDC client** - needs External Secrets + Akeyless.
  Terraform creates the Secret directly instead.
- **`server.insecure=true`** - only made sense behind a TLS-terminating
  Gateway. argocd-server serves its own (self-signed) TLS here.
- `root` / `cluster-resources` / `projects/` - Argo CD only, for now.
