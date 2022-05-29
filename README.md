# Testing the Kube Prometheus Stack deployment
Create a namespace for the deployment
```
kubectl create ns monitoring
```

## Minikube
We have to tweak a bit the configuration to avoid the failure of some monitored targets in the provided serviceMonitors

Monitoring etcd requires using HTTPS with a certificate signed by the etcd CA. These certificates can be found in the minikube node and we need to put them in a secret that will be mounted by the Prometheus instance
```
mkdir etcd-certs
minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/ca.crt > etcd-certs/ca.crt
minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/server.crt > etcd-certs/server.crt
minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/server.key > etcd-certs/server.key
kubectl create secret generic etcd-certs -n monitoring --from-file=etcd-certs
rm -rf etcd-certs
```

In minikube, the components kube-controller-manager and kube-scheduler are static Pods (started by kubelet). By default they listen in the node loopback address (127.0.0.1) and cannot be directly monitored by Prometheus. One possible solution to that is to change the IP address in the manifests to listen in all the network interfaces (0.0.0.0)
```
minikube ssh -- sudo sed -i 's/bind-address=127.0.0.1/bind-address=0.0.0.0/' \
  /etc/kubernetes/manifests/kube-controller-manager.yaml \
  /etc/kubernetes/manifests/kube-scheduler.yaml

```

Ensure the changes are applied by restarting the kubelet. This will force kubelet to recreate all the pods not running
```
minikube ssh -- sudo systemctl restart kubelet
```

Before installing the Helm chart, edit the `values_minikube.yaml` and replace the following settings if needed:
* The ingresses hosts which point to the default Minikube IP (192.168.49.2) 
* The grafana serviceMonitor label `release: kps` must match the Helm release name

Deploy the Kube Prometheus stack with the provided custom values
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kps prometheus-community/kube-prometheus-stack -n monitoring -f values_minikube.yaml
```

### Argo CD deployment
Find the Application manifests in the argocd directory

**NOTE:** There is an issue with the creation of Kube Prometheus Stack CRDs. A possible solution is [this workaround](https://blog.ediri.io/kube-prometheus-stack-and-argocd-23-how-to-remove-a-workaround)
