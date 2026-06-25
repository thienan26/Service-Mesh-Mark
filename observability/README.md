# Observability

MeshMart exposes two layers of metrics:

| Layer | Source | What it measures |
| --- | --- | --- |
| Infrastructure | Istio sidecar proxies | Request count, latency, and error rate at the network level |
| Application | `prometheus-client` in each service | Orders, payments, stock, reviews, and checkout behavior |

## Local Docker Compose

When running `docker compose up`, Prometheus and Grafana start with the microservices.

| Tool | URL | Purpose |
| --- | --- | --- |
| Grafana | http://localhost:13000 | Auto-loaded MeshMart dashboard |
| Prometheus | http://localhost:19090 | Raw metric explorer and scrape status |

Grafana loads the MeshMart dashboard automatically under `Dashboards -> MeshMart -> MeshMart - Application Overview`.

## Kubernetes And Istio Add-ons

```powershell
$istioDir = "C:\Users\ASUS\AppData\Local\Microsoft\WinGet\Packages\Istio.Istio_Microsoft.Winget.Source_8wekyb3d8bbwe\istio-1.29.2"
kubectl apply -f "$istioDir\samples\addons\prometheus.yaml"
kubectl apply -f "$istioDir\samples\addons\grafana.yaml"
kubectl apply -f "$istioDir\samples\addons\jaeger.yaml"
kubectl apply -f "$istioDir\samples\addons\kiali.yaml"
kubectl apply -f istio/telemetry-tracing.yaml
```

Port-forward dashboards:

```powershell
kubectl port-forward -n istio-system svc/prometheus 19090:9090
kubectl port-forward -n istio-system svc/grafana 13000:3000
kubectl port-forward -n istio-system svc/tracing 16686:80
kubectl port-forward -n istio-system svc/kiali 20001:20001
```

Dashboard URLs:

| Tool | URL |
| --- | --- |
| Prometheus | http://127.0.0.1:19090 |
| Grafana | http://127.0.0.1:13000 |
| Jaeger | http://127.0.0.1:16686/jaeger/ |
| Kiali | http://127.0.0.1:20001/kiali/ |

## Application Metrics

| Service | Metric examples |
| --- | --- |
| order-service | `meshmart_orders_total`, `meshmart_order_duration_seconds`, `meshmart_order_amount_usd`, `meshmart_idempotency_hits_total` |
| product-service | `meshmart_product_stock_total`, `meshmart_reviews_total`, `meshmart_inventory_operations_total` |
| payment-service | `meshmart_payments_total`, `meshmart_payment_duration_seconds`, `meshmart_payment_amount_usd` |
| notification-service | `meshmart_notifications_total`, `meshmart_notifications_sent_total_gauge` |

## Useful Prometheus Queries

```promql
sum(meshmart_orders_total{status="confirmed"}) / sum(meshmart_orders_total) * 100
histogram_quantile(0.95, rate(meshmart_order_duration_seconds_bucket[5m]))
histogram_quantile(0.95, rate(meshmart_payment_duration_seconds_bucket[5m]))
meshmart_product_stock_total
sum by (status) (rate(meshmart_orders_total[1m])) * 60
rate(meshmart_inventory_operations_total{operation="release"}[5m])
istio_requests_total
histogram_quantile(0.95, sum(rate(istio_request_duration_milliseconds_bucket[1m])) by (le, destination_workload))
```
