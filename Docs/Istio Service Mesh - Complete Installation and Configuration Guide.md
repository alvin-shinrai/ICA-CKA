

## Table of Contents

1. [Istio Architecture and Components]
    - [Control Plane Components]
    - [Data Plane Components]
    - [Envoy Proxy Features]
2. [Installation Methods]
    - [Prerequisites]
    - [istioctl Installation]
    - [Helm Installation]
    - [Operator Installation]
    - [Configuration Profiles]
3. [Traffic Management]
    - [Gateway]
    - [VirtualService]
    - [DestinationRule]
    - [ServiceEntry]
    - [Sidecar]
    - [WorkloadEntry]
4. [Security]
    - [Mutual TLS (mTLS)]
    - [Authentication]
    - [Authorization]
    - [Security Policies]
5. [Observability]
    - [Metrics Collection]
    - [Distributed Tracing]
    - [Access Logs]
    - [Telemetry Configuration]
6. [Advanced Features]
    - [Multi-cluster Deployment]
    - [Workload Identity]
    - [External Services]
    - [Circuit Breaking]
    - [Fault Injection]
7. [Monitoring and Observability Tools]
    - [Kiali]
    - [Jaeger]
    - [Grafana]
    - [Prometheus]
8. [Troubleshooting]
    - [Common Issues]
    - [Debugging Commands]
    - [Proxy Configuration]
