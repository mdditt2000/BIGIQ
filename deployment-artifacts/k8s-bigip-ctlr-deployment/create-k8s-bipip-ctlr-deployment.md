## Create kubernetes bigip container ingress service

```
kubectl create secret generic bigip-login -n kube-system --from-literal=username=admin --from-literal=password=admin
kubectl create secret generic bigiq-login -n kube-system --from-literal=username=cis --from-literal=password=cis
kubectl create serviceaccount k8s-bigip-ctlr -n kube-system
kubectl create clusterrolebinding k8s-bigip-ctlr-clusteradmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8s-bigip-ctlr
kubectl create -f k8s-bigip-ctlr-deployment.yaml
kubectl create -f f5-bigip-node.yaml
```
