# K8s-powered-by-GenAI

PartyRock app "Kubernetes Manifest Generator": https://partyrock.aws/u/fspiess/MnqyMNxWc/Kubernetes-Manifest-Generator

## Entry for text box "Kubernetes Requirements":
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

This manifest file includes:

* A Deployment that runs three replicas of an Nginx container.
* A ConfigMap that contains the HTML content for the "Hello World" page.
* A Service that exposes the web server.

Key features and best practices:

* Uses the latest stable API versions.
* Includes resource requests and limits for the container.
* Implements readiness and liveness probes for better health checking.
* Uses a ConfigMap to manage the HTML content, making it easy to update without changing the container image.
* Follows naming conventions and uses labels consistently.
* Separates different resources with --- for better readability.

To apply this manifest, go the the Linux terminal and save it to a file (e.g., hello-world-web.yaml) and run:

```
kubectl apply -f hello-world-web.yaml
```

## Entry in text box "Kubernetes Manifest assistant":
```
How can I do port forwarding to reach the above website from outside of the worker nodes?
```
Result, to be posted into the Linux terminal:
```
kubectl port-forward service/hello-world-service 8080:80
```
Actually, this command thould be ended with a " &" to make sure that it runs in the background, and we get back our Linux terminal prompt:

```
kubectl port-forward service/hello-world-service 8080:80 &
```

Now, by performing a "wget" for the related service, you can download the index.html from the webserver:
```
wget http://localhost:8080
```


## New requirement, again to be added in the "Kubernetes Manifest assistant" text box:
```
Instead of serving static content, my webserver should now respond with a random number that changes after each request.
```
Answer from the PartyRock app:

Certainly! To serve a random number that changes with each request, we'll need to modify our setup. Instead of using a static nginx server with a ConfigMap, we'll create a simple web application that generates a random number for each request. Here's how we can modify the manifest to achieve this:

* We'll use a Python web application with Flask.
* We'll create a custom Docker image for this application.
* We'll update our Kubernetes manifest to use this new image.

Here's the step-by-step process:

First, create a Python script (app.py) for the web application:
``` python
from flask import Flask
import random

app = Flask(__name__)

@app.route('/')

def hello():
    return str(random.randint(1, 1000000))

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
```
Create a Dockerfile:
```
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
EXPOSE 8080
CMD ["python", "app.py"]
```
Build and push the Docker image to a registry (replace your-registry with your actual registry):
```
docker build -t your-registry/random-number-app:v1 .
```
```
docker push your-registry/random-number-app:v1
```
Now, let's update the Kubernetes manifest:
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: random-number-app
  labels:
    app: random-number-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: random-number-app
  template:
    metadata:
      labels:
        app: random-number-app
    spec:
      containers:
      - name: random-number-container
        image: your-registry/random-number-app:v1
        ports:
        - containerPort: 8080
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
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: random-number-service
spec:
  selector:
    app: random-number-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
Key changes:

* Updated the image to use our custom random number application.
* Changed the container port to 8080 to match our Flask application.
* Updated the probes to use port 8080.
* Removed the ConfigMap as it's no longer needed.
* Updated the Service to forward to port 8080.

Apply this updated manifest in the Linux terminal:
```
kubectl apply -f random-number-app.yaml
```
Now, each time you access the service, you'll get a different random number. You can use the same methods we discussed earlier (NodePort, LoadBalancer, Ingress, or port-forwarding) to access this service from outside the cluster.

Would you like me to explain any part of this setup in more detail?
