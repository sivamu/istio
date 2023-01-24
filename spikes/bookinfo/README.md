# Istio Session Demos

## Enable mTLS

```bash

# Deploy apps
kubectl create ns foo
kubectl apply -f <(istioctl kube-inject -f ./deploy/httpbin/httpbin.yaml) -n foo
kubectl apply -f <(istioctl kube-inject -f ./deploy/sleep/sleep.yaml) -n foo
kubectl create ns bar
kubectl apply -f <(istioctl kube-inject -f ./deploy/httpbin/httpbin.yaml) -n bar
kubectl apply -f <(istioctl kube-inject -f ./deploy/sleep/sleep.yaml) -n bar
kubectl create ns legacy
kubectl apply -f ./deploy/httpbin/httpbin.yaml -n legacy
kubectl apply -f ./deploy/sleep/sleep.yaml -n legacy

# Test connectivity between sidecars
for from in "foo" "bar" "legacy"; do for to in "foo" "bar" "legacy"; do kubectl exec "$(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name})" -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done

# Apply mTLS within foo namespace
kubectl apply -f ./deploy/mtls-config.yaml

# Test connectivity between sidecars
for from in "foo" "bar" "legacy"; do for to in "foo" "bar" "legacy"; do kubectl exec "$(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name})" -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done

# Delete mTLS rule
kubectl delete -f ./deploy/mtls-config.yaml

```

## AuthN and AuthZ using JWTs

```bash

# Create ingress for httpbin service
kubectl apply -f ./deploy/httpbin/httpbin-ingress.yaml

# Test httpbin service endpoint
curl "http://localhost:30000/headers" -s -o /dev/null -w "%{http_code}\n"

# Apply authentication (AuthN) policy in ingressgateway
kubectl apply -f ./deploy/jwt-authn-policy.yaml

# Use invalid bearer token
curl --header "Authorization: Bearer cafebabe" "http://localhost:30000/headers" -s -o /dev/null -w "%{http_code}\n"

# Use proper authN token
TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.16/security/tools/jwt/samples/demo.jwt -s)
curl --header "Authorization: Bearer $TOKEN" "http://localhost:30000/headers" -s -o /dev/null -w "%{http_code}\n"

# Apply authorization (AuthZ) policy in ingressgateway
# reject requests without a valid token, for the `/headers` path
kubectl apply -f ./deploy/jwt-authz-policy.yaml

# Test requests against /headers path
curl "http://localhost:30000/headers" -s -o /dev/null -w "%{http_code}\n"
curl --header "Authorization: Bearer $TOKEN" "http://localhost:30000/headers" -s -o /dev/null -w "%{http_code}\n"

# Test requests against /ip path
curl "http://localhost:30000/ip" -s -o /dev/null -w "%{http_code}\n"
curl --header "Authorization: Bearer $TOKEN" "http://localhost:30000/ip" -s -o /dev/null -w "%{http_code}\n"

```
