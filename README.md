# K8s-powered-by-GenAI

PartyRock app "Kubernetes Manifest Generator": https://partyrock.aws/u/fspiess/MnqyMNxWc/Kubernetes-Manifest-Generator

Entry for text box "Kubernetes Requirements":
```
I want to run a webserver that serves a web page with content "Hello World".
```

Manifest file output:
``` yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-web
  labels:
    app: hello-world-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-web
  template:
    metadata:
      labels:
        app: hello-world-web
    spec:
      containers:
      - name: hello-world-container
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        volumeMounts:
        - name: html-content
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-content
        configMap:
          name: hello-world-html
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world-html
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Hello World</title>
    </head>
    <body>
      <h1>Hello World</h1>
    </body>
    </html>
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  selector:
    app: hello-world-web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
