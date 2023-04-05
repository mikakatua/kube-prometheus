# Testing the Kube Prometheus Stack deployment
Create a namespace for the deployment
```
kubectl create ns monitoring
```

## Minikube
We have to tweak a bit the configuration to avoid the failure of some monitored targets in the provided serviceMonitors

In Minikube, etcd runs as a static Pod (started by kubelet). The monitoring url is listening to the node loopback address (http://127.0.0.1:2381) and cannot be directly monitored by Prometheus. To fix that, we can change the IP address in the etcd manifest to listen to all the network interfaces (0.0.0.0)
```
minikube ssh -- sudo sed -i 's|listen-metrics-urls=http://127.0.0.1:2381|listen-metrics-urls=http://0.0.0.0:2381|' \
  /etc/kubernetes/manifests/etcd.yaml
```

The same happens with the components kube-controller-manager and kube-scheduler. We can apply the same fix
```
minikube ssh -- sudo sed -i 's/bind-address=127.0.0.1/bind-address=0.0.0.0/' \
  /etc/kubernetes/manifests/kube-controller-manager.yaml \
  /etc/kubernetes/manifests/kube-scheduler.yaml
```

The kubelet will automatically restart the Pods applying the changes in the manifests

Before installing the Helm chart, edit the `values_minikube.yaml` and replace the following settings if needed:
* The ingresses hosts which point to the default Minikube IP (192.168.49.2)
* The grafana serviceMonitor label `release: kps` must match the Helm release name

Deploy the Kube Prometheus stack with the provided custom values
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kps prometheus-community/kube-prometheus-stack -n monitoring --version 45.9.0 -f values_minikube.yaml
```

### Argo CD deployment
Find the Application manifests in the argocd directory

**NOTE:** There is an issue with the creation of Kube Prometheus Stack CRDs. A possible solution is [this workaround](https://blog.ediri.io/kube-prometheus-stack-and-argocd-23-how-to-remove-a-workaround)
