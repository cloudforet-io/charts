## On-premise Architecture
![On-premise Architecture](../images/on_premise_architecture.png)

## Configuration
[Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) is a resource that manages external access to the services in a cluster, typically HTTP.

You can create the ingress using the following steps.

### 1) Install Nginx Ingress Controller
Nginx Ingress Controller is required to automatically create the load balancer through the `kubernetes ingress`.
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/)
  
### 2) Prepare Certificates and Create Secret
First, you need to prepare the certificate for the following FQDN.
- `console.example.com`
- `*.api.example.com`
- `*.grpc.example.com` (optional)
- `webhook.example.com` (optional)

After that, download the [certificate_secret.yaml](../../examples/ingress/on_premise/certificate_secret.yaml) and replace the certificate and key with your own.

```bash
kubectl apply -f certificate_secret.yaml -n spaceone
```

### 3) Create Ingress
First, you need to download all the ingress files and replace the `hostname` with your own.

- Console: [console_ingress.yaml](../../examples/ingress/on_premise/console_ingress.yaml)
- REST API: [rest_api_ingress.yaml](../../examples/ingress/on_premise/rest_api_ingress.yaml)
- gRPC API: [grpc_api_ingress.yaml](../../examples/ingress/on_premise/grpc_api_ingress.yaml)
- Monitoring Webhook: [monitoring_webhook_ingress.yaml](../../examples/ingress/on_premise/monitoring_webhook_ingress.yaml)

```bash
kubectl apply -f console_ingress.yaml -n spaceone
kubectl apply -f console_api_ingress.yaml -n spaceone
kubectl apply -f grpc_api_ingress.yaml -n spaceone
kubectl apply -f monitoring_webhook_ingress.yaml -n spaceone
```

> `gRPC API` and `Monitoring Webhook` are optional. If you don't need it, you don't need to apply it.

### 4) Connect to the Console
You can access the console using the following URL.
- https://console.example.com