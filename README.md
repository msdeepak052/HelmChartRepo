# Helm Chart
---

## 🎯 What is Helm?

> **Helm** is the **package manager for Kubernetes**.

Just like:

* `apt` for Ubuntu
* `yum` for RHEL
* `npm` for Node.js

👉 **Helm helps you package, share, and deploy Kubernetes applications easily.**

---

## ⚙️ Helm Architecture

Helm has:

* **Charts**: Package of YAML manifests + metadata
* **Templates**: Variables and conditionals (`{{ }}`) in YAML
* **values.yaml**: User-defined config to inject into templates


![image](https://github.com/user-attachments/assets/2ce924e1-0f96-4e72-8b5e-67b85869479d)


---

## 🔁 Common Helm Workflow

```bash
# 1. Add a chart repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# 2. Search for a chart
helm search repo nginx

# 3. Install a chart
helm install my-nginx bitnami/nginx

# 4. List releases
helm list

# 5. Uninstall
helm uninstall my-nginx
```

---

## 📦 Chart Directory Structure (Custom Helm Chart)

```bash
helm create mychart
```

This creates:

```
mychart/
├── Chart.yaml            # Metadata
├── values.yaml           # Configurable variables
├── charts/               # Dependencies (optional)
├── templates/            # Kubernetes YAML files (templated)
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl      # Template functions/macros
```

---

## ✏️ Step-by-Step: Build Your Own Helm Chart

Let’s build a chart called `webapp-chart` for a simple Nginx deployment.

---

### ✅ Step 1: Create Chart

```bash
helm create webapp-chart
```

---

### ✅ Step 2: Modify `values.yaml`

Edit this section:

```yaml
replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80
```

---

### ✅ Step 3: Customize Deployment Template

File: `templates/deployment.yaml`

Look for this section and understand the templating:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}

  containers:
    - name: {{ .Chart.Name }}
      image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

👉 These variables pull values from `values.yaml`.

---

### ✅ Step 4: Install the Chart

```bash
helm install my-nginx ./webapp-chart
```

> This creates a deployment, service, etc., using your chart.

---

### ✅ Step 5: Customize More Using CLI Overrides

```bash
helm install my-nginx ./webapp-chart \
  --set replicaCount=3 \
  --set image.tag=1.21
```

---

## 🔍 Inspect Rendered YAML (Dry Run)

```bash
helm template my-nginx ./webapp-chart
```

or

```bash
helm install my-nginx ./webapp-chart --dry-run --debug
```

---

## 🚀 Example Full Manifest After Render

When rendered, the `deployment.yaml` would look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: webapp-chart
  template:
    metadata:
      labels:
        app.kubernetes.io/name: webapp-chart
    spec:
      containers:
        - name: webapp-chart
          image: nginx:latest
          ports:
            - containerPort: 80
```

---

## 🧹 Cleanup

```bash
helm uninstall my-nginx
```

---

## 📦 Package the Chart

```bash
helm package ./webapp-chart
```

This generates a `.tgz` file that you can share.

---

## 🌐 Host Your Own Helm Repo (Optional)

```bash
helm repo index ./charts
```

You can now host the chart on GitHub Pages, S3, etc.

---

Perfectly structured steps, Deepak! 👏 Here's the **final enhancement** to your Helm setup for the `travelsapp`, including a **ready-to-use Ingress template**, and some extra **best practices** you might want to apply.

---

## ✅ Final Helm Chart Touch – Add `templates/ingress.yaml`

If you're enabling Ingress (step 10), you must define this file properly:

### `templates/ingress.yaml`

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  labels:
    app: {{ .Release.Name }}
  annotations:
    {{- range $key, $value := .Values.ingress.annotations }}
    {{ $key }}: {{ $value | quote }}
    {{- end }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ $.Release.Name }}-service
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}

  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
{{- end }}
```

---

## ✅ Optional: Add a NOTES.txt (for user instructions post-install)

**`templates/NOTES.txt`**

```txt
1. Get the application URL by running these commands:

{{- if .Values.ingress.enabled }}
  export HOST={{ (index .Values.ingress.hosts 0).host }}
  echo "Visit http://$HOST"
{{- else }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
  kubectl port-forward $POD_NAME 8080:80
  echo "Visit http://localhost:8080"
{{- end }}
```

> This will print handy post-install instructions.

---

## ✅ Bonus: Custom `helm install` with overridden values

You can override values **without modifying the chart**:

```bash
helm install travelsapp ./travelsapp \
  --set image.tag=1.2 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=travelsapp.deepak.in \
  --set service.type=LoadBalancer
```

---

## 📦 Optional: Push to ArtifactHub or your own Helm repo

```bash
# Package
helm package travelsapp

# Create Helm repo metadata
helm repo index .

# You can host this on GitHub Pages, S3, etc.
```

---

## ✅ Final Manifest Output Summary (Generated by Helm)

After a full install, Helm will generate and deploy:

| Resource   | File                                    |
| ---------- | --------------------------------------- |
| Deployment | `templates/deployment.yaml`             |
| Service    | `templates/service.yaml`                |
| Ingress    | `templates/ingress.yaml` *(if enabled)* |
| Notes/help | `templates/NOTES.txt`                   |

---



## 🧠 Summary

| Task          | Helm Command                            |
| ------------- | --------------------------------------- |
| Add repo      | `helm repo add`                         |
| Install chart | `helm install`                          |
| Create chart  | `helm create`                           |
| Customize     | Edit `values.yaml` & `templates/*.yaml` |
| Render YAML   | `helm template`                         |
| Delete        | `helm uninstall`                        |

---

Would you like a real app example (like a Flask or Node app with Helm and Ingress), or a CI/CD pipeline deploying a Helm chart via Jenkins or GitHub Actions?

Let me know!

