# CKA Repository Structure

## 01-storage-10%
- 01-storageclass-fast.yaml
- 02-storageclass-standard.yaml
- 03-pvc-database.yaml
- 04-pvc-web-app.yaml
- 05-pod-with-pvc.yaml
- 06-pv-static.yaml
- 07-pod-with-hostpath.yaml
- 08-pod-with-emptydir.yaml

## 02-troubleshooting-30%
- 01-broken-pod.yaml
- 02-broken-deployment.yaml
- 03-broken-service.yaml
- 04-broken-networkpolicy.yaml
- 05-resource-quota-issue.yaml
- 06-broken-rbac.yaml
- 07-broken-configmap.yaml
- 08-broken-secret.yaml
- 09-node-issue-simulation.yaml
- 10-etcd-backup-restore.sh

## 03-workloads-scheduling-15%
- 01-deployment-basic.yaml
- 02-deployment-with-resources.yaml
- 03-pod-with-nodeselector.yaml
- 04-pod-with-node-affinity.yaml
- 05-pod-with-pod-affinity.yaml
- 06-pod-with-taints-tolerations.yaml
- 07-hpa-basic.yaml
- 08-hpa-advanced.yaml
- 09-configmap-basic.yaml
- 10-secret-basic.yaml
- 11-pod-with-configmap-secret.yaml

## 04-cluster-architecture-installation-configuration-25%
- 01-serviceaccount.yaml
- 02-role-basic.yaml
- 03-rolebinding-basic.yaml
- 04-clusterrole-basic.yaml
- 05-clusterrolebinding-basic.yaml
- 06-rbac-developer.yaml
- 07-rbac-admin.yaml
- 08-kubeadm-config.yaml
- 09-kubeadm-join.yaml
- 10-crd-basic.yaml
- 11-custom-resource.yaml
- kubeadm-upgrade.sh
- etcd-backup.sh
- etcd-restore.sh

## 05-services-networking-20%
- 01-service-clusterip.yaml
- 02-service-nodeport.yaml
- 03-service-loadbalancer.yaml
- 04-ingress-basic.yaml
- 05-ingress-tls.yaml
- 06-networkpolicy-deny-all.yaml
- 07-networkpolicy-allow-specific.yaml
- 08-networkpolicy-namespace.yaml
- 09-networkpolicy-egress.yaml
- 10-coredns-config.yaml
- 11-gateway-api-gateway.yaml
- 12-gateway-api-httproute.yaml