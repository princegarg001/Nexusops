# 🚀 Enterprise Multi-Cloud GitOps Platform with DevSecOps
## Part 2: Service Mesh, Security, Observability & Advanced Features

> ## 💚 AWS FREE TIER + LOCAL DEVELOPMENT VERSION
> **All components work on your local Kubernetes + AWS Free Tier services!**
> 
> | Component | Where It Runs | Cost |
> |-----------|---------------|------|
> | Istio, OPA, Trivy | Local Minikube | 💚 FREE |
> | Prometheus, Grafana | Local Minikube | 💚 FREE |
> | Container Images | AWS ECR | 💚 FREE (500MB) |
> | CI/CD Pipelines | GitHub Actions | 💚 FREE (2000 mins) |

---

## 📋 Table of Contents - Part 2

| Phase | Component | Cost | AWS Free Tier? |
|-------|-----------|------|----------------|
| 5 | [Istio Service Mesh](#phase-5-istio-service-mesh) | 💚 FREE | Local only |
| 6 | [DevSecOps Security](#phase-6-devsecops---security-scanning) | 💚 FREE | + ECR scanning |
| 7 | [Observability Stack](#phase-7-observability-stack) | 💚 FREE | + CloudWatch |
| 8 | [Autoscaling](#phase-8-autoscaling-configuration) | 💚 FREE | Local demo |
| 9 | [CI/CD Pipelines](#phase-9-cicd-pipelines) | 💚 FREE | ✅ Uses ECR |
| 10 | [Sample Applications](#phase-10-sample-applications) | 💚 FREE | ✅ Pushed to ECR |
| 11 | [Testing & Validation](#phase-11-testing--validation) | 💚 FREE | ✅ Yes |
| 12 | [Production Checklist](#phase-12-production-checklist) | - | Reference only |

---

## 🕸️ Phase 5: Istio Service Mesh

### 5.1 Istio Installation

```yaml
# service-mesh/istio-install/istio-operator.yaml

apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-control-plane
spec:
  profile: default
  
  # Global mesh configuration
  meshConfig:
    # Enable access logging
    accessLogFile: /dev/stdout
    accessLogFormat: |
      [%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%" "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"
    
    # Enable tracing
    enableTracing: true
    defaultConfig:
      tracing:
        sampling: 100.0
        zipkin:
          address: jaeger-collector.observability.svc:9411
      
      # Enable metrics
      proxyStatsMatcher:
        inclusionPrefixes:
          - "cluster.outbound"
          - "cluster.inbound"
        inclusionRegexps:
          - ".*circuit_breakers.*"
          
    # Outbound traffic policy
    outboundTrafficPolicy:
      mode: REGISTRY_ONLY
    
    # Enable auto mTLS
    enableAutoMtls: true
    
    # Trust domain for SPIFFE
    trustDomain: cluster.local

  components:
    # Ingress Gateway
    ingressGateways:
      - name: istio-ingressgateway
        enabled: true
        k8s:
          service:
            type: LoadBalancer
            ports:
              - port: 15021
                targetPort: 15021
                name: status-port
              - port: 80
                targetPort: 8080
                name: http2
              - port: 443
                targetPort: 8443
                name: https
              - port: 31400
                targetPort: 31400
                name: tcp
          hpaSpec:
            minReplicas: 2
            maxReplicas: 5
            metrics:
              - type: Resource
                resource:
                  name: cpu
                  targetAverageUtilization: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 2000m
              memory: 1024Mi
          
    # Egress Gateway
    egressGateways:
      - name: istio-egressgateway
        enabled: true
        k8s:
          hpaSpec:
            minReplicas: 1
            maxReplicas: 3

    # Istiod (Control Plane)
    pilot:
      enabled: true
      k8s:
        env:
          - name: PILOT_TRACE_SAMPLING
            value: "100"
        hpaSpec:
          minReplicas: 2
          maxReplicas: 5
          metrics:
            - type: Resource
              resource:
                name: cpu
                targetAverageUtilization: 80
        resources:
          requests:
            cpu: 500m
            memory: 2Gi

  values:
    global:
      # Multi-cluster mesh configuration
      meshID: multi-cloud-mesh
      multiCluster:
        clusterName: "" # Set per cluster
      network: "" # Set per cluster
      
      # Proxy configuration
      proxy:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 2000m
            memory: 1024Mi
        
        # Enable protocol sniffing
        protocolDetectionTimeout: 100ms
        
      # Enable Prometheus metrics
      proxy_init:
        resources:
          limits:
            cpu: 2000m
            memory: 1024Mi
          requests:
            cpu: 10m
            memory: 10Mi

    # Kiali dashboard
    kiali:
      enabled: true
      dashboard:
        auth:
          strategy: anonymous

    # Grafana
    grafana:
      enabled: true
      
    # Tracing
    tracing:
      enabled: true
      provider: jaeger
```

### 5.2 Install Istio - LOCAL VERSION (FREE) ⭐

> **Use this simplified installation for your local Minikube/Kind cluster!**

```powershell
# Step 1: Download and install Istio CLI
# Windows (PowerShell)
$ISTIO_VERSION = "1.20.0"
Invoke-WebRequest -Uri "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-win.zip" -OutFile "istio.zip"
Expand-Archive -Path "istio.zip" -DestinationPath "."
$env:PATH += ";$PWD\istio-$ISTIO_VERSION\bin"

# Or use Chocolatey (easier)
choco install istioctl -y

# Step 2: Verify your cluster is ready
istioctl x precheck

# Step 3: Install Istio with DEMO profile (includes all addons, perfect for learning)
istioctl install --set profile=demo -y

# Step 4: Verify installation
kubectl get pods -n istio-system

# Step 5: Enable automatic sidecar injection
kubectl label namespace default istio-injection=enabled
kubectl label namespace apps istio-injection=enabled

# Step 6: Install Kiali, Prometheus, Grafana, Jaeger addons
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/prometheus.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/grafana.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/jaeger.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml

# Step 7: Wait for everything to be ready
kubectl wait --for=condition=available deployment --all -n istio-system --timeout=300s

# Step 8: Access dashboards (open in browser)
# Kiali (Service Mesh visualization)
istioctl dashboard kiali

# Grafana (Metrics dashboards)  
istioctl dashboard grafana

# Jaeger (Distributed tracing)
istioctl dashboard jaeger
```

### 5.2-CLOUD Install Istio Script (For Future Cloud Deployment)

<details>
<summary>📦 Click to expand cloud Istio installation script</summary>

```bash
# scripts/install-istio.sh

#!/bin/bash

set -e

ISTIO_VERSION="1.20.0"

echo "📥 Downloading Istio ${ISTIO_VERSION}..."
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=${ISTIO_VERSION} sh -

cd istio-${ISTIO_VERSION}
export PATH=$PWD/bin:$PATH

echo "✅ Verifying Istio installation prerequisites..."
istioctl x precheck

echo "🚀 Installing Istio..."
istioctl install -f ../service-mesh/istio-install/istio-operator.yaml -y

echo "🏷️ Enabling automatic sidecar injection for default namespace..."
kubectl label namespace default istio-injection=enabled --overwrite

echo "📊 Installing Istio addons (Kiali, Prometheus, Grafana, Jaeger)..."
kubectl apply -f samples/addons/

echo "⏳ Waiting for Istio components to be ready..."
kubectl wait --for=condition=available --timeout=300s deployment --all -n istio-system

echo "✅ Istio installation complete!"
istioctl version
```

### 5.3 Gateway Configuration

```yaml
# service-mesh/gateway/gateway.yaml

apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: main-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
    # HTTP server (redirect to HTTPS)
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*.yourdomain.com"
      tls:
        httpsRedirect: true
        
    # HTTPS server
    - port:
        number: 443
        name: https
        protocol: HTTPS
      hosts:
        - "*.yourdomain.com"
      tls:
        mode: SIMPLE
        credentialName: wildcard-tls-secret
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: frontend-vs
  namespace: apps
spec:
  hosts:
    - "app.yourdomain.com"
  gateways:
    - istio-system/main-gateway
  http:
    - match:
        - uri:
            prefix: /api
      route:
        - destination:
            host: backend
            port:
              number: 8080
    - route:
        - destination:
            host: frontend
            port:
              number: 80
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: frontend-dr
  namespace: apps
spec:
  host: frontend
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: UPGRADE
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    loadBalancer:
      simple: ROUND_ROBIN
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

### 5.4 Traffic Management - Canary Release

```yaml
# service-mesh/traffic-management/canary-release.yaml

apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: backend-canary
  namespace: apps
spec:
  hosts:
    - backend
  http:
    # Canary routing based on header
    - match:
        - headers:
            x-canary:
              exact: "true"
      route:
        - destination:
            host: backend
            subset: canary
            
    # A/B testing based on user
    - match:
        - headers:
            x-user-group:
              exact: "beta"
      route:
        - destination:
            host: backend
            subset: canary
            
    # Weighted traffic split (90/10)
    - route:
        - destination:
            host: backend
            subset: stable
          weight: 90
        - destination:
            host: backend
            subset: canary
          weight: 10
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: backend-versions
  namespace: apps
spec:
  host: backend
  subsets:
    - name: stable
      labels:
        version: stable
    - name: canary
      labels:
        version: canary
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
        maxRequestsPerConnection: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 10s
      baseEjectionTime: 30s
      maxEjectionPercent: 100
```

### 5.5 Circuit Breaker Configuration

```yaml
# service-mesh/traffic-management/circuit-breaker.yaml

apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: circuit-breaker
  namespace: apps
spec:
  host: backend
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
        connectTimeout: 30s
      http:
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
        maxRequestsPerConnection: 10
        maxRetries: 3
        
    outlierDetection:
      # Evict hosts after 5 consecutive 5xx errors
      consecutive5xxErrors: 5
      # Check every 10 seconds
      interval: 10s
      # Eject for 30 seconds initially
      baseEjectionTime: 30s
      # Maximum 50% of hosts can be ejected
      maxEjectionPercent: 50
      # Minimum number of hosts to keep
      minHealthPercent: 30
      
    # Retry policy
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: gateway-error,connect-failure,refused-stream
---
# Timeout configuration
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: backend-timeout
  namespace: apps
spec:
  hosts:
    - backend
  http:
    - timeout: 10s
      retries:
        attempts: 3
        perTryTimeout: 3s
        retryOn: 5xx,reset,connect-failure
      route:
        - destination:
            host: backend
```

### 5.6 Zero-Trust Security (mTLS & Authorization)

```yaml
# service-mesh/security/peer-authentication.yaml

# Strict mTLS for entire mesh
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
---
# Namespace-specific mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: apps-mtls
  namespace: apps
spec:
  mtls:
    mode: STRICT
  portLevelMtls:
    8080:
      mode: STRICT
```

```yaml
# service-mesh/security/authorization-policy.yaml

# Deny all by default
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: apps
spec:
  {}
---
# Allow frontend to access backend
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: apps
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
    - from:
        - source:
            principals:
              - "cluster.local/ns/apps/sa/frontend"
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/*"]
---
# Allow backend to access database
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-backend-to-database
  namespace: apps
spec:
  selector:
    matchLabels:
      app: database
  action: ALLOW
  rules:
    - from:
        - source:
            principals:
              - "cluster.local/ns/apps/sa/backend"
      to:
        - operation:
            ports: ["5432"]
---
# Allow ingress gateway
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-ingress-gateway
  namespace: apps
spec:
  selector:
    matchLabels:
      app: frontend
  action: ALLOW
  rules:
    - from:
        - source:
            namespaces: ["istio-system"]
```

```yaml
# service-mesh/security/request-authentication.yaml

apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: apps
spec:
  selector:
    matchLabels:
      app: backend
  jwtRules:
    - issuer: "https://auth.yourdomain.com/"
      jwksUri: "https://auth.yourdomain.com/.well-known/jwks.json"
      audiences:
        - "api.yourdomain.com"
      forwardOriginalToken: true
      outputPayloadToHeader: x-jwt-payload
---
# Require JWT for specific paths
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: apps
spec:
  selector:
    matchLabels:
      app: backend
  action: ALLOW
  rules:
    - from:
        - source:
            requestPrincipals: ["https://auth.yourdomain.com/*"]
      to:
        - operation:
            paths: ["/api/protected/*"]
    - to:
        - operation:
            paths: ["/api/public/*", "/health", "/ready"]
```

---

## 🔐 Phase 6: DevSecOps - Security Scanning

### 6.1 Trivy Operator Installation

```yaml
# security/trivy/trivy-operator.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: trivy-system
  labels:
    app.kubernetes.io/name: trivy-operator
---
# Install via Helm
# helm repo add aqua https://aquasecurity.github.io/helm-charts/
# helm install trivy-operator aqua/trivy-operator \
#   --namespace trivy-system \
#   --set trivy.ignoreUnfixed=true \
#   --set operator.scanJobTimeout=10m
```

```yaml
# security/trivy/trivy-values.yaml

# Helm values for Trivy Operator

trivy:
  # Vulnerability database
  dbRepository: ghcr.io/aquasecurity/trivy-db
  
  # Ignore unfixed vulnerabilities
  ignoreUnfixed: true
  
  # Severity to report
  severity: UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL
  
  # Scan timeout
  timeout: 10m0s
  
  # Resources
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi

operator:
  # Scan job timeout
  scanJobTimeout: 10m
  
  # Concurrent scans
  scanJobsConcurrentLimit: 5
  
  # Scan on namespace creation
  vulnerabilityScannerEnabled: true
  configAuditScannerEnabled: true
  rbacAssessmentScannerEnabled: true
  infraAssessmentScannerEnabled: true
  clusterComplianceEnabled: true
  
  # Metrics
  metricsVulnerabilityId: true
  
  # Batch delete delay
  batchDeleteDelay: 10s
  batchDeleteLimit: 10

serviceMonitor:
  enabled: true
  interval: 60s

# Compliance reports
compliance:
  # NSA CISA Kubernetes Hardening Guide
  specs:
    - nsa
    - cis
```

### 6.2 OPA Gatekeeper Installation & Policies

```yaml
# security/opa-gatekeeper/gatekeeper-install.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: gatekeeper-system
---
# Install via Helm
# helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
# helm install gatekeeper gatekeeper/gatekeeper \
#   --namespace gatekeeper-system \
#   --set replicas=3 \
#   --set auditInterval=60
```

```yaml
# security/opa-gatekeeper/constraints/require-labels.yaml

# Constraint Template
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        
        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("Missing required labels: %v", [missing])
        }
---
# Constraint
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-app-labels
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
      - apiGroups: ["apps"]
        kinds: ["Deployment", "StatefulSet", "DaemonSet"]
    excludedNamespaces:
      - kube-system
      - gatekeeper-system
      - istio-system
  parameters:
    labels:
      - "app"
      - "version"
      - "environment"
```

```yaml
# security/opa-gatekeeper/constraints/container-limits.yaml

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8scontainerlimits
spec:
  crd:
    spec:
      names:
        kind: K8sContainerLimits
      validation:
        openAPIV3Schema:
          type: object
          properties:
            cpu:
              type: string
            memory:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8scontainerlimits
        
        missing_limits[container] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
        }
        
        missing_limits[container] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
        }
        
        violation[{"msg": msg}] {
          container := missing_limits[_]
          msg := sprintf("Container <%v> does not have resource limits defined", [container.name])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sContainerLimits
metadata:
  name: container-must-have-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces:
      - kube-system
      - gatekeeper-system
```

```yaml
# security/opa-gatekeeper/constraints/allowed-repos.yaml

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sallowedrepos
spec:
  crd:
    spec:
      names:
        kind: K8sAllowedRepos
      validation:
        openAPIV3Schema:
          type: object
          properties:
            repos:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sallowedrepos
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          satisfied := [good | repo = input.parameters.repos[_] ; good = startswith(container.image, repo)]
          not any(satisfied)
          msg := sprintf("Container <%v> has an invalid image repo <%v>, allowed repos are %v", [container.name, container.image, input.parameters.repos])
        }
        
        violation[{"msg": msg}] {
          container := input.review.object.spec.initContainers[_]
          satisfied := [good | repo = input.parameters.repos[_] ; good = startswith(container.image, repo)]
          not any(satisfied)
          msg := sprintf("InitContainer <%v> has an invalid image repo <%v>, allowed repos are %v", [container.name, container.image, input.parameters.repos])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAllowedRepos
metadata:
  name: allowed-container-registries
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces:
      - kube-system
      - gatekeeper-system
      - istio-system
  parameters:
    repos:
      # AWS ECR
      - "123456789.dkr.ecr.us-east-1.amazonaws.com/"
      # Azure ACR
      - "multicloudgitopsacr.azurecr.io/"
      # GCP Artifact Registry
      - "us-central1-docker.pkg.dev/your-project/"
      # Docker Hub official images
      - "docker.io/library/"
```

```yaml
# security/opa-gatekeeper/constraints/no-privileged.yaml

apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8spsprivilegedcontainer
spec:
  crd:
    spec:
      names:
        kind: K8sPSPPrivilegedContainer
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8spsprivilegedcontainer
        
        violation[{"msg": msg, "details": {}}] {
          c := input.review.object.spec.containers[_]
          c.securityContext.privileged
          msg := sprintf("Privileged container is not allowed: %v, securityContext: %v", [c.name, c.securityContext])
        }
        
        violation[{"msg": msg, "details": {}}] {
          c := input.review.object.spec.initContainers[_]
          c.securityContext.privileged
          msg := sprintf("Privileged initContainer is not allowed: %v, securityContext: %v", [c.name, c.securityContext])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPPrivilegedContainer
metadata:
  name: no-privileged-containers
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces:
      - kube-system
      - istio-system
```

### 6.3 Falco Runtime Security

```yaml
# security/falco/falco-daemonset.yaml

# Install via Helm
# helm repo add falcosecurity https://falcosecurity.github.io/charts
# helm install falco falcosecurity/falco \
#   --namespace falco \
#   --create-namespace \
#   -f falco-values.yaml
```

```yaml
# security/falco/falco-values.yaml

# Falco configuration
falco:
  jsonOutput: true
  jsonIncludeOutputProperty: true
  httpOutput:
    enabled: true
    url: "http://falco-sidekick.falco.svc:2801"

  # Custom rules
  rulesFile:
    - /etc/falco/falco_rules.yaml
    - /etc/falco/falco_rules.local.yaml
    - /etc/falco/k8s_audit_rules.yaml
    - /etc/falco/rules.d

# Falco Sidekick for alerting
falcosidekick:
  enabled: true
  
  config:
    slack:
      webhookurl: "https://hooks.slack.com/services/XXX/YYY/ZZZ"
      outputformat: "all"
      minimumpriority: "warning"
      
    prometheus:
      enabled: true
      
    alertmanager:
      hostport: "alertmanager.observability.svc:9093"
      minimumpriority: "warning"

# Custom rules
customRules:
  custom-rules.yaml: |-
    - rule: Shell in Container
      desc: A shell was spawned in a container
      condition: >
        spawned_process and
        container and
        shell_procs and
        proc.tty != 0 and
        container_entrypoint
      output: >
        Shell spawned in container (user=%user.name container=%container.name shell=%proc.name parent=%proc.pname)
      priority: WARNING
      tags: [container, shell, mitre_execution]

    - rule: Sensitive File Access
      desc: A process accessed a sensitive file
      condition: >
        open_read and
        container and
        (fd.name startswith /etc/shadow or
         fd.name startswith /etc/passwd or
         fd.name startswith /etc/kubernetes or
         fd.name contains /serviceaccount/token)
      output: >
        Sensitive file accessed (user=%user.name file=%fd.name container=%container.name)
      priority: WARNING
      tags: [container, filesystem, sensitive_data]

    - rule: Crypto Mining Detection
      desc: Potential crypto mining activity detected
      condition: >
        spawned_process and
        container and
        (proc.name in (cryptonight, minerd, xmrig, cpuminer) or
         proc.cmdline contains "stratum+tcp" or
         proc.cmdline contains "pool.minergate")
      output: >
        Crypto mining detected (user=%user.name command=%proc.cmdline container=%container.name)
      priority: CRITICAL
      tags: [container, cryptomining]
```

### 6.4 HashiCorp Vault for Secrets Management

```yaml
# security/vault/vault-install.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: vault
---
# Install via Helm
# helm repo add hashicorp https://helm.releases.hashicorp.com
# helm install vault hashicorp/vault \
#   --namespace vault \
#   -f vault-values.yaml
```

```yaml
# security/vault/vault-values.yaml

global:
  enabled: true
  tlsDisable: false

server:
  image:
    repository: hashicorp/vault
    tag: "1.15.2"

  # HA configuration
  ha:
    enabled: true
    replicas: 3
    
    raft:
      enabled: true
      setNodeId: true
      
      config: |
        ui = true
        
        listener "tcp" {
          tls_disable = 0
          address = "[::]:8200"
          cluster_address = "[::]:8201"
          tls_cert_file = "/vault/userconfig/vault-server-tls/tls.crt"
          tls_key_file  = "/vault/userconfig/vault-server-tls/tls.key"
        }
        
        storage "raft" {
          path = "/vault/data"
        }
        
        service_registration "kubernetes" {}

  # Resources
  resources:
    requests:
      memory: 256Mi
      cpu: 250m
    limits:
      memory: 512Mi
      cpu: 500m

  # Audit logging
  auditStorage:
    enabled: true
    size: 10Gi

# Injector for sidecar injection
injector:
  enabled: true
  replicas: 2
  
  # Agent configuration
  agentDefaults:
    cpuRequest: "25m"
    cpuLimit: "500m"
    memRequest: "64Mi"
    memLimit: "128Mi"

# CSI Provider
csi:
  enabled: true

# UI
ui:
  enabled: true
  serviceType: ClusterIP
```

```yaml
# security/vault/secret-policies.yaml

# Kubernetes auth method configuration
# vault auth enable kubernetes
# vault write auth/kubernetes/config \
#   kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

# Application policy
# vault policy write app-policy - <<EOF
path "secret/data/apps/*" {
  capabilities = ["read"]
}

path "secret/metadata/apps/*" {
  capabilities = ["list"]
}

path "database/creds/app-role" {
  capabilities = ["read"]
}
# EOF

# vault write auth/kubernetes/role/app \
#   bound_service_account_names=app-sa \
#   bound_service_account_namespaces=apps \
#   policies=app-policy \
#   ttl=1h
```

---

## 📊 Phase 7: Observability Stack

### 7.0 Quick Local Installation (FREE) ⭐

> **For local Minikube/Kind clusters - use this simplified setup!**

```powershell
# Add Helm repositories
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Prometheus + Grafana stack (all-in-one)
helm install monitoring prometheus-community/kube-prometheus-stack `
  --namespace monitoring `
  --create-namespace `
  --set prometheus.prometheusSpec.retention=7d `
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=10Gi

# Wait for deployment
kubectl wait --for=condition=available deployment --all -n monitoring --timeout=300s

# Access Grafana dashboard
# Default credentials: admin / prom-operator
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80

# Open in browser: http://localhost:3000

# Access Prometheus UI
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090

# Open in browser: http://localhost:9090
```

### 7.0.1 Install Jaeger for Distributed Tracing (Local)

```powershell
# Install Jaeger Operator
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.51.0/jaeger-operator.yaml -n observability

# Wait for operator
kubectl wait --for=condition=available deployment/jaeger-operator -n observability --timeout=120s

# Create a simple Jaeger instance
@"
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: observability
spec:
  strategy: allInOne
  allInOne:
    image: jaegertracing/all-in-one:latest
"@ | kubectl apply -f -

# Access Jaeger UI
kubectl port-forward svc/jaeger-query -n observability 16686:16686

# Open in browser: http://localhost:16686
```

### 7.0.2 Install Loki for Log Aggregation (Local)

```powershell
# Install Loki stack (Loki + Promtail)
helm install loki grafana/loki-stack `
  --namespace monitoring `
  --set grafana.enabled=false `
  --set prometheus.enabled=false

# Logs are now available in Grafana!
# Add Loki as data source in Grafana → Connections → Data Sources → Add Loki
# URL: http://loki:3100
```

---

### 7.1 Prometheus Stack Installation (Cloud Version)

<details>
<summary>📦 Click to expand cloud Prometheus configuration</summary>

```yaml
# observability/prometheus/prometheus-operator.yaml

# Install via Helm
# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# helm install prometheus prometheus-community/kube-prometheus-stack \
#   --namespace observability \
#   --create-namespace \
#   -f prometheus-values.yaml
```

```yaml
# observability/prometheus/prometheus-values.yaml

# Prometheus configuration
prometheus:
  prometheusSpec:
    replicas: 2
    retention: 30d
    retentionSize: "50GB"
    
    # Storage
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: gp3
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 100Gi
    
    # Additional scrape configs
    additionalScrapeConfigs:
      - job_name: 'istio-mesh'
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
        metrics_path: /stats/prometheus
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_container_port_name]
            action: keep
            regex: '.*-envoy-prom'
    
    # Resources
    resources:
      requests:
        memory: 2Gi
        cpu: 500m
      limits:
        memory: 4Gi
        cpu: 2000m

    # Pod monitor selector
    podMonitorSelectorNilUsesHelmValues: false
    serviceMonitorSelectorNilUsesHelmValues: false

# Alertmanager configuration
alertmanager:
  alertmanagerSpec:
    replicas: 2
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: gp3
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi
              
  config:
    global:
      resolve_timeout: 5m
      slack_api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
      
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack-notifications'
      routes:
        - match:
            severity: critical
          receiver: 'pagerduty-critical'
        - match:
            severity: warning
          receiver: 'slack-notifications'
          
    receivers:
      - name: 'slack-notifications'
        slack_configs:
          - channel: '#alerts'
            send_resolved: true
            title: '{{ template "slack.default.title" . }}'
            text: '{{ template "slack.default.text" . }}'
            
      - name: 'pagerduty-critical'
        pagerduty_configs:
          - service_key: '<pagerduty-service-key>'
            severity: critical

# Grafana configuration
grafana:
  enabled: true
  replicas: 2
  
  adminPassword: "admin" # Change in production
  
  persistence:
    enabled: true
    size: 10Gi
    
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
        - name: 'default'
          orgId: 1
          folder: ''
          type: file
          disableDeletion: false
          editable: true
          options:
            path: /var/lib/grafana/dashboards/default
        - name: 'istio'
          orgId: 1
          folder: 'Istio'
          type: file
          options:
            path: /var/lib/grafana/dashboards/istio
        - name: 'kubernetes'
          orgId: 1
          folder: 'Kubernetes'
          type: file
          options:
            path: /var/lib/grafana/dashboards/kubernetes

  dashboards:
    default:
      node-exporter:
        gnetId: 1860
        revision: 27
        datasource: Prometheus
    kubernetes:
      k8s-cluster:
        gnetId: 7249
        revision: 1
        datasource: Prometheus
      k8s-resources-cluster:
        gnetId: 11802
        revision: 1
        datasource: Prometheus
    istio:
      istio-mesh:
        gnetId: 7639
        revision: 194
        datasource: Prometheus
      istio-service:
        gnetId: 7636
        revision: 194
        datasource: Prometheus

  sidecar:
    dashboards:
      enabled: true
      searchNamespace: ALL
    datasources:
      enabled: true

# Node exporter
nodeExporter:
  enabled: true

# Kube state metrics
kubeStateMetrics:
  enabled: true
```

### 7.2 Custom Alert Rules

```yaml
# observability/prometheus/prometheus-rules.yaml

apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-alerts
  namespace: observability
  labels:
    release: prometheus
spec:
  groups:
    - name: kubernetes.alerts
      rules:
        - alert: PodCrashLooping
          expr: rate(kube_pod_container_status_restarts_total[15m]) * 60 * 5 > 0
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping"
            description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} has restarted {{ $value }} times in the last 15 minutes"

        - alert: PodNotReady
          expr: sum by (namespace, pod) (kube_pod_status_phase{phase=~"Pending|Unknown"}) > 0
          for: 15m
          labels:
            severity: warning
          annotations:
            summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is not ready"
            
        - alert: HighMemoryUsage
          expr: (container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100 > 90
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "High memory usage in {{ $labels.namespace }}/{{ $labels.pod }}"
            description: "Memory usage is above 90% (current: {{ $value }}%)"

        - alert: HighCPUUsage
          expr: (rate(container_cpu_usage_seconds_total[5m]) / container_spec_cpu_quota * container_spec_cpu_period) * 100 > 90
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "High CPU usage in {{ $labels.namespace }}/{{ $labels.pod }}"
            
    - name: istio.alerts
      rules:
        - alert: IstioHighRequestLatency
          expr: histogram_quantile(0.99, sum(rate(istio_request_duration_milliseconds_bucket[5m])) by (le, destination_service_name)) > 1000
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "High request latency for {{ $labels.destination_service_name }}"
            description: "P99 latency is {{ $value }}ms"

        - alert: IstioHighErrorRate
          expr: sum(rate(istio_requests_total{response_code=~"5.*"}[5m])) by (destination_service_name) / sum(rate(istio_requests_total[5m])) by (destination_service_name) * 100 > 5
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate for {{ $labels.destination_service_name }}"
            description: "Error rate is {{ $value }}%"

    - name: security.alerts
      rules:
        - alert: TrivyCriticalVulnerability
          expr: trivy_vulnerability_id{severity="Critical"} > 0
          for: 1m
          labels:
            severity: critical
          annotations:
            summary: "Critical vulnerability found"
            description: "Critical vulnerability {{ $labels.vulnerability_id }} found in {{ $labels.resource_name }}"

        - alert: GatekeeperViolation
          expr: increase(gatekeeper_violations[5m]) > 0
          for: 1m
          labels:
            severity: warning
          annotations:
            summary: "OPA Gatekeeper policy violation"
            description: "Policy violations detected in namespace {{ $labels.namespace }}"
```

### 7.3 Jaeger Distributed Tracing

```yaml
# observability/jaeger/jaeger-operator.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: tracing
---
# Install Jaeger Operator
# kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.51.0/jaeger-operator.yaml -n observability
---
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: observability
spec:
  strategy: production
  
  collector:
    replicas: 2
    maxReplicas: 5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 512Mi
    options:
      collector:
        zipkin:
          host-port: ":9411"

  query:
    replicas: 2
    options:
      query:
        base-path: /jaeger
    metricsStorage:
      type: prometheus

  storage:
    type: elasticsearch
    options:
      es:
        server-urls: https://elasticsearch.observability.svc:9200
        index-prefix: jaeger
        tls:
          ca: /es/certificates/ca.crt
    secretName: jaeger-es-secret

  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - jaeger.yourdomain.com
```

### 7.4 Loki for Log Aggregation

```yaml
# observability/loki/loki-stack.yaml

# Install via Helm
# helm repo add grafana https://grafana.github.io/helm-charts
# helm install loki grafana/loki-stack \
#   --namespace observability \
#   -f loki-values.yaml
```

```yaml
# observability/loki/loki-values.yaml

loki:
  enabled: true
  
  config:
    auth_enabled: false
    
    ingester:
      chunk_idle_period: 3m
      chunk_block_size: 262144
      chunk_retain_period: 1m
      max_transfer_retries: 0
      lifecycler:
        ring:
          kvstore:
            store: inmemory
          replication_factor: 1
    
    limits_config:
      enforce_metric_name: false
      reject_old_samples: true
      reject_old_samples_max_age: 168h
      ingestion_rate_mb: 16
      ingestion_burst_size_mb: 24
    
    schema_config:
      configs:
        - from: 2020-10-24
          store: boltdb-shipper
          object_store: filesystem
          schema: v11
          index:
            prefix: index_
            period: 24h
    
    storage_config:
      boltdb_shipper:
        active_index_directory: /data/loki/boltdb-shipper-active
        cache_location: /data/loki/boltdb-shipper-cache
        cache_ttl: 24h
        shared_store: filesystem
      filesystem:
        directory: /data/loki/chunks
    
    chunk_store_config:
      max_look_back_period: 0s
    
    table_manager:
      retention_deletes_enabled: true
      retention_period: 720h

  persistence:
    enabled: true
    size: 50Gi

promtail:
  enabled: true
  
  config:
    snippets:
      extraRelabelConfigs:
        - action: replace
          source_labels:
            - __meta_kubernetes_pod_node_name
          target_label: node_name
        - action: replace
          source_labels:
            - __meta_kubernetes_namespace
          target_label: namespace
        - action: replace
          source_labels:
            - __meta_kubernetes_pod_name
          target_label: pod
        - action: replace
          source_labels:
            - __meta_kubernetes_pod_container_name
          target_label: container

grafana:
  enabled: false # Using kube-prometheus-stack Grafana
  
  # Add Loki as datasource in Grafana
  sidecar:
    datasources:
      enabled: true
```

### 7.5 Custom Grafana Dashboards

```json
// observability/grafana/dashboards/multi-cloud-overview.json

{
  "dashboard": {
    "title": "Multi-Cloud GitOps Overview",
    "uid": "multicloud-overview",
    "tags": ["kubernetes", "multi-cloud", "gitops"],
    "timezone": "browser",
    "panels": [
      {
        "title": "Cluster Health",
        "type": "stat",
        "gridPos": { "x": 0, "y": 0, "w": 8, "h": 4 },
        "targets": [
          {
            "expr": "sum(kube_node_status_condition{condition=\"Ready\",status=\"true\"}) by (cluster)",
            "legendFormat": "{{cluster}}"
          }
        ],
        "options": {
          "colorMode": "background",
          "graphMode": "none",
          "textMode": "value_and_name"
        }
      },
      {
        "title": "Pod Status by Cluster",
        "type": "piechart",
        "gridPos": { "x": 8, "y": 0, "w": 8, "h": 8 },
        "targets": [
          {
            "expr": "sum(kube_pod_status_phase) by (cluster, phase)",
            "legendFormat": "{{cluster}} - {{phase}}"
          }
        ]
      },
      {
        "title": "Request Rate by Service",
        "type": "timeseries",
        "gridPos": { "x": 0, "y": 8, "w": 12, "h": 8 },
        "targets": [
          {
            "expr": "sum(rate(istio_requests_total[5m])) by (destination_service_name)",
            "legendFormat": "{{destination_service_name}}"
          }
        ]
      },
      {
        "title": "Error Rate by Service",
        "type": "timeseries",
        "gridPos": { "x": 12, "y": 8, "w": 12, "h": 8 },
        "targets": [
          {
            "expr": "sum(rate(istio_requests_total{response_code=~\"5.*\"}[5m])) by (destination_service_name) / sum(rate(istio_requests_total[5m])) by (destination_service_name) * 100",
            "legendFormat": "{{destination_service_name}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "mode": "absolute",
              "steps": [
                { "color": "green", "value": null },
                { "color": "yellow", "value": 1 },
                { "color": "red", "value": 5 }
              ]
            }
          }
        }
      },
      {
        "title": "Security Vulnerabilities",
        "type": "stat",
        "gridPos": { "x": 0, "y": 16, "w": 6, "h": 4 },
        "targets": [
          {
            "expr": "sum(trivy_vulnerability_id{severity=\"Critical\"})",
            "legendFormat": "Critical"
          }
        ],
        "options": {
          "colorMode": "background",
          "textMode": "value"
        },
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "mode": "absolute",
              "steps": [
                { "color": "green", "value": 0 },
                { "color": "red", "value": 1 }
              ]
            }
          }
        }
      },
      {
        "title": "Argo CD Sync Status",
        "type": "table",
        "gridPos": { "x": 6, "y": 16, "w": 18, "h": 8 },
        "targets": [
          {
            "expr": "argocd_app_info",
            "format": "table",
            "instant": true
          }
        ],
        "transformations": [
          {
            "id": "organize",
            "options": {
              "includeByName": {
                "name": true,
                "namespace": true,
                "sync_status": true,
                "health_status": true,
                "dest_server": true
              }
            }
          }
        ]
      }
    ]
  }
}
```

---

## 📈 Phase 8: Autoscaling Configuration

### 8.1 Horizontal Pod Autoscaler (HPA)

```yaml
# kubernetes/apps/backend/hpa.yaml

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: apps
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  minReplicas: 3
  maxReplicas: 20
  
  metrics:
    # CPU-based scaling
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
          
    # Memory-based scaling
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
          
    # Custom metric: requests per second
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: 1000
          
    # Istio metric: request duration
    - type: External
      external:
        metric:
          name: istio_request_duration_milliseconds_bucket
          selector:
            matchLabels:
              destination_service_name: backend
        target:
          type: AverageValue
          averageValue: 200m
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
        - type: Pods
          value: 4
          periodSeconds: 60
      selectPolicy: Min
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
```

### 8.2 Vertical Pod Autoscaler (VPA)

```yaml
# kubernetes/apps/backend/vpa.yaml

apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-vpa
  namespace: apps
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: backend
  
  updatePolicy:
    updateMode: "Auto"
    
  resourcePolicy:
    containerPolicies:
      - containerName: backend
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 4
          memory: 8Gi
        controlledResources: ["cpu", "memory"]
        controlledValues: RequestsAndLimits
      - containerName: istio-proxy
        mode: "Off"
```

### 8.3 Cluster Autoscaler

```yaml
# kubernetes/base/cluster-autoscaler/cluster-autoscaler.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - name: cluster-autoscaler
          image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.28.0
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/multicloud-eks
            - --balance-similar-node-groups
            - --scale-down-enabled=true
            - --scale-down-delay-after-add=10m
            - --scale-down-unneeded-time=10m
            - --scale-down-utilization-threshold=0.5
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          env:
            - name: AWS_REGION
              value: us-east-1
```

### 8.4 KEDA Event-Driven Autoscaling

```yaml
# kubernetes/apps/backend/keda-scaler.yaml

apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: backend-scaledobject
  namespace: apps
spec:
  scaleTargetRef:
    name: backend
  pollingInterval: 15
  cooldownPeriod: 300
  minReplicaCount: 3
  maxReplicaCount: 50
  
  triggers:
    # Prometheus trigger
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.observability.svc:9090
        metricName: http_requests_total
        threshold: '1000'
        query: sum(rate(http_requests_total{service="backend"}[2m]))
        
    # RabbitMQ trigger
    - type: rabbitmq
      metadata:
        host: amqp://rabbitmq.apps.svc:5672
        queueName: backend-queue
        queueLength: '50'
        
    # CPU trigger as fallback
    - type: cpu
      metricType: Utilization
      metadata:
        value: "70"

  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
            - type: Percent
              value: 10
              periodSeconds: 60
```

---

## 🔄 Phase 9: CI/CD Pipelines

### 9.0 AWS ECR + GitHub Actions Pipeline (FREE TIER) ⭐

> **This pipeline pushes to AWS ECR (free tier) and can trigger Argo CD deployments!**

```yaml
# .github/workflows/build-and-push-ecr.yaml
# 💚 This uses AWS Free Tier (500MB ECR storage)

name: Build and Push to AWS ECR

on:
  push:
    branches: [main]
    paths:
      - 'applications/**'
  workflow_dispatch:

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: multicloud-gitops/app

jobs:
  build-and-push:
    name: Build & Push to ECR
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Build the Docker image
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG ./applications/demo-app
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest ./applications/demo-app
          
          # Push to ECR
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
          
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Update Kubernetes manifests
        run: |
          # Update image tag in k8s manifests for Argo CD to pick up
          sed -i "s|image:.*|image: ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}|g" kubernetes/apps/demo-app/deployment.yaml
          
      - name: Commit and push manifest changes
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add kubernetes/
          git diff --quiet && git diff --staged --quiet || git commit -m "Update image to ${{ github.sha }}"
          git push
```

### 9.0.1 Setting Up GitHub Secrets for AWS

```markdown
Go to your GitHub repository → Settings → Secrets and variables → Actions

Add these secrets:
┌─────────────────────────┬────────────────────────────────────────────┐
│ Secret Name             │ Value                                      │
├─────────────────────────┼────────────────────────────────────────────┤
│ AWS_ACCESS_KEY_ID       │ Your IAM user access key                   │
│ AWS_SECRET_ACCESS_KEY   │ Your IAM user secret key                   │
└─────────────────────────┴────────────────────────────────────────────┘

These are the keys you created in Phase 1.5.5 (github-actions-deployer user)
```

### 9.0.2 Pull Images from ECR to Local Kubernetes

```powershell
# On your local machine - authenticate Docker with ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com

# Create Kubernetes secret for ECR (so your local cluster can pull images)
kubectl create secret docker-registry ecr-secret \
  --docker-server=$(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) \
  -n apps

# Reference this secret in your deployments:
# spec:
#   imagePullSecrets:
#     - name: ecr-secret
```

---

### 9.1 GitHub Actions - Full CI Pipeline (Reference)

<details>
<summary>📦 Click to expand full CI pipeline with all checks</summary>

```yaml
# .github/workflows/ci-pipeline.yaml

name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  AZURE_REGION: eastus
  GCP_REGION: us-central1

jobs:
  # Job 1: Code Quality & Security Scan
  code-quality:
    name: Code Quality & Security
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci
        working-directory: ./applications/demo-frontend

      - name: Run linting
        run: npm run lint
        working-directory: ./applications/demo-frontend

      - name: Run unit tests
        run: npm test -- --coverage
        working-directory: ./applications/demo-frontend

      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      - name: Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  # Job 2: Build & Push Container Images
  build-images:
    name: Build Container Images
    needs: code-quality
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [demo-frontend, demo-backend, demo-api]
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: |
            ${{ secrets.AWS_ECR_REGISTRY }}/${{ matrix.app }}
            ${{ secrets.AZURE_ACR_REGISTRY }}/${{ matrix.app }}
            ${{ secrets.GCP_ARTIFACT_REGISTRY }}/${{ matrix.app }}
          tags: |
            type=sha,prefix=
            type=ref,event=branch
            type=semver,pattern={{version}}

      # AWS ECR Login
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # Azure ACR Login
      - name: Login to Azure ACR
        uses: azure/docker-login@v1
        with:
          login-server: ${{ secrets.AZURE_ACR_REGISTRY }}
          username: ${{ secrets.AZURE_ACR_USERNAME }}
          password: ${{ secrets.AZURE_ACR_PASSWORD }}

      # GCP Artifact Registry Login
      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Login to GCP Artifact Registry
        run: gcloud auth configure-docker ${{ env.GCP_REGION }}-docker.pkg.dev

      # Build and push
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: ./applications/${{ matrix.app }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # Job 3: Container Security Scan
  security-scan:
    name: Container Security Scan
    needs: build-images
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [demo-frontend, demo-backend, demo-api]
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ secrets.AWS_ECR_REGISTRY }}/${{ matrix.app }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          ignore-unfixed: true

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # Job 4: Infrastructure Validation
  validate-infrastructure:
    name: Validate Infrastructure
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Terraform Format Check
        run: terraform fmt -check -recursive
        working-directory: ./infrastructure

      - name: Terraform Validate (AWS)
        run: |
          terraform init -backend=false
          terraform validate
        working-directory: ./infrastructure/aws/eks

      - name: Terraform Validate (Azure)
        run: |
          terraform init -backend=false
          terraform validate
        working-directory: ./infrastructure/azure/aks

      - name: Terraform Validate (GCP)
        run: |
          terraform init -backend=false
          terraform validate
        working-directory: ./infrastructure/gcp/gke

      - name: Run Checkov IaC Security Scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: infrastructure/
          framework: terraform
          output_format: sarif
          output_file_path: checkov-results.sarif

      - name: Upload Checkov results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: checkov-results.sarif

  # Job 5: Kubernetes Manifest Validation
  validate-manifests:
    name: Validate Kubernetes Manifests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install kubeval
        run: |
          wget https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz
          tar xf kubeval-linux-amd64.tar.gz
          sudo mv kubeval /usr/local/bin/

      - name: Validate Kubernetes manifests
        run: |
          find kubernetes/ -name '*.yaml' -type f | xargs kubeval --strict

      - name: Install Datree
        run: curl https://get.datree.io | /bin/bash

      - name: Run Datree policy check
        run: |
          datree test kubernetes/**/*.yaml --only-k8s-files

  # Job 6: Update GitOps Repository
  update-gitops:
    name: Update GitOps Manifests
    needs: [build-images, security-scan, validate-manifests]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITOPS_TOKEN }}

      - name: Update image tags in Kustomize
        run: |
          cd kubernetes/overlays/aws
          kustomize edit set image frontend=${{ secrets.AWS_ECR_REGISTRY }}/demo-frontend:${{ github.sha }}
          kustomize edit set image backend=${{ secrets.AWS_ECR_REGISTRY }}/demo-backend:${{ github.sha }}
          
          cd ../azure
          kustomize edit set image frontend=${{ secrets.AZURE_ACR_REGISTRY }}/demo-frontend:${{ github.sha }}
          kustomize edit set image backend=${{ secrets.AZURE_ACR_REGISTRY }}/demo-backend:${{ github.sha }}
          
          cd ../gcp
          kustomize edit set image frontend=${{ secrets.GCP_ARTIFACT_REGISTRY }}/demo-frontend:${{ github.sha }}
          kustomize edit set image backend=${{ secrets.GCP_ARTIFACT_REGISTRY }}/demo-backend:${{ github.sha }}

      - name: Commit and push changes
        run: |
          git config --global user.name 'GitHub Actions'
          git config --global user.email 'actions@github.com'
          git add -A
          git commit -m "chore: update image tags to ${{ github.sha }}" || exit 0
          git push

  # Job 7: Notify on completion
  notify:
    name: Send Notifications
    needs: [update-gitops]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Send Slack notification
        uses: slackapi/slack-github-action@v1.24.0
        with:
          payload: |
            {
              "text": "CI Pipeline completed for ${{ github.repository }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*CI Pipeline Status*\n• Repository: ${{ github.repository }}\n• Branch: ${{ github.ref_name }}\n• Commit: ${{ github.sha }}\n• Status: ${{ job.status }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 9.2 Infrastructure Deployment Pipeline

```yaml
# .github/workflows/infrastructure-deploy.yaml

name: Infrastructure Deployment

on:
  push:
    branches: [main]
    paths:
      - 'infrastructure/**'
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      cloud:
        description: 'Cloud provider'
        required: true
        type: choice
        options:
          - all
          - aws
          - azure
          - gcp

env:
  TF_VERSION: 1.6.0

jobs:
  terraform-plan:
    name: Terraform Plan
    runs-on: ubuntu-latest
    strategy:
      matrix:
        cloud: [aws, azure, gcp]
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials
        if: matrix.cloud == 'aws'
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Azure Login
        if: matrix.cloud == 'azure'
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: GCP Auth
        if: matrix.cloud == 'gcp'
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Terraform Init & Plan (VPC/VNet)
        run: |
          cd infrastructure/${{ matrix.cloud }}/vpc || cd infrastructure/${{ matrix.cloud }}/vnet
          terraform init
          terraform plan -out=tfplan
          
      - name: Terraform Init & Plan (Kubernetes)
        run: |
          cd infrastructure/${{ matrix.cloud }}/eks || cd infrastructure/${{ matrix.cloud }}/aks || cd infrastructure/${{ matrix.cloud }}/gke
          terraform init
          terraform plan -out=tfplan

      - name: Upload Plan
        uses: actions/upload-artifact@v4
        with:
          name: tfplan-${{ matrix.cloud }}
          path: infrastructure/${{ matrix.cloud }}/**/tfplan

  terraform-apply:
    name: Terraform Apply
    needs: terraform-plan
    runs-on: ubuntu-latest
    environment: production
    strategy:
      matrix:
        cloud: [aws, azure, gcp]
      max-parallel: 1
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Download Plan
        uses: actions/download-artifact@v4
        with:
          name: tfplan-${{ matrix.cloud }}
          path: infrastructure/${{ matrix.cloud }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure Credentials
        # Add cloud-specific credential configuration here
        run: echo "Configuring credentials for ${{ matrix.cloud }}"

      - name: Terraform Apply
        run: |
          cd infrastructure/${{ matrix.cloud }}/eks || cd infrastructure/${{ matrix.cloud }}/aks || cd infrastructure/${{ matrix.cloud }}/gke
          terraform init
          terraform apply -auto-approve tfplan
```

---

## 🚀 Phase 10: Sample Applications

### 10.1 Demo Frontend Application

```dockerfile
# applications/demo-frontend/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost/health || exit 1

USER nginx

CMD ["nginx", "-g", "daemon off;"]
```

```yaml
# applications/demo-frontend/k8s/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: apps
  labels:
    app: frontend
    version: stable
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        version: stable
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "80"
    spec:
      serviceAccountName: frontend
      securityContext:
        runAsNonRoot: true
        runAsUser: 101
        fsGroup: 101
      containers:
        - name: frontend
          image: frontend:latest
          ports:
            - containerPort: 80
              name: http
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /var/cache/nginx
      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir: {}
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - frontend
                topologyKey: kubernetes.io/hostname
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: apps
  labels:
    app: frontend
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
      name: http
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: frontend
  namespace: apps
```

### 10.2 Demo Backend Application

```dockerfile
# applications/demo-backend/Dockerfile

FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

FROM gcr.io/distroless/static:nonroot

WORKDIR /app

COPY --from=builder /app/main .

EXPOSE 8080

USER nonroot:nonroot

ENTRYPOINT ["/app/main"]
```

```go
// applications/demo-backend/main.go

package main

import (
	"encoding/json"
	"log"
	"net/http"
	"os"
	"time"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
	httpRequestsTotal = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Total number of HTTP requests",
		},
		[]string{"method", "path", "status"},
	)

	httpRequestDuration = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Name:    "http_request_duration_seconds",
			Help:    "HTTP request duration in seconds",
			Buckets: prometheus.DefBuckets,
		},
		[]string{"method", "path"},
	)
)

func init() {
	prometheus.MustRegister(httpRequestsTotal)
	prometheus.MustRegister(httpRequestDuration)
}

func main() {
	http.HandleFunc("/health", healthHandler)
	http.HandleFunc("/ready", readyHandler)
	http.HandleFunc("/api/data", metricsMiddleware(dataHandler))
	http.Handle("/metrics", promhttp.Handler())

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}

	log.Printf("Starting server on port %s", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(map[string]string{"status": "healthy"})
}

func readyHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(map[string]string{"status": "ready"})
}

func dataHandler(w http.ResponseWriter, r *http.Request) {
	data := map[string]interface{}{
		"message":   "Hello from backend",
		"timestamp": time.Now().UTC(),
		"version":   os.Getenv("APP_VERSION"),
		"cluster":   os.Getenv("CLUSTER_NAME"),
	}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(data)
}

func metricsMiddleware(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next(w, r)
		duration := time.Since(start).Seconds()

		httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, "200").Inc()
		httpRequestDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration)
	}
}
```

```yaml
# applications/demo-backend/k8s/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: apps
  labels:
    app: backend
    version: stable
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        version: stable
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: backend
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        fsGroup: 65532
      containers:
        - name: backend
          image: backend:latest
          ports:
            - containerPort: 8080
              name: http
          env:
            - name: APP_VERSION
              value: "1.0.0"
            - name: CLUSTER_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['cluster']
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - backend
              topologyKey: topology.kubernetes.io/zone
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: apps
spec:
  selector:
    app: backend
  ports:
    - port: 8080
      targetPort: 8080
      name: http
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backend
  namespace: apps
```

---

## ✅ Phase 11: Testing & Validation

### 11.1 Integration Tests

```yaml
# .github/workflows/integration-tests.yaml

name: Integration Tests

on:
  workflow_run:
    workflows: ["CI Pipeline"]
    types: [completed]
    branches: [main]

jobs:
  integration-tests:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubeconfig
        run: |
          aws eks update-kubeconfig --region us-east-1 --name multicloud-eks

      - name: Wait for deployment
        run: |
          kubectl wait --for=condition=available --timeout=300s deployment/frontend -n apps
          kubectl wait --for=condition=available --timeout=300s deployment/backend -n apps

      - name: Run integration tests
        run: |
          # Test frontend accessibility
          FRONTEND_URL=$(kubectl get svc frontend -n apps -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
          curl -f http://$FRONTEND_URL/health
          
          # Test backend API
          BACKEND_URL=$(kubectl get svc backend -n apps -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
          curl -f http://$BACKEND_URL:8080/health
          curl -f http://$BACKEND_URL:8080/api/data

      - name: Run load tests
        uses: grafana/k6-action@v0.3.1
        with:
          filename: tests/load-test.js
          flags: --out influxdb=http://influxdb.observability.svc:8086/k6
```

### 11.2 Load Test Script

```javascript
// tests/load-test.js

import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const latency = new Trend('latency');

export const options = {
  stages: [
    { duration: '2m', target: 50 },   // Ramp up
    { duration: '5m', target: 50 },   // Stay at 50 users
    { duration: '2m', target: 100 },  // Ramp up to 100
    { duration: '5m', target: 100 },  // Stay at 100 users
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests should be below 500ms
    errors: ['rate<0.01'],             // Error rate should be below 1%
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://app.yourdomain.com';

export default function () {
  // Test frontend
  let frontendRes = http.get(`${BASE_URL}/`);
  check(frontendRes, {
    'frontend status is 200': (r) => r.status === 200,
    'frontend response time < 500ms': (r) => r.timings.duration < 500,
  });
  errorRate.add(frontendRes.status !== 200);
  latency.add(frontendRes.timings.duration);

  // Test backend API
  let apiRes = http.get(`${BASE_URL}/api/data`);
  check(apiRes, {
    'api status is 200': (r) => r.status === 200,
    'api response time < 200ms': (r) => r.timings.duration < 200,
    'api returns valid JSON': (r) => {
      try {
        JSON.parse(r.body);
        return true;
      } catch (e) {
        return false;
      }
    },
  });
  errorRate.add(apiRes.status !== 200);
  latency.add(apiRes.timings.duration);

  sleep(1);
}
```

### 11.3 Chaos Engineering with Litmus

```yaml
# tests/chaos/pod-delete-experiment.yaml

apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: backend-chaos
  namespace: apps
spec:
  appinfo:
    appns: 'apps'
    applabel: 'app=backend'
    appkind: 'deployment'
  chaosServiceAccount: litmus-admin
  experiments:
    - name: pod-delete
      spec:
        components:
          env:
            - name: TOTAL_CHAOS_DURATION
              value: '30'
            - name: CHAOS_INTERVAL
              value: '10'
            - name: FORCE
              value: 'false'
            - name: PODS_AFFECTED_PERC
              value: '50'
```

---

## 📋 Phase 12: Production Checklist

### 12.1 Security Checklist

```markdown
## Security Checklist

### Network Security
- [ ] Network policies implemented (default deny)
- [ ] Pod-to-pod encryption (mTLS via Istio)
- [ ] Ingress TLS configured
- [ ] Egress traffic controlled

### Container Security
- [ ] Images scanned with Trivy
- [ ] No critical/high vulnerabilities
- [ ] Non-root containers
- [ ] Read-only root filesystem
- [ ] Resource limits defined
- [ ] Security contexts configured

### Access Control
- [ ] RBAC configured
- [ ] Service accounts with minimal permissions
- [ ] Secrets managed via Vault
- [ ] No hardcoded credentials
- [ ] API authentication enabled (JWT)

### Compliance
- [ ] OPA Gatekeeper policies active
- [ ] Audit logging enabled
- [ ] Falco runtime monitoring active
- [ ] Compliance reports generated (CIS/NSA)
```

### 12.2 Operations Runbook

```markdown
## Operations Runbook

### Daily Checks
1. Review Grafana dashboards for anomalies
2. Check Argo CD sync status
3. Review security scan results
4. Verify backup completion

### Incident Response
1. **High CPU/Memory Alert**
   - Check HPA status: `kubectl get hpa -A`
   - Review pod metrics: `kubectl top pods -A`
   - Scale if needed: `kubectl scale deployment/<name> --replicas=<n>`

2. **Service Degradation**
   - Check Istio traffic: `istioctl proxy-status`
   - Review circuit breaker: `istioctl x describe pod <pod>`
   - Check Jaeger for slow traces

3. **Security Alert**
   - Review Falco alerts
   - Check Trivy scan results
   - Isolate affected pods if needed

### Disaster Recovery
1. **Cluster Failure**
   - Traffic automatically routes to healthy clusters
   - Verify DNS failover
   - Monitor replacement cluster provisioning

2. **Data Recovery**
   - Restore from Velero backup
   - Verify data integrity
   - Update DNS if needed
```

### 12.3 Makefile for Common Operations

```makefile
# Makefile

.PHONY: all deploy clean test

# Variables
CLUSTER_AWS := multicloud-eks
CLUSTER_AZURE := multicloud-aks
CLUSTER_GCP := multicloud-gke

# Deploy infrastructure
deploy-infra:
	@echo "Deploying infrastructure..."
	cd infrastructure/aws/vpc && terraform apply -auto-approve
	cd infrastructure/aws/eks && terraform apply -auto-approve
	cd infrastructure/azure/vnet && terraform apply -auto-approve
	cd infrastructure/azure/aks && terraform apply -auto-approve
	cd infrastructure/gcp/vpc && terraform apply -auto-approve
	cd infrastructure/gcp/gke && terraform apply -auto-approve

# Install Argo CD
install-argocd:
	@echo "Installing Argo CD..."
	kubectl create namespace argocd --dry-run=client -o yaml | kubectl apply -f -
	helm repo add argo https://argoproj.github.io/argo-helm
	helm upgrade --install argocd argo/argo-cd -n argocd -f gitops/argocd/values.yaml

# Install Istio
install-istio:
	@echo "Installing Istio..."
	istioctl install -f service-mesh/istio-install/istio-operator.yaml -y
	kubectl label namespace apps istio-injection=enabled --overwrite

# Install observability stack
install-observability:
	@echo "Installing observability stack..."
	helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
	helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
		-n observability --create-namespace -f observability/prometheus/prometheus-values.yaml
	helm repo add grafana https://grafana.github.io/helm-charts
	helm upgrade --install loki grafana/loki-stack -n observability -f observability/loki/loki-values.yaml

# Install security tools
install-security:
	@echo "Installing security tools..."
	helm repo add aqua https://aquasecurity.github.io/helm-charts
	helm upgrade --install trivy-operator aqua/trivy-operator -n trivy-system --create-namespace
	helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
	helm upgrade --install gatekeeper gatekeeper/gatekeeper -n gatekeeper-system --create-namespace

# Deploy applications via GitOps
deploy-apps:
	@echo "Deploying applications via Argo CD..."
	kubectl apply -f gitops/applications/app-of-apps.yaml

# Run tests
test:
	@echo "Running tests..."
	kubectl apply -f tests/chaos/pod-delete-experiment.yaml
	k6 run tests/load-test.js

# Get cluster credentials
get-credentials:
	aws eks update-kubeconfig --region us-east-1 --name $(CLUSTER_AWS) --alias aws
	az aks get-credentials --resource-group multicloud-gitops-rg --name $(CLUSTER_AZURE) --context azure
	gcloud container clusters get-credentials $(CLUSTER_GCP) --region us-central1

# Clean up
clean:
	@echo "Cleaning up..."
	kubectl delete -f gitops/applications/app-of-apps.yaml
	helm uninstall argocd -n argocd
	helm uninstall prometheus -n observability
	cd infrastructure/aws/eks && terraform destroy -auto-approve
	cd infrastructure/azure/aks && terraform destroy -auto-approve
	cd infrastructure/gcp/gke && terraform destroy -auto-approve

# Status check
status:
	@echo "=== Cluster Status ==="
	kubectl get nodes -o wide
	@echo "\n=== Argo CD Status ==="
	argocd app list
	@echo "\n=== Istio Status ==="
	istioctl proxy-status
	@echo "\n=== Pod Status ==="
	kubectl get pods -A | grep -v Running | grep -v Completed
```

---

## 🔧 Troubleshooting Guide

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Pods stuck in Pending | Resource constraints | Check node capacity, add nodes |
| Istio sidecar not injecting | Namespace not labeled | `kubectl label ns <ns> istio-injection=enabled` |
| Argo CD sync failed | Git credentials expired | Update repository secret |
| High latency | Circuit breaker tripped | Review destination rules |
| OPA blocking deployments | Policy violation | Check constraint violations |
| Trivy scan failing | Database not updated | Restart trivy-operator |

### Useful Commands

```bash
# Debug Istio
istioctl analyze
istioctl proxy-config cluster <pod>.<namespace>
istioctl x describe pod <pod>

# Debug Argo CD
argocd app get <app-name>
argocd app sync <app-name> --prune

# Debug networking
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash

# Check security
kubectl get vulnerabilityreports -A
kubectl get constrainttemplates
kubectl get constraints

# Performance
kubectl top nodes
kubectl top pods -A --sort-by=memory
```

---

## 💰 Cost Optimization

### 🆓 Your Current Setup: AWS FREE TIER + LOCAL = $0!

> **Congratulations!** By using AWS Free Tier + local Kubernetes, you're getting real cloud experience for **$0**!

| Component | Where It Runs | Cloud Cost | Your Cost |
|-----------|---------------|------------|-----------|
| Kubernetes Cluster | Local (Minikube) | $200-400/mo | 💚 **FREE** |
| Container Registry | AWS ECR | $10-50/mo | 💚 **FREE** (500MB) |
| Storage/State | AWS S3 | $5-25/mo | 💚 **FREE** (5GB) |
| CI/CD | GitHub Actions | $0-100/mo | 💚 **FREE** (2000 mins) |
| Monitoring Stack | Local cluster | $50-200/mo | 💚 **FREE** |
| Service Mesh | Local cluster | Included | 💚 **FREE** |
| **Total/month** | | **$325-907** | **$0** |

### AWS Free Tier Services You're Using

| Service | Free Tier Limit | Your Usage | Status |
|---------|-----------------|------------|--------|
| ECR | 500 MB/month | ~50-100 MB | ✅ Well under limit |
| S3 | 5 GB storage | ~10-50 MB | ✅ Well under limit |
| IAM | Unlimited | 1-2 users | ✅ Always free |
| CloudWatch | 10 custom metrics | Optional | ✅ Free tier |

### ⚠️ What to Avoid (Costs Money!)

| Service | Monthly Cost | Recommendation |
|---------|--------------|----------------|
| EKS (Managed K8s) | ~$73/month | ❌ Use Minikube instead |
| EC2 (beyond t2.micro) | Varies | ❌ Not needed |
| NAT Gateway | ~$45/month | ❌ Avoid |
| Load Balancers | ~$20/month | ❌ Use local NodePort |
| RDS | ~$15+/month | ❌ Use local containers |

### 💡 When to Consider Paid Cloud

Only move to paid cloud when:
1. ✅ You've mastered local setup completely
2. ✅ You have a job/project requiring cloud
3. ✅ You've used up free tier credits ($300 AWS)
4. ✅ You need production-grade features

### Cloud Cost Estimates (For Future Reference)

<details>
<summary>📦 Click to see full cloud cost breakdown</summary>

| Component | AWS | Azure | GCP | Total |
|-----------|-----|-------|-----|-------|
| Kubernetes (3 nodes) | $200 | $180 | $190 | $570 |
| Load Balancers | $50 | $45 | $40 | $135 |
| Storage (100GB) | $25 | $20 | $22 | $67 |
| Data Transfer | $50 | $45 | $40 | $135 |
| **Total/month** | **$325** | **$290** | **$292** | **~$907** |

</details>

### Cost Saving Tips (For Future Cloud Use)

1. **Use Spot/Preemptible instances** for non-critical workloads (60-80% savings)
2. **Right-size nodes** using VPA recommendations
3. **Enable cluster autoscaler** to scale down during off-hours
4. **Use reserved instances** for baseline capacity
5. **Implement pod disruption budgets** to safely use spot instances
6. **Clean up unused resources** regularly
7. **Set billing alerts** at $1, $10, $50 thresholds

---

## 🎉 Conclusion

### What You've Achieved (AWS Free Tier + Local = $0!)

Congratulations! You now have a complete enterprise-grade GitOps platform with:

✅ **AWS Integration** (ECR for images, S3 for state)  
✅ **GitOps Deployments** (Argo CD with ApplicationSets)  
✅ **Service Mesh** (Istio with mTLS, traffic management)  
✅ **DevSecOps** (Trivy, OPA Gatekeeper, Falco, Vault)  
✅ **Observability** (Prometheus, Grafana, Jaeger, Loki)  
✅ **Autoscaling** (HPA, VPA, Cluster Autoscaler, KEDA)  
✅ **CI/CD Pipelines** (GitHub Actions with security scanning)

This platform demonstrates advanced DevOps practices and is ready for production deployment!

---

## 📚 Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/) ⭐ *For local setup*
- [Kind Documentation](https://kind.sigs.k8s.io/) ⭐ *Alternative local setup*
- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Istio Documentation](https://istio.io/latest/docs/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/)

---

## 🚀 Your Learning Path (Zero Cost)

```
✅ Week 1: Set up Minikube + basic Kubernetes
✅ Week 2: Install & configure Argo CD (GitOps)
✅ Week 3: Add Istio service mesh
✅ Week 4: Set up security scanning (Trivy, OPA)
✅ Week 5: Configure monitoring (Prometheus, Grafana)
✅ Week 6: Deploy sample apps & test everything

🎯 RESULT: Production-ready skills, $0 spent!

📈 NEXT STEPS (When ready):
   → Use $800 in free cloud credits
   → Deploy same configs to AWS/Azure/GCP
   → Experience real multi-cloud environment
```

---

**Happy Learning! 🚀 Remember: You're building real enterprise skills at zero cost!**
