# Helm Example for Travels app

### To package your application (travelsapp) using Helm, you need to follow these full steps:

---

✅ Step 1: Install Helm (if not already installed)

On Linux/macOS:

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Verify:

helm version


---

✅ Step 2: Create a Helm Chart for travelsapp

helm create travelsapp

This generates a directory structure like:

```
travelsapp/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl

```


---

✅ Step 3: Customize Chart.yaml

Update the Chart.yaml file:

```
apiVersion: v2
name: travelsapp
description: A Helm chart for Kubernetes deployment of Travels App
type: application
version: 0.1.0
appVersion: "1.0"

```


---

✅ Step 4: Customize values.yaml

Set your app-specific values:

replicaCount: 2

image:
  repository: your-dockerhub-username/travelsapp
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"


---

✅ Step 5: Edit templates/deployment.yaml

Helm already generates this, but make sure it uses your values.yaml fields:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
          resources:
            {{- toYaml .Values.resources | nindent 12 }}


---

✅ Step 6: Edit templates/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 80


---

✅ Step 7: Package the Chart

helm package travelsapp

This generates a .tgz file, e.g.:

travelsapp-0.1.0.tgz


---

✅ Step 8: Install or Upgrade the Chart in Kubernetes

# Install
helm install travelsapp travelsapp-0.1.0.tgz

# OR upgrade
helm upgrade travelsapp travelsapp-0.1.0.tgz


---

✅ Step 9: Verify Installation

helm list
kubectl get all


---

✅ Step 10: Optional - Enable Ingress

Edit values.yaml:

ingress:
  enabled: true
  className: nginx
  annotations: {}
  hosts:
    - host: travelsapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls: []

And customize templates/ingress.yaml.


---

