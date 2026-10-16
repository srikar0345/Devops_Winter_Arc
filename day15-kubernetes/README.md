# Day 15: Kubernetes Fundamentals

## 1. Objective

The objective of this assignment is to understand the fundamentals of Kubernetes by deploying an Nginx application on a local Kubernetes cluster, exposing the application through a Service, and scaling the Deployment to three replicas.

## 2. Environment and Technologies

- **Container Platform:** Docker Desktop
- **Orchestration Platform:** Kubernetes
- **Command-Line Tool:** kubectl v1.34.1
- **Application Image:** nginx:1.27
- **Configuration Format:** YAML

## 3. Kubernetes Core Concepts

- **Cluster:** A complete Kubernetes environment used to manage and run containerized applications.
- **Control Plane:** Responsible for managing the cluster and maintaining its desired state.
- **Node:** A machine within the cluster that runs Kubernetes workloads.
- **Pod:** The smallest deployable unit in Kubernetes, containing one or more containers.
- **Deployment:** Manages replicated Pods and maintains the desired number of application replicas.
- **Service:** Provides a stable network endpoint for accessing Pods that match its selector.

## 4. Implementation Procedure

### 4.1 Cluster Verification

Verified the Kubernetes client installation and confirmed that the local Kubernetes cluster was operational.

Commands executed:

```bash
kubectl version --client
kubectl cluster-info
kubectl get nodes
```

The Docker Desktop node was reported in the `Ready` state.

### 4.2 Deployment Configuration

Created `deployment.yaml` to deploy Nginx using the `nginx:1.27` image. The initial configuration specified two replicas.

Applied the configuration and inspected the Deployment and Pods to confirm successful creation.

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods -o wide
```

Both Nginx Pods reached the `Running` state.

### 4.3 Service Configuration

Created `service.yaml` to expose the Nginx application through a NodePort Service.

The Service configuration specified:
- Service type: `NodePort`
- Service port: `80`
- Target port: `80`
- NodePort: `30080`
- Pod selector: `app: nginx`

Applied the Service configuration and verified its status.

```bash
kubectl apply -f service.yaml
kubectl get services
```

### 4.4 Application Access Verification

Used Kubernetes port-forwarding to access the Nginx application locally.

```bash
kubectl port-forward service/nginx-service 8080:80
```

In a separate terminal, executed:

```bash
curl http://localhost:8080
```

The command returned the Nginx welcome page HTML, confirming that the application was accessible through the Service.

### 4.5 Deployment Scaling

Scaled the Nginx Deployment from two replicas to three.

```bash
kubectl scale deployment nginx-deployment --replicas=3
```

Verified the resulting state:

```bash
kubectl get deployment nginx-deployment
kubectl get pods
```

The Deployment reported three desired, three updated, and three available replicas. All three Nginx Pods reached the `Running` state.

## 5. Verification Results

The following results were observed during execution:

| Verification | Result |
|---|---|
| Kubernetes cluster connectivity | Successful |
| Node readiness | Ready |
| Nginx Deployment creation | Successful |
| Initial Nginx replicas | 2 |
| Service creation | Successful |
| Nginx application access | Successful |
| Final Nginx replicas | 3 |
| Final replica availability | 3 of 3 |

## 6. Troubleshooting and Diagnostic Commands

The following commands can be used to inspect Kubernetes resources and investigate common issues.

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl logs <pod-name>
```

- `kubectl get pods` displays Pod status and restart counts.
- `kubectl describe pod` provides detailed Pod information and related events.
- `kubectl get events` displays cluster events in chronological order.
- `kubectl logs` retrieves container logs to assist with application troubleshooting.

## 7. Cleanup Procedure

The following commands remove the Deployment and Service created for this assignment.

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

**Note:** Execute these commands only when the application is no longer required. They remove the resources defined in the respective manifests.

## 8. Conclusion

This assignment provided practical experience with Kubernetes cluster verification, YAML-based resource configuration, Deployment and Service creation, application access, and replica scaling.

The Nginx application was deployed successfully on the local Kubernetes cluster, accessed through port-forwarding, and scaled to three available replicas. These exercises demonstrate the fundamental role of Kubernetes in managing and scaling containerized workloads.
