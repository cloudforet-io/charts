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
- 1.10.0
    - console: 1.10.0.5
    - inventory: 1.10.0.6
    - identity: 1.10.0.1

### DB patch
- Update statistics history label
```
use statistics;
db.history.updateMany({'values.label': 'Compute'}, {$set: {'values.label': 'Server’}})
```

- Reset table schema for Server Table
```
use config;
db.user_config.find({name: /console:USER:.*:page-schema:inventory.*:table/})
db.user_config.deleteMany({name: /console:USER:.*:page-schema:inventory.*:table/})
```

- Remove some fields from Cloud Service
```
use inventory;
db.cloud_service.updateMany({}, {$unset: {"collection_info.pinned_keys": ""}})
db.cloud_service.updateMany({}, {$unset: {"collection_info.change_history”: ""}})
db.cloud_service.updateMany({}, {$unset: {"launched_at": ""}})
```

- Remove server data from Collection_State
```
use inventory
db.collection_state.find({resource_type: "inventory.Server"})
db.collection_state.deleteMany({resource_type: "inventory.Server"})
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
