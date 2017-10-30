# kubernetes-keycloak

This repo contains manifests to run Keycloak by RedHat in Kubernetes.

## How To Use the Manifests
If you have your own Postgres Database running, ignore the database manifest named `postgres.yaml`.
The `postgres.yaml` creates a single Postresql database instance for testing.
Checkout the [Configuration](#Configuration) section below, before creating anything.

You should also before `kubectl create`ing the manifests, modify the `ingress.yaml` or simply remove it, but then access to the Keycloak instance will be harder.

It doesn't matter in which order you create the manifests, for testing running the following is enough:
```
kubectl create -f .
```

If you need help, please let me know through an issue.

## Configuration
### Environment variables
The environment variables can be set in the `statefulset.yaml`.

| Name | Description | Default |
| ------------- |-------------| -----|
| `POSTGRES_HOST` | Postgres Database address | `postgres` |
| `POSTGRES_PORT` | Postgres Database port | `5432` |
| `POSTGRES_DATABASE` | Postgres Database name | `keycloak` |
| `POSTGRES_USER` | Postgres Database user | `keycloak` |
| `POSTGRES_PASSWORD` | Postgres Database password | `password` |
| `PROXY_ADDRESS_FORWARDING` | Enable proxy in front of Keycloak JBoss | `false` |
| `KEYCLOAK_LOGLEVEL` | Set Keycloak log level | `INFO` |
| `KEYCLOAK_USER` | First Keycloak user username (no management access) | `` |
| `KEYCLOAK_PASSWORD` | First Keycloak user password (no management access) | `` |
| `KEYCLOAK_MGMT_USER` | Management user username | `` |
| `KEYCLOAK_MGMT_PASSWORD` | Management user password | `` |
| `KEYCLOAK_OWNERS_COUNT` | The cache/sessions infiniband owner/"replica" count (should be `replicas` count) | `2` |
| `BASE_SCRIPT_DIR` | DON'T change unless you know what you are doing | `/scripts` |
| `AUTO_INJECT_HOSTS` | Set to anything but empty to enable injection of new hosts during runtime  | `` (empty) |
| `MY_POD_IP` |  | Kubernetes Downward API `status.podIP` |

### ConfigMap variables
The `ConfigMap` variables can be set in the `configmap.yaml`.

| Name | Description | Default |
| ---- | ----------- | ------- |
| `REPLICAS` | Set to `replicas` count of the `StatefulSet`. Has to be the same! | `2` |
