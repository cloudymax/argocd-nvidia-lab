# argocd-nvidia-lab

An ArgoCD [app-of-apps] to deploy a Nvidia-based HPC cluster on [smol-k8s-lab].

## Contents

1. Kube-prometheus-stack
2. Loki-stack
3. Nvidia GPU Operator
4. Extra Scrape targets (wip)
5. Dashboards (wip)
6. Production ClusterIssuer (not set as default in apps yet)

## Extras

 - nvidia-glx-desktop
 - nvidia-egl-desktop
 - games-on-whales
 - novnc-desktop
 - juicefs (wip)
 - knative


<!-- external link references -->
[smol-k8s-lab]: https://github.com/small-hack/smol-k8s-lab "Get started with k0s, k3s, or kind to bootstrap simple projects on kubernetes with a friendly smol vibe. Great for testing webapps and benchmarking."
[app-of-apps]: https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/ "ArgoCD App-of-Apps pattern documentation"

## Scrape Configs

Add this yaml for a prometheus scrape configs as a secret

```yaml
- job_name: "nginx-ingress"
  static_configs:
  - targets: ["ingress-nginx-controller-metrics.ingress-nginx.svc.cluster.local:10254"]
- job_name: "postgres"
  static_configs:
  - targets: ["postgres-postgresql-metrics.default.svc.cluster.local:9187"]
```
```

```bash
kubectl create ns monitoring
kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml \
    --dry-run=client -oyaml > additional-scrape-configs.yaml

kubectl apply -f additional-scrape-configs.yaml -n monitoring
```

## NGINX MOD Security + Metrics

Update the Nginx configmap:

```bash
kubectl edit configmap -n ingress-nginx ingress-nginx-controller
```

```yaml
apiVersion: v1
data:
  # ...
  enable-owasp-modsecurity-crs: "true"
  enable-modsecurity: "true"
  load-balance: "ewma"
  # ...
```

```bash
helm upgrade ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx \
--set controller.metrics.enabled=true \
--set-string controller.podAnnotations."prometheus\.io/scrape"="true" \
--set-string controller.podAnnotations."prometheus\.io/port"="10254"
```

- Install the ArgoCD CLI and log in

```bash
# Install the argocd CLI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

```bash
# Login to argocd via cli
argocd login argocd.cloudydev.net --username admin \
    --grpc-web \
    --password $(bw get password argocd.cloudydev.net)
```

- Deploy the app of apps

```bash
# deploy the app of apps
kubectl config set-context --current --namespace=argocd
argocd app create app-of-apps --repo https://github.com/cloudymax/argocd-nvidia-lab.git \
    --path templates/ \
    --dest-server https://kubernetes.default.svc
    
argocd app sync app-of-apps
```

## Kubevirt Dashboard

- https://grafana.com/grafana/dashboards/11748-kubevirt/
