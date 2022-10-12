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
## [Upgrade aws-load-balancer-controller(>= v2.4.1)](https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases)
> ingress apiVersion in helm template has been updated from `extensions/v1beta1` to `networking.k8s.io/v1` and aws-load-balancer-controller must be upgraded to support this template.
```
helm update repo
```
```
helm upgrade aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<cluster-name>
```

## Changed Configuration
### image versions
- 1.9.4
- hotfix
  - console 1.9.4.4
  - console-api 1.9.4.2
  - cost-analysis 1.9.4.2

### {all}.yaml
- [Delete] {service}.ingress.annotation.`kubernetes.io/ingress.class: alb`
- [Modify] {service}.ingress.path
```diff
    ingress:
...
        annotations:
-           kubernetes.io/ingress.class: alb
            alb.ingress.kubernetes.io/listen-ports: *
            alb.ingress.kubernetes.io/inbound-cidrs: *
            alb.ingress.kubernetes.io/scheme: *
            alb.ingress.kubernetes.io/target-type: *
            alb.ingress.kubernetes.io/certificate-arn: *
            alb.ingress.kubernetes.io/healthcheck-path: *
            alb.ingress.kubernetes.io/load-balancer-attributes: *
            external-dns.alpha.kubernetes.io/hostname: *
        servicePort: *
+       path: /
-       path: /*
```

### frontend.yaml
- [Add] console.production_json.BILLING_ENABLED
```diff
console:
  enabled: true
  developer: false
  name: console
  replicas: 1
  image:
      name: public.ecr.aws/megazone/spaceone/console
      version: 1.9.4.2
  imagePullPolicy: IfNotPresent

  # For production.json (nodejs)
  # Domain name of console-api (usually ALB of console-api)
#######################
# TODO: Update value
#  - ENDPOINT
#  - GTAG_ID (if you have google analytics ID)
#  - AMCHARTS_LICENSE (for commercial use only)
#######################
  production_json:
    CONSOLE_API:
        ENDPOINT: https://console-api.spaceone.dev
    CONTACT_LINK: <enter_your_site_url_or_email> # Change the contact us link of sign in page
+   BILLING_ENABLED: [] # If you don't want to activate the billing page
+   BILLING_ENABLED: 
+   - <domain_id>
```
- [Add] console-api.production_json.billingV2
```diff
console-api:
  enabled: true
  developer: false
  name: console-api
  replicas: 1
  image:
      name: public.ecr.aws/megazone/spaceone/console-api
      version: 1.9.3.1
  imagePullPolicy: IfNotPresent

###############################################
# TODO: Update value
#  - cors
###############################################

  production_json:
      cors:
      - http://*
      - https://*
#      - https://*.console.spaceone.dev
#      - https://*.console.spaceone.dev:8080
      redis:
          host: redis
          port: 6379
          db: 15
+     billingV2: [] # If you don't want to activate the billing page
+     billingV2: 
+     - <domain_id>      
...
```

## DB patch
- config

```
use config;
db.user_config.dropIndex('name_1_domain_id_1')
})
```

## [New domain-initializer](https://github.com/spaceone-dev/spaceone-initializer#spaceone-initializer)

### root-domain.yaml
```
enabled: true
image:
    name: spaceone/spacectl
    version: 1.9.4
main:
  import:
    - /root/spacectl/apply/root_domain.yaml
    - /root/spacectl/apply/marketplace.yaml
    - /root/spacectl/apply/role.yaml
  var:
    domain:
      root: root
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
### user-domain.yaml
```
enabled: true
image:
    name: spaceone/spacectl
    version: 1.9.4
main:
  import:
    - /root/spacectl/apply/user_domain.yaml
    - /root/spacectl/apply/statistics.yaml
  var:
    domain:
      user: spaceone
    default_language: ko
    default_timezone: Asia/Seoul
    domain_owner:
      id: admin
      password: Admin123!@#
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
### domain.yaml(root and user)
```
enabled: true
image:
    name: spaceone/spacectl
    version: 1.9.4
main:
  import:
    - /root/spacectl/apply/root_domain.yaml 
    - /root/spacectl/apply/marketplace.yaml
    - /root/spacectl/apply/role.yaml
    - /root/spacectl/apply/user_domain.yaml
    - /root/spacectl/apply/statistics.yaml
  var:
    domain:
      root: root
      user: spaceone
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

## Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
