# Notes

If you want your argo app to use a local values.yaml file, you must also have the chart locally as `Chart.yaml` (capitalization matters).
Mixing remote charts and local values files is not supported. Its a known-issue that is currently a "wont-fix" for the ArgoCD team.

If you want to use a remote chart source you will need to put the helm values into the application yaml files the ugly way. sorry folx.
