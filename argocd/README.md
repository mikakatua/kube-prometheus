1. Create the project `monitoring` in Argo CD
2. Add the following repositories in Argo CD:
  - https://github.com/mikakatua/kube-prometheus-test.git (git)
  - https://charts.rancher.io (helm)
3. Add some custom health checks:
  - Enable Argo CD Application health assessment. This is necessary because we are using app-of-apps pattern and orchestrating synchronization using sync waves. More details [here](https://argo-cd.readthedocs.io/en/stable/operator-manual/upgrading/1.7-1.8/#health-assessement-of-argoprojioapplication-crd-has-been-removed)
  - Disable Prometheus health assessment. This is necessary because the Prometheus resource does not provide a health check and Sync status will be "Progressing" forever. More details [here](https://github.com/argoproj/argo-cd/issues/11261) and [here](https://github.com/argoproj/argo-cd/issues/11782)
```
kubectl -n argocd patch cm argocd-cm --patch-file argocd-cm_patch.yaml
```
4. Deploy the app-of-apps 
```
kubectl -n argocd apply -f app-of-apps.yaml
```
