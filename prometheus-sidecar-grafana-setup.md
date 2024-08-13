# Setup Guide: Prometheus Sidecar and Grafana for GCP Managed Prometheus

## Prerequisites
- GKE cluster running with the monitoring namespace
- `gcloud` CLI installed and configured
- `kubectl` configured to communicate with your GKE cluster
- Helm 3 installed

## Step 1: Enable Google Cloud Managed Service for Prometheus

First, enable the necessary APIs in your GCP project:

```bash
gcloud services enable monitoring.googleapis.com
gcloud services enable opsconfigmonitoring.googleapis.com
```

## Step 2: Install GMP Operator

The GMP (Google Managed Prometheus) Operator will help deploy and manage the Prometheus sidecar/collector.

1. Add the Google Cloud Helm repository:

```bash
helm repo add gmp https://googlecloudplatform.github.io/cloud-ops-sandbox/charts
helm repo update
```

2. Install the GMP Operator:

```bash
helm install gmp-operator gmp/gmp-operator \
  --namespace monitoring \
  --create-namespace
```

## Step 3: Create a PodMonitoring resource

Create a file named `pod-monitoring.yaml` with the following content:

```yaml
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: prometheus-self-monitoring
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: prometheus
  endpoints:
  - port: metrics
    interval: 30s
```

Apply this configuration:

```bash
kubectl apply -f pod-monitoring.yaml
```

## Step 4: Create a GMP Collector

Create a file named `gmp-collector.yaml` with the following content:

```yaml
apiVersion: monitoring.googleapis.com/v1
kind: GmpCollector
metadata:
  name: gmp-collector
  namespace: monitoring
spec:
  collectors:
    - type: prometheus
      prometheusCollector:
        scrapeInterval: 60s
        scrapeTimeout: 10s
  gcpConfig:
    projectId: your-gcp-project-id
    serviceAccountName: your-gcp-service-account@your-project.iam.gserviceaccount.com
```

Replace `your-gcp-project-id` and `your-gcp-service-account@your-project.iam.gserviceaccount.com` with your actual GCP project ID and service account.

Apply this configuration:

```bash
kubectl apply -f gmp-collector.yaml
```

## Step 5: Update Grafana configuration

Update your Grafana configuration to use Google Cloud Managed Prometheus as a data source. Edit your `grafana-values.yaml` file (or create a new one if you don't have it):

```yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Google Cloud Managed Prometheus
      type: prometheus
      url: https://monitoring.googleapis.com/v1/projects/your-gcp-project-id/locations/global/prometheus
      access: proxy
      jsonData:
        tokenUri: https://oauth2.googleapis.com/token
        authenticationType: gce
        defaultProject: your-gcp-project-id

# Other Grafana configurations...
```

Replace `your-gcp-project-id` with your actual GCP project ID.

## Step 6: Update Grafana

If you've already installed Grafana, update it with the new configuration:

```bash
helm upgrade grafana grafana/grafana \
  --namespace monitoring \
  -f grafana-values.yaml
```

If you haven't installed Grafana yet, install it:

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --create-namespace \
  -f grafana-values.yaml
```

## Step 7: Verify the setup

1. Check if the GMP Collector is running:

```bash
kubectl get pods -n monitoring | grep gmp-collector
```

2. Check if Grafana is running:

```bash
kubectl get pods -n monitoring | grep grafana
```

3. Access Grafana UI (use the method appropriate for your setup, e.g., port-forwarding or ingress)

4. In Grafana, go to Configuration > Data Sources and verify that the Google Cloud Managed Prometheus data source is available and working.

## Step 8: Create dashboards

Now you can create dashboards in Grafana using the Google Cloud Managed Prometheus data source. You can import existing Prometheus dashboards or create custom ones based on your metrics.

Remember to use PromQL queries compatible with Google Cloud Managed Prometheus when creating your dashboards.
