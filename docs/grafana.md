# Grafana
Grafana is an open source analytics and interactive visualization web application, providing charts, graphs, and alerts for monitoring and observability requirements. You'll configure Grafana to connect to Prometheus. 

- Using Helm, install Grafana using the publicly available Grafana Helm Chart. You will deploy Grafana into the monitoring namespace -
```bash
{
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana --namespace monitoring grafana/grafana --version 6.1.14
}
```
- Confirm that the Grafana deployment has been rolled out successfully. In the terminal execute the following command:
```bash
kubectl get deployment grafana -n monitoring -w
```
- The Grafana web admin interface now needs to be exposed to the Internet. To do so, create a new NodePort based Service, exposing the web admin interface on port 30300. In the terminal execute the following command:
```bash
{
kubectl expose deployment grafana --type=NodePort --name=grafana-main --port=30300 --target-port=3000 -n monitoring
kubectl patch service grafana-main -n monitoring -p '{"spec":{"ports":[{"nodePort": 30300, "port": 30300, "protocol": "TCP", "targetPort": 3000}]}}'
}
```
- Extract the default admin password which will be required to login. In the terminal execute the following command:
```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
- Get the public IP address of the Kubernetes cluster that Grafana has been deployed into. In the terminal execute the following command:
```bash
export | grep K8S_CLUSTER_PUBLICIP
```
- Having successfully authenticated, the Welcome to Grafana home page is displayed. Within the Data Sources section, click on the Add your first data source option. In the Add data source view, select the Prometheus option by clicking on it's Select button. 
-  In the Data Sources / Prometheus configuration view update the HTTP URL to be the same URL that you previously used to browse to the Prometheus web admin interface. Leave all other default settings as is. In particular its important to leave the Name field set to Prometheus. Complete the Prometheus data source setup by clicking  on the Save & Test button at the bottom. 
- In the IDE, open the project/code/grafana directory and click on the dashboard.json file to open it within the editor pane. In the editor pane, select all of the configuration and copy it to the clipboard.
- Return to Grafana and this time select the Create icon (+) on the main left hand side menu, and then select the dashboard Import option 
- In the Import view, paste in the copied Grafana dashboard json into the Import via panel json area and then click the Load button
- Under Import Options, accept all defaults without changing anything, and then click the Import button
- Grafana will now load the prebuilt dashboard and start rendering visualisations using live monitoring data streams taken from Prometheus. The dashboard view automatically refreshes every 5 seconds