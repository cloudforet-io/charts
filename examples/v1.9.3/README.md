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
- 1.9.3
- hotfix
  - console 1.9.3.2
  - console-api 1.9.3.1

### root-domain.yaml(Only when creating the initial root domain)
- [ADD] main.import.role.yaml
```diff
enabled: true
image:
    name: spaceone/spacectl
    version: 1.9.3
domain: root
main:
  import:
    - /root/spacectl/apply/root_domain.yaml 
    - /root/spacectl/apply/marketplace.yaml
+   - /root/spacectl/apply/role.yaml
    - /root/spacectl/apply/hyperbilling.yaml # If you have a hyperbilling account
  var:
    domain_name: root
    default_language: ko
    default_timezone: Asia/Seoul
    domain_owner:
      id: admin
      password: Admin123!@#
    user:
      id: root_api_key
    consul_server: spaceone-consul-server
    marketplace_endpoint: grpc://repository.portal.spaceone.dev:50051
    project_admin_policy_type: MANAGED
    project_admin_policy_id: policy-managed-project-admin
    project_viewer_policy_type: MANAGED
    project_viewer_policy_id: policy-managed-project-viewer
    domain_admin_policy_type: MANAGED
    domain_admin_policy_id: policy-managed-domain-admin
    domain_viewer_policy_type: MANAGED
    domain_viewer_policy_id: policy-managed-domain-viewer
  tasks: []
```

### user-domain.yaml(New)
```diff
+enabled: true
+image:
+    name: spaceone/spacectl
+    version: 1.9.3
+domain: user
+main:
+  import:
+    - /root/spacectl/apply/local_domain.yaml
+    - /root/spacectl/apply/statistics.yaml
+
+  var:
+    domain_name: spaceone
+    default_language: ko
+    default_timezone: Asia/Seoul
+    domain_owner: admin
+    domain_owner_password: Admin123!@#
+    project_admin_policy_type: MANAGED
+    project_admin_policy_id: policy-managed-project-admin
+    project_viewer_policy_type: MANAGED
+    project_viewer_policy_id: policy-managed-project-viewer
+    domain_admin_policy_type: MANAGED
+    domain_admin_policy_id: policy-managed-domain-admin
+    domain_viewer_policy_type: MANAGED
+    domain_viewer_policy_id: policy-managed-domain-viewer
+
+
+  tasks: []
```

### minikube.yaml
- [Modify] console.production_json.DOMAIN_NAME
```diff
console:
  enabled: true
  developer: false
  name: console
  replicas: 1
  image:
      name: spaceone/console
      version: 1.9.3
  imagePullPolicy: IfNotPresent

  production_json:
      CONSOLE_API:
        ENDPOINT: http://localhost:8081
+     DOMAIN_NAME: root
-     DOMAIN_NAME: spaceone
      DOMAIN_NAME_REF: localhost
```

## DB patch
- cost-analysis

```
use cost-analysis;
db.budget.find().forEach(function (item){
    var start = item.start.toISOString().substr(0,7); 
    var end = item.end.toISOString().substr(0,7);
    db.budget.updateOne({_id: item._id}, {$set: {start: start, end: end}})
})
```

## Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
