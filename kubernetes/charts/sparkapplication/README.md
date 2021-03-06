# Helm Chart for Spark applications using the Spark Operator

This is the Helm chart to create Spark applications to work with the [Kubernetes Operator for Apache Spark](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator).

### Prerequisites

As the Spark Operator requires Kubernetes 1.13 or above, this chart needs the same. It also requires an instance of the Spark Operator in your cluster. You can deploy one with the provided [Helm Chart](https://github.com/helm/charts/tree/master/incubator/sparkoperator)

### Installing the Chart

The chart can be found in our public Helm repository. Add the repository:

```bash
$ helm repo add <repo_name> https://artifactory.vgt.vito.be/helm-charts
```

Install the chart with:

```bash
$ helm install <repo_name>/sparkapplication --generate-name --namespace <namespace>
```

There are 4 required parameters to be set:
  * image
  * imageVersion
  * mainApplicationFile
  * serviceAccount

### Parameters

| Parameter             | Description                                         | Default          |
|-----------------------|-----------------------------------------------------|------------------|
| `driver.envVars`      | Environment variables for the driver                |                  |
| `driver.cores`        | How many cores the driver can use                   | `2`              |
| `driver.memory`       | Memory limimt for the driver                        | `"4096m"`        |
| `driver.userId`       | User to run the container as                        |                  |
| `executor.cores`      | How many cores the driver can use                   | `2`              |
| `executor.envVars`    | Environment variables for the driver                |                  |
| `executor.instances`  | Number of executors                                 | `1`              |
| `executor.memory`     | Memory limit for the driver                         | `"4096m"`        |
| `fileDependencies`    | File dependencies for the application               |                  |
| `hostNetwork`         | Use the host network                                | `false`          |
| `image`               | Image for the Spark application                     |                  |
| `imageVersion`        | Version for the Spark application image             |                  |
| `imagePullPolicy`     | Docker image pull policy                            | `"IfNotPresent"` |
| `ingress.annotations` | Annotations for the ingress resource                |                  |
| `ingress.enabled`     | Enable ingress                                      | `false`          |
| `ingress.hosts.[0]`   | host header for access                              |                  |
| `ingress.tls`         | Utilize TLS backend in ingress                      | `false`          |
| `jarDependencies`     | Jar dependencies for the application                |                  |
| `jmxExporterJar`      | The Prometheus jar to use for monitoring            |                  |
| `jmxPort`             | Port for serving the Prometheus metrics             | `8090`           |
| `mainApplicationFile` | The start script of the Spark application           |                  |
| `pythonVersion`       | The Python version                                  | `3`              |
| `serviceAccount`      | Service account of the Spark Operator               |                  |
| `service.enabled`     | Enable service                                      | `false`          |
| `service.port`        | A port that point to your spark application         |                  |
| `service.type`        | The type of the service                             |                  |
| `sparkConf`           | Spark configuration settings                        |                  |
| `sparkVersion`        | The Spark version to use                            | `"2.4.5"`        |
| `volumes`             | Volumes to be consumed by the application           |                  |
| `volumeMounts`        | The volumes that should be mounted in the container |                  |

### Sample values

Following is an example `values.yaml` file:

```yaml
---
image: "vito-docker.artifactory.vgt.vito.be/openeo-geotrellis"
imageVersion: "0.1.13"
jmxExporterJar: "/opt/jmx_prometheus_javaagent-0.13.0.jar"
mainApplicationFile: "local:///usr/local/lib/python3.7/dist-packages/openeogeotrellis/deploy/kubernetes.py"
serviceAccount: "spark-operator-spark"
volumes:
  - name: "eodata"
    hostPath:
      path: "/eodata"
      type: "DirectoryOrCreate"
volumeMounts:
  - name: "eodata"
    mountPath: "/eodata"
executor:
  memory: "512m"
  cpu: 1
driver:
  memory: "512m"
  cpu: 1
  envVars:
    KUBE_OPENEO_API_PORT: "50001"
    DRIVER_IMPLEMENTATION_PACKAGE: "openeogeotrellis"
    TRAVIS: "1"
    OPENEO_CATALOG_FILES: "/opt/layercatalog.json"
sparkConf:
  "spark.executorEnv.DRIVER_IMPLEMENTATION_PACKAGE": "openeogeotrellis"
  "spark.extraListeners": "org.openeo.sparklisteners.CancelRunawayJobListener"
  "spark.appMasterEnv.DRIVER_IMPLEMENTATION_PACKAGE": "openeogeotrellis"
jarDependencies:
  - 'local:///opt/geotrellis-extensions-1.4.0-SNAPSHOT.jar'
  - 'local:///opt/geotrellis-backend-assembly-0.4.5-openeo.jar'
fileDependencies:
  - 'local:///opt/layercatalog.json'
service:
  enabled: true
  port: 50001
ingress:
  annotations:
    kubernetes.io/ingress.class: traefik
  enabled: true
  hosts:
  - host: openeo.example.com
    paths:
      - '/'
```
