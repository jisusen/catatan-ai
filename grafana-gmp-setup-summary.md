# Grafana and Google Managed Prometheus (GMP) Setup Summary

## Ports

1. Grafana:
   - Default HTTP port: 3000
   - Default HTTPS port: 3443

2. GMP Collector (Prometheus sidecar):
   - Metrics port: 9090 (typically only accessed within the cluster)

3. Kubernetes API Server:
   - HTTPS port: 6443 (for secure communication, typically handled by GKE setup)

## IAM Roles

1. For GMP Collector:
   - roles/monitoring.metricWriter
   - roles/monitoring.viewer
   - roles/logging.logWriter

2. For Grafana (to query GMP):
   - roles/monitoring.viewer

3. For GKE nodes (if using GKE's workload identity):
   - roles/iam.workloadIdentityUser

## Firewall Rules

1. Allow health checks:
   - Source: 130.211.0.0/22, 35.191.0.0/16
   - Destination: GKE Nodes
   - Ports: TCP 3000 (Grafana), TCP 9090 (GMP Collector)

2. Allow internal access (if needed):
   - Source: Your VPC CIDR range
   - Destination: GKE Nodes
   - Ports: TCP 3000 (Grafana)

3. Allow access from bastion/jump host (if applicable):
   - Source: Bastion VM IP
   - Destination: GKE Nodes
   - Ports: TCP 3000 (Grafana), TCP 9090 (GMP Collector)

Note: Adjust these based on your specific network setup and security requirements.
