# Parameters

## Global Parameters

| Parameter                     | Description                           | Default               |
|-------------------------------|---------------------------------------|-----------------------|
| `global.namespace`            | Namespace for all resources           | `spaceone`            |
| `global.supervisor_namespace` | Namespace for supervisor resource     | `spaceone-supervisor` |
| `global.shared_conf`          | Shared configuration for all services | `{...}`               |

### Global Shared Configuration

| Parameter                                                  | Description                | Default                                      |
|------------------------------------------------------------|----------------------------|----------------------------------------------|
| `global.shared_conf.CASHES`                                | Cache configuration        | `{...}`                                      |
| `global.shared_conf.CASHES.default.backend`                | Cache backend              | `spaceone.core.cache.redis_cache.RedisCache` |
| `global.shared_conf.CASHES.default.host`                   | Cache host                 | `redis`                                      |
| `global.shared_conf.CASHES.default.port`                   | Cache port                 | `6379`                                       |
| `global.shared_conf.CASHES.default.db`                     | Redis DB                   | `0`                                          |
| `global.shared_conf.CASHES.default.encoding`               | Redis encoding             | `utf-8`                                      |
| `global.shared_conf.CASHES.default.socket_timeout`         | Redis socket timeout       | `10`                                         |
| `global.shared_conf.CASHES.default.socket_connect_timeout` | Redis socket timeout       | `10`                                         |
| `global.shared_conf.DATABASE`                              | Database configuration     | `{...}`                                      |
| `global.shared_conf.DATABASE.default.username`             | Database username          | `admin`                                      |
| `global.shared_conf.DATABASE.default.password`             | Database password          | `password`                                   |
| `global.shared_conf.DATABASE.default.host`                 | Database host              | `mongodb`                                    |
| `global.shared_conf.DATABASE.default.port`                 | Database port              | `27017`                                      |
| `global.shared_conf.TOKEN`                                 | System token configuration | `""`                                         |

## Console Parameters

| Parameter                 | Description           | Default            |
|---------------------------|-----------------------|--------------------|
| `console.image.name`      | Console image name    | `spaceone/console` |
| `console.image.version`   | Console image tag     | `x.y.z`            |
| `console.production_json` | Console Configuration | `{...}`            |

### Console Production JSON

| Parameter                                         | Description          | Default                              |
|---------------------------------------------------|----------------------|--------------------------------------|
| `console.production_json.CONSOLE_API.ENDPOINT`    | Console API Endpoint | `http://console.api.example.com`     |
| `console.production_json.CONSOLE_API_V2.ENDPOINT` | Console API Endpoint | `http://console-v2.api.example.com`  |
| `console.production_json.DOCS`                    | Docs URL             | `[...]`                              |
| `console.production_json.DOCS[].label`            | Document Label       | `User Guide`                         |
| `console.production_json.DOCS[].url`              | Document URL         | `https://cloudforet.io/docs/guides/` |
| `console.production_json.CONTACT_LINK`            | Contact Link         | `""`                                 |