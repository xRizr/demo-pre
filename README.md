# Azure Identity demo — app template

This repo is a **GitHub template repository**. Each per-project repo created from it
inherits everything below, then Terraform commits a single `values.yaml` at the root.
That commit triggers `.github/workflows/deploy.yml`, which does:

```
helm upgrade --install <projectName> ./chart -n <namespace> --values ./values.yaml
```

## Structure

```
.github/workflows/deploy.yml   # OIDC -> azure/login -> helm upgrade
chart/                         # Helm chart with templates for SA, Deployment,
  Chart.yaml                   #   Service (ClusterIP), HTTPRoute (shared Gateway).
  values.yaml                  # defaults, overridden per project
  templates/
    serviceaccount.yaml
    deployment.yaml
    service.yaml
    httproute.yaml
values.yaml                    # ← committed by Terraform per project
```

## Identities

- **Deploy identity** — UAMI federated to `repo:<org>/<repo>:environment:production`.
  GitHub Actions assumes it via OIDC, gets namespace-scoped Azure RBAC.
- **Runtime identity** — different UAMI federated to the AKS OIDC issuer (subject
  `system:serviceaccount:<ns>:web`). Pod uses it to read Storage at runtime.

Two identities, one federation mechanism, zero stored secrets.

## Exposure

Apps are exposed via Gateway API on a shared Envoy Gateway. HTTPRoute matches
hostname `<projectName>.<gateway_ip>.nip.io` against the shared Gateway's listener.
No per-project LoadBalancer IPs.
