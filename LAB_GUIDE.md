# Kubernetes Minikube Lab Guide (Windows)

This guide documents the step-by-step process to deploy the "MyService" application on a local Minikube cluster. This setup mimics a production environment using Deployments, Services (LoadBalancer), and Ingress on Windows.

## Prerequisites
- Docker Desktop installed and running.
- Minikube installed.
- PowerShell terminal.

---

## Step 1: Start Minikube
**Why:** First, we need a working Kubernetes cluster. Accessing the cluster requires Minikube to be running.
**Command:**
```powershell
minikube start
```

## Step 2: Connect Docker to Minikube
**Why:** By default, `docker build` builds images on your host Windows machine. Minikube runs in a separate isolated container/VM and cannot see your host images. This command forces your current PowerShell terminal to talk directly to the Docker engine *inside* Minikube.
**Command:**
```powershell
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
```
*Verification: Run `docker images` and you should see k8s containers.*

## Step 3: Build the Docker Image
**Why:** Kubernetes needs a container image to run the app. We build it inside Minikube so it's instantly available to the cluster without pushing to Docker Hub.
**Command:**
```powershell
docker build -t franckfozie2023/myservice:1 ./MyService
```

## Step 4: Deploy the Application
**Why:** The `Deployment` creates and manages the Pods (containers) for your application.
**Command:**
```powershell
kubectl apply -f myservice-deployment.yml
```

## Step 5: Expose the Service (LoadBalancer)
**Why:** To access the application from outside the cluster, we use a Service. We use `type: LoadBalancer` with a specific port configuration to avoid conflicts with local services (like Jenkins on port 8080).
**Command:**
```powershell
kubectl apply -f myservice-loadbalancing-service.yml
```
*Note: We configured the external port to be `8085` to avoid conflicts.*

## Step 6: Start Minikube Tunnel
**Why:** On Windows (and macOS), Minikube cannot assign real LoadBalancer IPs automatically. The `tunnel` command acts as a bridge that routes traffic from your computer to the cluster services.
**Command:**
(This MUST run in a separate terminal window and stay open)
```powershell
minikube tunnel
```
*Test: Open `http://127.0.0.1:8085` in your browser.*

## Step 7: Configure Ingress (Domain Name)
**Why:** In production, we use domain names, not IPs. The Ingress Controller acts as a reverse proxy (like Nginx) to route traffic based on domains.

**1. Enable Ingress Addon:**
```powershell
minikube addons enable ingress
```

**2. Apply Ingress Rules:**
**Why:** Tells the cluster that `myservice.info` should route to our service.
```powershell
kubectl apply -f ingress.yml
```

**3. Configure DNS (Hosts File):**
**Why:** Your computer doesn't know where `myservice.info` is. We manually map it to localhost.
*   Open Notepad as **Administrator**.
*   Edit file: `C:\Windows\System32\drivers\etc\hosts`
*   Add the line:
    ```
    127.0.0.1 myservice.info
    ```

## Final Test
Open your browser and navigate to:
[http://myservice.info](http://myservice.info)

---

## Useful Debug Commands
- Check pods: `kubectl get pods`
- Check services: `kubectl get svc`
- Check logs: `kubectl logs <pod-name>`
- Restart deployment (after code change): `kubectl rollout restart deployment/myservice`
