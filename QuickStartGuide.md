# BIG-IQ and Container Ingress Controler Intergration Quick Start Guide

This page is created to document BIG-IQ with intergration of CIS and BIGIP. Please open issues on my github page on contact me at m.dittmer@f5.com. This intergration is experimental and for demo purposes.

# Note

Environment parameters

* K8s 1.14 
* CIS 1.11
* AS3: 3.13.1 LTS
* BIG-IQ 7.1
* BIG-IP 14.1

## Prerequisite

Since we using BIG-IQ, BIG-IQ will prepare the BIG-IP with AS3

## Create CIS Controller, BIG-IP, BIG-IQ credentials and RBAC Authentication

```
args: [
        "--bigip-username=$(BIGIP_USERNAME)",
        "--bigip-password=$(BIGIP_PASSWORD)",
        # Replace with the IP address or hostname of your BIG-IP device
        "--bigip-url=192.168.200.82",
        # Replace with the name of the BIG-IP partition you want to manage
        "--bigip-partition=openshift",
        "--pool-member-type=cluster",
        # Replace with the path to the BIG-IP VXLAN connected to the
        # OpenShift HostSubnet
        "--openshift-sdn-name=/Common/openshift_vxlan",
        "--manage-routes=true",
        "--namespace=f5demo",
        "--route-vserver-addr=10.192.75.107",
        "--log-level=DEBUG",
        # Self-signed cert
        "--insecure=true",
        "--agent=as3"
       ]
```
```
oc create secret generic bigip-login --namespace kube-system --from-literal=username=admin --from-literal=password=f5PME123
oc create serviceaccount bigip-ctlr -n kube-system
oc create -f f5-kctlr-openshift-clusterrole.yaml
oc create -f f5-k8s-bigip-ctlr-openshift.yaml
oc adm policy add-cluster-role-to-user cluster-admin -z bigip-ctlr -n kube-system
```
