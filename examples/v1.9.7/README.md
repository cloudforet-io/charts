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
- 1.9.7
    - console: 1.10.0
    - config: 1.10.0
    - cost-analysis: 1.10.0
    - identity: 1.10.0
    - inventory: 1.10.0
    - monitoring: 1.10.0
    - notification: 1.10.0
    - plugin: 1.10.0
    - repository: 1.10.0
    - secret: 1.10.0
    - statistics: 1.10.0

### frontend.yaml

- [DELETE] console.production_json.DISABLED_MENU
``` diff
-   DISABLED_MENU:
-   - alert_manager
-   - asset_inventory.collector_history
-   - asset_inventory.collector
```

- [DELETE] console-api.production_json.billingV2
```diff
-     billingV2:
-     - domain-id
-     - domain-id
-     - domain-id
```
### values.yaml

- [DELETE] billing(deprecated)
```diff
-billing:
-    enabled: false
-    replicas: 1
-    image:
-      name: public.ecr.aws/megazone/spaceone/billing
-      version: 1.9.7
-
-    pod:
-        spec: {}
```

## DB patch
```
use identity
db.policy.dropIndex('policy_id_1_domain_id_1')
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
