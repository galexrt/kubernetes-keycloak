# kubernetes-keycloak

This repo contains manifests to run Keycloak by RedHat in Kubernetes.

## How To Use the Manifests
If you have your own Postgres Database running, ignore the database manifest named `postgres.yaml`.
The `postgres.yaml` creates a single Postresql database instance for testing.
Checkout the [Configuration](#Configuration) section below, before creating anything.

You should also before `kubectl create`ing the manifests, modify the `ingress.yaml` or simply remove it, but then access to the Keycloak instance will be harder.

It doesn't matter in which order you create the manifests, for testing running the following is enough:
```
kubectl create -f . --namespace default
```

If you need help, please let me know through an issue.

## Default credentials
> **NOTE** To disable user creation, leave the specific `*_USER` and `*_PASSWORD` environment variables empty (only works for `KEYCLOAK_*` variables).

To change the usernames, edit the `*_USER` variables in the `ConfigMap` `keycloak-env` which can be found in [`configmap.yaml`](configmap.yaml).

To change the passwords, edit the `*_PASSWORD` variables in the `Secret` `keycloak-secret`, which can be found in [`secret.yaml`](secret.yaml). The passwords/secrets need to be `base64` encoded (example `echo -n YOUR_PASSWORD | base64 -w0`).

### Keycloak
* Username (`KEYCLOAK_USER`): `keycloak`
* Password (`KEYCLOAK_PASSWORD`): `keycloak123`

### Management
* Username (`KEYCLOAK_MGMT_USER`): `keycloak`
* Password (`KEYCLOAK_MGMT_PASSWORD`): `keycloak123`

### Postgres (Example)
See [`postgres.yaml`](postgres.yaml) `env` vars for username and password.

## Configuration
### Environment variables
The environment variables can be set in the `statefulset.yaml`.

| Name                       | Description                                                                      | Default                                |
| -------------------------- | -------------------------------------------------------------------------------- | -------------------------------------- |
| `POSTGRES_HOST`            | Postgres Database address                                                        | `postgres`                             |
| `POSTGRES_PORT`            | Postgres Database port                                                           | `5432`                                 |
| `POSTGRES_DATABASE`        | Postgres Database name                                                           | `keycloak`                             |
| `POSTGRES_USER`            | Postgres Database user                                                           | `keycloak`                             |
| `POSTGRES_PASSWORD`        | Postgres Database password                                                       | `password`                             |
| `PROXY_ADDRESS_FORWARDING` | Enable proxy in front of Keycloak JBoss                                          | `false`                                |
| `KEYCLOAK_LOGLEVEL`        | Set Keycloak log level                                                           | `INFO`                                 |
| `KEYCLOAK_USER`            | First Keycloak user username (no management access)                              | ``                                     |
| `KEYCLOAK_PASSWORD`        | First Keycloak user password (no management access)                              | ``                                     |
| `KEYCLOAK_MGMT_USER`       | Management user username                                                         | ``                                     |
| `KEYCLOAK_MGMT_PASSWORD`   | Management user password                                                         | ``                                     |
| `KEYCLOAK_OWNERS_COUNT`    | The cache/sessions infiniband owner/"replica" count (should be `replicas` count) | `2`                                    |
| `BASE_SCRIPT_DIR`          | DON'T change unless you know what you are doing                                  | `/scripts`                             |
| `MY_POD_IP`                | The Pod IP                                                                       | Kubernetes Downward API `status.podIP` |

## Exposing to the outside
An appropiate `Ingress` can be found here: [`ingress.yaml`](ingress.yaml).

The service which exposes Keycloak HTTP port only is named `keycloak-external`.

## Upgrade procedure
> **NOTE** This procedure has **not** been tested to work in "100%" cases!
>
> **NOTE** This procedure has been tested with a replicas of `2` deployment of Keycloak.

### Without migrations
Update the image tag in the `StatefulSet` and replace (`kubectl replace`) the `StatefulSet`.
That is it. The `Pod`s should one by one get recreated with the image.

### With migrations
> **WARNING** This procedure only needs to be done only when new migrations are added to the `bin/migrate-standalone-ha.cli` file (which can be found in the release tarball of Keycloak)!
>
> **WARNING** **This only needs to be done in one `Pod`!**

Update the image tag in the `StatefulSet`, replace (`kubectl replace`) the `StatefulSet`, wait for the highest count `Pod` to get terminated and started again, immediately run the following command in the highest count Keycloak `Pod`:

```
kubectl exec --namespace default -it keycloak-1 -- bash -c 'cd /opt/jboss && bin/jboss-cli.sh --file=bin/migrate-standalone-ha.cli'
```

(Where `keycloak-1` would be the highest count `Pod`, for example for `replicas: 10`, it is `keycloak-9`)

After the successfull run of the exec, you need to delete the `Pod` you execed into.
