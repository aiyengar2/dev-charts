# cattle-pushprox

A Rancher chart based on [PushProx](https://github.com/prometheus-community/PushProx) that sets up a deployment of the PushProx proxy and several PushProx clients on a Kubernetes cluster.

Installs [cattle-pushprox](https://github.com/rancher/dev-charts/tree/master/packages/cattle-pushprox) to create a DaemonSet of clients that can access their host's network and register with a PushProx proxy Deployment. A [Prometheus Operator](https://github.com/coreos/prometheus-operator) ServiceMonitor CR is also included that is configured to scrape the metrics from each of the clients through the proxy.

The clients and proxy are created based on of a Rancher fork of the Prometheus Community [PushProx](https://github.com/prometheus-community/PushProx) project.

Using an instance of this chart is suitable for the following scenarios:
- You need to scrape metrics from a port that should not be accessible outside of the host (i.e. scraping `etcd` metrics in a hardened cluster)
- You need to scrape metrics on a host that are not exposed outside of 127.0.0.1 (i.e. scraping `kube-proxy` metrics)
- You need to scrape metrics through HTTPS using certs hosted directly on hostPath
- You need to scrape metrics from Kubernetes components that require authorization via a service account (i.e. permissions to make request to `/metrics`)
- You need to scrape metrics without access to cacerts (i.e. insecureSkipVerify)

## Configuration

The following tables list the configurable parameters of the cattle-pushprox chart and their default values.

### General

#### Required
| Parameter | Description | Example |
| ----- | ----------- | ------ |
| `component` | The component you are planning to monitor | `kube-etcd`
| `metricsPort` | The port on the host that contains the metrics you want to scrape (i.e. `http://<HOST_IP>:<metricsPort>/metrics`) | `2379` |

#### Optional
| Parameter | Description | Default |
| ----- | ----------- | ------ |
| `serviceMonitor.enabled` | Deploys a [Prometheus Operator](https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#servicemonitor) ServiceMonitor CR that is configured to scrape metrics on the hosts in which PushProx clients are deployed through the PushProx proxy. Also deploys a Service that points to all pods with the expected PushProx client name that exposes the `metricsPort` selected. | `true` |
| `clients.enabled` | Deploys a DaemonSet of PushProx clients that are each capable of scraping endpoints on the hostNetwork it is deployed on. | `true` |
| `clients.port` |  The port where the client will publish PushProx client-specific metrics | `9369` |
| `clients.proxyUrl` | Overrides the default proxyUrl setting of `http://pushprox-{{ .Values.component }}-proxy.{{ . Release.Naemspace }}.svc.cluster.local:{{ .Values.proxy.port }}"` with the proxyUrl specified | `""` |
| `clients.useLocalhost` | Sets a flag on each client deployment to redirect scrapes from the `HOST_IP` to `127.0.0.1` to allow users to scrape metrics on the host that are not exposed outside the host | `false` |
| `clients.https.enabled` | Enables scraping metrics via HTTPS using the provided TLS certs that exist on each host | `false` |
| `clients.https.useServiceAccountCredentials` | If set to true, the client will create a service account with adequate permissions and set a flag on the client to use the service account token provided by it to make authorized scrape requests | `false` |
| `clients.https.insecureSkipVerify` | If set to true, the client will disable SSL security checks | `false` |
| `clients.https.certDir` | The path on the host where TLS certs can be found. This path is mounted as a volume on an `initContainer` which copies only the necessary files over to an EmptyDir volume used by each client. Required and only used if `clients.https.enabled` is set. | `""` |
| `clients.https.certFile` | The path to the TLS cert file located within `clients.https.certDir`. Required and only used if `clients.https.enabled` is set.  | `""` |
| `clients.https.keyFile` | The path to the TLS key file located within `clients.https.certDir`. Required and only used if `clients.https.enabled` is set.  | `""` |
| `clients.https.caCertFile` | The path to the TLS cacert file located within `clients.https.certDir`. Required and only used if `clients.https.enabled` is set.  | `""` |
| `clients.resources` | Set resource limits and requests for the client container | `{}` |
| `clients.nodeSelector` | Select which nodes to deploy the PushProx clients on | `{}` |
| `clients.tolerations` | Specify tolerations for PushProx clients | `[]` |
| `proxy.enabled` | Deploys a Deployment of a PushProx proxy that each PushProx client will register with | `true` |
| `proxy.port` | The port that each PushProx client will register with to allow metrics to be scraped from the host | `8080` |
| `proxy.resources` | Set resource limits and requests for the proxy container | `{}` |
| `proxy.nodeSelector` | Select which nodes the PushProx proxy can be deployed on | `{}` |
| `proxy.tolerations` | Specify tolerations (if necessary) to allow the proxy to be deployed on the selected node | `[]` |

*Tip: The filepaths set in `clients.https.<cert|key|caCert>File` can include wildcard characters*