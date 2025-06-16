---

````markdown
# 📊 Flask App with Metrics using Prometheus & Grafana on Minikube

This mini-project demonstrates how to deploy a Flask application exposing custom metrics, which are scraped by Prometheus and visualized using Grafana — all running on a local Minikube Kubernetes cluster.

---

## 📁 Project Structure

```bash
.
├── Dockerfile
├── flask_app.py
├── flask-app.yaml
└── README.md
````

---

## 🚀 1. Run the Flask App Locally (Optional)

You can verify that the app works locally before deploying:

```bash
python flask_app.py
```

---

## 🐳 2. Deploy Flask App on Minikube

### 🔧 Step-by-step:

1. **Start Minikube**

   ```bash
   minikube start
   ```

2. **Build Docker Image Inside Minikube**

   ```bash
   eval $(minikube docker-env)
   docker build -t flask-metrics-app:latest .
   ```

3. **(OR) Load Prebuilt Image into Minikube**

   ```bash
   minikube image load flask-metrics-app:latest
   ```

4. **Deploy the App**

   ```bash
   kubectl apply -f flask-app.yaml
   ```

5. **Access the Flask App**

   ```bash
   minikube service flask-metrics-app --url
   ```

---

## 📈 3. Install Prometheus Using Helm

1. **Install Helm**
   On Windows:

   ```bash
   choco install kubernetes-helm
   ```

2. **Add Prometheus Repo & Install**

   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
   ```

3. **Access Prometheus Dashboard**

   ```bash
   kubectl port-forward -n monitoring svc/prometheus-server 9090:80
   ```

   Open: [http://localhost:9090](http://localhost:9090)

---

## ⚙️ 4. Configure Prometheus to Scrape Flask Metrics

1. **Get Flask App Cluster IP**

   ```bash
   kubectl get svc
   ```

   Example: `10.107.61.186:5000`

2. **Edit Prometheus ConfigMap**

   ```bash
   kubectl edit configmap prometheus-server -n monitoring
   ```

3. **Add Target Under `scrape_configs`:**

   ```yaml
   scrape_configs:
     - job_name: 'flask-app'
       static_configs:
         - targets: ['10.107.61.186:5000']
   ```

4. **Restart Prometheus**

   ```bash
   kubectl rollout restart deployment prometheus-server -n monitoring
   ```

---

## 📊 5. Install Grafana via Helm

1. **Add Grafana Repo**

   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo update
   ```

2. **Install Grafana**

   ```bash
   helm install grafana grafana/grafana -n monitoring --create-namespace
   ```

3. **Port Forward Grafana**

   ```bash
   kubectl port-forward svc/grafana -n monitoring 3000:80
   ```

   Open: [http://localhost:3000](http://localhost:3000)

---

## 🔐 6. Login to Grafana

* **Username**: `admin`
* **Password**: Run the following:

  ```bash
  # Bash
  kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

  # PowerShell
  kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}"
  ```

  Then decode it using:

  ```powershell
  [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("<YOUR_CODE>"))
  ```

---

## 📡 7. Connect Prometheus as a Data Source in Grafana

1. Go to: `Connections > Data Sources > Add Data Source`
2. Choose **Prometheus**
3. Set URL:

   ```
   http://prometheus-server.monitoring.svc.cluster.local:80
   ```
4. Click **Save & Test**

---

## 📈 8. Create Grafana Dashboards

1. Go to `Dashboard > New > New Panel`
2. Add queries using Prometheus metrics (e.g., `total_api_requests_total`)
3. Customize your visualizations

---

## ✅ Result

You now have a fully functional Flask app monitored by Prometheus and visualized in Grafana — all running locally in Minikube! 🎯

---

## 📎 References

* [Prometheus Helm Chart](https://github.com/prometheus-community/helm-charts)
* [Grafana Helm Chart](https://github.com/grafana/helm-charts)
* [Kubernetes Docs](https://kubernetes.io/docs/)

---

## 💡 Tip: Clean Up

To delete everything:

```bash
kubectl delete -f flask-app.yaml
helm uninstall prometheus -n monitoring
helm uninstall grafana -n monitoring
kubectl delete namespace monitoring
```

---

🧠 **Author**: *Keshav Prasad*
📅 **Date**: *June 2025*

```

---

Let me know if you want a PDF version, GitHub README enhancements (like shields or badges), or integration with GitHub Actions for CI/CD.
```
