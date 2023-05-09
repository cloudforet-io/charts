## AWS Architecture


## Prerequisites
Before creating the ingress, you need to prepare for the following.
- EKS Cluster 
  - ALB Ingress Controller
  - External DNS
- Route53
- ACM

After that, create the ingress.
- Console: [console_ingress.yaml](console_ingress.yaml)
- gRPC: [grpc_ingress.yaml](grpc_ingress.yaml)
- Monitoring Webhook: [monitoring_webhook_ingress.yaml](monitoring_webhook_ingress.yaml)

```bash
kubectl apply -f console_ingress.yaml -n spaceone
kubectl apply -f grpc_ingress.yaml -n spaceone
kubectl apply -f monitoring_webhook_ingress.yaml -n spaceone
```