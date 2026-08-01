# 📸 Observability & Verification Evidence Guide

This directory documents evidence of the fully configured and validated Istio service mesh (v1.24.3) running on Kubernetes.

---

## 📋 Evaluation Checklist & Proof Matrix

| # | Feature / Requirement | Verification Command | Expected Output | Evidence File |
|---|---|---|---|---|
| **1** | **Sidecar Injection** | `kubectl get pods -n default` | `2/2` READY containers for all pods | `pods-2-2-containers.png` |
| **2** | **Ingress Gateway** | `curl -s -I http://$GATEWAY_URL/productpage` | `HTTP/1.1 200 OK` | `gateway-http-200.png` |
| **3** | **Canary Deployment** | `kubectl get vs reviews -o yaml` | `weight: 90` (v1), `weight: 10` (v2) | `canary-traffic-split.png` |
| **4** | **Destination Rules** | `kubectl get destinationrule` | Subsets for `reviews`, `ratings`, `details`, `productpage` | `destination-rules.png` |
| **5** | **Strict mTLS** | `kubectl get peerauthentication` | `mode: STRICT` across `default` namespace | `mtls-strict-mode.png` |
| **6** | **Zero-Trust RBAC** | `kubectl get authorizationpolicy` | 4 ALLOW policies matching call graph principals | `zero-trust-rbac.png` |
| **7** | **Circuit Breaking** | `kubectl get dr ratings-destination -o yaml` | `outlierDetection` with 5 consecutive errors | `circuit-breaker-config.png` |
| **8** | **Timeouts & Retries** | `kubectl get vs ratings -o yaml` | `timeout: 2s`, `attempts: 3`, `perTryTimeout: 1s` | `timeouts-retries.png` |
| **9** | **Fault Injection** | `kubectl get vs ratings -o yaml` | 5s delay (50%), HTTP 500 abort (10%) | `fault-injection.png` |
| **10**| **Mesh Validation** | `istioctl analyze` | `✔ No validation issues found` | `istioctl-analyze-clean.png` |

---

## 🎨 Observability Dashboards

### 1. Kiali Topology Graph (`istioctl dashboard kiali`)
- **Namespace:** Select `default`
- **Display Options:** Enable *Traffic Animation*, *Security* (mTLS padlocks), and *Traffic Rate*.
- **Key Visuals:**
  - mTLS lock icons on all inter-service edges (`productpage` ➔ `details`, `productpage` ➔ `reviews`, `reviews` ➔ `ratings`).
  - Weighted traffic split (90% / 10%) on `reviews-v1` vs `reviews-v2`.

### 2. Jaeger Distributed Tracing (`istioctl dashboard jaeger`)
- **Service:** Select `productpage.default`
- **Operation:** Select `productpage.default:9080/productpage`
- **Key Visuals:**
  - Multi-span traces showing request propagation: `productpage` ➔ `reviews` ➔ `ratings` and `productpage` ➔ `details`.
  - Retry spans triggered by ratings fault injection (timeouts at 2000ms).

---

## 🔬 CLI Verification Snippets

### Verification 1: Strict mTLS Transport Encryption
```bash
istioctl x describe pod $(kubectl get pod -l app=productpage -o jsonpath='{.items[0].metadata.name}')
# Output confirms mTLS mode: STRICT
```

### Verification 2: Zero-Trust Access Control (RBAC)
```bash
kubectl get authorizationpolicy -n default
# Shows: productpage-viewer, details-viewer, reviews-viewer, ratings-viewer
```

### Verification 3: Complete Static Mesh Analysis
```bash
istioctl analyze --use-kube=false istio-configs/ app-manifests/
# Output: ✔ No validation issues found
```
