
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





kubeadm init —pod-network-cidr 10.244.0.0/16 —apiserver-advertise-address 10.91.145.132

kubeadm join —token 99905e.a41d71ba9f5cf96b 10.91.145.132:6443 —discovery-token-ca-cert-hash sha256:1717f9252bbff3b0a123011696e06cb273f456505a5920978bd521f9cb42b1b0

curl https://rawgit.com/coreos/flannel/master/Documentation/kube-flannel.yml \
|  sed “s/amd64/arm/g” | sed “s/vxlan/host-gw/g” \
  > kube-flannel.yaml
kubectl apply -f kube-flannel.yaml

kubectl describe —namespace=kube-system configmaps/kube-flannel-cfg
kubectl describe —namespace=kube-system daemonsets/kube-flannel-ds
kubectl —namespace=kube-system describe pod kube-dns-66ffd5c588-frf55

kubectl get pods —all-namespaces

curl -OL https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard-arm.yaml
kubectl apply -f kubernetes-dashboard-arm.yaml

curl -sSL \
  https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard-arm.yaml \
  | kubectl apply -f -

ssh -L8001:localhost:8001 pirate@10.91.145.132
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/



Sample application:

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




Tear down

kubectl drain k8snode02 —delete-local-data —force —ignore-daemonsets
kubectl delete node k8snode02