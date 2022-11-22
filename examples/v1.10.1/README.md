# Kubernetes namespace

~~~
kubectl create ns spaceone
kubectl create ns root-supervisor

alias kcd='kubectl config set-context $(kubectl config current-context) --namespace'
kcd spaceone
~~~

# Helm repo

~~~
helm repo add spaceone https://spaceone-dev.github.io/charts
helm repo update
helm search repo
~~~

# Install

## Type 1. (for developer)
* update values.yaml
* update frontend.yaml
* MongoDB is a temporary POD deployment.

** frontend.yaml **
To exact configuration, update frontend.yaml file.

For the HTTPS connection, you have to prepare certificate a.k.a. AWS Certificate ARN.

If your domain is ***example.com***, you have to create certificate for ****.console.example.com*** and ***console-api.example.com***.


You have to update your domain settings.

| Component |	Key 				| Description |
| --- 		| --- 				| --- |
| console	| ENDPOINT 			| ENDPOINT of console-api FQDN |
| console	| host				| FQDN of User domains |
| console	| alb.ingress.kubernetes.io/certificate-arn |  Certificate ARN |
| console 	| external-dns.alpha.kubernetes.io/hostname | External DNS name of console	|
| console-api	| host				| Hostname of console-api |
| console-api	| alb.ingress.kubernetes.io/certificate-arn |  Certificate ARN |
| console-api	| external-dns.alpha.kubernetes.io/hostname | External DNS name of console-api	|

~~~
kcd spaceone

helm install spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

~~~


## Type 2. (for production)

For production level, install mongodb cluster or AWS document DB

If you use mongodb cluster,
host is "localhost" in database.yaml
Use TYPE 2. global varable in values.yaml

~~~
kcd spaceone

helm install spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone

~~~

# Upgrade
## Changed Configuration
### image versions
- 1.10.1
    - console: 1.10.1.4

### new service
- [ADD] board
```diff
+board:
+    enabled: true
+    grpc: true
+    scheduler: false
+    worker: false
+    replicas: 1
+    image:
+      name: public.ecr.aws/megazone/spaceone/board
+      version: 1.10.1
+
+    pod:
+      spec:
+        nodeSelector:
+          Category: core
```
- [ADD] file-manager
```diff
+file-manager:
+    enabled: true
+    grpc: true
+    replicas: 1
+    image:
+      name: public.ecr.aws/megazone/spaceone/file-manager
+      version: 1.10.1
+
+    volumeMounts:
+        application: []
+        application_worker: []
+        application_scheduler: []
+        application_rest: []
+
+    pod:
+        spec: {}
```

### value update
- [ADD] monitoring.application_grpc.INSTALLED_DATA_SOURCE_PLUGINS
```diff
+       - name: AWS CloudTrail
+         plugin_info:
+           plugin_id: plugin-aws-cloudtrail-mon-datasource
+           provider: aws
+       - name: Azure Activity Log
+         plugin_info:
+           plugin_id: plugin-azure-activity-log-mon-datasource
+           provider: azure
```

- [ADD] identity.application_grpc.ENDPOINTS
```diff
      ENDPOINTS:
      (..omit..)
+     - service: board
+       name: Board Service
+       endpoint: grpc+ssl://board.own.your.domain.com:443/v1
+     - service: file_manager
+       name: File Manager
+       endpoint: grpc+ssl://file-manager.own.your.domain.com:443/v1
```

- [ADD] identity.application_grpc.INTERNAL_ENDPOINTS
```diff
      INTERNAL_ENDPOINTS:
      (..omit..)
+     - service: board
+       name: Board Service
+       endpoint: grpc://board.spaceone.svc.cluster.local:50051/v1
+     - service: file_manager
+       name: File Manager
+       endpoint: grpc://file-manager.spaceone.svc.cluster.local:50051/v1
```

### DB Patch
- command 
```shell
migrate.py 1.10.1 -f ./sample_migration_config.yml -d
```
- document : https://github.com/cloudforet-io/db-migration

## Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
