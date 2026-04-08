# llm-eks-ollama
📘 LLM Deployment on EKS using Ollama + Open WebUI (Helm)
🚀 Project Overview

This project demonstrates how to deploy a local LLM (Llama3) on Kubernetes using:

Ollama → LLM runtime
Open WebUI → ChatGPT-like UI
Helm → Package management
EKS (KodeKloud Playground) → Kubernetes cluster
🧠 Architecture
User (Browser)
     ↓
Open WebUI (NodePort / Port-forward)
     ↓
Ollama Service (ClusterIP)
     ↓
LLM Model (llama3)
⚙️ Prerequisites
Kubernetes Cluster (EKS / KodeKloud)
kubectl configured
Helm installed
kubectl get nodes
helm version
📦 Step 1: Create Namespace
kubectl create namespace ollama
🚀 Step 2: Add Helm Repositories
helm repo add ollama-helm https://otwld.github.io/ollama-helm/
helm repo add open-webui https://open-webui.github.io/helm-charts
helm repo update
🧾 Step 3: Create values.yaml (Ollama)
replicaCount: 1

image:
  repository: ollama/ollama
  pullPolicy: IfNotPresent

ollama:
  port: 11434
  gpu:
    enabled: false
  models:
    pull:
      - llama3

service:
  type: ClusterIP
  port: 11434

resources:
  requests:
    cpu: "1"
    memory: "2Gi"
  limits:
    cpu: "2"
    memory: "4Gi"

persistentVolume:
  enabled: false   # Disabled for playground

autoscaling:
  enabled: false
🚀 Step 4: Deploy Ollama
helm upgrade --install ollama ollama-helm/ollama \
  -f values.yaml \
  -n ollama
🔍 Step 5: Verify Ollama
kubectl get pods -n ollama
kubectl logs -f deploy/ollama -n ollama

👉 Look for:

pulling llama3...
🧾 Step 6: Create webui-values.yaml
replicaCount: 1

image:
  repository: ghcr.io/open-webui/open-webui
  tag: latest

service:
  type: NodePort
  port: 80
  nodePort: 30080

ollama:
  enabled: true
  baseUrl: http://ollama:11434

resources:
  requests:
    cpu: "200m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"

persistence:
  enabled: false

pipelines:
  enabled: false

autoscaling:
  enabled: false
🚀 Step 7: Deploy Open WebUI
helm upgrade --install open-webui open-webui/open-webui \
  -f webui-values.yaml \
  -n ollama
🌐 Step 8: Access UI
Option 1: NodePort
kubectl get svc -n ollama
kubectl get nodes -o wide

Open:

http://<NODE-IP>:30080
Option 2: Port Forward
kubectl port-forward svc/open-webui 8080:80 -n ollama

Open:

http://localhost:8080
🔐 Step 9: Login & Test
Create account
Select model → llama3
Ask:
Explain Kubernetes in simple terms
⚠️ Troubleshooting
❌ PVC Error
pod has unbound PersistentVolumeClaims

👉 Fix:

persistentVolume:
  enabled: false
❌ No Model in UI
kubectl exec -it deploy/ollama -n ollama -- ollama list

If empty:

ollama pull llama3
❌ Connection Issue

Check:

baseUrl: http://ollama:11434
