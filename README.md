# Cloudforet Helm Charts 
A Helm Chart for Cloudforet **`1.11.3`**.

## Prerequisites
- Kubernetes 1.21+
- Helm 3.2.0+
- MongoDB 5.0+ (optional)

> MongoDB is installed in the Kubernetes cluster by default. If you have an external MongoDB, you can use it.  

### Docker Images
You can download the docker images from [Docker Hub](https://hub.docker.com/u/spaceone).

| Image Name                  | Version             | Required |
|:----------------------------|:--------------------|:--------:|
| spaceone/console            | 1.11.0.27           |   True   |
| spaceone/console-api        | 1.11.0.3            |   True   |
| spaceone/identity           | 1.11.0              |   True   |
| spaceone/secret             | 1.11.0              |   True   |
| spaceone/repository         | 1.11.0              |   True   |
| spaceone/plugin             | 1.11.0              |   True   |
| spaceone/config             | 1.11.0              |   True   |
| spaceone/inventory          | 1.11.3              |   True   |
| spaceone/monitoring         | 1.11.3              |   True   |
| spaceone/statistics         | 1.11.0              |   True   |
| spaceone/cost-analysis      | 1.11.3              |   True   |
| spaceone/notification       | 1.11.0              |   True   |
| spaceone/board              | 1.11.0              |   True   |
| spaceone/file-manager       | 1.11.0              |   True   |
| spaceone/dashboard          | 1.11.4              |   True   |
| spaceone/console-api-v2     | 1.11.2              |   True   |
| spaceone/supervisor         | 1.11.0              |   True   |
| spaceone/spacectl           | 1.11.2              |   True   |
| spaceone/marketplace-assets | 1.11.0.1            |   True   |
| spaceone/docs               | 0.1.20230502.153847 |   True   |
| mongo                       | latest              |          |
| redis                       | latest              |          |

> Warning: If your Kubernetes cluster can't access the internet, you need to download the docker images and push them to your private docker registry.


## Installation
### 1) Add Helm Repository
```bash
helm repo add cloudforet https://cloudforet-io.github.io/charts
helm repo update
helm search repo
```

### 2) Create Namespaces
```bash
kubectl create ns spaceone
kubectl create ns spaceone-plugin
```
If you want to use only one namespace, you don't create the `spaceone-plugin` namespace.

### 3) Create Role and RoleBinding
First, download the [rbac.yaml](examples/rbac.yaml) file and execute the following command.

```bash
kubectl apply -f rbac.yaml -n spaceone-plugin
```

### 3) Install Cloudforet Chart
```bash
helm install cloudforet cloudforet/spaceone -n spaceone
```

After executing the above command, check the status of the pod.
```bash
kubectl get pod -n spaceone

NAME                                      READY   STATUS             RESTARTS      AGE
board-5746fd9657-vtd45                    1/1     Running            0             57s
config-5d4c4b7f58-z8k9q                   1/1     Running            0             58s
console-6b64cf66cb-q8v54                  1/1     Running            0             59s
console-api-7c95848cb8-sgt56              2/2     Running            0             58s
console-api-v2-rest-7d64bc85dd-987zn      2/2     Running            0             56s
cost-analysis-7b9d64b944-xw9qg            1/1     Running            0             59s
cost-analysis-scheduler-ff8cc758d-lfx4n   0/1     Error              3 (37s ago)   55s
cost-analysis-worker-559b4799b9-fxmxj     1/1     Running            0             58s
cost-analysis-worker-559b4799b9-nf5vs     1/1     Running            0             58s
cost-analysis-worker-559b4799b9-swzw8     1/1     Running            0             58s
cost-analysis-worker-559b4799b9-x8f4j     1/1     Running            0             58s
dashboard-b4cc996-mgwj9                   1/1     Running            0             56s
docs-5fb4cc56c7-68qbk                     1/1     Running            0             59s
identity-6fc984459d-zk8r9                 1/1     Running            0             56s
inventory-67498999d6-722bw                1/1     Running            0             57s
inventory-scheduler-5dc6856d44-4spvm      0/1     CrashLoopBackOff   3 (18s ago)   59s
inventory-worker-68d9fcf5fb-x6knb         1/1     Running            0             55s
marketplace-assets-8675d44557-ssm92       1/1     Running            0             59s
mongodb-7c9794854-cdmwj                   1/1     Running            0             59s
monitoring-fdd44bdbf-pcgln                1/1     Running            0             59s
notification-5b477f6c49-gzfl8             1/1     Running            0             59s
notification-scheduler-675696467-gn24j    1/1     Running            0             59s
notification-worker-d88bb6df6-pjtmn       1/1     Running            0             57s
plugin-556f7bc49b-qmwln                   1/1     Running            0             57s
plugin-scheduler-86c4c56d84-cmrmn         0/1     CrashLoopBackOff   3 (13s ago)   59s
plugin-worker-57986dfdd6-v9vqg            1/1     Running            0             58s
redis-75df77f7d4-lwvvw                    1/1     Running            0             59s
repository-5f5b7b5cdc-lnjkl               1/1     Running            0             57s
secret-77ffdf8c9d-48k46                   1/1     Running            0             55s
spacectl-5664788d5d-dtwpr                 1/1     Running            0             59s
statistics-67b77b6654-p9wcb               1/1     Running            0             56s
statistics-scheduler-586875947c-8zfqg     0/1     Error              3 (30s ago)   56s
statistics-worker-68d646fc7-knbdr         1/1     Running            0             58s
supervisor-scheduler-6744657cb6-tpf78     2/2     Running            0             59s
```

> Warning: Scheduler pods are in `CrashLoopBackOff` or `Error` state. This is because the setup is not complete.

### 4) Initialize the Configuration  
Reference: [spaceone-initializer](https://github.com/cloudforet-io/spaceone-initializer)

Check the above reference and choose one mode for your environment.
- [Default Mode (with Marketplace)](https://github.com/cloudforet-io/spaceone-initializer#default-mode-with-marketplace)
- [Local Mode (without Marketplace)](https://github.com/cloudforet-io/spaceone-initializer#local-mode-without-marketplace)

And copy the values to the `initializer.yaml` file and edit the values for your environment.

After that, execute the following command.
```bash
helm install initializer cloudforet/spaceone-initializer -n spaceone -f initializer.yaml
```

### 5) Set the Helm Values and Upgrade the Chart
Complete the initialization, you can get the system token from the initializer pod logs.
```bash
kubectl logs initializer-5f5b7b5cdc-lnjkl -n spaceone

...
TASK [Print Admin API Key] *********************************************************************************************
"{TOKEN}"

FINISHED [ ok=23, skipped=0 ] ******************************************************************************************

FINISH SPACEONE INITIALIZE
```

Create the `values.yaml` file and edit the values.
```yaml
console:
    production_json:
        CONSOLE_API:
            ENDPOINT: https://console.api.example.com       # Change the endpoint
        CONSOLE_API_V2:
            ENDPOINT: https://console-v2.api.example.com    # Change the endpoint

global:
    shared_conf:
        TOKEN: '{TOKEN}'                                    # Change the system token
```

For more advanced configuration, please refer the [documents](docs/parameters.md) and examples.
- [All Values](examples/values/all.yaml)
- [External Database](examples/values/external_db.yaml)
- [One Namespace](examples/values/one_namespace.yaml)


After that, execute the following command.
```bash
helm upgrade cloudforet cloudforet/spaceone -n spaceone -f values.yaml
kubectl delete po -n spaceone -l app.kubernetes.io/instance=cloudforet
```

### 6) Check the status of the pods
```bash
kubectl get pod -n spaceone
```

If all pods are in `Running` state, the setup is complete.

## Configure Ingress
After the installation, you need to configure the ingress to access the console.

### AWS Environment
Before creating the ingress, you need to prepare for the following.
- EKS Cluster 
  - ALB Ingress Controller
  - External DNS
- Route53
- ACM

After that, create the ingress.
- Console: [console_ingress.yaml](examples/ingress/aws/console_ingress.yaml)
- gRPC: [grpc_ingress.yaml](examples/ingress/aws/grpc_ingress.yaml)
- Monitoring Webhook: [monitoring_webhook_ingress.yaml](examples/ingress/aws/monitoring_webhook_ingress.yaml)

```bash
kubectl apply -f console_ingress.yaml -n spaceone
kubectl apply -f grpc_ingress.yaml -n spaceone
kubectl apply -f monitoring_webhook_ingress.yaml -n spaceone
```

### On-premise Environment
Before creating the ingress, you need to prepare for the following.
- Kubernetes Cluster
  - Nginx Ingress Controller
  - Cert-Manager (Optional)