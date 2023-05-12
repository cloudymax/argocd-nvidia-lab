# argocd-nvidia-lab

An ArgoCD [app-of-apps] to deploy a Nvidia-based HPC cluster on [smol-k8s-lab].

## Contents
| Role | Service |
| ---  | ---     |
| Metrics & Dashboards | Kube-prometheus-stack + Extra Scrape targets |
| Log Aggregation | Loki-stack + Extra Scrape targets |
| GPU Management | Nvidia GPU Operator |
| SSO/Oauth | Vouch Proxy |
| Virtual Machines | Kubervirt |

## Reccomended Dashboards
- https://grafana.com/grafana/dashboards/15172-node-exporter-for-prometheus-dashboard-based-on-11074/
- https://grafana.com/grafana/dashboards/14314-kubernetes-nginx-ingress-controller-nextgen-devops-nirvana/

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
- job_name: "nvidia-smi"
  static_configs:
  - targets: ["nvidia-dcgm-exporter.default.svc.cluster.local:9400"]
```

```bash
kubectl create secret generic additional-scrape-configs --from-file=prometheus-additional.yaml \
    --dry-run=client -oyaml > additional-scrape-configs.yaml
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
  allow-snippet-annotations: "true"
  enable-modsecurity: "true"
  enable-owasp-modsecurity-crs: "true"
  load-balance: ewma
  modsecurity-snippet: |-
    SecRuleEngine DetectionOnly
    SecAuditEngine RelevantOnly
    SecStatusEngine On
    SecRequestBodyAccess On
    SecAuditLog /dev/stdout
    SecAuditLogFormat JSON
    SecRuleRemoveById 920420
    SecRuleRemoveById 920350
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

## Install the ArgoCD CLI and log in

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

## Postgres secret

```
kubectl create secret generic postgres \
    --namespace cosmos \
    --from-literal=POSTGRES_USER_PASSWORD="reopen-tapping-customize" \
    --from-literal=POSTGRES_ADMIN_PASSWORD="flattered-tropics-tidings" \
    --from-literal=POSTGRES_REPLICATION_PASSWORD="blurb-barber-liable"
```
## Deploy the app of apps

```bash
# deploy the app of apps
kubectl config set-context --current --namespace=argocd
argocd app create app-of-apps --repo https://github.com/cloudymax/argocd-nvidia-lab.git \
    --path templates/ \
    --dest-server https://kubernetes.default.svc
    
argocd app sync app-of-apps
```

## Install Argo Rollouts

Other install methods: https://github.com/argoproj/argo-rollouts/blob/master/docs/installation.md#kubectl-plugin-installation

Inatlls plugin
```bash
brew install argoproj/tap/kubectl-argo-rollouts
```

Watch a rollout
```bash
kubectl argo rollouts get rollout rollout-bluegreen -n nginx-hello-bluegreen --watch
```

Change rollout image:
```bash
kubectl argo rollouts set image rollout-bluegreen nginx-hello-bluegreen=argoproj/rollouts-demo:yellow -n nginx-hello-bluegreen
```

Promote rollout:
```bash
kubectl argo rollouts promote rollout-bluegreen -n nginx-hello-bluegreen
```

## Argo Workflows CLI
Check for latest release at https://github.com/argoproj/argo-workflows/releases

- Linux
 
```bash
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.4.7/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

## validate virtualization support

```bash
virt-host-validate qemu
```

## Kubevirt Dashboard

- https://grafana.com/grafana/dashboards/11748-kubevirt/


