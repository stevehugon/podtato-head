# Deliver with Helm

Here's how to deliver podtato-head using [Helm](https://helm.sh).

## Deliver

Install from the local chart:

```
helm install podtato-head ./delivery/chart
```

This will install the chart in this directory with release name `podtato-head`.

> NOTE: You can instruct helm to wait for the resources to be ready before marking the release as successful by adding the `--wait` option to the previous command.

The installation can be customized by changing the following parameters via `--set` or a custom `values.yaml` file:

| Parameter                       | Description                                                     | Default                      |
| ------------------------------- | ----------------------------------------------------------------| -----------------------------|
| `replicaCount`                  | Number of replicas of the container                             | `1`                          |
| `images.repositoryDirname`      | Prefix for image repos                                          | `ghcr.io/podtato-head`       |
| `images.pullPolicy`             | Podtato Head Container pull policy                              | `IfNotPresent`               |
| `images.pullSecrets`            | Podtato Head Pod pull secret                                    | ``                           |
| `<service>.repositoryBasename`  | Leaf part of name of image repo for <service>                   | `podtato-main` etc.          |
| `<service>.tag`                 | Tag of image repo for <service>                                 | `v1-latest-dev`              |
| `<service>.serviceType`         | Service type for <service>                                      | `LoadBalancer` for main      |
| `<service>.servicePort`         | Service port for <service>                                      | `9000`-`9005`
| `serviceAccount.create`         | Whether or not to create dedicated service account              | `true`                       |
| `serviceAccount.name`           | Name of the service account to use                              | `default`                    |
| `serviceAccount.annotations`    | Annotations to add to a created service account                 | `{}`                         |
| `podAnnotations`                | Map of annotations to add to the pods                           | `{}`                         |
| `ingress.enabled`               | Enables Ingress                                                 | `false`                      |
| `ingress.annotations`           | Ingress annotations                                             | `{}`                         |
| `ingress.hosts`                 | Ingress accepted hostnames                                      | `[]`                         |
| `ingress.tls`                   | Ingress TLS configuration                                       | `[]`                         |
| `autoscaling.enabled`           | Enable horizontal pod autoscaler                                | `false`                      |
| `autoscaling.targetCPUUtilizationPercentage`  | Target CPU utilization                            | `80`                         |
| `autoscaling.targetMemoryUtilizationPercentage`  | Target Memory utilization                      | `80`                         |
| `autoscaling.minReplicas`       | Min replicas for autoscaling                                    | `1`                          |
| `autoscaling.maxReplicas`       | Max replicas for autoscaling                                    | `100`                        |
| `tolerations`                   | List of node taints to tolerate                                 | `[]`                         |
| `resources`                     | Resource requests and limits                                    | `{}`                         |
| `nodeSelector`                  | Labels for pod assignment                                       | `{}`                         |
| `service.type`                  | Kubernetes Service type                                         | `ClusterIP`                  |
| `service.port`                  | The port the service will use                                   | `9000`                       |

## Test

Verify the release succeeded:

```
helm list
kubectl get pods
kubectl get services
```

### Test the API endpoint

If using a ClusterIP-type service, run `kubectl port-forward` in the background
and connect through that:

> NOTE: Find and kill the port-forward process afterwards using `ps` and `kill`.

```
# Choose below the IP address of your machine you want to use to access application 
ADDR=0.0.0.0
# Choose below the port of your machine you want to use to access application 
PORT=9000
kubectl port-forward --address ${ADDR} svc/podtato-main ${PORT}:9000 &
```

Now test the API itself with a browser at `http://<uid>.int.be.continental.cloud:9000/`

## Update

To update the application version, you can choose one of the following methods :

- update `<service>.tag` in `values.yaml` for each service and run `helm upgrade podtato-head ./delivery/chart`
- run `helm upgrade podtato-head ./delivery/chart --set main.tag=v0.1.1 --set leftLeg.tag=v0.1.1 ...`

A new revision is then installed.

> NOTE : to ensure idempotency between the first installation and the following updates, you should use the following command :

```
helm upgrade --install podtato-head ./delivery/chart
```

## Rollback

To rollback to a previous revision, run :

```
# Check revision history
helm history podtato-head

# Rollback to the revision 1
helm rollback podtato-head 1

# Check the revision
helm status podtato-head
```

## Uninstall

```
helm uninstall podtato-head
```