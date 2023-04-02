# Fluent-bit - Loki plugin - Prometheus node-exporter
DaemonSet stack logging and metrics

# Integrate Fluent-bit with Loki and Prometheus, and visualize logs and metrics using Grafana.

1- create name space for monitoring stack
```
kubectl create namespace observation
```
2- Install grafana using official Chart
```
helm upgrade --install grafana grafana/grafana --namespace observation --set persistence.enabled=true --set persistence.storageClassName="linode-block-storage" --set adminPassword='rMWKVIuZpNUR' --set service.type=LoadBalancer
```

3- Install loki from grafana loki-stack chart
```
helm upgrade --install loki grafana/loki-stack --namespace observation --set promtail.enabled=false --set backend.podAnnotations="prometheus.io/port: "3100"" 
```
3.1- Patch Loki StatfullSet to make sure that Promethuse scarp metrics from port 3100
```
kubectl patch statefulset loki -n observation --type json -p '[{"op": "replace", "path": "/spec/template/metadata/annotations/prometheus.io~1port", "value": "3100"}]'
```

4- Insatll Prometheus from offical community chart with node-exporter disabled because it will be configured with fluent bit in the next step.
```
helm upgrade --install prometheus prometheus-community/prometheus --namespace observation --set server.service.type=LoadBalancer --set alertmanager.persistentVolume.storageClass="linode-block-storage" --set server.persistentVolume.storageClass="linode-block-storage" --set prometheus-node-exporter.enabled=false 
```

5- Install fluent-bit from this Chart.
```
git clone https://github.com/abdo-farag/fluent-bit-stack.git
cd fluent-bit-stack
helm upgrade --install fluent-bit . --namespace observation
```

## Uninsatll all deployments
```
helm uninstall fluent-bit -n observation
helm uninstall prometheus -n observation
helm uninstall loki -n observation
helm uninstall grafana -n observation
kubectl delete namespace observation
```
