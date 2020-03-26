## Monitoring multi-cluster Istio with Prometheus

Based on any pattern of multi-cluster Istio setup, we can setup
prometheus either externally or locally to scrape cluster specific metrics.

### External Prometheus

- Enable prometheus addon on remote cluster by:

```bash
istioctl manifest apply --set addonComponents.prometheus.enabled=true
```

- Follow this instruction: [Remotely Accessing Telemetry Addons](https://istio.io/docs/tasks/observability/gateways/)
To make cluster-local prometheus accessible via IP_IngressGateway:15030

- Edit prometheus.yaml file for the external prometheus

```bash
scrape_configs:
  - job_name: 'federate-{{CLUSTER_NAME}}'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="pilot"}'
        - '{job="envoy-stats"}'

    static_configs:
      - targets:
        - '{{GATEWAY_IP_ADDR}}:15030'
        labels:
          cluster: {{CLUSTER_NAME}}
```

### Local Prometheus 

We aim to gather both local and remote prometheus scraped metrics from the main cluster. So, we need
to edit prometheus.yaml in main cluster, we can do that via config map:

```bash
kubectl -n istio-system edit cm prometheus -o yaml
``` 

(1) Add job as the above scrape_config for remote prometheus
(2) Add local job:

```bash
- job_name: 'federate-local'
  honor_labels: true
  metrics_path: '/federate'
  metrics_relabel_configs:
  - replacement: {{CLUSTER_NAME}}
    targetLabel: cluster
  kubernetes_sd_configs:
  - role: pod
    namespaces:
      names: ['istio-system']
  params:
    'match[]':
    - '{__name__=~"istio_(.*)"}'
    - '{__name__=~"pilot(.*)"}'
```

### Verification

Verification can be done by visiting prometheus UI.