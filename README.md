# node-app-helm
To deploy a basic node.js application using helm

# Node.js Kubernetes Deployment with Helm

## Introduction
This project demonstrates how to deploy a simple Node.js application on a Kubernetes cluster using Helm. The application is exposed via an Nginx ingress controller.

## Prerequisites
- A local Kubernetes environment (e.g., Minikube or kind).
- Helm installed and configured.
- Basic knowledge of Nginx and Node.js.

## Steps Followed

### 1. Setting Up the Kubernetes Cluster
- Enable Kubernetes in Docker Desktop:
  - Open Docker Desktop.
  - Go to "Settings" > "Kubernetes".
  - Check "Enable Kubernetes".
  - Click "Apply & Restart".

- Ensure `kubectl` is configured:

  kubectl config use-context docker-desktop

### 2. Create a basic Node.js application:
- Copy the code and Dockerfile from node-app and build the image using below command

    docker build -t node-app2 .
    docker run -p 3000:3000 node-app2

### 3. Set up the nginx ingress controller using below commands
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm install ingress-nginx ingress-nginx/ingress-nginx 


### 4. Create a helm chart directory and copy all the files from helm-chart directory
- Run below command to deploy the application using helm after going to the helm directory
    helm install node-app . 

### 5. To update, make changes in maybe image pull policy and run below command
    helm upgrade node-app .

### 6. To rollback
    helm rollback node-app <revision-number>
    ### You can get revision number using helm history node-app

For Production grade setup, we should add below objects and strategies to our deployment:

1. liveness/readiness probe
### Inside code

// Liveness probe endpoint
app.get('/healthz', (req, res) => {
  res.status(200).send('OK');
});

// Readiness probe endpoint
app.get('/readyz', (req, res) => {
  // Perform some checks here if necessary
  res.status(200).send('OK');

### Inside deployment

livenessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10
readinessProbe:
            httpGet:
              path: /readyz
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10

2. RollingUpdateStrategy

strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

3. HPA Integration

### Configure resource limits and requests

resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"

Adding hpa.yaml

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nodejs-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nodejs-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

4. Enabling monitoring by having metrics endpoints and kube-prometheus-stack integration using service monitors

5. Enabling alerts using alertmanager

6. Creating Notes.txt

7. Creating helpers.tpl

{{- define "nodejs-chart.fullname" -}}
{{- include "nodejs-chart.name" . }}-{{ .Release.Name | trunc 63 | trimSuffix "-" }}
{{- end -}}

{{- define "nodejs-chart.name" -}}
{{- if .Values.nameOverride }}
{{- .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- else -}}
{{- .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end -}}
{{- end -}}



