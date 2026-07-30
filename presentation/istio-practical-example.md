---
marp: true
theme: default
paginate: true
header: 'Istio — Practical Examples'
footer: 'Simple guide for everyone'
style: |
  section { font-size: 26px; }
  h1 { color: #2563eb; }
  h2 { color: #1e40af; }
  code { background: #f1f5f9; font-size: 21px; }
  table { font-size: 21px; }
---

# Istio
## Practical Examples (Beginner Friendly)

**Understand service mesh with simple stories and real YAML**

---

# Who Is This For?

This guide is for you if:

- You heard of **Istio** but it feels complicated
- You run microservices on **Kubernetes**
- You want **routing, security, and observability** without rewriting apps

**You do NOT need to be a networking expert.**

---

# The Problem (Without Istio)

Imagine a **web shop** with 3 services:

```
Frontend  →  Orders  →  Payments
```

Without Istio, you must solve in app code:

- How do I send 10% traffic to a new version?
- How do I encrypt service-to-service traffic?
- How do I see which service is slow?
- How do I stop a broken service from cascading failures?

**Result:** every team rewrites the same networking logic.

---

# What Is a Service Mesh?

A **service mesh** is a dedicated layer for service-to-service communication.

It adds:

- Smart traffic routing
- mTLS encryption
- Retries / timeouts / circuit breaking
- Metrics, traces, and logs

**Without changing your application code.**

---

# What Is Istio?

**Istio** is a popular open-source service mesh for Kubernetes.

| Attribute | Detail |
|-----------|--------|
| **Type** | Service mesh |
| **Data plane** | Envoy proxies |
| **Control plane** | istiod |
| **Docs** | https://istio.io |

**One sentence:** Istio sits between your services and makes traffic smarter and safer.

---

# Real-Life Analogy

Think of an **airport**:

| Airport part | Istio part | Job |
|--------------|------------|-----|
| Control tower | **istiod** (control plane) | Decides rules and routes |
| Planes + staff | Your apps | Do business work |
| Air traffic radios | **Envoy sidecars** | Actually move traffic safely |
| Airport entrance | **Ingress Gateway** | Lets outside visitors in |

Apps fly. Istio manages the air traffic.

---

# Architecture in One Picture

```
                 ┌────────────────────┐
                 │      istiod        │
                 │  (control plane)   │
                 └─────────┬──────────┘
                           │ configures
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      ┌─────────┐     ┌─────────┐     ┌─────────┐
      │ Frontend│     │ Orders  │     │Payments │
      │ + Envoy │────▶│ + Envoy │────▶│ + Envoy │
      └─────────┘     └─────────┘     └─────────┘
           ▲
           │
     Ingress Gateway (from internet)
```

---

# Two Planes (Remember This)

| Plane | Components | Job |
|-------|------------|-----|
| **Control plane** | `istiod` | Config, certificates, service discovery |
| **Data plane** | Envoy proxies | Handle real request traffic |

- Control plane = **brain**
- Data plane = **hands**

Apps talk normally. Proxies intercept and apply rules.

---

# Sidecar Explained Simply

A **sidecar** is an extra container next to your app in the same Pod.

```
Pod
├── shop-app      (your code)
└── istio-proxy   (Envoy sidecar)
```

All inbound/outbound traffic goes through Envoy.

**Benefit:** add Istio features without rewriting the app.

---

# Example 1: Enable Mesh for a Namespace

Label a namespace so Istio injects sidecars automatically:

```bash
kubectl label namespace shop istio-injection=enabled
```

Then redeploy your apps:

```bash
kubectl apply -n shop -f frontend.yaml
kubectl apply -n shop -f orders.yaml
kubectl apply -n shop -f payments.yaml
```

Each Pod now gets an Envoy sidecar.

---

# Example 1: Check Sidecar Was Injected

```bash
kubectl get pods -n shop
```

You should see **2/2 Ready** (app + proxy):

```
NAME                         READY   STATUS
frontend-7d8f9b6c4-abc12     2/2     Running
orders-5c7d8e9f1-def34       2/2     Running
payments-6a1b2c3d4-ghi56     2/2     Running
```

`2/2` = sidecar is working.

---

# Core Istio Objects (Cheat Sheet)

| Object | Simple meaning |
|--------|----------------|
| **Gateway** | Door from outside into the mesh |
| **VirtualService** | Traffic routing rules (who goes where) |
| **DestinationRule** | Destination policies (subsets, load balancing) |
| **PeerAuthentication** | mTLS rules between services |
| **AuthorizationPolicy** | Who is allowed to call whom |
| **ServiceEntry** | Register external services |

---

# Example 2: Ingress Gateway (Let Traffic In)

```yaml
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: shop-gateway
  namespace: shop
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "shop.example.com"
```

**Plain English:** accept HTTP traffic for `shop.example.com`.

---

# Example 2: VirtualService Bound to Gateway

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: shop
  namespace: shop
spec:
  hosts:
  - "shop.example.com"
  gateways:
  - shop-gateway
  http:
  - route:
    - destination:
        host: frontend
        port:
          number: 80
```

**Flow:** Internet → Gateway → VirtualService → `frontend` service.

---

# Request Path Example

```
User browser
   │
   ▼
shop.example.com
   │
   ▼
Istio Ingress Gateway
   │
   ▼
VirtualService (route rules)
   │
   ▼
frontend Pod (+ Envoy)
   │
   ▼
orders Pod (+ Envoy)
```

Istio controls every hop.

---

# Example 3: Canary Release (10% New Version)

You have two versions of Orders:

- `v1` = stable
- `v2` = new candidate

Goal: send **90% to v1**, **10% to v2**.

---

# Example 3: DestinationRule (Define Subsets)

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: orders
  namespace: shop
spec:
  host: orders
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**Meaning:** group Pods by `version` label.

---

# Example 3: VirtualService (Split Traffic)

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: orders
  namespace: shop
spec:
  hosts:
  - orders
  http:
  - route:
    - destination:
        host: orders
        subset: v1
      weight: 90
    - destination:
        host: orders
        subset: v2
      weight: 10
```

**Result:** gradual rollout without changing app code.

---

# Example 4: A/B Test by Header

Send testers with a special header to `v2`:

```yaml
http:
- match:
  - headers:
      end-user:
        exact: qa-team
  route:
  - destination:
      host: orders
      subset: v2
- route:
  - destination:
      host: orders
      subset: v1
```

**Meaning:**
- header `end-user: qa-team` → v2
- everyone else → v1

---

# Example 5: Timeouts and Retries

```yaml
http:
- route:
  - destination:
      host: payments
      port:
        number: 8080
  timeout: 3s
  retries:
    attempts: 3
    perTryTimeout: 1s
    retryOn: 5xx,connect-failure,refused-stream
```

If Payments is slow/flaky:
- wait max 3 seconds
- retry up to 3 times

Protects users from temporary failures.

---

# Example 6: Circuit Breaker (Stop Cascades)

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: payments
  namespace: shop
spec:
  host: payments
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
```

If Payments fails repeatedly, Istio temporarily ejects bad pods.

---

# Security: mTLS in Simple Words

**mTLS** = mutual TLS

- Services prove identity to each other
- Traffic is encrypted in transit

Without Istio: hard to implement per service.
With Istio: certificates managed by `istiod` automatically.

---

# Example 7: Require mTLS in Namespace

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default
  namespace: shop
spec:
  mtls:
    mode: STRICT
```

**Modes:**
- `DISABLE` — no mTLS
- `PERMISSIVE` — accept both plain and mTLS (migration)
- `STRICT` — mTLS only (recommended after migration)

---

# Example 8: Who Can Call Whom

Only Frontend can call Orders:

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: orders-allow-frontend
  namespace: shop
spec:
  selector:
    matchLabels:
      app: orders
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "cluster.local/ns/shop/sa/frontend"
```

**Plain English:** deny unknown callers by default identity rules.

---

# Example 9: Observe Traffic (Why Teams Love Istio)

Istio sidecars generate:

- **Metrics** (request rate, errors, latency)
- **Traces** (request path across services)
- **Access logs**

Common tools:
- Prometheus + Grafana
- Kiali (service graph UI)
- Jaeger / Zipkin (tracing)

You can answer: "Where is latency coming from?"

---

# Example 10: External API (ServiceEntry)

Call Stripe (outside cluster) through the mesh:

```yaml
apiVersion: networking.istio.io/v1
kind: ServiceEntry
metadata:
  name: stripe
  namespace: shop
spec:
  hosts:
  - api.stripe.com
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

Now Istio can observe/control egress to Stripe.

---

# Ambient Mode (Optional Modern Path)

Istio also supports **ambient mode**:

| Mode | How it works | Best when |
|------|--------------|-----------|
| **Sidecar** | proxy in every Pod | full L7 features per workload |
| **Ambient** | node proxy (+ optional waypoint) | lower resource cost, gradual adoption |

For beginners, start with **sidecar mode** (most tutorials use it).

---

# Istio vs Ingress Controller vs API Gateway

| Tool | Main job |
|------|----------|
| **Kubernetes Ingress / Gateway API** | Get traffic into cluster |
| **API Gateway** | Auth, rate limit, API product features |
| **Istio** | Service-to-service mesh + edge gateway + security + observability |

They can work together. Istio focuses on **east-west** (service ↔ service) and can also handle north-south ingress.

---

# Practical Day Story

1. Deploy `orders:v2` with label `version: v2`
2. Add DestinationRule subsets
3. Route 10% traffic to v2
4. Watch Kiali / Grafana metrics
5. If healthy, move to 50%, then 100%
6. If errors rise, instantly set weight back to 100% v1

**No redeploy of callers. No app code change.**

---

# Common Mistakes (And Fixes)

| Mistake | Fix |
|---------|-----|
| Namespace not labeled for injection | `kubectl label ns shop istio-injection=enabled` |
| Pods show `1/1` not `2/2` | restart pods after enabling injection |
| VirtualService host mismatch | host must match Kubernetes Service name / Gateway host |
| Canary not working | check Pod labels match DestinationRule subsets |
| Apps can't talk after STRICT mTLS | migrate with `PERMISSIVE` first |

---

# Mini Lab (Try Yourself)

```bash
# 1) Install Istio (example)
istioctl install --set profile=demo -y

# 2) Enable injection
kubectl create namespace shop
kubectl label namespace shop istio-injection=enabled

# 3) Deploy sample app (Bookinfo is official demo)
kubectl apply -n shop -f samples/bookinfo/platform/kube/bookinfo.yaml

# 4) Open Gateway + VirtualService for Bookinfo
kubectl apply -n shop -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

Official getting started: https://istio.io/latest/docs/setup/getting-started/

---

# Cheat Sheet

```
istiod            = brain (control plane)
Envoy sidecar     = traffic handler next to app
Gateway           = entry door
VirtualService    = routing rules
DestinationRule   = subsets + destination policies
PeerAuthentication= mTLS mode
AuthorizationPolicy = allow/deny who can call
```

**Remember:** Istio adds networking superpowers without rewriting apps.

---

# Practice Questions

1. What is a sidecar?
2. VirtualService vs DestinationRule?
3. How do you send 10% traffic to v2?
4. What does `STRICT` mTLS mean?
5. Why do Pods become `2/2 Ready`?

---

# Answer Key

1. Extra Envoy container in the same Pod handling traffic
2. VirtualService = how to route; DestinationRule = destination policies/subsets
3. DestinationRule subsets + VirtualService weights 90/10
4. Only encrypted mutual TLS traffic is accepted
5. App container + Istio proxy container

---

# Final Summary

If you remember only 4 things:

1. **Istio is a service mesh** for Kubernetes traffic
2. **Sidecars** intercept traffic without code changes
3. **VirtualService + DestinationRule** control routing and canaries
4. **mTLS + policies** secure service-to-service calls

Start small: inject one namespace → add Gateway → try a 10% canary.

---

# Thank You

Next steps:
1. Run the Bookinfo demo
2. Practice canary routing
3. Enable `PERMISSIVE` then `STRICT` mTLS

Docs: https://istio.io/latest/docs/

Related presentations in this repo:
- `presentation/practical-example.md` (Kargo + Argo CD)
- `presentation/helm-practical-example.md` (Helm)
