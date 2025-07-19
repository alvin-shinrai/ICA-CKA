## ICA Structure

01-installation-upgrades-configuration-20%/

    01-istiooperator-basic.yaml
    02-istiooperator-custom.yaml
    03-istiooperator-ambient.yaml
    04-helm-values-basic.yaml
    05-helm-values-custom.yaml
    06-sidecar-injection-namespace.yaml
    07-ambient-mode-namespace.yaml
    install-istioctl.sh
    install-helm.sh
    upgrade-canary.sh
    upgrade-inplace.sh

02-traffic-management-35%/

    01-gateway-basic.yaml
    02-gateway-https.yaml
    03-gateway-multiple-hosts.yaml
    04-virtualservice-basic.yaml
    05-virtualservice-path-routing.yaml
    06-virtualservice-header-routing.yaml
    07-virtualservice-traffic-split.yaml
    08-virtualservice-canary.yaml
    09-virtualservice-fault-injection.yaml
    10-virtualservice-retry-timeout.yaml
    11-destinationrule-basic.yaml
    12-destinationrule-subsets.yaml
    13-destinationrule-circuit-breaker.yaml
    14-destinationrule-outlier-detection.yaml
    15-serviceentry-external-http.yaml
    16-serviceentry-external-https.yaml
    17-virtualservice-mirror.yaml
    18-virtualservice-external.yaml

03-securing-workloads-25%/

    01-peerauthentication-strict.yaml
    02-peerauthentication-permissive.yaml
    03-peerauthentication-service.yaml
    04-authorizationpolicy-deny-all.yaml
    05-authorizationpolicy-allow-specific.yaml
    06-authorizationpolicy-service-to-service.yaml
    07-authorizationpolicy-jwt.yaml
    08-requestauthentication-jwt.yaml
    09-requestauthentication-custom.yaml
    10-gateway-tls-simple.yaml
    11-gateway-tls-passthrough.yaml
    12-gateway-tls-mutual.yaml
    tls-secret.yaml
    jwt-secret.yaml

04-troubleshooting-20%/

    01-broken-gateway.yaml
    02-broken-virtualservice.yaml
    03-broken-destinationrule.yaml
    04-broken-authorizationpolicy.yaml
    05-broken-peerauthentication.yaml
    06-broken-serviceentry.yaml
    07-sidecar-injection-issue.yaml
    08-mtls-issue.yaml
    debug-commands.sh
    proxy-config-commands.sh

