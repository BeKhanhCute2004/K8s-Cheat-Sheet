# Kubernetes Ultimate Comprehensive Cheat Sheet

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Pods](#pods)
3. [Deployments](#deployments)
4. [Services](#services)
5. [ConfigMaps and Secrets](#configmaps-and-secrets)
6. [Volumes](#volumes)
7. [Namespaces](#namespaces)
8. [Labels and Selectors](#labels-and-selectors)
9. [ReplicaSets](#replicasets)
10. [StatefulSets](#statefulsets)
11. [DaemonSets](#daemonsets)
12. [Jobs and CronJobs](#jobs-and-cronjobs)
13. [Resource Management](#resource-management)
14. [Probes](#probes)
15. [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler)
16. [Vertical Pod Autoscaler](#vertical-pod-autoscaler)
17. [Ingress](#ingress)
18. [Network Policies](#network-policies)
19. [RBAC](#rbac)
20. [Service Accounts](#service-accounts)
21. [Taints and Tolerations](#taints-and-tolerations)
22. [Affinity](#affinity)
23. [Node Management](#node-management)
24. [Storage](#storage)
25. [Security](#security)
26. [Monitoring and Logging](#monitoring-and-logging)
27. [Debugging and Troubleshooting](#debugging-and-troubleshooting)
28. [Helm](#helm)
29. [Custom Resource Definitions](#custom-resource-definitions)
30. [Operators](#operators)
31. [Multi-Cluster Management](#multi-cluster-management)
32. [Backup and Restore](#backup-and-restore)
33. [Cluster Upgrades](#cluster-upgrades)
34. [Advanced Topics](#advanced-topics)

---

## Core Concepts

### Kubernetes Architecture Components
```bash
# Master Components
# - kube-apiserver: Exposes the Kubernetes API
# - etcd: Consistent and highly-available key value store
# - kube-scheduler: Watches for newly created Pods and selects nodes for them
# - kube-controller-manager: Runs controller processes
# - cloud-controller-manager: Interacts with cloud provider APIs

# Node Components
# - kubelet: Agent that runs on each node
# - kube-proxy: Network proxy that runs on each node
# - Container Runtime: Software responsible for running containers
```
![Kubernetes Architecture](./kubectl_advance_cheatsheet_1.jpg)

### Cluster Information Commands
```bash
# Cluster information
kubectl cluster-info
kubectl cluster-info dump

# API server health
kubectl get --raw='/healthz?verbose'
kubectl get --raw='/livez'
kubectl get --raw='/readyz'

# Version information
kubectl version --short
kubectl version --output=yaml

# Get component statuses (deprecated)
kubectl get componentstatuses

# Get API resources
kubectl api-resources
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
kubectl api-versions

# Explain resource fields
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
```

---

## Pods

### Create Pod - Imperative Commands
```bash
# Create a simple pod
kubectl run nginx-pod --image=nginx

# Create pod with port
kubectl run nginx-pod --image=nginx --port=80

# Create pod with labels
kubectl run nginx-pod --image=nginx --labels="app=web,env=prod"

# Create pod with environment variables
kubectl run nginx-pod --image=nginx --env="ENV=production" --env="LOG_LEVEL=info"

# Create pod with resource limits
kubectl run nginx-pod --image=nginx --requests="cpu=100m,memory=256Mi" --limits="cpu=200m,memory=512Mi"

# Create pod and expose service
kubectl run nginx-pod --image=nginx --port=80 --expose

# Create pod with restart policy
kubectl run nginx-pod --image=nginx --restart=Never

# Create pod in specific namespace
kubectl run nginx-pod --image=nginx --namespace=dev

# Dry run - generate YAML without creating
kubectl run nginx-pod --image=nginx --dry-run=client -o yaml

# Create pod from YAML file
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

# Create pod with specific service account
kubectl run nginx-pod --image=nginx --serviceaccount=my-service-account

# Create pod with node selector
kubectl run nginx-pod --image=nginx --overrides='{"spec": {"nodeSelector": {"disktype": "ssd"}}}'

# Create pod with tolerations
kubectl run nginx-pod --image=nginx --overrides='{"spec": {"tolerations": [{"key": "key", "operator": "Equal", "value": "value", "effect": "NoSchedule"}]}}'

# Create pod with security context
kubectl run secure-pod --image=nginx --overrides='{"spec": {"securityContext": {"runAsUser": 1000, "runAsGroup": 3000, "fsGroup": 2000}}}'

# Create pod with init containers
kubectl run init-pod --image=nginx --overrides='{"spec": {"initContainers": [{"name": "init-container", "image": "busybox", "command": ["sh", "-c", "echo Initializing..."]}]}}'

# Create pod with volumes
kubectl run volume-pod --image=nginx --overrides='{"spec": {"volumes": [{"name": "data", "emptyDir": {}}], "containers": [{"name": "nginx", "image": "nginx", "volumeMounts": [{"name": "data", "mountPath": "/data"}]}]}}'
```

### Pod YAML Examples
```yaml
# Simple Pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
    env: prod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.21
    ports:
    - containerPort: 80
```

```yaml
# Multi-container Pod
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo "logging" >> /var/log/app.log; sleep 30; done']
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
  volumes:
  - name: shared-logs
    emptyDir: {}
```

```yaml
# Pod with Resource Limits and Environment Variables
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app-container
    image: nginx:1.21
    env:
    - name: ENV
      value: "production"
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
    - containerPort: 80
      name: http
```

```yaml
# Pod with Security Context
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: secure-container
    image: nginx:1.21
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```

```yaml
# Pod with Init Containers
apiVersion: v1
kind: Pod
meta
  name: init-pod
spec:
  initContainers:
  - name: init-container
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  containers:
  - name: app-container
    image: nginx:1.21
```

```yaml
# Pod with Lifecycle Hooks
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-pod
spec:
  containers:
  - name: lifecycle-container
    image: nginx:1.21
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "nginx -s quit; while kill -0 $!; do sleep 1; done"]
```

```yaml
# Pod with Node Affinity
apiVersion: v1
kind: Pod
meta
  name: node-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
  containers:
  - name: nginx
    image: nginx:1.21
```

```yaml
# Pod with Tolerations
apiVersion: v1
kind: Pod
metadata:
  name: tolerant-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.21
```

### Pod Management Commands
```bash
# Get pods
kubectl get pods
kubectl get pods -n namespace-name
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide

# Get pods with more details
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Describe pod
kubectl describe pod nginx-pod

# Get pod logs
kubectl logs nginx-pod
kubectl logs nginx-pod -f
kubectl logs nginx-pod --previous
kubectl logs -l app=nginx
kubectl logs nginx-pod --since=1h
kubectl logs nginx-pod --since-time=2023-01-01T10:00:00Z

# Execute commands in pod
kubectl exec -it nginx-pod -- /bin/bash
kubectl exec nginx-pod -- ls /app
kubectl exec -c container-name nginx-pod -- ps aux

# Delete pod
kubectl delete pod nginx-pod
kubectl delete pod nginx-pod --force --grace-period=0
kubectl delete pods -l app=nginx

# Port forward
kubectl port-forward pod/nginx-pod 8080:80
kubectl port-forward service/nginx-service 8080:80

# Copy files
kubectl cp local-file nginx-pod:/remote/path
kubectl cp nginx-pod:/remote/path local-file

# Watch pods
kubectl get pods --watch
kubectl get pods -w

# Get pods by field selector
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.name=nginx-pod

# Sort pods
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by=.status.startTime

# Get pods with custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
kubectl get pods -o custom-columns-file=template.txt
```

---

## Deployments

### Create Deployment - Imperative Commands
```bash
# Create a deployment
kubectl create deployment nginx-deployment --image=nginx

# Create deployment with replicas
kubectl create deployment nginx-deployment --image=nginx --replicas=3

# Create deployment with port
kubectl create deployment nginx-deployment --image=nginx
kubectl expose deployment nginx-deployment --port=80 --target-port=80

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update deployment image
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Rollback deployment
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# View rollout status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Pause/resume deployment
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment

# Create deployment with specific service account
kubectl create deployment nginx-deployment --image=nginx --serviceaccount=my-service-account

# Create deployment with resource limits
kubectl create deployment nginx-deployment --image=nginx --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi -o yaml

# Create deployment with environment variables
kubectl create deployment nginx-deployment --image=nginx --dry-run=client -o yaml | kubectl set env -f - ENV=production -o yaml

# Create deployment from YAML file
kubectl create -f deployment.yaml
kubectl apply -f deployment.yaml
```

### Deployment YAML Examples
```yaml
# Basic Deployment
apiVersion: apps/v1
kind: Deployment
meta
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```yaml
# Deployment with Rolling Update Strategy
apiVersion: apps/v1
kind: Deployment
meta
  name: app-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```yaml
# Deployment with Environment Variables and ConfigMap
apiVersion: apps/v1
kind: Deployment
meta
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.21
        env:
        - name: PORT
          value: "8080"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: database-config
              key: host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: password
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

```yaml
# Deployment with Multiple Containers
apiVersion: apps/v1
kind: Deployment
meta
  name: multi-container-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multi-container
  template:
    meta
      labels:
        app: multi-container
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
      - name: logger
        image: busybox
        command: ['sh', '-c', 'while true; do echo "logging" >> /var/log/app.log; sleep 30; done']
        volumeMounts:
        - name: shared-logs
          mountPath: /var/log
      volumes:
      - name: shared-logs
        emptyDir: {}
```

```yaml
# Deployment with Node Selector
apiVersion: apps/v1
kind: Deployment
meta
  name: node-selector-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-selector
  template:
    metadata:
      labels:
        app: node-selector
    spec:
      nodeSelector:
        disktype: ssd
      containers:
      - name: app
        image: nginx:1.21
```

```yaml
# Deployment with Affinity
apiVersion: apps/v1
kind: Deployment
meta
  name: affinity-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: affinity
  template:
    metadata:
      labels:
        app: affinity
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - affinity
              topologyKey: kubernetes.io/hostname
      containers:
      - name: app
        image: nginx:1.21
```

### Deployment Management Commands
```bash
# Get deployments
kubectl get deployments
kubectl get deployments -n namespace-name
kubectl get deploy

# Describe deployment
kubectl describe deployment nginx-deployment

# Edit deployment
kubectl edit deployment nginx-deployment

# Delete deployment
kubectl delete deployment nginx-deployment

# Get deployment YAML
kubectl get deployment nginx-deployment -o yaml

# Apply deployment from file
kubectl apply -f deployment.yaml

# Check deployment status
kubectl get deployment nginx-deployment -o wide

# Get deployment history
kubectl rollout history deployment/nginx-deployment

# Get specific revision
kubectl rollout history deployment/nginx-deployment --revision=2

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl scale deployment nginx-deployment --current-replicas=2 --replicas=5

# Autoscale deployment
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10

# Update deployment image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Patch deployment
kubectl patch deployment nginx-deployment -p '{"spec":{"replicas":5}}'

# Annotate deployment
kubectl annotate deployment nginx-deployment description="My web deployment"

# Label deployment
kubectl label deployment nginx-deployment version=v2.0

# Get deployment events
kubectl get events --field-selector involvedObject.name=nginx-deployment

# Watch deployments
kubectl get deployments --watch
```

---

## Services

### Create Service - Imperative Commands
```bash
# Expose deployment as ClusterIP service
kubectl expose deployment nginx-deployment --port=80 --target-port=8080

# Expose deployment as NodePort service
kubectl expose deployment nginx-deployment --type=NodePort --port=80

# Expose deployment as LoadBalancer service
kubectl expose deployment nginx-deployment --type=LoadBalancer --port=80

# Create service with specific port mapping
kubectl expose deployment nginx-deployment --port=80 --target-port=8080 --name=nginx-service

# Create service without selector
kubectl create service clusterip nginx-service --tcp=80:8080

# Create headless service
kubectl create service clusterip nginx-service --clusterip="None" --tcp=80:8080

# Create service with specific type
kubectl create service nodeport nginx-service --tcp=80:8080 --node-port=30007

# Create service from YAML file
kubectl create -f service.yaml
kubectl apply -f service.yaml
```

### Service YAML Examples
```yaml
# ClusterIP Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

```yaml
# NodePort Service
apiVersion: v1
kind: Service
meta
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30007
  type: NodePort
```

```yaml
# LoadBalancer Service
apiVersion: v1
kind: Service
meta
  name: nginx-loadbalancer
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

```yaml
# Headless Service
apiVersion: v1
kind: Service
meta
  name: nginx-headless
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  clusterIP: None
```

```yaml
# Service with Multiple Ports
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: web
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
  - name: https
    protocol: TCP
    port: 443
    targetPort: 8443
```

```yaml
# Service with External IPs
apiVersion: v1
kind: Service
metadata:
  name: external-ip-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  externalIPs:
  - 192.168.1.100
  - 192.168.1.101
```

```yaml
# Service with Session Affinity
apiVersion: v1
kind: Service
meta
  name: sticky-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

### Service Management Commands
```bash
# Get services
kubectl get services
kubectl get svc
kubectl get services -n namespace-name

# Describe service
kubectl describe service nginx-service

# Get service details
kubectl get service nginx-service -o wide
kubectl get service nginx-service -o yaml

# Delete service
kubectl delete service nginx-service

# Port forward to service
kubectl port-forward service/nginx-service 8080:80

# Get service endpoints
kubectl get endpoints nginx-service
kubectl describe endpoints nginx-service

# Test service connectivity
kubectl run debug-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://nginx-service

# Get service by type
kubectl get services --field-selector spec.type=LoadBalancer

# Watch services
kubectl get services --watch

# Patch service
kubectl patch service nginx-service -p '{"spec":{"type":"NodePort"}}'

# Annotate service
kubectl annotate service nginx-service description="My web service"

# Label service
kubectl label service nginx-service version=v1.0
```

---

## ConfigMaps and Secrets

### Create ConfigMap - Imperative Commands
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config --from-literal=database.host=localhost --from-literal=database.port=5432

# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.properties

# Create ConfigMap from directory
kubectl create configmap app-config --from-file=./config/

# Create ConfigMap from multiple files
kubectl create configmap app-config --from-file=key1=value1.txt --from-file=key2=value2.txt

# Create ConfigMap from environment file
kubectl create configmap app-config --from-env-file=.env

# Create ConfigMap from YAML file
kubectl create -f configmap.yaml
kubectl apply -f configmap.yaml

# Create ConfigMap with dry run
kubectl create configmap app-config --from-literal=key=value --dry-run=client -o yaml
```

### ConfigMap YAML Examples
```yaml
# ConfigMap with literal values
apiVersion: v1
kind: ConfigMap
meta
  name: app-config

  database.host: "localhost"
  database.port: "5432"
  log.level: "info"
```

```yaml
# ConfigMap with file content
apiVersion: v1
kind: ConfigMap
meta
  name: nginx-config

  nginx.conf: |
    server {
        listen 80;
        server_name localhost;
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
```

```yaml
# ConfigMap with multiple files
apiVersion: v1
kind: ConfigMap
meta
  name: app-config-files
data:
  app.properties: |
    app.name=MyApp
    app.version=1.0
  logging.properties: |
    level=INFO
    file=/var/log/app.log
```

### Create Secret - Imperative Commands
```bash
# Create secret from literal values
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=secretpassword

# Create secret from file
kubectl create secret generic ssl-cert --from-file=tls.crt --from-file=tls.key

# Create secret with specific type
kubectl create secret tls nginx-tls --cert=cert.pem --key=key.pem

# Create docker registry secret
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=user --docker-password=password --docker-email=email@example.com

# Create secret from YAML file
kubectl create -f secret.yaml
kubectl apply -f secret.yaml

# Create secret from environment file
kubectl create secret generic app-secret --from-env-file=.env

# Create secret with dry run
kubectl create secret generic test-secret --from-literal=key=value --dry-run=client -o yaml

# Create secret from base64 encoded data
kubectl create secret generic encoded-secret --from-literal=username=YWRtaW4= --from-literal=password=c2VjcmV0
```

### Secret YAML Examples
```yaml
# Generic Secret
apiVersion: v1
kind: Secret
meta
  name: db-secret
type: Opaque

  username: YWRtaW4=  # base64 encoded "admin"
  password: c2VjcmV0cGFzc3dvcmQ=  # base64 encoded "secretpassword"
```

```yaml
# TLS Secret
apiVersion: v1
kind: Secret
meta
  name: nginx-tls
type: kubernetes.io/tls

  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJV...
```

```yaml
# Docker Registry Secret
apiVersion: v1
kind: Secret
meta
  name: regcred
type: kubernetes.io/dockerconfigjson

  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJ1c2VyIiwicGFzc3dvcmQiOiJwYXNzd29yZCIsImF1dGgiOiJiWmtiaGRrSmxibk5sY25RdVkyOXQifX19
```

### Using ConfigMaps and Secrets in Pods
```yaml
# Pod using ConfigMap and Secret
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.host
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: db-secret
```

```yaml
# Pod using ConfigMap as environment variables
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: db-secret
```

```yaml
# Pod using ConfigMap subPath
apiVersion: v1
kind: Pod
meta
  name: subpath-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config/database.properties
      subPath: database.properties
  volumes:
  - name: config-volume
    configMap:
      name: app-config-files
```

### ConfigMap and Secret Management Commands
```bash
# Get ConfigMaps and Secrets
kubectl get configmaps
kubectl get secrets
kubectl get cm
kubectl get secrets -n namespace-name

# Describe ConfigMap/Secret
kubectl describe configmap app-config
kubectl describe secret db-secret

# Get values (Secrets show only keys)
kubectl get configmap app-config -o yaml
kubectl get secret db-secret -o yaml

# Get specific key from ConfigMap
kubectl get configmap app-config -o jsonpath='{.data.database\.host}'

# Get decoded secret value
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode

# Delete ConfigMap/Secret
kubectl delete configmap app-config
kubectl delete secret db-secret

# Edit ConfigMap/Secret
kubectl edit configmap app-config
kubectl edit secret db-secret

# Patch ConfigMap
kubectl patch configmap app-config -p '{"data":{"log.level":"debug"}}'

# Watch ConfigMaps/Secrets
kubectl get configmaps --watch
kubectl get secrets --watch

# Get secrets by type
kubectl get secrets --field-selector type=kubernetes.io/tls

# Create secret from file content
kubectl create secret generic file-secret --from-file=key=file.txt

# Create secret with multiple keys
kubectl create secret generic multi-secret --from-literal=user=admin --from-literal=pass=secret
```

---

## Volumes

### Volume YAML Examples
```yaml
# emptyDir Volume
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: writer
    image: nginx
    volumeMounts:
    - name: cache-volume
      mountPath: /cache
    - name: tmp-volume
      mountPath: /tmp
  volumes:
  - name: cache-volume
    emptyDir: {}
  - name: tmp-volume
    emptyDir:
      medium: Memory
```

```yaml
# hostPath Volume
apiVersion: v1
kind: Pod
meta
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: host-data
      mountPath: /data
  volumes:
  - name: host-data
    hostPath:
      path: /data
      type: Directory
```

```yaml
# PersistentVolume and PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
---
apiVersion: v1
kind: Pod
meta
  name: pvc-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: pvc-example
```

```yaml
# NFS Volume
apiVersion: v1
kind: Pod
meta
  name: nfs-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: nfs-volume
      mountPath: /mnt/nfs
  volumes:
  - name: nfs-volume
    nfs:
      server: nfs-server.example.com
      path: /share/nfs
```

```yaml
# AWS EBS Volume
apiVersion: v1
kind: Pod
meta
  name: ebs-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: ebs-volume
      mountPath: /data
  volumes:
  - name: ebs-volume
    awsElasticBlockStore:
      volumeID: aws://<availability-zone>/<volume-id>
      fsType: ext4
```

```yaml
# GCE Persistent Disk
apiVersion: v1
kind: Pod
metadata:
  name: gce-pd-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: gce-pd-volume
      mountPath: /data
  volumes:
  - name: gce-pd-volume
    gcePersistentDisk:
      pdName: my-disk
      fsType: ext4
```

```yaml
# Azure Disk Volume
apiVersion: v1
kind: Pod
meta
  name: azure-disk-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: azure-disk-volume
      mountPath: /data
  volumes:
  - name: azure-disk-volume
    azureDisk:
      diskName: myDisk
      diskURI: /subscriptions/{guid}/resourceGroups/{resource-group}/providers/Microsoft.Compute/disks/myDisk
```

```yaml
# CSI Volume
apiVersion: v1
kind: Pod
metadata:
  name: csi-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: csi-volume
      mountPath: /data
  volumes:
  - name: csi-volume
    csi:
      driver: com.example.team/csi-driver
      volumeHandle: data-id
      readOnly: true
```

### Volume Management Commands
```bash
# Get PersistentVolumes
kubectl get pv
kubectl get persistentvolumes

# Get PersistentVolumeClaims
kubectl get pvc
kubectl get persistentvolumeclaims

# Describe PV/PVC
kubectl describe pv pv-example
kubectl describe pvc pvc-example

# Get PV/PVC in YAML format
kubectl get pv pv-example -o yaml
kubectl get pvc pvc-example -o yaml

# Delete PV/PVC
kubectl delete pv pv-example
kubectl delete pvc pvc-example

# Watch PV/PVC
kubectl get pv --watch
kubectl get pvc --watch

# Get PV by status
kubectl get pv --field-selector status.phase=Available

# Patch PVC
kubectl patch pvc pvc-example -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'

# Get storage classes
kubectl get storageclass
kubectl get sc

# Describe storage class
kubectl describe storageclass standard
```

---

## Namespaces

### Create Namespace - Imperative Commands
```bash
# Create namespace
kubectl create namespace dev
kubectl create namespace prod

# Create namespace with labels
kubectl create namespace staging --dry-run=client -o yaml | kubectl label -f - environment=staging --local=true -o yaml | kubectl create -f -

# Create namespace from YAML
kubectl apply -f namespace.yaml

# Create namespace with annotations
kubectl create namespace test --dry-run=client -o yaml | kubectl annotate -f - description="Testing namespace" --local=true -o yaml | kubectl create -f -
```

### Namespace YAML Examples
```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    environment: development
  annotations:
    description: "Development environment"
```

```yaml
# Namespace with Resource Quota
apiVersion: v1
kind: Namespace
metadata:
  name: prod
---
apiVersion: v1
kind: ResourceQuota
meta
  name: compute-resources
  namespace: prod
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "10"
```

### Namespace Management Commands
```bash
# Get namespaces
kubectl get namespaces
kubectl get ns

# Describe namespace
kubectl describe namespace dev

# Set namespace context
kubectl config set-context --current --namespace=dev

# Delete namespace
kubectl delete namespace dev

# Get namespace with labels
kubectl get namespaces --show-labels

# Label namespace
kubectl label namespace dev environment=development

# Annotate namespace
kubectl annotate namespace dev description="Development environment"

# Get resources in specific namespace
kubectl get all -n dev

# Watch namespaces
kubectl get namespaces --watch

# Get namespace by label
kubectl get namespaces -l environment=development
```

---

## Labels and Selectors

### Label Management Commands
```bash
# Add labels to existing resources
kubectl label pod nginx-pod env=production
kubectl label deployment nginx-deployment version=v1.0

# Update label
kubectl label pod nginx-pod env=staging --overwrite

# Remove label
kubectl label pod nginx-pod env-

# Get resources by label
kubectl get pods -l env=production
kubectl get pods -l 'env in (production, staging)'
kubectl get pods --selector=app=nginx

# Get resources by multiple labels
kubectl get pods -l app=nginx,env=production

# Get resources without specific label
kubectl get pods -l '!env'

# Get resources with label existence
kubectl get pods -l env

# Patch resource with labels
kubectl patch pod nginx-pod -p '{"metadata":{"labels":{"tier":"frontend"}}}'
```

### Label YAML Examples
```yaml
# Pod with multiple labels
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
  labels:
    app: web
    env: production
    version: v1.0
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

```yaml
# Deployment with recommended labels
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recommended-deployment
  labels:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/instance: myapp-instance
    app.kubernetes.io/version: v1.0.0
    app.kubernetes.io/component: web
    app.kubernetes.io/part-of: myplatform
    app.kubernetes.io/managed-by: helm
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: myapp
      app.kubernetes.io/instance: myapp-instance
  template:
    metadata:
      labels:
        app.kubernetes.io/name: myapp
        app.kubernetes.io/instance: myapp-instance
        app.kubernetes.io/version: v1.0.0
        app.kubernetes.io/component: web
    spec:
      containers:
      - name: app
        image: myapp:v1.0.0
```

---

## ReplicaSets

### ReplicaSet YAML Examples
```yaml
# ReplicaSet
apiVersion: apps/v1
kind: ReplicaSet
meta
  name: nginx-replicaset
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```yaml
# ReplicaSet with MatchExpressions
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset-expressions
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
    matchExpressions:
    - key: version
      operator: In
      values:
      - v1.0
      - v2.0
  template:
    metadata:
      labels:
        app: nginx
        version: v1.0
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### ReplicaSet Management Commands
```bash
# Get ReplicaSets
kubectl get replicasets
kubectl get rs

# Describe ReplicaSet
kubectl describe replicaset nginx-replicaset

# Scale ReplicaSet
kubectl scale replicaset nginx-replicaset --replicas=5

# Delete ReplicaSet
kubectl delete replicaset nginx-replicaset

# Get ReplicaSet with pods
kubectl get rs -o wide

# Watch ReplicaSets
kubectl get rs --watch

# Patch ReplicaSet
kubectl patch rs nginx-replicaset -p '{"spec":{"replicas":3}}'

# Get ReplicaSet by label
kubectl get rs -l app=nginx
```

---

## StatefulSets

### StatefulSet YAML Examples
```yaml
# StatefulSet with Headless Service
apiVersion: v1
kind: Service
meta
  name: nginx-statefulset-service
spec:
  clusterIP: None
  selector:
    app: nginx-statefulset
  ports:
  - port: 80
    name: web
---
apiVersion: apps/v1
kind: StatefulSet
meta
  name: nginx-statefulset
spec:
  serviceName: nginx-statefulset-service
  replicas: 3
  selector:
    matchLabels:
      app: nginx-statefulset
  template:
    meta
      labels:
        app: nginx-statefulset
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - meta
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

```yaml
# StatefulSet with Pod Management Policy
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: parallel-statefulset
spec:
  serviceName: nginx-service
  replicas: 3
  podManagementPolicy: Parallel
  selector:
    matchLabels:
      app: parallel
  template:
    meta
      labels:
        app: parallel
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

```yaml
# StatefulSet with Update Strategy
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: update-statefulset
spec:
  serviceName: nginx-service
  replicas: 3
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2
  selector:
    matchLabels:
      app: update
  template:
    metadata:
      labels:
        app: update
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### StatefulSet Management Commands
```bash
# Get StatefulSets
kubectl get statefulsets
kubectl get sts

# Describe StatefulSet
kubectl describe statefulset nginx-statefulset

# Scale StatefulSet
kubectl scale statefulset nginx-statefulset --replicas=5

# Delete StatefulSet
kubectl delete statefulset nginx-statefulset

# Delete StatefulSet and its pods
kubectl delete statefulset nginx-statefulset --cascade=orphan

# Watch StatefulSets
kubectl get sts --watch

# Get StatefulSet pods
kubectl get pods -l app=nginx-statefulset

# Patch StatefulSet
kubectl patch sts nginx-statefulset -p '{"spec":{"replicas":2}}'

# Update StatefulSet image
kubectl patch sts nginx-statefulset -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.22"}]}}}}'
```

---

## DaemonSets

### DaemonSet YAML Examples
```yaml
# DaemonSet
apiVersion: apps/v1
kind: DaemonSet
meta
  name: fluentd-daemonset
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.12
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

```yaml
# DaemonSet with Node Selector
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-selector-daemonset
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    meta
      labels:
        app: monitoring
    spec:
      nodeSelector:
        disktype: ssd
      containers:
      - name: monitor
        image: prometheus/node-exporter
```

### DaemonSet Management Commands
```bash
# Get DaemonSets
kubectl get daemonsets
kubectl get ds

# Describe DaemonSet
kubectl describe daemonset fluentd-daemonset

# Delete DaemonSet
kubectl delete daemonset fluentd-daemonset

# Get DaemonSet pods
kubectl get pods -l name=fluentd

# Watch DaemonSets
kubectl get ds --watch

# Patch DaemonSet
kubectl patch ds fluentd-daemonset -p '{"spec":{"template":{"spec":{"containers":[{"name":"fluentd","image":"fluent/fluentd:v1.13"}]}}}}'

# Rollout restart DaemonSet
kubectl rollout restart daemonset fluentd-daemonset
```

---

## Jobs and CronJobs

### Job YAML Examples
```yaml
# Simple Job
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-job
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

```yaml
# Job with Parallelism
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 5
  parallelism: 3
  template:
    spec:
      containers:
      - name: job-container
        image: busybox
        command: ["echo", "Hello Kubernetes!"]
      restartPolicy: OnFailure
```

```yaml
# Job with Active Deadline
apiVersion: batch/v1
kind: Job
metadata:
  name: deadline-job
spec:
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: slow-job
        image: busybox
        command: ["sleep", "300"]
      restartPolicy: OnFailure
```

### CronJob YAML Examples
```yaml
# CronJob
apiVersion: batch/v1
kind: CronJob
meta
  name: hello-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```yaml
# CronJob with Concurrency Policy
apiVersion: batch/v1
kind: CronJob
metadata:
  name: concurrent-cronjob
spec:
  schedule: "*/5 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: long-job
            image: busybox
            command: ["sleep", "300"]
          restartPolicy: OnFailure
```

```yaml
# CronJob with History Limits
apiVersion: batch/v1
kind: CronJob
metadata:
  name: history-cronjob
spec:
  schedule: "0 */1 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: busybox
            command: ["rm", "-rf", "/tmp/old-files"]
          restartPolicy: OnFailure
```

### Job and CronJob Management Commands
```bash
# Get Jobs and CronJobs
kubectl get jobs
kubectl get cronjobs
kubectl get cj

# Describe Job/CronJob
kubectl describe job pi-job
kubectl describe cronjob hello-cronjob

# Delete Job/CronJob
kubectl delete job pi-job
kubectl delete cronjob hello-cronjob

# Watch Jobs/CronJobs
kubectl get jobs --watch
kubectl get cronjobs --watch

# Get Job logs
kubectl logs job/pi-job

# Manual trigger CronJob
kubectl create job --from=cronjob/hello-cronjob manual-job

# Suspend/Resume CronJob
kubectl patch cronjob hello-cronjob -p '{"spec":{"suspend":true}}'
kubectl patch cronjob hello-cronjob -p '{"spec":{"suspend":false}}'

# Get CronJob next schedule
kubectl get cronjob hello-cronjob -o jsonpath='{.status.nextScheduleTime}'
```

---

## Resource Management

### Resource Quotas and Limits YAML Examples
```yaml
# Resource Quota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "10"
```

```yaml
# Limit Range
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

```yaml
# Object Count Quota
apiVersion: v1
kind: ResourceQuota
meta
  name: object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
```

### Resource Management Commands
```bash
# Get Resource Quotas
kubectl get resourcequotas
kubectl describe resourcequota compute-resources

# Get Limit Ranges
kubectl get limitranges
kubectl describe limitrange mem-limit-range

# Create Resource Quota from YAML
kubectl apply -f resourcequota.yaml

# Create Limit Range from YAML
kubectl apply -f limitrange.yaml

# Get quota usage
kubectl get resourcequota compute-resources -o yaml

# Delete Resource Quota/Limit Range
kubectl delete resourcequota compute-resources
kubectl delete limitrange mem-limit-range
```

---

## Probes

### Probe YAML Examples
```yaml
# Pod with Liveness and Readiness Probes
apiVersion: v1
kind: Pod
meta
  name: probe-pod
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

```yaml
# Pod with Exec Probe
apiVersion: v1
kind: Pod
meta
  name: exec-probe-pod
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

```yaml
# Pod with TCP Probe
apiVersion: v1
kind: Pod
meta
  name: tcp-probe-pod
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

```yaml
# Pod with Startup Probe
apiVersion: v1
kind: Pod
metadata:
  name: startup-probe-pod
spec:
  containers:
  - name: slow-app
    image: myapp:slow-start
    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 300
      periodSeconds: 10
```

---

## Horizontal Pod Autoscaler

### HPA YAML Examples
```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
meta
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

```yaml
# HPA with Custom Metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
meta
  name: custom-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metric:
        name: packets-per-second
      target:
        type: AverageValue
        averageValue: 1k
```

### HPA Management Commands
```bash
# Get HPAs
kubectl get hpa

# Describe HPA
kubectl describe hpa nginx-hpa

# Create HPA imperatively
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10

# Delete HPA
kubectl delete hpa nginx-hpa

# Get HPA metrics
kubectl get hpa nginx-hpa -o yaml

# Watch HPAs
kubectl get hpa --watch
```

---

## Vertical Pod Autoscaler

### VPA YAML Examples
```yaml
# Vertical Pod Autoscaler
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
meta
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: nginx
      maxAllowed:
        cpu: 1000m
        memory: 1Gi
      minAllowed:
        cpu: 100m
        memory: 128Mi
```

### VPA Management Commands
```bash
# Get VPAs
kubectl get vpa

# Describe VPA
kubectl describe vpa nginx-vpa

# Delete VPA
kubectl delete vpa nginx-vpa
```

---

## Ingress

### Ingress YAML Examples
```yaml
# Ingress Controller (NGINX example)
apiVersion: networking.k8s.io/v1
kind: IngressClass
meta
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
meta
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

```yaml
# Ingress with TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
meta
  name: tls-ingress
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

```yaml
# Ingress with Multiple Hosts
apiVersion: networking.k8s.io/v1
kind: Ingress
meta
  name: multi-host-ingress
spec:
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

### Ingress Management Commands
```bash
# Get Ingresses
kubectl get ingress
kubectl get ing

# Describe Ingress
kubectl describe ingress example-ingress

# Delete Ingress
kubectl delete ingress example-ingress

# Get Ingress classes
kubectl get ingressclasses

# Describe Ingress class
kubectl describe ingressclass nginx
```

---

## Network Policies

### Network Policy YAML Examples
```yaml
# Deny all ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

```yaml
# Allow specific ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-ingress
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 8080
```

```yaml
# Allow egress to specific namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
meta
  name: allow-egress-to-database
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database-namespace
    ports:
    - protocol: TCP
      port: 5432
```

```yaml
# Allow traffic from specific IP block
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-cidr
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    ports:
    - protocol: TCP
      port: 80
```

### Network Policy Management Commands
```bash
# Get Network Policies
kubectl get networkpolicies
kubectl get netpol

# Describe Network Policy
kubectl describe networkpolicy allow-app-ingress

# Delete Network Policy
kubectl delete networkpolicy allow-app-ingress

# Watch Network Policies
kubectl get netpol --watch
```

---

## RBAC

### RBAC YAML Examples
```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
meta
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

```yaml
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# ServiceAccount with RoleBinding
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
meta
  name: app-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
meta
  name: app-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

### RBAC Management Commands
```bash
# Get Roles and RoleBindings
kubectl get roles
kubectl get rolebindings

# Get ClusterRoles and ClusterRoleBindings
kubectl get clusterroles
kubectl get clusterrolebindings

# Describe RBAC resources
kubectl describe role pod-reader
kubectl describe clusterrole secret-reader

# Create RBAC resources
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml

# Delete RBAC resources
kubectl delete role pod-reader
kubectl delete rolebinding read-pods

# Get RBAC resources by namespace
kubectl get roles -n default
kubectl get rolebindings -n default

# Watch RBAC resources
kubectl get roles --watch
kubectl get clusterroles --watch
```

---

## Service Accounts

### Service Account YAML Examples
```yaml
# Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
  namespace: default
automountServiceAccountToken: false
```

```yaml
# Service Account with Image Pull Secret
apiVersion: v1
kind: ServiceAccount
metadata:
  name: image-pull-sa
  namespace: default
imagePullSecrets:
- name: regcred
```

```yaml
# Service Account with Secret
apiVersion: v1
kind: ServiceAccount
metadata:
  name: secret-sa
  namespace: default
secrets:
- name: my-secret
```

### Service Account Management Commands
```bash
# Get Service Accounts
kubectl get serviceaccounts
kubectl get sa

# Create Service Account
kubectl create serviceaccount build-robot

# Describe Service Account
kubectl describe serviceaccount build-robot

# Delete Service Account
kubectl delete serviceaccount build-robot

# Get Service Account token
kubectl get secret $(kubectl get sa build-robot -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode

# Add image pull secret to Service Account
kubectl patch serviceaccount default -p '{"imagePullSecrets":[{"name":"regcred"}]}'

# Watch Service Accounts
kubectl get sa --watch
```

---

## Taints and Tolerations

### Taint Management Commands
```bash
# Add taint to node
kubectl taint nodes node1 key=value:NoSchedule

# Remove taint from node
kubectl taint nodes node1 key:NoSchedule-

# View node taints
kubectl describe node node1 | grep Taints

# Add multiple taints
kubectl taint nodes node1 key1=value1:NoSchedule key2=value2:PreferNoSchedule

# Add taint with effect NoExecute
kubectl taint nodes node1 key=value:NoExecute

# Remove all taints with specific key
kubectl taint nodes node1 key-
```

### Toleration YAML Examples
```yaml
# Pod with Tolerations
apiVersion: v1
kind: Pod
meta
  name: tolerant-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoExecute"
    tolerationSeconds: 3600
  containers:
  - name: nginx
    image: nginx
```

```yaml
# Pod with Operator Exists
apiVersion: v1
kind: Pod
metadata:
  name: exists-toleration-pod
spec:
  tolerations:
  - key: "key"
    operator: "Exists"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

```yaml
# Pod with Multiple Tolerations
apiVersion: v1
kind: Pod
meta
  name: multi-toleration-pod
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key2"
    operator: "Equal"
    value: "value2"
    effect: "NoExecute"
    tolerationSeconds: 3600
  containers:
  - name: nginx
    image: nginx
```

---

## Affinity

### Affinity YAML Examples
```yaml
# Node Affinity
apiVersion: v1
kind: Pod
meta
  name: node-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```

```yaml
# Pod Affinity
apiVersion: v1
kind: Pod
meta
  name: pod-affinity-pod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
  containers:
  - name: nginx
    image: nginx
```

```yaml
# Pod Anti-Affinity
apiVersion: v1
kind: Pod
meta
  name: pod-anti-affinity-pod
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - web
          topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
```

```yaml
# Combined Node and Pod Affinity
apiVersion: v1
kind: Pod
meta
  name: combined-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - web
          topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
```

---

## Node Management

### Node Management Commands
```bash
# Get nodes
kubectl get nodes
kubectl get nodes -o wide

# Describe node
kubectl describe node node-name

# Cordon node (mark as unschedulable)
kubectl cordon node-name

# Uncordon node (mark as schedulable)
kubectl uncordon node-name

# Drain node (evict pods)
kubectl drain node-name --ignore-daemonsets --delete-emptydir-data

# Uncordon after drain
kubectl uncordon node-name

# Add label to node
kubectl label node node-name disktype=ssd

# Remove label from node
kubectl label node node-name disktype-

# Add annotation to node
kubectl annotate node node-name description="High performance node"

# Get node by label
kubectl get nodes -l disktype=ssd

# Get node resources
kubectl top nodes

# Patch node
kubectl patch node node-name -p '{"spec":{"unschedulable":true}}'

# Watch nodes
kubectl get nodes --watch
```

---

## Storage

### Storage Class YAML Examples
```yaml
# Storage Class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

```yaml
# Default Storage Class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

### Storage Management Commands
```bash
# Get storage classes
kubectl get storageclass
kubectl get sc

# Describe storage class
kubectl describe storageclass fast-ssd

# Set default storage class
kubectl patch storageclass fast-ssd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Delete storage class
kubectl delete storageclass fast-ssd

# Get persistent volumes
kubectl get pv

# Get persistent volume claims
kubectl get pvc

# Watch storage resources
kubectl get sc --watch
kubectl get pv --watch
kubectl get pvc --watch
```

---

## Security

### Pod Security Policy YAML Examples
```yaml
# Pod Security Policy (deprecated in v1.21+)
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: MustRunAsNonRoot
  fsGroup:
    rule: RunAsAny
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
```

### Security Context YAML Examples
```yaml
# Pod Security Context
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox
    securityContext:
      allowPrivilegeEscalation: false
```

### Network Policy YAML Examples
```yaml
# Deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

---

## Monitoring and Logging

### Monitoring Commands
```bash
# Get resource usage
kubectl top nodes
kubectl top pods
kubectl top pods -n namespace-name

# Get events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector type=Warning

# Get logs
kubectl logs pod-name
kubectl logs pod-name -f
kubectl logs -l app=nginx
kubectl logs pod-name --since=1h

# Port forward
kubectl port-forward pod-name 8080:80
kubectl port-forward service/service-name 8080:80
```

---

## Debugging and Troubleshooting

### Debugging Commands
```bash
# Get cluster info
kubectl cluster-info

# Get component statuses
kubectl get componentstatuses

# Get API resources
kubectl api-resources

# Explain resource fields
kubectl explain pod.spec.containers

# Check API server health
kubectl get --raw='/healthz?verbose'

# Execute commands in pod
kubectl exec -it pod-name -- /bin/bash
kubectl exec pod-name -- ps aux

# Copy files
kubectl cp local-file pod-name:/remote/path
kubectl cp pod-name:/remote/path local-file

# Port forward
kubectl port-forward pod-name 8080:80

# Get raw API data
kubectl get --raw='/api/v1/pods'
kubectl get --raw='/api/v1/namespaces/default/pods'

# Debug with temporary pod
kubectl run debug --image=busybox --rm -it --restart=Never -- sh
```

---

## Helm

### Helm Commands
```bash
# Add repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repositories
helm repo update

# Search for charts
helm search repo nginx
helm search hub wordpress

# Install chart
helm install my-release bitnami/nginx

# Install chart with values
helm install my-release bitnami/nginx -f values.yaml

# Install chart with set values
helm install my-release bitnami/nginx --set service.type=LoadBalancer

# List releases
helm list
helm list --all-namespaces

# Get release status
helm status my-release

# Upgrade release
helm upgrade my-release bitnami/nginx

# Upgrade with values
helm upgrade my-release bitnami/nginx -f values.yaml

# Rollback release
helm rollback my-release 1

# Delete release
helm uninstall my-release

# Get release values
helm get values my-release

# Get release manifest
helm get manifest my-release

# Get release notes
helm get notes my-release

# Template chart (dry run)
helm template my-release bitnami/nginx

# Package chart
helm package my-chart/

# Lint chart
helm lint my-chart/

# Create new chart
helm create my-chart

# Dependency management
helm dependency update
helm dependency build
```

---

## Custom Resource Definitions

### CRD YAML Examples
```yaml
# Custom Resource Definition
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
meta
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronSpec:
                type: string
              image:
                type: string
              replicas:
                type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

### CRD Management Commands
```bash
# Get CRDs
kubectl get crds

# Describe CRD
kubectl describe crd crontabs.stable.example.com

# Delete CRD
kubectl delete crd crontabs.stable.example.com

# Get custom resources
kubectl get crontabs
kubectl get ct
```

---

## Operators

### Operator Management Commands
```bash
# Install Operator Lifecycle Manager (OLM)
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.20.0/install.sh | bash -s v0.20.0

# Get Operators
kubectl get operators
kubectl get csv
kubectl get subscriptions
kubectl get installplans

# Install Operator
kubectl apply -f operator.yaml
```

---

## Multi-Cluster Management

### Multi-Cluster Commands
```bash
# Get contexts
kubectl config get-contexts

# Switch context
kubectl config use-context cluster-name

# Set current context
kubectl config set-context --current --namespace=dev

# Get current context
kubectl config current-context

# View config
kubectl config view

# Merge kubeconfig files
export KUBECONFIG=file1:file2:kubeconfig

# Set cluster
kubectl config set-cluster cluster-name --server=https://cluster-url --certificate-authority=ca.crt

# Set credentials
kubectl config set-credentials user-name --client-certificate=cert.crt --client-key=key.key

# Set context
kubectl config set-context context-name --cluster=cluster-name --user=user-name --namespace=default
```

---

## Backup and Restore

### Backup Commands
```bash
# Backup etcd
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/etcd-snapshot.db

# Backup resources
kubectl get all --all-namespaces -o yaml > backup.yaml

# Backup specific resources
kubectl get deployments,services,configmaps,secrets -n namespace-name -o yaml > namespace-backup.yaml
```

### Restore Commands
```bash
# Restore etcd
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot restore /tmp/etcd-snapshot.db

# Restore resources
kubectl apply -f backup.yaml
```

---

## Cluster Upgrades

### Upgrade Commands
```bash
# Check current version
kubectl version

# Add new repository
apt-get install ca-certificates curl gnupg lsb-release apt-transport-https gpg
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/`enter new version`/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Update repository
apt update -y

# Upgrade kubeadm
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.24.0-00 && \
apt-mark hold kubeadm

# Plan upgrade
kubeadm upgrade plan

# Apply upgrade
kubeadm upgrade apply v1.24.0

# Upgrade node
kubeadm upgrade node

# Upgrade kubelet and kubectl
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.24.0-00 kubectl=1.24.0-00 && \
apt-mark hold kubelet kubectl

# Restart kubelet
systemctl daemon-reload
systemctl restart kubelet
```

---

## Advanced Topics

### Advanced Commands
```bash
# JSONPath queries
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.startTime}{"\n"}{end}'
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
kubectl get pods -o custom-columns-file=template.txt

# Sort output
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by=.status.startTime

# Field selectors
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.name!=pod-name

# Label selectors
kubectl get pods -l app=nginx
kubectl get pods -l 'app in (nginx, apache)'
kubectl get pods -l 'app=nginx,env=production'

# Watch resources
kubectl get pods --watch
kubectl get pods -w

# Wait for condition
kubectl wait --for=condition=Ready pod/pod-name --timeout=60s
kubectl wait --for=jsonpath='{.status.phase}'=Running pod/pod-name

# Apply with server-side
kubectl apply --server-side -f manifest.yaml

# Diff resources
kubectl diff -f manifest.yaml

# Kustomize
kubectl apply -k ./kustomize/
kubectl diff -k ./kustomize/
```
