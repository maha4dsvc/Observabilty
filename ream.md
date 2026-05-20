
- https://docs.aws.amazon.com/eks/latest/eksctl/installation.html 

- Install EKSCTL:
- Download   EKSCTL 
https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_windows_arm64.zip
- Extract zip into program file 
- Set  classpath 

## Or 

# Open Powershell as administrator  and execute command 
```bash
$ErrorActionPreference = "Stop"; `
$installPath = "C:\Program Files\eksctl"; `
$zipPath = "$env:TEMP\eksctl_windows_amd64.zip"; `
Write-Host "📦 Downloading eksctl..." -ForegroundColor Cyan; `
Invoke-WebRequest -Uri "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_windows_amd64.zip" -OutFile $zipPath; `
Write-Host "📂 Extracting..."; `
Expand-Archive -Force $zipPath -DestinationPath $env:TEMP; `
New-Item -ItemType Directory -Force -Path $installPath | Out-Null; `
Move-Item -Force "$env:TEMP\eksctl.exe" "$installPath\eksctl.exe"; `
Unblock-File "$installPath\eksctl.exe"; `
$envPath = [Environment]::GetEnvironmentVariable("Path", "Machine"); `
if ($envPath -notlike "*$installPath*") { `
    Write-Host "🔧 Adding eksctl to PATH..."; `
    [Environment]::SetEnvironmentVariable("Path", "$envPath;$installPath", "Machine"); `
} `
Write-Host "✅ eksctl installed successfully! Please restart PowerShell, then run:" -ForegroundColor Green; `
Write-Host "   eksctl version"
```


# For login into AWS account from  your laptop by CLI 
- Create  IAM admin user
- Install  chocolatey on your laptop by  powershell as an  administrator 
- Install AWSCLI by using Chocolatey (choco install awscli)
- Close  and reopen powershell
- Try to login with aws ( aws configure)
- Then we can execute aws cli commands

##Install Kubectle on your laptop:
```bash
choco install kubernetes-cli -y
Kubectl 
```
# Step 1: Create EKS Cluster

```bash
eksctl create cluster --name=observability1  --region=us-west-2 --zones=us-west-2a,us-west-2b --without-nodegroup
```

```bash
eksctl utils associate-iam-oidc-provider --region us-west-2 --cluster observability1 --approve
```

```bash
eksctl create nodegroup --cluster=observability1 --region=us-west-2   --name=observability1-ng-private                    --node-type=t3.medium --nodes-min=2 --nodes-max=3 --node-volume-size=20 --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access --node-private-networking
```

```bash
# Update ./kube/config file
aws eks update-kubeconfig --name observability1
```

Step 2: Install kube-prometheus-stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

# Step 3: Deploy the chart into a new namespace "monitoring"
```bash
kubectl create ns monitoring
```
```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring -f ./custom_kube_prometheus_stack.yml
```

```bash
kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-user}" | base64 --decode ; echo 
```

# Step 4: Verify the Installation
```bash
kubectl get all -n monitoring
```

### Prometheus UI: 
```bash
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
```
### Grafana UI: password is prom-operator 
```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
```
### Alertmanager UI: 
```bash
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
```

### see Exports:

```bash
kubectl get pods -n monitoring
```

# Step 5: Clean UP
### Uninstall helm chart: 

```bash
helm uninstall monitoring --namespace monitoring
```
### Delete namespace: 
```bash
kubectl delete ns monitoring
```
### Delete Cluster & everything else: 

```bash
eksctl delete cluster --name observability1
```




