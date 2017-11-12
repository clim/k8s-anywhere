# Kubernetes Dashboard

## Deploy Kubernetes Dashboard
```
curl -OL https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard-arm.yaml
curl -OL https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/alternative/kubernetes-dashboard-arm.yaml
curl -OL https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
curl -sSL https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml \
  | sed "s/amd64/arm/g" | > kubernetes-dashboard.yml

https://raw.githubusercontent.com/kubernetes/dashboard/v1.7.1/src/deploy/recommended/kubernetes-dashboard-arm.yaml

kubectl apply -f kubernetes-dashboard-arm.yaml

curl -sSL \
  https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard-arm.yaml \
  | kubectl apply -f -

ssh -L8001:localhost:8001 pirate@10.91.145.132
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```
