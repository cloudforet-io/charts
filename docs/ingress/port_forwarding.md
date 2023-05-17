## Port Forwarding (No Ingress) Architecture


## Configuration
You can create the ingress using the following steps.

### 1) Change production_json in values.yaml
```yaml
console:
    production_json:
        CONSOLE_API:
            ENDPOINT: https://localhost:8081
        CONSOLE_API_V2:
            ENDPOINT: https://localhost:8082
```
After editing the `values.yaml` file, upgrade the helm chart.
```bash
helm upgrade cloudforet cloudforet/spaceone -n spaceone -f values.yaml
kubectl delete po -n spaceone -l app.kubernetes.io/name=console
```

### 2) Port Forwarding
Since the ingress is not used, you need to port forward the console, console-api and console-api-v2-rest services.

```bash
kubectl port-forward svc/console -n spaceone 8080:80 --address=0.0.0.0 &
kubectl port-forward svc/console-api -n spaceone 8081:80 --address=0.0.0.0 &
kubectl port-forward svc/console-api-v2-rest -n spaceone 8082:80 --address=0.0.0.0 &
```

### 3) Connect to the Console
You can access the console using the following URL.
- http://localhost:8080

### 4) Close Port Forwarding
```bash
kill $(ps aux | grep 'kubectl port-forward' | awk '{print $2}')
```
