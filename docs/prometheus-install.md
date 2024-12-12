# Prometheus Installation
- Create a new ```monitoring``` namespace within the cluster - 
```bash
kubectl create ns monitoring
```
- Using Helm, install Prometheus using the publicly available Prometheus Helm Chart. Deploy Prometheus into the monitoring namespace within the lab provided cluster. In the terminal execute the following commands:
```bash
{
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install prometheus --namespace monitoring --values ./code/prometheus/values.yml prometheus-community/prometheus --version 13.0.0
}
```
- Confirm that Prometheus has been successfully rolled out within the cluster. In the terminal execute the following command:
```bash
kubectl get deployment -n monitoring -w
```
- Patch the Prometheus Node Exporter DaemonSet to ensure that Prometheus can collect Memory and CPU node metrics. In the terminal execute the following command:
```bash
kubectl patch daemonset prometheus-node-exporter -n monitoring -p '{"spec":{"template":{"metadata":{"annotations":{"prometheus.io/scrape": "true"}}}}}'
```
-  The Prometheus web admin interface now needs to be exposed to the Internet so that you can browse to it. To do so, create a new NodePort based Service, and expose the web admin interface on port 30900. In the terminal execute the following command:
```bash
{
kubectl expose deployment prometheus-server --type=NodePort --name=prometheus-main --port=30900 --target-port=9090 -n monitoring
kubectl patch service prometheus-main -n monitoring -p '{"spec":{"ports":[{"nodePort": 30900, "port": 30900, "protocol": "TCP", "targetPort": 9090}]}}'
}
```
- Get the public IP address of the Kubernetes cluster that Prometheus has been deployed into. In the terminal execute the following command:
```bash
export | grep K8S_CLUSTER_PUBLICIP
```