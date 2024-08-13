# Grafana Setup Guide for GKE

## Prerequisites
- Helm 3 installed on your local machine
- `kubectl` configured to communicate with your GKE cluster
- Necessary permissions to create resources in the monitoring namespace

## Step 1: Add the Grafana Helm repository

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## Step 2: Create a values file for Grafana configuration

Create a file named `grafana-values.yaml` with the following content:

```yaml
persistence:
  enabled: true
  storageClassName: "standard" # Use appropriate storage class

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: "gce-internal"
    # Add any other necessary annotations for your setup

datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.monitoring.svc.cluster.local
      access: proxy
    - name: Jaeger
      type: jaeger
      url: http://jaeger-query.monitoring.svc.cluster.local:16686

# If you're using GCP managed Prometheus, add this datasource
    - name: Cloud Monitoring
      type: stackdriver
      access: proxy
      jsonData:
        tokenUri: https://oauth2.googleapis.com/token
        authenticationType: gce

# Basic security setup - change in production
adminUser: admin
adminPassword: strongpassword

# Resources
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

Adjust the values according to your specific needs and environment.

## Step 3: Install Grafana using Helm

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --create-namespace \
  -f grafana-values.yaml
```

## Step 4: Verify the installation

```bash
kubectl get pods -n monitoring
kubectl get services -n monitoring
kubectl get ingress -n monitoring
```

## Step 5: Access Grafana

If you've set up an internal load balancer, you can access Grafana via the internal IP. To get the IP:

```bash
kubectl get ingress -n monitoring grafana -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

Use this IP to access Grafana in your browser.

## Step 6: Log in to Grafana

Use the adminUser and adminPassword you set in the values file to log in.

## Step 7: Configure additional data sources and dashboards

Once logged in, you can:
1. Verify the pre-configured data sources (Prometheus, Jaeger, Cloud Monitoring)
2. Import dashboards for your specific needs
3. Create custom dashboards for your applications

## Note on Security
For production environments, consider:
- Using Kubernetes secrets for sensitive information
- Implementing more robust authentication methods
- Setting up proper network policies
- Regularly updating Grafana and reviewing security settings
