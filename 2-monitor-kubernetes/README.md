# Exercise: Monitor Kubernetes System With Prometheus and Grafana

Design and run a monitoring stack for a single-node Kubernetes lab cluster. By the end of the exercise you will have Prometheus scraping Kubernetes control-plane and node metrics, and Grafana dashboards visualizing cluster health.


## Learning goals

- Deploy Prometheus and Grafana on k3s using upstream Helm charts
- Collect node resource metrics (CPU, memory, filesystem) and core Kubernetes metrics (APIServer, kubelet, scheduler, etc.)
- Validate data flow end-to-end (Prometheus targets → Grafana dashboards)
- Practice common day-2 operations: troubleshooting pods, exposing services securely, and cleaning up

## Preparation

- Define a namespace for your team: ex. `g1`, `g2`, etc
- Copy the provided config file to `$HOME/.kube/config`
- Test the connection: `kubectl get nodes`
- Create namespace for your group: `kubectl create ns g1`

NOTE: if you cannot connect to the cluster (timeout), try using the guest WI-FI. The main network seems to filter access to port 6443 which is needed to connect to Kubernetes.

## Exercise

Create a pipeline with the following blocks:

1. **Deploy**: deploy Prometheus and Grafana using Helm

- Create namespace for your chosen team name
- Switch to the namespace
- Add Helm charts:
    ```sh
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    ```
- Install the chart (update with your group number)

    ```shell
    helm install monitoring-g1 prometheus-community/kube-prometheus-stack -n g1 --values clusterroles-g1.yml --values prometheus-kubernetes.yml
    ```

- Wait for rollout to complete, you should see a message like this:

```text
NAME: monitoring-g1
LAST DEPLOYED: Fri Apr 24 17:54:50 2026
NAMESPACE: g1
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace g1 get pods -l "release=monitoring-g1"

Get Grafana 'admin' user password by running:

  kubectl --namespace g1 get secrets monitoring-g1-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace g1 get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=monitoring-g1" -oname)
  kubectl --namespace g1 port-forward $POD_NAME 3000

Get your grafana admin user password by running:

  kubectl get secret --namespace g1 -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo


Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

## Experiment with Prometheus

Run the following commands from your machine (you will need to have kubectl installed and the kubeconfig file in `$HOME/.kube/config`)

For example, for group 1

   ```bash
   kubectl port-forward -n g1 svc/monitoring-g1-kube-prometh-prometheus 9090:9090
   ```

Or group 2

   ```bash
   kubectl port-forward -n g2 svc/monitoring-g2-kube-prometh-prometheus 9090:9090
   ```

2. Visit `http://localhost:9090/targets` and explore what is being scraped.
3. Query live data (e.g., `node_cpu_seconds_total`) to confirm metrics are flowing.

## Experiment with Grafana

Run the following commands from your machine (you will need to have kubectl installed and the kubeconfig file in `$HOME/.kube/config`)

Find the admin password. The command is in the output of the Grafana installation (replace with your group namespace)

```shell
kubectl get secret --namespace g0 -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo
```

Take note of the random password generated.

Export the service locally:

```shell
export POD_NAME=$(kubectl --namespace g1 get pod -l "app.kubernetes.io/name=grafana" -oname)
kubectl port-forward -n g1 $POD_NAME 3000
```

Visit `http://localhost:3000/` and login with admin and the password you obtainer earlier.

Select "Dashboards" from the left menu -> "Kubernetes / Compute Resources / Cluster". You should see the CPU/MEM utilization for the cluster

Explore all the other dashboards

Navigate to explore: here you can run the same queries you ran on exercise 1

## Cleanup

Add a promotion to your pipeline to cleanup, with the following blocks:

1. **Destroy**: destroy  Prometheus and Grafana using Helm

- Uninstall the chart:

```shell
helm uninstall -n g1 monitoring-g1
```

