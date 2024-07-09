# TD2 - A better observability tool

This exercise will guide you through the process of setting up alerts for the Squash-TM application deployed on Minikube. Alerts are crucial for proactive monitoring, helping you to identify and respond to potential issues before they impact users.

## 1 - Lets focus on Prometheus

```Bash
# 1 Add a LoadBalancer
helm upgrade --install prometheus prometheus-community/prometheus --create-namespace --namespace monitoring -f 01-values-prometheus.yaml

# Get the new IP
kubectl get svc -A
```

1) Try to create an alert from a query you tried before.
2) Go on grafana and use the explorer to create a query.
3) Add the query as an alert in the `02-values-prometheus-with-alerts.yaml` file.

## 2 - A different management on Grafana

On Grafana, in the alerting section, create:
- One alert based on the Mariadb memory >80%
- One alert based on the orchestrator CPU <20%
- Add a chat webhook for the alerts