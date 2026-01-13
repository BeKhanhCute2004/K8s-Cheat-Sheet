# Kubernetes Comprehensive Cheat Sheet

This cheat sheet provides a detailed overview of common Kubernetes commands and their declarative YAML equivalents.

## Table of Contents

1.  [`kubectl` Basic & Context Commands](#1-kubectl-basic--context-commands)
2.  [Labels and Selectors](#2-labels-and-selectors)
3.  [Namespaces](#3-namespaces)
4.  [Pods](#4-pods)
5.  [Deployments](#5-deployments)
6.  [Services](#6-services)
7.  [ConfigMaps and Secrets](#7-configmaps-and-secrets)
8.  [Volumes, PersistentVolumes (PV), and PersistentVolumeClaims (PVC)](#8-volumes-persistentvolumes-pv-and-persistentvolumeclaims-pvc)
9.  [ReplicaSets](#9-replicasets)
10. [StatefulSets](#10-statefulsets)
11. [DaemonSets](#11-daemonsets)
12. [Jobs and CronJobs](#12-jobs-and-cronjobs)
13. [Resource Management (Requests & Limits)](#13-resource-management-requests--limits)
14. [Probes (Health Checks)](#14-probes-health-checks)
15. [Horizontal Pod Autoscaler (HPA)](#15-horizontal-pod-autoscaler-hpa)
16. [Ingress](#16-ingress)
17. [Network Policies](#17-network-policies)
18. [RBAC (Role-Based Access Control)](#18-rbac-role-based-access-control)
19. [Taints and Tolerations](#19-taints-and-tolerations)
20. [Affinity and Anti-Affinity](#20-affinity-and-anti-affinity)
21. [Debugging and Monitoring](#21-debugging-and-monitoring)
22. [Package Management with Helm](#22-package-management-with-helm)
23. [Custom Resource Definitions (CRDs)](#23-custom-resource-definitions-crds)

---

## 1. `kubectl` Basic & Context Commands

These are essential commands for interacting with your cluster.

| Command | Description | Example |
|---|---|---|
| `kubectl version` | Display client and server versions. | `kubectl version` |
| `kubectl cluster-info` | Display information about the master and services. | `kubectl cluster-info` |
| `kubectl config get-contexts` | List all available cluster contexts. | `kubectl config get-contexts` |
| `kubectl config current-context`| Get the current context. | `kubectl config current-context` |
| `kubectl config use-context <name>`| Switch to a different context. | `kubectl config use-context docker-desktop` |
| `kubectl get all --all-namespaces`| List all resources in all namespaces. | `kubectl get all -A` |

---

## 2. Labels and Selectors

Labels are key/value pairs attached to objects like Pods. They are used to organize and to select subsets of objects. Selectors are the core grouping mechanism in Kubernetes, used by services and controllers to identify the Pods they operate on.

- **Labels:** Defined in the `metadata` section of an object.
  ```yaml
  metadata:
    labels:
      app: my-app
      tier: frontend
  ```
- **Selectors:** Used in the `spec` of a controller or service to select Pods with matching labels.
  ```yaml
  # Example in a Service
  spec:
    selector:
      app: my-app # Selects any pod with the label 'app: my-app'
  ```

---

## 3. Namespaces

Namespaces are used to create virtual clusters within your physical cluster, providing a scope for names.

### Imperative Command

```bash
# Create a namespace
kubectl create namespace <namespace-name>

# Example
kubectl create namespace dev
```

### Declarative (YAML)

**`namespace.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

---

## 4. Pods

A Pod is the smallest and simplest unit in the Kubernetes object model that you create or deploy.

### Imperative Command

```bash
# Run a pod with a specific image
kubectl run <pod-name> --image=<image-name>

# Example
kubectl run my-nginx-pod --image=nginx:latest
```

### Declarative (YAML)

**`pod.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

## 5. Deployments

Deployments provide declarative updates for Pods and ReplicaSets, managing stateless applications.

### Imperative Command

```bash
# Create a deployment
kubectl create deployment <deployment-name> --image=<image-name>

# Scale a deployment
kubectl scale deployment <deployment-name> --replicas=<replica-count>

# Set a new image for a deployment
kubectl set image deployment/<deployment-name> <container-name>=<new-image-tag>

# View the rollout history of a deployment
kubectl rollout history deployment/<deployment-name>

# View the status of a rollout
kubectl rollout status deployment/<deployment-name>

# Undo a rollout
kubectl rollout undo deployment/<deployment-name>
```

### Declarative (YAML)

**`deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector: ##Helps the Deployment find and manage its pods.
    matchLabels:
      app: nginx
  template: #Defines how the pods should be created.
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

### Deployment Strategies

Kubernetes supports different strategies for updating Pods. The strategy is specified in the `.spec.strategy` field.

#### Rolling Update

This is the default strategy. It gradually replaces old Pods with new ones, ensuring that the application remains available during the update. You can control the process with `maxSurge` and `maxUnavailable`.

*   **maxSurge:** The maximum number of pods that can be created over the desired number of pods.
*   **maxUnavailable:** The maximum number of pods that can be unavailable during the update.

**`deployment-rolling-update.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
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
        image: nginx:1.19.10 # New version
        ports:
        - containerPort: 80
```

#### Blue-Green Deployment

This strategy involves deploying the new version of the application alongside the old version. Once the new version is tested and verified, traffic is switched from the old version to the new one. This is not a native Kubernetes strategy, but can be achieved by using two deployments and a service.

**Conceptual Example:**

1.  **Deployment `blue`:** Running the current version of the application, labeled `version: blue`.
2.  **Deployment `green`:** Running the new version of the application, labeled `version: green`.
3.  **Service:** A service that selects pods based on the `version` label.

To switch traffic, you update the service's selector from `version: blue` to `version: green`.

**`service-for-blue-green.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue # Initially points to the blue deployment
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

#### Canary Deployment

A canary deployment is a strategy where you release a new version of your application to a small subset of users. This allows you to test the new version in a production environment with minimal impact. This can be achieved in Kubernetes by using two deployments with different numbers of replicas.

**Conceptual Example:**

1.  **Deployment `stable`:** Running the current version with a large number of replicas.
2.  **Deployment `canary`:** Running the new version with a small number of replicas.
3.  **Service:** A service that selects pods from both deployments.

---

## 6. Services

A Service is an abstraction that defines a logical set of Pods and a policy by which to access them. This enables loose coupling between microservices.

### Service Types

There are four main types of Services:

1.  **ClusterIP:** (Default) Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
2.  **NodePort:** Exposes the Service on each Node’s IP at a static port (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service routes, is automatically created. You can contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.
3.  **LoadBalancer:** Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.
4.  **ExternalName:** Maps the Service to the contents of the `externalName` field (e.g., `foo.bar.example.com`), by returning a `CNAME` record with its value. No proxying of any kind is set up.

---

### Imperative Commands

```bash
# Expose a deployment with a ClusterIP service (default)
kubectl expose deployment my-nginx-deployment --port=80 --target-port=80

# Expose a deployment with a NodePort service
kubectl expose deployment my-nginx-deployment --type=NodePort --port=80 --target-port=80

# Expose a deployment with a LoadBalancer service
kubectl expose deployment my-nginx-deployment --type=LoadBalancer --port=80 --target-port=80
```

---

### Declarative (YAML) Examples

#### ClusterIP

This is the default service type. It provides a stable internal IP address and DNS name for your service, making it accessible from other pods within the same cluster.

**`service-clusterip.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-clusterip
spec:
  # type: ClusterIP # This is the default, so it can be omitted
  selector:
    app: nginx # Selects pods with the label "app: nginx"
  ports:
  - protocol: TCP
    port: 80 # The port the service will be available on within the cluster
    targetPort: 80 # The port on the pod that the service will forward traffic to
```

#### NodePort

This service type exposes the service on a static port on each worker node's IP address. It's useful for development or when you need to expose a service directly without a load balancer.

**`service-nodeport.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    # nodePort: 30080 # Optional: specify a port in the range 30000-32767. If not specified, a random port will be assigned.
```
**Note:** You can access this service from outside the cluster using `<NodeIP>:<nodePort>`.

#### LoadBalancer

This service type is the standard way to expose a service to the internet. It provisions an external load balancer (if supported by your cloud provider) and automatically routes traffic to your service.

**`service-loadbalancer.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80 # The port on the load balancer
    targetPort: 80 # The port on the pod
```
**Note:** After creating this service, you can get the external IP address by running `kubectl get service my-nginx-loadbalancer`.

#### ExternalName

This service type acts as a DNS alias. It doesn't proxy traffic but instead returns a CNAME record pointing to an external service. This is useful for providing a consistent internal name for an external service.

**`service-externalname.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: my.database.example.com
```
**Note:** When a pod looks up `my-external-service`, it will get a CNAME record for `my.database.example.com`.


---

## 7. ConfigMaps and Secrets

- **ConfigMap:** Stores non-confidential data in key-value pairs.
- **Secret:** Stores sensitive information, such as passwords or API keys.

### Imperative Command

```bash
# Create a ConfigMap from a literal value
kubectl create configmap app-config --from-literal=app.color=blue

# Create a Secret from a literal value
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password='S3cr3t'
```

### Declarative (YAML) - Secret

**`secret.yaml`**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  # Values must be base64 encoded
  username: YWRtaW4=
  password: UzNjcmV0
```

---

## 8. Volumes, PersistentVolumes (PV), and PersistentVolumeClaims (PVC)

- **Volume:** A directory accessible to containers in a Pod. Its lifecycle is tied to the Pod. Common types include `emptyDir`, `hostPath`, `configMap`, `secret`.
- **PersistentVolume (PV):** A piece of storage in the cluster provisioned by an administrator. It is a resource in the cluster, independent of any individual Pod.
- **PersistentVolumeClaim (PVC):** A request for storage by a user. It consumes PV resources. Pods request storage by claiming a PVC.

### Declarative (YAML) - PVC and a Pod that uses it

**`pvc.yaml`**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**`pod-with-pvc.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-with-storage
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - name: my-storage
        mountPath: /data
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: my-pvc
```

---

## 8a. Storage Deep Dive

Kubernetes provides a powerful volume management system. Here’s a deeper look at how storage is managed.

### StorageClasses

A `StorageClass` provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. This allows for dynamic provisioning of `PersistentVolumes` (PVs).

**`storageclass.yaml`**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

### Dynamic vs. Static Provisioning

*   **Static Provisioning:** A cluster administrator creates a number of PVs. They carry the details of the real storage which is available for use by cluster users. They exist in the Kubernetes API and are available for consumption.
*   **Dynamic Provisioning:** When none of the static PVs the administrator created match a user’s `PersistentVolumeClaim`, the cluster may try to dynamically provision a volume specially for the PVC. This provisioning is based on `StorageClasses`.

### Access Modes

A `PersistentVolume` can be mounted on a host in any way supported by the resource provider. The access modes are:

*   **ReadWriteOnce (RWO):** The volume can be mounted as read-write by a single node.
*   **ReadOnlyMany (ROX):** The volume can be mounted read-only by many nodes.
*   **ReadWriteMany (RWX):** The volume can be mounted as read-write by many nodes.
*   **ReadWriteOncePod (RWOP):** The volume can be mounted as read-write by a single Pod. This is a new feature and might not be available in all clusters.

---

## 9. ReplicaSets

A ReplicaSet ensures that a specified number of replica Pods are running at any given time. **Note:** It's highly recommended to use Deployments, which manage ReplicaSets for you.

### Declarative (YAML)

**`replicaset.yaml`**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

---

## 10. StatefulSets

Manages the deployment and scaling of a set of Pods for stateful applications, providing guarantees about the ordering and uniqueness of these Pods.

### Declarative (YAML)

**Requires a Headless Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-sts
```
**`statefulset.yaml`**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx-headless"
  replicas: 2
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 11. DaemonSets

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. Useful for cluster-wide agents like log collectors or monitoring agents.

### Declarative (YAML)

**`daemonset.yaml`**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-agent
spec:
  selector:
    matchLabels:
      name: fluentd-agent
  template:
    metadata:
      labels:
        name: fluentd-agent
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1
```

---

## 12. Jobs and CronJobs

- **Job:** Creates one or more Pods and ensures that a specified number of them successfully terminate. For run-to-completion tasks.
- **CronJob:** Creates Jobs on a repeating schedule.

### Declarative (YAML) - CronJob

**`cronjob.yaml`**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *" # Runs every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command:
            - /bin/sh
            - -c
            - date; echo "Hello from the CronJob"
          restartPolicy: OnFailure
```

---

## 13. Resource Management (Requests & Limits)

Specify CPU and memory needs for each container to ensure reliable performance and cluster stability.

- **requests:** Guaranteed resources for a container.
- **limits:** Maximum resources a container can use.

**`pod-with-resources.yaml`**
```yaml
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

---

## 14. Probes (Health Checks)

Probes are used by the kubelet to check the health of a container.

- **livenessProbe:** Checks if the container is running. If it fails, the container is restarted.
- **readinessProbe:** Checks if the container is ready to serve traffic. If it fails, it's removed from service endpoints.
- **startupProbe:** Checks if the application within the container has started.

**`pod-with-probes.yaml`**
```yaml
spec:
  containers:
  - name: my-app
    image: my-app-image
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

---

## 15. Horizontal Pod Autoscaler (HPA)

Automatically scales the number of pods in a deployment based on observed CPU utilization or other metrics.

### Imperative Command
```bash
kubectl autoscale deployment <name> --cpu-percent=50 --min=1 --max=10
```

### Declarative (YAML)
**`hpa.yaml`**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## 16. Ingress

Manages external access to services in a cluster, typically HTTP/HTTPS. **Requires an Ingress Controller.**

### Declarative (YAML)
**`ingress.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: "myapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-nginx-service
            port:
              number: 80
```

---

## 16a. Ingress Deep Dive

An Ingress provides a way to manage external access to services in a cluster, typically for HTTP and HTTPS traffic. It acts as a reverse proxy, routing traffic from outside the cluster to services within the cluster.

### Ingress Controllers

To use Ingress, you need an Ingress controller. The controller is responsible for fulfilling the Ingress, usually with a load balancer. Popular Ingress controllers include:

*   **NGINX Ingress Controller:** A popular choice, maintained by the Kubernetes community.
*   **Traefik:** A modern HTTP reverse proxy and load balancer.
*   **HAProxy Ingress:** Another robust and high-performance option.

### TLS/SSL Termination

You can secure an Ingress by specifying a Secret that contains a TLS private key and certificate.

**`ingress-with-tls.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: my-tls-secret
  rules:
  - host: "myapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-nginx-service
            port:
              number: 80
```
**Note:** The `my-tls-secret` must be created beforehand and contain `tls.crt` and `tls.key`.

### Host-based and Path-based Routing

Ingress allows you to route traffic to different services based on the hostname and path.

**`ingress-routing.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: complex-routing-ingress
spec:
  rules:
  - host: "foo.example.com"
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "bar.example.com"
    http:
      paths:
      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
```

---

## 17. Network Policies

Provide firewall-like capabilities to control traffic flow between Pods.

### Declarative (YAML) - Allow ingress from specific pods
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

---

## 17a. Security Context

A security context defines privilege and access control settings for a Pod or Container. This is crucial for ensuring that your workloads run with the minimum privileges they need.

### Pod and Container Security Context

You can set a security context at both the Pod and the Container level. If settings are specified at both levels, the container-level settings take precedence.

**`pod-with-security-context.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: my-secure-container
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      runAsUser: 1001 # Overrides the pod-level setting
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

### Capabilities

With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user. For example, you can grant the `NET_BIND_SERVICE` capability to a container to allow it to bind to a port below 1024, without giving it full root access.

**`pod-with-capabilities.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-with-capabilities
spec:
  containers:
  - name: my-container
    image: nginx
    securityContext:
      capabilities:
        add: ["NET_BIND_SERVICE"]
        drop: ["ALL"]
```

---

## 18. RBAC (Role-Based Access Control)

Manages who can do what within the cluster.

- **Role/ClusterRole:** A set of permissions.
- **RoleBinding/ClusterRoleBinding:** Grants the permissions in a Role to a set of users.

### Declarative (YAML) - Role and RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
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

---

## 19. Taints and Tolerations

- **Taints:** Applied to Nodes to repel Pods.
- **Tolerations:** Applied to Pods to allow them to be scheduled on Nodes with matching taints.

### Imperative Command
```bash
kubectl taint nodes node1 app=blue:NoSchedule
```
### Declarative (YAML) - Pod with a Toleration
```yaml
spec:
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

---

## 20. Affinity and Anti-Affinity

A property of Pods that attracts them to or repels them from a set of other objects.

- **Node Affinity:** Attracts a Pod to a set of nodes based on node labels.
- **Pod Affinity / Anti-Affinity:** Attracts/repels a Pod from other Pods based on pod labels.

### Declarative (YAML) - Pod with Node Affinity
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

---

## 20a. Advanced Scheduling

Kubernetes offers several ways to control how pods are scheduled onto nodes. This allows you to optimize for cost, performance, and availability.

### Node Selectors

The simplest way to constrain a Pod to run on a particular Node is to use a `nodeSelector`. This field in the Pod spec specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels.

**`pod-with-nodeselector.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-with-nodeselector
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

### Node Affinity and Anti-Affinity

Node affinity is a more expressive way to constrain which nodes your pod can be scheduled on. You can express rules like "this pod should run on a node with an Intel CPU" or "this pod should not run on a node in availability zone us-west-2a".

*   `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can’t schedule the Pod unless the rule is met.
*   `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler will try to find a node that meets the rule, but if it can’t, it will still schedule the Pod.

**`pod-with-node-affinity.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

### Pod Affinity and Anti-Affinity

Pod affinity and anti-affinity allow you to constrain which nodes your pod can be scheduled on based on the labels of pods already running on that node. This is useful for co-locating pods that communicate frequently, or for spreading pods across nodes to improve availability.

**`pod-with-pod-affinity.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
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
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```

---

## 21. Debugging and Monitoring

| Command | Description |
|---|---|
| `kubectl get events --sort-by='.lastTimestamp'` | List all events, sorted by time. Crucial for finding problems. |
| `kubectl describe <resource> <name>` | The best all-purpose command for inspecting a resource and finding issues. |
| `kubectl logs <pod-name>` | View the logs from a pod (`-f` to follow). |
| `kubectl exec -it <pod-name> -- /bin/sh` | Get an interactive shell inside a running pod. |
| `kubectl port-forward <pod-name> <local>:<remote>` | Forward a local port to a port on a pod. |
| `kubectl top pod` / `kubectl top node` | View CPU and Memory usage. |

---

## 22. Package Management with Helm

Helm is the package manager for Kubernetes. It uses "charts" to define, install, and upgrade applications.

| Command | Description |
|---|---|
| `helm repo add <name> <url>` | Add a chart repository. |
| `helm search repo <keyword>` | Search for charts. |
| `helm install <release> <chart>` | Install a chart. |
| `helm list` | List all releases. |
| `helm upgrade <release> <chart>` | Upgrade a release. |
| `helm uninstall <release>` | Uninstall a release. |

---

## 23. Custom Resource Definitions (CRDs)

Custom Resource Definitions (CRDs) are a powerful feature of Kubernetes that allow you to create your own custom resource types. This enables you to extend the Kubernetes API to meet your specific needs.

### What are CRDs?

A CRD is a blueprint for a new resource type. Once you create a CRD, you can create and manage custom objects of that type just like you would with built-in Kubernetes objects like Pods and Deployments.

### Example

Here is an example of a simple CRD and a custom resource.

**`crd.yaml`**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
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

**`custom-resource.yaml`**
```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-crontab
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```