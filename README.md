# portal-api-chart

Helm chart for **portal-api-server** — the bouc.io user-facing portal API
(personal instructions, billing/subscription records, payment methods).

Part of the [bouc.io AI platform](../../../documentation/getting-started/README.md#ai-assistant-platform).

## What it deploys

A stateless Deployment plus a ClusterIP Service, backed by a bundled **PostgreSQL** subchart.

| Object | Name |
|---|---|
| Deployment | `<release>-portal-api-chart` |
| Service | `<release>-portal-api-chart` |
| ServiceAccount | `<release>-portal-api-chart` (when `serviceAccount.create`) |
| HorizontalPodAutoscaler | `<release>-portal-api-chart` (when `autoscaling.enabled`) |
| Ingress | `<release>-portal-api-chart` (when `ingress.enabled`) |

The container name inside the pod is `{{ .Chart.Name }}`, i.e. `portal-api-chart`, so use
`-c portal-api-chart` with `kubectl exec`.

## Values

Values live in three files. There is no plain `values.yaml`.

| File | Purpose |
|---|---|
| `base.values.yaml` | Common defaults |
| `lcl.values.yaml` | Local (docker-desktop / Pi) overlay |
| `snbx.values.yaml` | Sandbox cluster overlay |

> In the cluster, FluxCD supplies values from generated ConfigMaps via `valuesFrom:`, not from these
> files directly. They are the source the ConfigMaps are generated from.

App env vars are documented in `portal-api-server/.env.example`.

## Probes

`livenessProbe` and `readinessProbe` are fixed in the Deployment template, not values-driven:
`/v1/portal/health/live` and `/v1/portal/health/ready` on the HTTP port. Note the paths carry the
`/v1/portal` prefix, unlike the other API charts.

## Local usage

```bash
helm dependency update                       # fetch the postgresql subchart
helm lint . -f base.values.yaml -f lcl.values.yaml
helm template test . -f base.values.yaml -f lcl.values.yaml
helm install portal-api . -f base.values.yaml -f lcl.values.yaml
```

The values files layer: `base` first, then exactly one environment file.

> The chart must be published to the chart registry by CI before FluxCD can reconcile it. Pushing
> chart source to git is not enough.

## License

[Elastic License 2.0](./LICENSE) — source-available; not OSI open source.
