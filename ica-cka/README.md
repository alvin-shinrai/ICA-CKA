01-secure-multi-tier-application
kubernetes/

    01-namespace.yaml
    02-serviceaccounts.yaml
    03-rbac.yaml
    04-frontend-deployment.yaml
    05-backend-deployment.yaml
    06-database-deployment.yaml
    07-frontend-service.yaml
    08-backend-service.yaml
    09-database-service.yaml
    10-database-pvc.yaml
    11-networkpolicy-deny-all.yaml
    12-networkpolicy-frontend-backend.yaml
    13-networkpolicy-backend-database.yaml
    14-configmap.yaml
    15-secret.yaml

istio/

    01-gateway.yaml
    02-virtualservice-frontend.yaml
    03-virtualservice-backend.yaml
    04-destinationrule-frontend.yaml
    05-destinationrule-backend.yaml
    06-peerauthentication-strict.yaml
    07-authorizationpolicy-deny-all.yaml
    08-authorizationpolicy-frontend-backend.yaml
    09-authorizationpolicy-backend-database.yaml

scripts/

    deploy-k8s.sh
    deploy-istio.sh
    test-connectivity.sh
    cleanup.sh

02-canary-deployment-observability
kubernetes/

    01-namespace.yaml
    02-webapp-v1-deployment.yaml
    03-webapp-v2-deployment.yaml
    04-webapp-service.yaml
    05-webapp-hpa.yaml
    06-configmap.yaml
    07-secret.yaml
    08-ingress.yaml
    09-tls-secret.yaml
    10-prometheus-rbac.yaml

    istio/

        01-gateway.yaml
        02-virtualservice-canary.yaml
        03-destinationrule-versions.yaml
        04-virtualservice-fault-injection.yaml
        05-virtualservice-retry.yaml
        06-serviceentry-external.yaml
        07-virtualservice-mirror.yaml

    monitoring/

        01-prometheus-config.yaml
        02-grafana-config.yaml
        03-jaeger-config.yaml
        04-kiali-config.yaml
        05-servicemonitor.yaml

    scripts/

        deploy-app.sh
        deploy-monitoring.sh
        generate-traffic.sh
        canary-promotion.sh
        rollback.sh

03-multi-environment-gitops
base/

01-namespace.yaml
02-deployment.yaml
03-service.yaml
04-configmap.yaml
05-secret.yaml
06-rbac.yaml
07-networkpolicy.yaml
08-pvc.yaml
kustomization.yaml

overlays/dev/

01-namespace-patch.yaml
02-deployment-patch.yaml
03-resource-quota.yaml
04-limitrange.yaml
05-istio-peerauthentication.yaml
06-istio-authorizationpolicy.yaml
07-istio-gateway.yaml
08-istio-virtualservice.yaml
kustomization.yaml

overlays/staging/

01-namespace-patch.yaml
02-deployment-patch.yaml
03-resource-quota.yaml
04-limitrange.yaml
05-istio-peerauthentication.yaml
06-istio-authorizationpolicy.yaml
07-istio-gateway.yaml
08-istio-virtualservice.yaml
kustomization.yaml

overlays/prod/

01-namespace-patch.yaml
02-deployment-patch.yaml
03-resource-quota.yaml
04-limitrange.yaml
05-istio-peerauthentication.yaml
06-istio-authorizationpolicy.yaml
07-istio-gateway.yaml
08-istio-virtualservice.yaml
09-istio-destinationrule.yaml
kustomization.yaml

scripts/

deploy-dev.sh
deploy-staging.sh
deploy-prod.sh
promote-staging.sh
promote-prod.sh

04-high-availability-disaster-recovery
kubernetes/

01-kubeadm-ha-config.yaml
02-etcd-cluster.yaml
03-loadbalancer-config.yaml
04-persistent-storage.yaml
05-backup-cronjob.yaml
06-monitoring-stack.yaml
07-cluster-autoscaler.yaml
08-pod-disruption-budget.yaml

istio/

01-istiod-ha.yaml
02-gateway-ha.yaml
03-circuit-breaker-config.yaml
04-retry-policies.yaml
05-outlier-detection.yaml
06-cross-cluster-config.yaml
07-mesh-expansion.yaml

backup/

etcd-backup.sh
etcd-restore.sh
velero-backup.yaml
velero-restore.yaml
disaster-recovery-plan.md

scripts/

setup-ha-cluster.sh
test-failover.sh
backup-cluster.sh
restore-cluster.sh
health-check.sh

05-security-hardening-compliance
kubernetes/

01-pod-security-standards.yaml
02-security-context.yaml
03-admission-controllers.yaml
04-rbac-least-privilege.yaml
05-network-policies-strict.yaml
06-secrets-encryption.yaml
07-audit-policy.yaml
08-service-account-tokens.yaml

istio/

01-strict-mtls-mesh.yaml
02-authorization-policies-strict.yaml
03-jwt-authentication.yaml
04-oauth-integration.yaml
05-security-monitoring.yaml
06-workload-identity.yaml
07-external-ca-config.yaml

security/

01-falco-config.yaml
02-opa-gatekeeper.yaml
03-cert-manager.yaml
04-vault-integration.yaml
05-security-scanning.yaml
06-compliance-checks.yaml

scripts/

security-scan.sh
compliance-check.sh
cert-rotation.sh
security-audit.sh
vulnerability-assessment.sh
