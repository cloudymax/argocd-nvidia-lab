# Notes


## Known Issues

1. Mixing remote charts and local values files is not supported. See [2789] and [7257].

    If you want your argo app to use a local `values.yaml` file, you must also have the chart locally as `Chart.yaml` (capitalization matters). See [the helm directory] for an example. The ArgoCD team has introduced a hacky [work-around] that instead of allowing local source to be specified requires defining a repo as a $VAR_NAME that can be referenced later in another block. I havent been able to get it to work though and it's not in the official docs anywhere yet.

    > TL;DR: If you want to use a remote chart source you will need to put the helm values into the application yaml files the ugly way. sorry folx.

<!-- external link references -->
[2789]: https://github.com/argoproj/argo-cd/issues/2789 "ArgoCD ticket 2789"
[7257]: https://github.com/argoproj/argo-cd/issues/7257 "ArgoCD ticket 7257"
[work-around]: https://github.com/argoproj/argo-cd/issues/2789#issuecomment-1446647827 "Workaround for issue 2789"
[the helm directory]: https://github.com/cloudymax/argocd-nvidia-lab/tree/main/prometheus/helm "Helm directory for this projects prometheus app"

