
Initialize Ansible:

Disable host key checking
[defaults]
host_key_checking = false

To check that all nodes are reacheable:
ansible all -m ping -i hosts -k

Upload ssh key to all nodes:
ansible-playbook -i hosts upload_ssy_keys.yml -k

Change hostname
ansible-playbook -e ‘reboot=true’ -i hosts change_hostname.yml

# Initialize the master node
```
kubeadm init —pod-network-cidr 10.244.0.0/16 —apiserver-advertise-address 10.91.145.132
```

## WIP

A longer route is to:
```
kubeadm token generate
```

Use the generate token for the --token parameter.

kubeadm messages when generating the certificates:
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [k8snode01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.91.145.132]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"


To generate the CA key hash:
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

Kubernetes document to generate certificates:
https://kubernetes.io/docs/concepts/cluster-administration/certificates/

openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt

Use the generated hash for the --discovery-token-ca-cert-hash parameter

# Install a pod network using Flannel
curl https://rawgit.com/coreos/flannel/master/Documentation/kube-flannel.yml \
|  sed "s/amd64/arm/g" | sed "s/vxlan/host-gw/g" \
  > kube-flannel.yaml
kubectl apply -f kube-flannel.yaml

NOTE: I get issues with Flannel. kube-dns doesn't come up to Ready state.


# Install a pod network using Weave Net
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"

# Configure worker nodes to join the cluster
```
kubeadm join --token 249e1b.192243f317d05c96 10.91.145.132:6443 --discovery-token-ca-cert-hash sha256:1c70d1d092cb6ae1fbfada6e74c5c5d5c3ac4d12b45c60f2566b52f6a4c2f864
```

kubectl describe —namespace=kube-system configmaps/kube-flannel-cfg
kubectl describe —namespace=kube-system daemonsets/kube-flannel-ds
kubectl —namespace=kube-system describe pod kube-dns-66ffd5c588-frf55

kubectl get pods —all-namespaces


# Deploy Kubernetes Dashboard
curl -OL https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard-arm.yaml
kubectl apply -f kubernetes-dashboard-arm.yaml

curl -sSL \
  https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard-arm.yaml \
  | kubectl apply -f -

ssh -L8001:localhost:8001 pirate@10.91.145.132
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/


# Deploy Sample Application

Create a namespace
kubectl create namespace sock-shop

Run the application
kubectl apply -n sock-shop -f “https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true”

To check the exposed port
kubectl -n sock-shop get svc front-end

To list all the pods for this namespace
kubectl get pods -n sock-shop

To uninstall the sample application:
kubectl delete namespace sock-shop



Sample 2:
kubectl run hypriot —image=hypriot/rpi-busybox-httpd —replicas=3 —port=80
kubectl expose deployment hypriot —port 80
kubectl get endpoints hypriot

Traefik load balancer:
kubectl apply -f https://raw.githubusercontent.com/hypriot/rpi-traefik/master/traefik-k8s-example.yaml
kubectl label node k8snode01 nginx-controller=traefik

hypriot-ingress.yml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hypriot
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: hypriot
          servicePort: 80

kubectl apply -f hypriot-ingress.yml




# Tear down

kubectl drain k8snode02 —delete-local-data —force —ignore-daemonsets
kubectl delete node k8snode02
