# Observability Evidence

This directory contains screenshots and proof of the working Istio service mesh.

## Required Screenshots

After deploying and generating traffic, capture the following:

1. **kiali-service-graph.png** — Kiali dashboard showing the service topology with mTLS padlock icons
2. **jaeger-distributed-trace.png** — Jaeger UI showing a trace spanning productpage → details → reviews → ratings
3. **canary-traffic-split.png** — Terminal output or Kiali graph showing the 90/10 traffic split on reviews
4. **pods-2-2-containers.png** — `kubectl get pods` output showing all pods with 2/2 containers
5. **mtls-verification.png** — Output of `istioctl x describe pod` showing STRICT mTLS

## How to Capture

```bash
# Open dashboards
istioctl dashboard kiali    # Screenshot the graph view
istioctl dashboard jaeger   # Screenshot a multi-service trace

# Terminal captures
kubectl get pods            # Screenshot 2/2 containers
istioctl proxy-status       # Screenshot all SYNCED proxies
```
