# argocd-nvidia-lab

An ArgoCD [app-of-apps] to deploy a Nvidia-based HPC cluster on [smol-k8s-lab].

## Contents

1. Kube-prometheus-stack (WiP)
2. Loki-stack (todo)
3. Nvidia GPU Operator (todo)
4. Extra Scrape targets (todo)
5. Dashboards (todo)
6. Production ClusterIssuer (todo)

<!-- external link references -->
[smol-k8s-lab]: https://github.com/small-hack/smol-k8s-lab "Get started with k0s, k3s, or kind to bootstrap simple projects on kubernetes with a friendly smol vibe. Great for testing webapps and benchmarking."
[app-of-apps]: https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/ "ArgoCD App-of-Apps pattern documentation"

## Scrape Configs

Add this yaml for a prometheus scrape configs as a secret

```yaml
- job_name: "postgres"
  static_configs:
  - targets: ["postgres-postgresql-metrics.cosmos.svc.cluster.local:9187"]
```

```bash
kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml \
    --dry-run=client -oyaml > additional-scrape-configs.yaml

kubectl apply -f additional-scrape-configs.yaml -n monitoring
```
