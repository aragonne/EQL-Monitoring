# TD3 - AlertManager and Alert Configuration

This guide will walk you through setting up and understanding alerts for the Squash-TM application deployed on Minikube. Alerting is a crucial component of proactive monitoring, enabling you to identify and respond to potential issues before they impact your users.

## Understanding AlertManager

AlertManager is a component of the Prometheus ecosystem that handles alerts sent by client applications such as Prometheus server. It takes care of deduplicating, grouping, and routing alerts to the correct receiver (such as email, Slack, or PagerDuty). Key features include:

- **Grouping**: Combines similar alerts to reduce notification noise
- **Inhibition**: Suppresses notifications for certain alerts if others are already firing
- **Silences**: Temporarily mute alerts based on specific criteria
- **High Availability**: Can be configured in a clustered mode for redundancy

## 1 - Prometheus AlertManager Configuration

The file `02-values-prometheus-with-alerts.yaml` contains example alert rules that demonstrate different types of alerts.

### Deploying Prometheus with Alert Rules

```bash
# Deploy Prometheus with LoadBalancer and alert rules
helm upgrade --install prometheus prometheus-community/prometheus --create-namespace --namespace monitoring -f 02-values-prometheus-with-alerts.yaml

# Get the new IP to access Prometheus
kubectl get svc -A | grep prometheus-server
```

### Alert Rules Explanation

The alert rules in `02-values-prometheus-with-alerts.yaml` are organized into groups:

#### Infrastructure Alerts
- **InstanceDown**: Detects when any monitored service is down for over 2 minutes
- **HighCPUUsage**: Alerts when CPU usage exceeds 80% for 5 minutes

#### Squash-TM Application Alerts
- **MariaDBHighMemoryUsage**: Triggers when MariaDB memory usage exceeds 80%
- **OrchestratorLowCPU**: Alerts on unexpectedly low CPU usage (<20%) which might indicate issues
- **SquashTMHighLatency**: Monitors API response time degradation

### Alert Rule Components

1. **Expression (expr)**: The Prometheus query that defines the alert condition
2. **Duration (for)**: How long the condition must be true before firing
3. **Labels**: Used for routing and severity indication
4. **Annotations**: Human-readable information and actions

## 2 - Creating Alerts in Grafana

Grafana offers a different approach to alert management with a unified alerting interface.

### Setting Up Grafana Alerts

In Grafana, navigate to Alerting > Alert Rules, and create:

1. **MariaDB Memory Alert**:
   - Condition: Memory usage > 80%
   - Query: `container_memory_usage_bytes{container="mariadb"} / container_spec_memory_limit_bytes{container="mariadb"} * 100 > 80`
   - Evaluation interval: 1m
   - Pending period: 10m

2. **Orchestrator CPU Alert**:
   - Condition: CPU usage < 20%
   - Query: `avg(rate(container_cpu_usage_seconds_total{container="orchestrator"}[5m])) * 100 < 20`
   - Evaluation interval: 1m
   - Pending period: 15m

3. **Setting Up a Chat Webhook**:
   - In Grafana, navigate to Alerting > Contact points
   - Add a new contact point with type 'Webhook'
   - Configure the webhook URL for your chat service (Slack, Teams, etc.)
   - Create a notification policy that routes alerts to this contact point

## 3 - Practical Exercise

1. Create your own alert based on a query you've tried before
2. Use Grafana Explorer to build and test the query
3. Add the query as an alert in the `02-values-prometheus-with-alerts.yaml` file
4. Test triggering the alert by creating the condition (e.g., high CPU load)
5. Observe how AlertManager handles the alert and sends notifications