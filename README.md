# Testing the Kube Prometheus Stack deployment
Create a namespace for the deployment
```
kubectl create ns monitoring
```

## Minikube
We have to tweak a bit the configuration to avoid the failure of some monitored targets:
```
mkdir etcd-certs
minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/ca.crt > etcd-certs/ca.crt
minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/server.crt > etcd-certs/server.crt
minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/server.key > etcd-certs/server.key
kubectl create secret generic etcd-certs -n monitoring --from-file=etcd-certs
rm -rf etcd-certs

minikube ssh -- sudo sed -i 's/bind-address=127.0.0.1/bind-address=0.0.0.0/' \
  /etc/kubernetes/manifests/kube-controller-manager.yaml \
  /etc/kubernetes/manifests/kube-scheduler.yaml

minikube ssh -- sudo systemctl restart kubelet
```

Deploy it with Helm. The Ingress configured in the values points to the default Miniube IP (192.168.49.2)
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kps prometheus-community/kube-prometheus-stack -n monitoring -f values_minikube.yaml
```
