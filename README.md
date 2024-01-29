# Kubernetes for Data Engineering

This repository contains the necessary configuration files and DAGs (Directed Acyclic Graphs) for setting up a robust data engineering environment using Kubernetes and Apache Airflow. It includes the setup for the Kubernetes Dashboard, which provides a user-friendly web interface for managing Kubernetes clusters, and Apache Airflow, a platform to programmatically author, schedule, and monitor workflows.

### DAGs

- `fetch_and_preview.py`: A DAG for fetching data and providing a preview.
- `hello.py`: A simple example DAG to demonstrate basic Airflow concepts.

### Kubernetes (k8s) Configuration

- `dashboard-adminuser.yaml`: YAML file for setting up an admin user serviceAccount for the Kubernetes Dashboard.
- `dashboard-clusterrole.yaml`: YAML file defining the cluster role for the Kubernetes Dashboard.
- `dashboard-secret.yaml`: YAML file for managing secrets used by the Kubernetes Dashboard.
- `recommended-dashboard.yaml`: YAML file for deploying the recommended Kubernetes Dashboard setup.
- `values.yaml`: YAML file containing values for customizing the Kubernetes setup.

## Getting Started

### Prerequisites

- A Kubernetes cluster
- `kubectl` installed and configured
- Helm (optional, but recommended for managing Kubernetes applications)

### Setup

1. **Deploy the Kubernetes Dashboard:**

   To deploy the Kubernetes Dashboard, apply the YAML files in the `k8s` directory:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
   ```

   This will set up the Kubernetes Dashboard with the necessary roles and permissions.

2. **Accessing the Kubernetes Dashboard:**

   To access the Dashboard, you may need to start a proxy server:

   ```bash
   kubectl proxy
   ```

   Then, access the Dashboard at: `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`.

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

   Use the token generated for the admin user to log in (`dashboard-secret.yaml`,`dashboard-clusterrole.yaml`).

   ```bash
   kubectl apply -f dashboard-adminuser.yaml
   kubectl apply -f dashboard-clusterrole.yaml 
   kubectl apply -f dashboard-secret.yaml  
   k get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
   ```

3. **Deploy Apache Airflow:**

   You can deploy Apache Airflow using Helm or by applying custom YAML files. For Helm:

   ```bash
   <!-- ariflow install -->
   helm repo add apache-airflow https://airflow.apache.org
   helm install airflow apache-airflow/airflow --namespace airflow --create-namespace --debug
   <!-- ariflow port-forward -->
   kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow (default id/pw : admin / admin)
   <!-- extract aiflow values -->
   helm show values apache-airflow/airflow > values.yaml
   <!-- get aiflow secret fernetkey -->
   kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode
   <!-- upgrade airflow -->
   helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace -f k8s/values.yaml
   <!-- install apache-airflow -->
   pip3 install apache-airflow
   helm install airflow apache-airflow/airflow -f k8s/values.yaml
   ```

   This will deploy Airflow with the settings defined in `values.yaml`.

4. **Adding DAGs to Airflow:**

   Copy your DAG files (e.g., `fetch_and_preview.py`, `hello.py`) into the DAGs folder of your Airflow deployment. The method of copying depends on your Airflow setup (e.g., using Persistent Volume, Git-sync).

### Usage

- **Kubernetes Dashboard:** Use the Dashboard to monitor and manage the Kubernetes cluster.
- **Apache Airflow:** Access the Airflow web UI to manage, schedule, and monitor workflows.
