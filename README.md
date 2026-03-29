# Selectel Billing Exporter

The exporter uses `https://api.selectel.ru/v3/balances` API to get balance info.

Selectel API token can be retrieved from https://my.selectel.ru/profile/apikeys

## Installing on Kubernetes

### helm

Helm chart available at https://github.com/mxssl/helm-charts/tree/main/charts/selectel-billing-exporter

### manual manifests creation

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: selectel-billing
  namespace: exporters
spec:
  selector:
    matchLabels:
      component: selectel-billing
  template:
    metadata:
      labels:
        component: selectel-billing
    spec:
      containers:
        - name: exporter
          image: mxssl/selectel-billing-exporter:1.1.5
          command: ["./app"]
          ports:
            - containerPort: 80
          env:
            - name: TOKEN
              value: <YOUR-TOKEN>

---
apiVersion: v1
kind: Service
metadata:
  name: selectel-billing
  namespace: exporters
spec:
  ports:
    - name: exporter
      port: 6789
      targetPort: 80
  selector:
    component: selectel-billing
```

```sh
kubectl apply -n exporters -f your-file.yaml
```

Metrics will be available at `selectel-billing.exporters.svc.cluster.local:6789/metrics`

## Prometheus setup

```yaml
- job_name: "selectel_billing"
  scrape_interval: 60m
  static_configs:
    - targets: ["exporter_address:6789"]
```

## Alertmanager alert example

```yaml
- alert: selectel_billing
  expr: selectel_billing_final_sum{job="selectel_billing"} / 100 < 10000
  for: 180s
  labels:
    severity: warning
  annotations:
    summary: "{{ $labels.instance }}: Selectel balance is below 10,000 rubles"
```

## Grafana dashboard

https://grafana.com/dashboards/9315