9. [Best Practices](https://claude.ai/chat/f7b0c796-da53-4dd6-9a84-90951d3e7420#best-practices)
    - [Production Deployment](https://claude.ai/chat/f7b0c796-da53-4dd6-9a84-90951d3e7420#production-deployment)
    - [Performance Tuning](https://claude.ai/chat/f7b0c796-da53-4dd6-9a84-90951d3e7420#performance-tuning)
    - [Security Hardening](https://claude.ai/chat/f7b0c796-da53-4dd6-9a84-90951d3e7420#security-hardening)

---

## Istio Architecture and Components

### Control Plane Components

Istio's control plane provides policy and configuration for the service mesh.

**Istiod**

- Unified control plane combining Pilot, Galley, and Citadel
- Service discovery and configuration
- Certificate management and distribution
- Sidecar injection

```bash
# Check Istiod status
kubectl get pods -n istio-system
kubectl logs -n istio-system deployment/istiod
```

### Data Plane Components

**Envoy Proxy**

- High-performance proxy deployed as sidecar
- Handles all network communication
- Implements traffic management, security, and observability

```bash
# Check sidecar injection
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].name}'

# Check Envoy configuration
kubectl exec -it <pod-name> -c istio-proxy -- pilot-agent request GET config_dump
```

### Envoy Proxy Features

- **Load Balancing**: Round-robin, least-request, random, ring-hash
- **Circuit Breaking**: Fail-fast mechanism
- **Health Checking**: Active and passive health checks
- **Timeouts and Retries**: Configurable timeout and retry policies
- **Rate Limiting**: Request rate limiting
- **TLS Termination**: SSL/TLS handling

---

## Installation Methods

### Prerequisites

**System Requirements:**

- Kubernetes cluster 1.22+
- 4 GB RAM minimum
- 2 CPU cores minimum
- Network connectivity between all nodes

**Pre-installation checks:**

```bash
# Check Kubernetes version
kubectl version --short

# Check cluster resources
kubectl top nodes
kubectl get nodes -o wide

# Check for existing Istio installation
kubectl get namespace istio-system
kubectl get crd | grep istio
```

### istioctl Installation

**Install istioctl:**

```bash
# Download and install istioctl
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0
export PATH=$PWD/bin:$PATH

# Verify installation
istioctl version --remote=false
```

**Install Istio:**

```bash
# Check pre-installation
istioctl x precheck

# Install with default profile
istioctl install --set values.defaultRevision=default

# Install with custom configuration
istioctl install --set values.pilot.traceSampling=100.0

# Verify installation
istioctl verify-install

# Check installation status
kubectl get pods -n istio-system
kubectl get svc -n istio-system
```

**Enable sidecar injection:**

```bash
# Label namespace for automatic injection
kubectl label namespace default istio-injection=enabled

# Verify injection
kubectl get namespace -L istio-injection

# Manual injection
istioctl kube-inject -f app.yaml | kubectl apply -f -
```

### Helm Installation

**Add Istio Helm repository:**

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

**Install Istio base:**

```bash
# Install Istio base (CRDs and cluster roles)
helm install istio-base istio/base -n istio-system --create-namespace

# Verify base installation
helm status istio-base -n istio-system
```

**Install Istiod:**

```bash
# Install Istiod
helm install istiod istio/istiod -n istio-system --wait

# Verify istiod installation
helm status istiod -n istio-system
kubectl get deployments -n istio-system --output wide
```

**Install Istio Gateway:**

```bash
# Install ingress gateway
helm install istio-ingress istio/gateway -n istio-ingress --create-namespace

# Custom gateway configuration
helm install istio-ingress istio/gateway -n istio-ingress \
  --create-namespace \
  --set service.type=NodePort \
  --set service.ports[0].port=80 \
  --set service.ports[0].name=http \
  --set service.ports[1].port=443 \
  --set service.ports[1].name=https
```

### Operator Installation

**Install Istio Operator:**

```bash
# Install operator
istioctl operator init

# Verify operator installation
kubectl get pods -n istio-operator
```

**Create IstioOperator resource:**

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: control-plane
  namespace: istio-system
spec:
  values:
    defaultRevision: "1-20-0"
    pilot:
      traceSampling: 100.0
      env:
        EXTERNAL_ISTIOD: false
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster1
      network: network1
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
        hpaSpec:
          maxReplicas: 5
          minReplicas: 1
          metrics:
          - type: Resource
            resource:
              name: cpu
              targetAverageUtilization: 80
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        service:
          type: NodePort
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
```

```bash
kubectl apply -f istio-operator.yaml
```

### Configuration Profiles

**Available profiles:**

```bash
# List available profiles
istioctl profile list

# View profile configuration
istioctl profile dump demo
istioctl profile dump minimal
istioctl profile dump production
```

**Profile descriptions:**

- **default**: Production deployment (recommended for production)
- **demo**: Demonstration with additional features enabled
- **minimal**: Minimal installation with only Istiod
- **external**: For external control plane deployments
- **empty**: No components deployed
- **preview**: Contains experimental features

**Custom profile installation:**

```bash
# Install with demo profile
istioctl install --set values.defaultRevision=default --set values.pilot.traceSampling=100.0

# Install with minimal profile
istioctl install --set values.defaultRevision=default --set components.pilot.k8s.resources.requests.memory=256Mi

# Install with custom values
istioctl install --set values.defaultRevision=default \
  --set values.pilot.traceSampling=100.0 \
  --set values.global.proxy.resources.requests.cpu=10m \
  --set values.global.proxy.resources.requests.memory=64Mi
```

---

## Traffic Management

### Gateway

Gateways manage inbound and outbound traffic for the mesh.

**Basic Gateway:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
  namespace: default
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - bookinfo.example.com
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: bookinfo-secret
    hosts:
    - bookinfo.example.com
```

**Multi-host Gateway:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: multi-host-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http-app1
      protocol: HTTP
    hosts:
    - app1.example.com
  - port:
      number: 80
      name: http-app2
      protocol: HTTP
    hosts:
    - app2.example.com
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: wildcard-secret
    hosts:
    - "*.example.com"
```

**TCP Gateway:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: tcp-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 31400
      name: tcp
      protocol: TCP
    hosts:
    - "*"
```

### VirtualService

VirtualServices define traffic routing rules.

**Basic HTTP routing:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - bookinfo.example.com
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

**Traffic splitting (Canary deployment):**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25
```

**Advanced routing with retry and timeout:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: gateway-error,connect-failure,refused-stream
```

**Fault injection:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
      abort:
        percentage:
          value: 0.1
        httpStatus: 500
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

### DestinationRule

DestinationRules configure traffic policies.

**Basic DestinationRule with subsets:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
      http:
        http1MaxPendingRequests: 10
        maxRequestsPerConnection: 2
    loadBalancer:
      simple: LEAST_CONN
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 20
```

**Circuit breaker configuration:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
```

**TLS configuration:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: external-service
spec:
  host: external-service.example.com
  trafficPolicy:
    tls:
      mode: SIMPLE
      sni: external-service.example.com
      caCertificates: /etc/ssl/certs/ca-bundle.crt
```

### ServiceEntry

ServiceEntry enables access to external services.

**External HTTP service:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-service
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http
    protocol: HTTP
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
```

**External database:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-database
spec:
  hosts:
  - database.example.com
  ports:
  - number: 3306
    name: mysql
    protocol: TCP
  location: MESH_EXTERNAL
  resolution: DNS
```

**Static IP external service:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: static-external
spec:
  hosts:
  - static-service.local
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: STATIC
  endpoints:
  - address: 192.168.1.100
  - address: 192.168.1.101
```

### Sidecar

Sidecar configurations optimize proxy resource usage.

**Default sidecar configuration:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Sidecar
metadata:
  name: default
  namespace: default
spec:
  egress:
  - hosts:
    - "./*"
    - "istio-system/*"
    - "production/*"
```

**Restricted sidecar configuration:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Sidecar
metadata:
  name: productpage-sidecar
  namespace: default
spec:
  workloadSelector:
    labels:
      app: productpage
  ingress:
  - port:
      number: 9080
      name: http
      protocol: HTTP
    defaultEndpoint: 127.0.0.1:9080
  egress:
  - port:
      number: 9080
      name: http
      protocol: HTTP
    hosts:
    - "./reviews.default.svc.cluster.local"
    - "./details.default.svc.cluster.local"
```

### WorkloadEntry

WorkloadEntry enables external workloads to join the mesh.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: WorkloadEntry
metadata:
  name: vm-workload
  namespace: default
spec:
  address: 192.168.1.10
  ports:
    http: 8080
    grpc: 9090
  labels:
    app: vm-app
    version: v1
```

---

## Security

### Mutual TLS (mTLS)

**Check mTLS status:**

```bash
# Check mesh-wide mTLS status
istioctl authn tls-check

# Check specific workload
istioctl authn tls-check productpage-v1-123456.default

# Verify mTLS configuration
istioctl proxy-config cluster productpage-v1-123456.default | grep reviews
```

**Mesh-wide mTLS policy:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

**Namespace-level mTLS:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: namespace-policy
  namespace: production
spec:
  mtls:
    mode: STRICT
```

**Workload-specific mTLS:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: productpage-peer-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  mtls:
    mode: STRICT
  portLevelMtls:
    9080:
      mode: DISABLE
```

### Authentication

**JWT authentication:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"
    audiences:
    - "bookinfo-api"
    forwardOriginalToken: true
```

**Multiple JWT issuers:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: multi-jwt
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  jwtRules:
  - issuer: "https://auth1.example.com"
    jwksUri: "https://auth1.example.com/.well-known/jwks.json"
  - issuer: "https://auth2.example.com"
    jwksUri: "https://auth2.example.com/.well-known/jwks.json"
    audiences:
    - "reviews-api"
```

### Authorization

**Allow all policy:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-all
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  rules:
  - {}
```

**Deny all policy:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  selector:
    matchLabels:
      app: ratings
```

**HTTP method-based authorization:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: reviews-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/productpage"]
    to:
    - operation:
        methods: ["GET"]
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/admin"]
    to:
    - operation:
        methods: ["GET", "POST", "PUT", "DELETE"]
```

**IP-based authorization:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: ip-allowlist
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  rules:
  - from:
    - source:
        ipBlocks: ["192.168.1.0/24", "10.0.0.0/8"]
  - to:
    - operation:
        paths: ["/health", "/status"]
```

**JWT claims-based authorization:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: jwt-claims-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  rules:
  - from:
    - source:
        requestPrincipals: ["https://auth.example.com/user-123"]
    when:
    - key: request.auth.claims[role]
      values: ["admin", "editor"]
  - from:
    - source:
        requestPrincipals: ["*"]
    to:
    - operation:
        methods: ["GET"]
    when:
    - key: request.auth.claims[role]
      values: ["viewer"]
```

### Security Policies

**Custom CA integration:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: custom-ca-policy
  namespace: production
spec:
  mtls:
    mode: STRICT
```

**Secret management for external CA:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cacerts
  namespace: istio-system
  labels:
    istio.io/key: true
type: Opaque
data:
  root-cert.pem: <base64-encoded-root-cert>
  cert-chain.pem: <base64-encoded-cert-chain>
  ca-cert.pem: <base64-encoded-ca-cert>
  ca-key.pem: <base64-encoded-ca-key>
```

---

## Observability

### Metrics Collection

**Enable Prometheus metrics:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: metrics
  namespace: istio-system
spec:
  metrics:
  - providers:
    - name: prometheus
  - overrides:
    - match:
        metric: ALL_METRICS
      disabled: false
    - match:
        metric: REQUEST_COUNT
      tagOverrides:
        destination_app:
          value: "{{.destination_app | default 'unknown'}}"
```

**Custom metrics:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: custom-metrics
  namespace: default
spec:
  metrics:
  - providers:
    - name: prometheus
  - overrides:
    - match:
        metric: REQUEST_COUNT
      tags:
        request_id: "%REQ(X-REQUEST-ID)%"
        user_agent: "%REQ(USER-AGENT)%"
    - match:
        metric: REQUEST_DURATION
      tags:
        response_code: "%RESPONSE_CODE%"
```

### Distributed Tracing

**Enable Jaeger tracing:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: tracing
  namespace: istio-system
spec:
  tracing:
  - providers:
    - name: jaeger
  - customTags:
      my_tag:
        literal:
          value: "custom_value"
      request_id:
        header:
          name: "x-request-id"
```

**Configure sampling rate:**

```bash
# Set sampling rate via MeshConfig
kubectl patch meshconfig default -n istio-system --type merge -p '{"spec":{"defaultConfig":{"tracing":{"sampling":100.0}}}}'

# Or via istioctl
istioctl install --set values.pilot.traceSampling=100.0
```

### Access Logs

**Enable access logs:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: access-logs
  namespace: istio-system
spec:
  accessLogging:
  - providers:
    - name: otel
```

**Custom access log format:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: custom-access-logs
  namespace: default
spec:
  accessLogging:
  - providers:
    - name: otel
  - format:
      text: |
        [%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%"
        %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT%
        %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%"
        "%REQ(USER-AGENT)%" "%REQ(X-REQUEST-ID)%"
        "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"
```

### Telemetry Configuration

**Disable telemetry for specific workloads:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: disable-health-checks
  namespace: default
spec:
  metrics:
  - providers:
    - name: prometheus
  - overrides:
    - match:
        metric: ALL_METRICS
        mode: CLIENT
      disabled: true
    - match:
        metric: ALL_METRICS
        mode: SERVER
      disabled: true
  selector:
    matchLabels:
      app: health-checker
```

**Resource-based telemetry:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: resource-telemetry
  namespace: production
spec:
  metrics:
  - providers:
    - name: prometheus
  - overrides:
    - match:
        metric: REQUEST_COUNT
      tags:
        source_workload: "%{SOURCE_WORKLOAD}"
        destination_service: "%{DESTINATION_SERVICE_NAME}"
        request_protocol: "%{REQUEST_PROTOCOL}"
```

---

## Advanced Features

### Multi-cluster Deployment

**Primary cluster setup:**

```bash
# Create cluster network
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
  labels:
    topology.istio.io/network: network1
EOF

# Install Istio with multi-cluster configuration
istioctl install --set values.pilot.env.EXTERNAL_ISTIOD=true
```

**Configure cross-cluster service discovery:**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istiod-gateway
  namespace: istio-system
spec:
  selector:
    istio: eastwestgateway
  servers:
  - port:
      number: 15010
      name: tls
      protocol: TLS
    tls:
      mode: PASSTHROUGH
    hosts:
    - "*"
---
apiVersion: v1
kind: Service
metadata:
  name: istiod-external
  namespace: istio-system
  labels:
    app: istiod
spec:
  type: LoadBalancer
  ports:
  - port: 15010
    name: tls
  selector:
    app: istiod
```

**Remote cluster setup:**

```bash
# Create namespace and secret
kubectl create namespace istio-system
kubectl create secret generic cacerts -n istio-system \
  --from-file=root-cert.pem \
  --from-file=cert-chain.pem \
  --from-file=ca-cert.pem \
  --from-file=ca-key.pem

# Install Istio on remote cluster
istioctl install --set values.istiodRemote.enabled=true \
  --set values.pilot.env.EXTERNAL_ISTIOD=true \
  --set values.global.meshID=mesh1 \
  --set values.global.remotePilotAddress=${DISCOVERY_ADDRESS}
```

### Workload Identity

**Configure workload identity:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: WorkloadGroup
metadata:
  name: vm-workloads
  namespace: default
spec:
  metadata:
    labels:
      app: vm-app
      version: v1
  template:
    ports:
      http: 8080
    serviceAccount: vm-service-account
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: vm-service-account
  namespace: default
```

### External Services

**Configure external service with circuit breaker:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-api
spec:
  hosts:
  - api.external.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: external-api-cb
spec:
  host: api.external.com
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
      http:
        http1MaxPendingRequests: 10
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
```

### Circuit Breaking

**Advanced circuit breaker configuration:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: circuit-breaker
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        http2MaxRequests: 1
        maxRequestsPerConnection: 1
        maxRetries: 3
        consecutiveGatewayErrors: 5
        interval: 10s
        baseEjectionTime: 30s
        maxEjectionPercent: 50
        minHealthPercent: 50
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
      minHealthPercent: 0
```

### Fault Injection

**Comprehensive fault injection:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: fault-injection
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
      abort:
        percentage:
          value: 0.1
        httpStatus: 500
    route:
    - destination:
        host: ratings
        subset: v1
  - match:
    - headers:
        test-user:
          regex: ".*"
    fault:
      delay:
        percentage:
          value: 1.0
        exponentialDelay: 2s
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

---

## Monitoring and Observability Tools

### Kiali

**Install Kiali:**

```bash
# Install Kiali operator
helm repo add kiali https://kiali.org/helm-charts
helm install kiali-operator kiali/kiali-operator -n kiali-operator --create-namespace

# Create Kiali CR
kubectl apply -f - <<EOF
apiVersion: kiali.io/v1alpha1
kind: Kiali
metadata:
  name: kiali
  namespace: istio-system
spec:
  installation_tag: "v1.75.0"
  istio_namespace: "istio-system"
  deployment:
    accessible_namespaces: ["**"]
    image_version: "v1.75.0"
  external_services:
    prometheus:
      url: "http://prometheus:9090"
    tracing:
      in_cluster_url: "http://jaeger-query:16686"
    grafana:
      in_cluster_url: "http://grafana:3000"
EOF
```

**Kiali configuration:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kiali
  namespace: istio-system
data:
  config.yaml: |
    auth:
      strategy: "anonymous"
    deployment:
      accessible_namespaces: ["**"]
      namespace: "istio-system"
    external_services:
      prometheus:
        url: "http://prometheus:9090"
      tracing:
        in_cluster_url: "http://jaeger-query:16686"
        use_grpc: true
      grafana:
        enabled: true
        in_cluster_url: "http://grafana:3000"
        url: "http://grafana.example.com"
```

### Jaeger

**Install Jaeger:**

```bash
# Install Jaeger operator
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.49.0/jaeger-operator.yaml -n observability

# Deploy Jaeger instance
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: istio-system
spec:
  strategy: production
  collector:
    maxReplicas: 5
    resources:
      limits:
        memory: "400Mi"
  storage:
    type: elasticsearch
    elasticsearch:
      nodeCount: 3
      storage:
        storageClassName: "fast-ssd"
        size: 10Gi
      redundancyPolicy: "SingleRedundancy"
  query:
    resources:
      limits:
        memory: "200Mi"
EOF
```

### Grafana

**Install Grafana:**

```bash
# Add Grafana Helm repository
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Grafana
helm install grafana grafana/grafana \
  -n istio-system \
  --set persistence.enabled=true \
  --set persistence.storageClassName="default" \
  --set persistence.size=10Gi \
  --set adminPassword='admin123'

# Get admin password
kubectl get secret --namespace istio-system grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

**Grafana Istio dashboards:**

```bash
# Import Istio dashboards
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/grafana.yaml
```

### Prometheus

**Install Prometheus:**

```bash
# Install Prometheus using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
  -n istio-system \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false
```

**Prometheus configuration for Istio:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: istio-system
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'istiod'
      kubernetes_sd_configs:
      - role: endpoints
        namespaces:
          names:
          - istio-system
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: istiod;http-monitoring
    - job_name: 'envoy-stats'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_container_port_name]
        action: keep
        regex: '.*-envoy-prom'
```

---

## Troubleshooting

### Common Issues

**Sidecar injection not working:**

```bash
# Check namespace label
kubectl get namespace -L istio-injection

# Check admission webhook
kubectl get mutatingwebhookconfiguration istio-sidecar-injector -o yaml

# Manual injection
istioctl kube-inject -f deployment.yaml | kubectl apply -f -

# Check webhook logs
kubectl logs -n istio-system deployment/istiod
```

**Service not accessible:**

```bash
# Check service endpoints
kubectl get endpoints

# Check DestinationRule
kubectl get destinationrule
kubectl describe destinationrule reviews

# Check VirtualService
kubectl get virtualservice
kubectl describe virtualservice reviews

# Test service resolution
kubectl exec -it productpage-v1-123456 -c istio-proxy -- curl reviews:9080
```

**mTLS issues:**

```bash
# Check mTLS status
istioctl authn tls-check productpage-v1-123456.default

# Check certificates
istioctl proxy-config secret productpage-v1-123456.default

# Check authentication policy
kubectl get peerauthentication
kubectl describe peerauthentication default
```

### Debugging Commands

**Proxy configuration:**

```bash
# Get all proxy config
istioctl proxy-config all productpage-v1-123456.default

# Get specific configurations
istioctl proxy-config cluster productpage-v1-123456.default
istioctl proxy-config listener productpage-v1-123456.default
istioctl proxy-config route productpage-v1-123456.default
istioctl proxy-config endpoint productpage-v1-123456.default

# Get bootstrap configuration
istioctl proxy-config bootstrap productpage-v1-123456.default

# Get secrets
istioctl proxy-config secret productpage-v1-123456.default
```

**Envoy admin interface:**

```bash
# Port forward to Envoy admin
kubectl port-forward productpage-v1-123456 15000:15000

# Access admin interface
curl http://localhost:15000/help
curl http://localhost:15000/config_dump
curl http://localhost:15000/clusters
curl http://localhost:15000/listeners
curl http://localhost:15000/stats
```

**Pilot debugging:**

```bash
# Check pilot logs
kubectl logs -n istio-system deployment/istiod

# Get pilot debug endpoints
kubectl port-forward -n istio-system deployment/istiod 8080:8080

# Debug endpoints
curl http://localhost:8080/debug/syncz
curl http://localhost:8080/debug/registryz
curl http://localhost:8080/debug/endpointz
curl http://localhost:8080/debug/configz
```

### Proxy Configuration

**Envoy filter for debugging:**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: debug-headers
  namespace: default
spec:
  workloadSelector:
    labels:
      app: productpage
  configPatches:
  - applyTo: HTTP_FILTER
    match:
      context: SIDECAR_INBOUND
      listener:
        filterChain:
          filter:
            name: envoy.filters.network.http_connection_manager
            subFilter:
              name: envoy.filters.http.router
    patch:
      operation: INSERT_BEFORE
      value:
        name: envoy.filters.http.wasm
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.http.wasm.v3.Wasm
          config:
            name: debug_headers
            root_id: debug_headers
            vm_config:
              vm_id: debug_headers
              runtime: envoy.wasm.runtime.null
              code:
                local:
                  inline_string: |
                    class RootContext : public Context {
                    public:
                      explicit RootContext(uint32_t id, RootContext* root) : Context(id, root) {}
                      FilterHeadersStatus onRequestHeaders(uint32_t, bool) override {
                        addRequestHeader("x-debug-request", "true");
                        return FilterHeadersStatus::Continue;
                      }
                    };
                    static RegisterContextFactory register_RootContext(CONTEXT_FACTORY(RootContext));
```

**Traffic analysis:**

```bash
# Analyze traffic with tcpdump
kubectl exec -it productpage-v1-123456 -c istio-proxy -- tcpdump -i any -n port 9080

# Get detailed proxy statistics
kubectl exec -it productpage-v1-123456 -c istio-proxy -- pilot-agent request GET stats/prometheus

# Check proxy health
kubectl exec -it productpage-v1-123456 -c istio-proxy -- pilot-agent request GET ready
```

---

## Best Practices

### Production Deployment

**Resource requests and limits:**

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: production
spec:
  values:
    global:
      proxy:
        resources:
          requests:
            cpu: 10m
            memory: 40Mi
          limits:
            cpu: 100m
            memory: 128Mi
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        hpaSpec:
          minReplicas: 2
          maxReplicas: 5
          metrics:
          - type: Resource
            resource:
              name: cpu
              targetAverageUtilization: 80
```

**Security hardening:**

```yaml
# Deny-all authorization policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: istio-system
spec: {}

---
# Strict mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

### Performance Tuning

**Optimize sidecar configuration:**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Sidecar
metadata:
  name: performance-optimization
  namespace: production
spec:
  egress:
  - hosts:
    - "./app1.production.svc.cluster.local"
    - "./app2.production.svc.cluster.local"
    - "istio-system/*"
```

**Disable unnecessary features:**

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: disable-access-logs
  namespace: production
spec:
  accessLogging:
  - providers:
    - name: otel
  - disabled: true
```

### Security Hardening

**Network policies:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: istio-system
  namespace: istio-system
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
  - from:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 15010
    - protocol: TCP
      port: 15011
    - protocol: TCP
      port: 8080
  egress:
  - {}
```

**Pod Security Standards:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
    istio-injection: enabled
```

