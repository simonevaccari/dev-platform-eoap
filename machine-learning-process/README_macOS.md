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

## **Troubleshooting**  

If **Code Server** or **Mlflow** are not reachable at `http://127.0.0.1:8000` and `http://127.0.0.1:5000` respectively, the ports may be in use by another process. To free the ports and allow Kubernetes to re-establish port forwarding, run:  

```sh
sudo fuser -k 8000/tcp
sudo fuser -k 5000/tcp
```

# Outstanding Issues

We're encountering problems when building and running Docer containers inside a Kubernetes pod created using our Minikube + Skaffold development workflow on macOS. These issues appear from architecture mismatches, specifically when attempting to use `amd64` containers on an Apple Silicon (`arm64`) machine (e.g. M1, M2, M3). 

To enable seamless cross-platform container builds and execution (e.g., building amd64 images on an arm64 host), we recommend setting up support for multi-architecture emulation using [tonigstiigi/binfmt](https://hub.docker.com/r/tonistiigi/binfmt), which is a Docker-compatible image that installs the required QEMU-based emulators via `binfmt_misc`.

1) **Enable Multi-Architecture Support via `binfmt_misc`**:

Run the following command once on your host machine to enable emulation of amd64 (and other platforms) via QEMU.

```
docker run --privileged --rm tonistiigi/binfmt --install all
```

This sets up the emulation layer required to run amd64 images on your arm64 system.

> Note: Steps 2 and 3 below **may not be directly relevant** when your container is building/running images *from inside a pod*. In that case, the key requirement is to ensure the Kubernetes pod itself is running in the correct configuration (e.g., with privileged security context and compatible Docker runtime settings) to support cross-platform Docker operations.

2) **(Optional) Pull the Correct Architecture Image**:

If you're pulling images manually for local testing or debugging, explicitly specify the target platform:
```
docker pull --platform=linux/amd64 <your_image>
```

> Tip: add this `--platform=linux/amd64` flag to your skaffold.yaml or Docker build scripts if needed to ensure consistency. Note it was added in the `skaffold dev` command above. 

3) **(Optional) Load Image into Minikube**

To make a locally built image available to Minikube (without pushing it to a registry), use:

```
minikube image load <your_image>
```

> Ensure Minikube is configured to use the same Docker context as your host (docker driver), or use the command below to configure your shell environment to point to Minikube's internal Docker daemon: 
> `eval $(minikube docker-env)`