# Repository Structure

# CKA Folder Structure
./cka/
├── 01-storage-10%/
├── 02-troubleshooting-30%/
├── 03-workloads-scheduling-15%/
├── 04-cluster-architecture-installation-configuration-25%/
└── 05-services-networking-20%/

# ICA Folder Structure  
./ica/
├── 01-installation-upgrades-configuration-20%/
├── 02-traffic-management-35%/
├── 03-securing-workloads-25%/
└── 04-troubleshooting-20%/

# Combined ICA-CKA Folder Structure
./ica-cka/
├── 01-secure-multi-tier-application/
├── 02-canary-deployment-observability/
├── 03-multi-environment-gitops/
├── 04-high-availability-disaster-recovery/
└── 05-security-hardening-compliance/


## CKA

### 01-storage-10%

- 01-storageclass-fast.yaml
- 02-storageclass-standard.yaml
- 03-pvc-database.yaml
- 04-pvc-web-app.yaml
- 05-pod-with-pvc.yaml
- 06-pv-static.yaml
- 07-pod-with-hostpath.yaml
- 08-pod-with-emptydir.yaml

### 02-troubleshooting-30%

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

### 03-workloads-scheduling-15%

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

### 04-cluster-architecture-installation-configuration-25%

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

### 05-services-networking-20%

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

## ICA

### 01-installation-upgrades-configuration-20%

- 01-istiooperator-basic.yaml
- 02-istiooperator-custom.yaml
- 03-istiooperator-ambient.yaml
- 04-helm-values-basic.yaml
- 05-helm-values-custom.yaml
- 06-sidecar-injection-namespace.yaml
- 07-ambient-mode-namespace.yaml
- install-istioctl.sh
- install-helm.sh
- upgrade-canary.sh
- upgrade-inplace.sh

### 02-traffic-management-35%

- 01-gateway-basic.yaml
- 02-gateway-https.yaml
- 03-gateway-multiple-hosts.yaml
- 04-virtualservice-basic.yaml
- 05-virtualservice-path-routing.yaml
- 06-virtualservice-header-routing.yaml
- 07-virtualservice-traffic-split.yaml
- 08-virtualservice-canary.yaml
- 09-virtualservice-fault-injection.yaml
- 10-virtualservice-retry-timeout.yaml
- 11-destinationrule-basic.yaml
- 12-destinationrule-subsets.yaml
- 13-destinationrule-circuit-breaker.yaml
- 14-destinationrule-outlier-detection.yaml
- 15-serviceentry-external-http.yaml
- 16-serviceentry-external-https.yaml
- 17-virtualservice-mirror.yaml
- 18-virtualservice-external.yaml

### 03-securing-workloads-25%

- 01-peerauthentication-strict.yaml
- 02-peerauthentication-permissive.yaml
- 03-peerauthentication-service.yaml
- 04-authorizationpolicy-deny-all.yaml
- 05-authorizationpolicy-allow-specific.yaml
- 06-authorizationpolicy-service-to-service.yaml
- 07-authorizationpolicy-jwt.yaml
- 08-requestauthentication-jwt.yaml
- 09-requestauthentication-custom.yaml
- 10-gateway-tls-simple.yaml
- 11-gateway-tls-passthrough.yaml
- 12-gateway-tls-mutual.yaml
- tls-secret.yaml
- jwt-secret.yaml

### 04-troubleshooting-20%

- 01-broken-gateway.yaml
- 02-broken-virtualservice.yaml
- 03-broken-destinationrule.yaml
- 04-broken-authorizationpolicy.yaml
- 05-broken-peerauthentication.yaml
- 06-broken-serviceentry.yaml
- 07-sidecar-injection-issue.yaml
- 08-mtls-issue.yaml
- debug-commands.sh
- proxy-config-commands.sh

## ICA-CKA Combined

### 01-secure-multi-tier-application

