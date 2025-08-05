# TD2 - A better observability tool

In this hands-on exercise, you will dive deeper into the world of observability by using the Grafana Stack. You'll deploy the Squash-TM application and utilize the powerful visualization and monitoring capabilities of Grafana to gain insights into the application's performance and health.

## 1 - Deploy the data collectors

Grafana is only a visual of the metrics. Those metrics comes from a collector Prometheus or Loki.

```Bash
# 1 Add the repo to your Helm repo list
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

# 2 Install loki
helm upgrade --install loki --create-namespace --namespace loki-stack grafana/loki -f 01-values-loki.yaml 

# 3 Install Prometheus 
helm upgrade --install prometheus prometheus-community/prometheus --create-namespace --namespace monitoring -f 02-values-prometheus.yaml
```

## 2 - Deploy Grafana
``` Bash
# 1 Deploy grafana
helm upgrade --install loki-grafana grafana/grafana --create-namespace --namespace loki-stack -f 03-values-grafana.yaml

# 2 Get your 'admin' user password by running:
kubectl get secret --namespace loki-stack loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
``` 

## 3 - Connect the data sources

Define loki datasource in grafana:

* https://<base_url>/grafana
* Admin credentials are in the shared keepass
* Add loki datasource in grafana :
  * > config > Data Sources
  * > Add Data Source
    * http://loki-gateway.loki-stack.svc.cluster.local/


The same with Prometheus
* https://<base_url>/grafana
* Admin credentials are in the shared keepass
* Add prometheus datasource in grafana :
  * > config > Data Sources ( in the side menu )
  * > Add Data Source
    * http://prometheus-server.monitoring.svc.cluster.local


Add your first dashboard:
* Dashboards > import
  * 15141 (Loki Kubernetes Logs)


## 4 - Use of the explorer
With the explorer, get the following informations:
- Use loki to get the TM logs
- Use promql to get the CPU usage of the orchestrator pod
- Use promql to get the TM memory

Make a dashbord that track the mariadb memory

Ints: 
- `(sum(rate(container_cpu_usage_seconds_total{pod=~"mariadb.*"}[5m])) / avg(kube_pod_container_resource_requests{pod=~"mariadb.*", resource="cpu"})) *100`
