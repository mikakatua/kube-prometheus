1. Create the project `monitoring` in Argo CD
2. Add the following repositories in Argo CD:
  - https://github.com/mikakatua/kube-prometheus-test.git (git)
  - https://charts.rancher.io (helm)
3. Enable Argo CD application health assessment. This is necessary because we are using app-of-apps pattern and orchestrating synchronization using sync waves. 
```
kubectl -n argocd patch cm argocd-cm --patch-file argocd-cm_patch.yaml
```
4. Deploy the app-of-apps 
```
kubectl -n argocd apply -f app-of-apps.yaml
```
