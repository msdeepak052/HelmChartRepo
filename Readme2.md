Here is your **Helm example for `travelsapp`** with proper **indentation and formatting**:

---

# 🚀 Helm Example for Travels App

### 🎯 Goal: Package and deploy your `travelsapp` using Helm.

---

![image](https://github.com/user-attachments/assets/1db59fd9-4f61-4a51-99c5-aec0d928787c)


### ✅ Step 1: Install Helm

**On Linux/macOS:**

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify installation:**

```bash
helm version
```

---

### ✅ Step 2: Create Helm Chart

```bash
helm create travelsapp
```

**Generated directory structure:**

```
travelsapp/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── _helpers.tpl
```

---

### ✅ Step 3: Customize `Chart.yaml`

```yaml
apiVersion: v2
name: travelsapp
description: A Helm chart for Kubernetes deployment of Travels App
type: application
version: 0.1.0
appVersion: "1.0"
```

---

### ✅ Step 4: Customize `values.yaml`

```yaml
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
```

---

### ✅ Step 5: Edit `templates/deployment.yaml`

```yaml
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
```

---

### ✅ Step 6: Edit `templates/service.yaml`

```yaml
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
```

---

### ✅ Step 7: Package the Chart

```bash
helm package travelsapp
```

**Generates:**

```
travelsapp-0.1.0.tgz
```

---

### ✅ Step 8: Install or Upgrade the Chart

```bash
# Install
helm install travelsapp travelsapp-0.1.0.tgz

# OR Upgrade
helm upgrade travelsapp travelsapp-0.1.0.tgz
```

---

### ✅ Step 9: Verify Installation

```bash
helm list
kubectl get all
```

---

### ✅ Step 10: (Optional) Enable Ingress

Update `values.yaml`:

```yaml
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
```

Then edit `templates/ingress.yaml` to reflect those values.

---

Let me know if you want a sample `ingress.yaml` template too!