**kubernetes/**

- 01-namespace.yaml
- 02-serviceaccounts.yaml
- 03-rbac.yaml
- 04-frontend-deployment.yaml
- 05-backend-deployment.yaml
- 06-database-deployment.yaml
- 07-frontend-service.yaml
- 08-backend-service.yaml
- 09-database-service.yaml
- 10-database-pvc.yaml
- 11-networkpolicy-deny-all.yaml
- 12-networkpolicy-frontend-backend.yaml
- 13-networkpolicy-backend-database.yaml
- 14-configmap.yaml
- 15-secret.yaml

**istio/**

- 01-gateway.yaml
- 02-virtualservice-frontend.yaml
- 03-virtualservice-backend.yaml
- 04-destinationrule-frontend.yaml
- 05-destinationrule-backend.yaml
- 06-peerauthentication-strict.yaml
- 07-authorizationpolicy-deny-all.yaml
- 08-authorizationpolicy-frontend-backend.yaml
- 09-authorizationpolicy-backend-database.yaml

**scripts/**

- deploy-k8s.sh
- deploy-istio.sh
- test-connectivity.sh
- cleanup.sh

### 02-canary-deployment-observability

**kubernetes/**

- 01-namespace.yaml
- 02-webapp-v1-deployment.yaml
- 03-webapp-v2-deployment.yaml
- 04-webapp-service.yaml
- 05-webapp-hpa.yaml
- 06-configmap.yaml
- 07-secret.yaml
- 08-ingress.yaml
- 09-tls-secret.yaml
- 10-prometheus-rbac.yaml

**istio/**

- 01-gateway.yaml
- 02-virtualservice-canary.yaml
- 03-destinationrule-versions.yaml
- 04-virtualservice-fault-injection.yaml
- 05-virtualservice-retry.yaml
- 06-serviceentry-external.yaml
- 07-virtualservice-mirror.yaml

**monitoring/**

- 01-prometheus-config.yaml
- 02-grafana-config.yaml
- 03-jaeger-config.yaml
- 04-kiali-config.yaml
- 05-servicemonitor.yaml

**scripts/**

- deploy-app.sh
- deploy-monitoring.sh
- generate-traffic.sh
- canary-promotion.sh
- rollback.sh

### 03-multi-environment-gitops

**base/**

- 01-namespace.yaml
- 02-deployment.yaml
- 03-service.yaml
- 04-configmap.yaml
- 05-secret.yaml
- 06-rbac.yaml
- 07-networkpolicy.yaml
- 08-pvc.yaml
- kustomization.yaml

**overlays/dev/**

- 01-namespace-patch.yaml
- 02-deployment-patch.yaml
- 03-resource-quota.yaml
- 04-limitrange.yaml
- 05-istio-peerauthentication.yaml
- 06-istio-authorizationpolicy.yaml
- 07-istio-gateway.yaml
- 08-istio-virtualservice.yaml
- kustomization.yaml

**overlays/staging/**

- 01-namespace-patch.yaml
- 02-deployment-patch.yaml
- 03-resource-quota.yaml
- 04-limitrange.yaml
- 05-istio-peerauthentication.yaml
- 06-istio-authorizationpolicy.yaml
- 07-istio-gateway.yaml
- 08-istio-virtualservice.yaml
- kustomization.yaml

**overlays/prod/**

- 01-namespace-patch.yaml
- 02-deployment-patch.yaml
- 03-resource-quota.yaml
- 04-limitrange.yaml
- 05-istio-peerauthentication.yaml
- 06-istio-authorizationpolicy.yaml
- 07-istio-gateway.yaml
- 08-istio-virtualservice.yaml
- 09-istio-destinationrule.yaml
- kustomization.yaml

**scripts/**

- deploy-dev.sh
- deploy-staging.sh
- deploy-prod.sh
- promote-staging.sh
- promote-prod.sh

### 04-high-availability-disaster-recovery

**kubernetes/**

- 01-kubeadm-ha-config.yaml
- 02-etcd-cluster.yaml
- 03-loadbalancer-config.yaml
- 04-persistent-storage.yaml
- 05-backup-cronjob.yaml
- 06-monitoring-stack.yaml
- 07-cluster-autoscaler.yaml
- 08-pod-disruption-budget.yaml

**istio/**

- 01-istiod-ha.yaml
- 02-gateway-ha.yaml
- 03-circuit-breaker-config.yaml
- 04-retry-policies.yaml
- 05-outlier-detection.yaml
- 06-cross-cluster-config.yaml
- 07-mesh-expansion.yaml

**backup/**

- etcd-backup.sh
- etcd-restore.sh
- velero-backup.yaml
- velero-restore.yaml
- disaster-recovery-plan.md

**scripts/**

- setup-ha-cluster.sh
- test-failover.sh
- backup-cluster.sh
- restore-cluster.sh
- health-check.sh

### 05-security-hardening-compliance

**kubernetes/**

- 01-pod-security-standards.yaml
- 02-security-context.yaml
- 03-admission-controllers.yaml
- 04-rbac-least-privilege.yaml
- 05-network-policies-strict.yaml
- 06-secrets-encryption.yaml
- 07-audit-policy.yaml
- 08-service-account-tokens.yaml

**istio/**

- 01-strict-mtls-mesh.yaml
- 02-authorization-policies-strict.yaml
- 03-jwt-authentication.yaml
- 04-oauth-integration.yaml
- 05-security-monitoring.yaml
- 06-workload-identity.yaml
- 07-external-ca-config.yaml

**security/**

- 01-falco-config.yaml
- 02-opa-gatekeeper.yaml
- 03-cert-manager.yaml
- 04-vault-integration.yaml
- 05-security-scanning.yaml
- 06-compliance-checks.yaml

**scripts/**

- security-scan.sh
- compliance-check.sh
- cert-rotation.sh
- security-audit.sh
- vulnerability-assessment.sh