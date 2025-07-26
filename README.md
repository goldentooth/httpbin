# HTTPBin

HTTP testing service for the Goldentooth Kubernetes cluster, providing a comprehensive HTTP request & response testing tool built in Go.

## Overview

HTTPBin is a simple HTTP service that echoes back request data, making it invaluable for testing HTTP clients, debugging network issues, and validating load balancer configurations in the Goldentooth cluster.

## Purpose

### Testing & Validation
- **LoadBalancer Testing**: Verify MetalLB service allocation and routing
- **Network Connectivity**: Test ingress/egress rules and firewall configurations
- **HTTP Client Development**: Debug and validate HTTP client implementations
- **Service Mesh Validation**: Test service-to-service communication patterns

### Monitoring & Observability
- **Health Checks**: Simple endpoint for monitoring system health
- **Request Tracing**: Analyze request headers, timing, and routing
- **Load Testing**: Generate predictable responses for performance testing
- **DNS Validation**: Verify ExternalDNS and service discovery functionality

## Deployment

### Helm Chart
HTTPBin is deployed via a custom Helm chart with:
- **Namespace**: Dedicated namespace for isolation
- **Service Type**: LoadBalancer for external access via MetalLB
- **Resource Limits**: Appropriate CPU/memory constraints
- **Health Checks**: Liveness and readiness probes

### Kubernetes Resources

#### `Deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpbin
  template:
    spec:
      containers:
      - name: httpbin
        image: kennethreitz/httpbin:latest
        ports:
        - containerPort: 80
```

#### `Service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "httpbin.goldentooth.net"
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: httpbin
```

## Available Endpoints

### Request Inspection
- **`GET /get`**: Returns GET request data including headers, args, origin
- **`POST /post`**: Returns POST request data including form data and JSON
- **`PUT /put`**: Returns PUT request data
- **`DELETE /delete`**: Returns DELETE request data
- **`PATCH /patch`**: Returns PATCH request data

### Response Testing
- **`GET /status/{code}`**: Returns HTTP status code (200, 404, 500, etc.)
- **`GET /headers`**: Returns request headers as JSON
- **`GET /ip`**: Returns client IP address
- **`GET /user-agent`**: Returns client User-Agent string
- **`GET /json`**: Returns sample JSON data

### Delay & Streaming
- **`GET /delay/{seconds}`**: Returns response after specified delay
- **`GET /stream/{lines}`**: Stream N lines of JSON data
- **`GET /bytes/{count}`**: Generate N random bytes
- **`GET /uuid`**: Generate a UUID

### HTTP Authentication
- **`GET /basic-auth/{user}/{pass}`**: Basic authentication challenge
- **`GET /bearer`**: Bearer token authentication
- **`GET /digest-auth/{qop}/{user}/{pass}`**: Digest authentication

## Testing Examples

### Basic Connectivity
```bash
# Test basic HTTP response
curl https://httpbin.goldentooth.net/get

# Test POST with data
curl -X POST https://httpbin.goldentooth.net/post \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

### MetalLB LoadBalancer Testing
```bash
# Test external IP allocation
kubectl get svc httpbin -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Test multiple requests to verify load balancing
for i in {1..10}; do
  curl -s https://httpbin.goldentooth.net/ip
done
```

### Performance Testing
```bash
# Test response delays
curl https://httpbin.goldentooth.net/delay/5

# Generate load for testing
ab -n 1000 -c 10 https://httpbin.goldentooth.net/get
```

## Integration with Cluster Services

### ExternalDNS
HTTPBin automatically gets DNS records created via service annotations:
```yaml
annotations:
  external-dns.alpha.kubernetes.io/hostname: "httpbin.goldentooth.net"
```

### Prometheus Monitoring
HTTPBin endpoints are monitored for:
- **Response Times**: Latency measurements for performance monitoring
- **Error Rates**: HTTP 4xx/5xx response tracking
- **Request Volume**: Traffic patterns and usage analytics
- **Availability**: Uptime and health check status

### Service Mesh Testing
HTTPBin serves as a test target for:
- **Consul Connect**: Service mesh connectivity validation
- **Envoy Proxy**: Sidecar proxy configuration testing
- **TLS Termination**: Certificate and encryption validation
- **Circuit Breakers**: Fault tolerance testing

## Operational Use Cases

### Network Troubleshooting
```bash
# Test DNS resolution
nslookup httpbin.goldentooth.net

# Test connectivity from cluster nodes
kubectl run test-pod --image=curlimages/curl -it --rm -- \
  curl http://httpbin.httpbin.svc.cluster.local/get
```

### Load Balancer Validation
```bash
# Verify HAProxy backend configuration
curl -H "Host: httpbin.goldentooth.net" http://10.4.11.100/get

# Test SSL termination
curl -I https://httpbin.goldentooth.net/status/200
```

### Development Testing
HTTPBin provides a reliable testing endpoint for:
- **CI/CD Pipelines**: Automated testing of HTTP clients
- **Development Workflows**: Local testing against cluster services
- **Integration Tests**: End-to-end HTTP communication validation
- **Performance Benchmarks**: Baseline measurements for optimization

## Security Considerations

- **Public Access**: HTTPBin is intentionally exposed for testing purposes
- **Rate Limiting**: Consider implementing rate limits for production use
- **Data Sensitivity**: Avoid sending sensitive data to HTTPBin endpoints
- **Monitoring**: Log access patterns for security analysis
