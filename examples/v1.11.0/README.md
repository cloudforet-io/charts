# Kubernetes namespace

~~~
kubectl create ns spaceone
kubectl create ns root-supervisor

alias kcd='kubectl config set-context $(kubectl config current-context) --namespace'
kcd spaceone
~~~

# Kubernetes role and role binding
- create rbac.yaml
~~~rbac.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: supervisor-plugin-control-role
  namespace: root-supervisor
rules:
- apiGroups:
  - ["*"]
  resources:
  - replicaSets
  - pods
  - deployments
  - services
  - endpoints
  verbs:
  - get
  - list
  - watch
  - create
  - delete
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: supervisor-role-binding
  namespace: root-supervisor
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: supervisor-plugin-control-role
subjects:
- kind: ServiceAccount
  name: default
  namespace: root-supervisor
~~~

- apply rbac.yaml
~~~
kubectl apply -f rbac.yaml
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
- 1.11.0
  - console: 1.11.0.7
  - console-api: 1.11.0.2
  - monitoring: 1.11.0.1
  - supervisor: 1.11.0.2
  - dashboard: 1.11.0.3
  - cost-analysis: 1.11.0.1
  - console-api-v2: 1.11.0.1
### new service
- [ADD] console-api-v2
```diff
+ console-api-v2:
+    enabled: true
+    scheduler: false
+    worker: false
+    rest: true
+    replicas: 4
+    image:
+      name: public.ecr.aws/megazone/spaceone/console-api-v2
+      version: 1.11.0

+    service:
+      rest:
+        type: ClusterIP
+        annotations:
+          nil: nil
+        ports:
+          - name: http
+            port: 80
+            targetPort: 80
+            protocol: TCP
+          - name: https
+            port: 443
+            targetPort: 80
+            nodePort: null
+            protocol: TCP

+    ingress:
+      rest:
+        enabled: true
+        annotations:
+          alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
+          alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
+          alb.ingress.kubernetes.io/inbound-cidrs: 0.0.0.0/0 # replace or leave out
+          alb.ingress.kubernetes.io/scheme: "internet-facing" # internet-facing
+          alb.ingress.kubernetes.io/target-type: instance # Your console-api-v2 should be NodePort for this configuration.
+          alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-northeast-2:111111111111:certificate/11111111-1111-111111111-111111111111
+          alb.ingress.kubernetes.io/healthcheck-path: "/check"
+          alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=60 # default 60
+          external-dns.alpha.kubernetes.io/hostname: "*.example.com"
+        servicePort: 80
+        path: /
```
- [ADD] dashboard
```diff
+ dashboard:
+    enabled: true
+    grpc: true
+    scheduler: false
+    worker: false
+    replicas: 4
+    image:
+      name: public.ecr.aws/megazone/spaceone/dashboard
+      version: 1.11.0.0
```

### value update
- [ADD] (Optional) console.production_json.DASHBOARD_ENABLED
```diff
  production_json:
    CONSOLE_API:
        ENDPOINT: https://console.api.example.com
    CONTACT_LINK: <enter_your_site_url_or_email> # Change the contact us link of sign in page
    DASHBOARD_ENABLED:
+   - <user_domain_id>
```

## DB Patch
- command
```
migrate.py 1.11.0 -f ./sample_migration_config.yml -d
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
