# Installation on a macOS 

1) Launch **Docker Desktop** and, if needed, manually increase the available disk space: navigate to **Docker Desktop → Settings → Resources → Disk image size**, and adjust as needed. 

> **Note**: Storage allocations should follow this order in size: **Minikube storage > Helm chart storage > Skaffold storage**. 

2) Start **Minikube** using the Docker driver, allocating sufficient memory and CPU: 

```
minikube start --memory=7000MB --cpus=4 --driver=docker`
```

3) Add **Helm repositories** and install the required chart:
 
```
helm repo add community-charts https://community-charts.github.io/helm-charts
helm repo add localstack https://helm.localstack.cloud
helm repo add rimusz https://charts.rimusz.net
helm repo update
helm upgrade --install hostpath-provisioner --namespace kube-system rimusz/hostpath-provisioner
```

4) Deploy using **Skaffold**, specifying the correct architecture compatible with macOS `linux/amd64`:
```
skaffold dev --platform linux/amd64
```

5) Access your services: After a few minutes, if the deploy is successful the terminal will display the URLs for accessing the services `code-server` and `mlflow`:

```
code-server: http://localhost:8000
mlflow: http://localhost:5000
```

### **Troubleshooting**  

If **Code Server** or **Mlflow** are not reachable at `http://127.0.0.1:8000` and `http://127.0.0.1:5000` respectively, the ports may be in use by another process. To free the ports and allow Kubernetes to re-establish port forwarding, run:  

```sh
sudo fuser -k 8000/tcp
sudo fuser -k 5000/tcp
```