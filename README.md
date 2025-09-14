# Voting Application on Kubernetes

A complete microservices-based voting application deployed on Kubernetes using **Deployments**, demonstrating a modern distributed architecture with separate services for voting, result display, background processing, and data storage.

## 🏗️ Architecture Overview

This application consists of 5 microservices working together to provide a complete voting system, all managed by Kubernetes Deployments for better reliability and scalability:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Vote App      │    │   Result App    │    │   Worker        │
│   (Frontend)    │    │   (Results)     │    │   (Processor)   │
│   Port: 31000   │    │   Port: 31001   │    │   (Background)  │
│   Deployment    │    │   Deployment    │    │   Deployment    │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Redis       │    │   PostgreSQL    │    │   Kubernetes    │
│   (Message      │    │   (Database)    │    │   Services      │
│    Queue)       │    │   Deployment    │    │   & Deployments │
│   Deployment    │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📁 Project Structure

```
voting-app-k8s/
├── deployments/              # Deployment and Service definitions
│   ├── db-deployment.yaml    # PostgreSQL database deployment
│   ├── redis-deployment.yaml # Redis cache/message queue deployment
│   ├── result-deployment.yaml # Result display application deployment
│   ├── vote-deployment.yaml  # Main voting application deployment
│   ├── worker-deployment.yaml # Background worker deployment
│   ├── db-service.yaml       # Database service (ClusterIP)
│   ├── redis-service.yaml    # Redis service (ClusterIP)
│   ├── result-service.yaml   # Result app service (NodePort)
│   └── voting-service.yaml   # Voting app service (NodePort)
├── pods/                     # Legacy pod definitions (for reference)
│   ├── db-pod.yaml          # PostgreSQL database pod
│   ├── redis-pod.yaml       # Redis cache/message queue pod
│   ├── result-app-pod.yaml  # Result display application pod
│   ├── voting-app-pod.yaml  # Main voting application pod
│   └── worker-pod.yaml      # Background worker pod
├── services/                 # Legacy service definitions (for reference)
│   ├── db-service.yaml      # Database service (ClusterIP)
│   ├── redis-service.yaml   # Redis service (ClusterIP)
│   ├── result-service.yaml  # Result app service (NodePort)
│   └── voting-service.yaml  # Voting app service (NodePort)
└── README.md                # This documentation
```

## 🚀 Services Description

### 1. **Voting Application** (`vote`)
- **Purpose**: Main user interface for casting votes
- **Type**: Deployment with 1 replica
- **Image**: `dockersamples/examplevotingapp_vote`
- **Port**: 80 (internal), 31000 (external NodePort)
- **Access**: External users can vote via `http://<node-ip>:31000`
- **Benefits**: Automatic pod restart, rolling updates, scalability

### 2. **Result Application** (`result`)
- **Purpose**: Displays real-time voting results and statistics
- **Type**: Deployment with 1 replica
- **Image**: `dockersamples/examplevotingapp_result`
- **Port**: 80 (internal), 31001 (external NodePort)
- **Access**: External users can view results via `http://<node-ip>:31001`
- **Benefits**: Automatic pod restart, rolling updates, scalability

### 3. **Worker Service** (`worker`)
- **Purpose**: Background processor that handles vote processing
- **Type**: Deployment with 1 replica
- **Image**: `dockersamples/examplevotingapp_worker`
- **Function**: Consumes vote messages from Redis and stores them in PostgreSQL
- **Access**: Internal only (no external ports)
- **Benefits**: Automatic pod restart, can scale to multiple workers

### 4. **Redis Cache** (`redis`)
- **Purpose**: Message queue and caching layer
- **Type**: Deployment with 1 replica
- **Image**: `redis:alpine`
- **Port**: 6379 (internal ClusterIP only)
- **Function**: Acts as message broker between voting app and worker
- **Benefits**: Automatic pod restart, data persistence (with proper configuration)

### 5. **PostgreSQL Database** (`db`)
- **Purpose**: Persistent storage for voting data
- **Type**: Deployment with 1 replica
- **Image**: `postgres:15-alpine`
- **Port**: 5432 (internal ClusterIP only)
- **Credentials**: postgres/postgres (⚠️ Change in production!)
- **Benefits**: Automatic pod restart, data persistence (with proper configuration)

## 🔄 Data Flow

1. **User votes** → Voting App (port 31000)
2. **Vote data** → Redis message queue
3. **Worker processes** → Consumes from Redis queue
4. **Data storage** → Worker stores votes in PostgreSQL
5. **Results display** → Result App (port 31001) reads from database

## 🛠️ Prerequisites

- Kubernetes cluster (minikube, kind, or cloud provider)
- `kubectl` configured to connect to your cluster
- Basic understanding of Kubernetes concepts (Pods, Services)

## 📦 Deployment Instructions

### 1. Deploy All Resources
```bash
# Deploy all deployments and services from the deployments directory
kubectl apply -f deployments/

# Verify deployments are created
kubectl get deployments

# Verify services are created
kubectl get services
```

### 2. Check Deployment Status
```bash
# Check deployment status
kubectl get deployments -o wide

# Check pod status (pods are managed by deployments)
kubectl get pods -o wide

# Check service endpoints
kubectl get endpoints

# View pod logs (if needed)
kubectl logs <pod-name>
```

### 3. Verify Application Health
```bash
# Check if all pods are running
kubectl get pods

# Check deployment rollout status
kubectl rollout status deployment/<deployment-name>

# View deployment details
kubectl describe deployment <deployment-name>
```

## 🌐 Accessing the Application

### For Minikube:
```bash
# Get minikube IP
minikube ip

# Access voting interface
http://$(minikube ip):31000

# Access results interface
http://$(minikube ip):31001
```

### For Other Kubernetes Clusters:
```bash
# Get node IP
kubectl get nodes -o wide

# Access via node IP
http://<NODE_IP>:31000  # Voting
http://<NODE_IP>:31001  # Results
```

## 🔧 Kubernetes Resources Explained

### Deployments vs Pods
- **Deployments**: Manage pod lifecycle, provide rolling updates, auto-restart failed pods
- **Pods**: Basic unit of deployment (managed by deployments in this setup)
- **Benefits**: Better reliability, scalability, and zero-downtime updates

### Service Types
- **ClusterIP Services**: Database & Redis - Internal cluster communication only
- **NodePort Services**: Voting & Result Apps - External access via node IP (31000, 31001)
- **Security**: Backend services isolated, frontend services accessible

### Deployment Features
- **Replica Management**: Each deployment maintains desired number of replicas
- **Rolling Updates**: Zero-downtime deployments with `kubectl rollout`
- **Auto-Recovery**: Failed pods are automatically restarted
- **Scaling**: Easy horizontal scaling with `kubectl scale`

## 🐛 Troubleshooting

### Common Issues:

1. **Deployments not creating pods**:
   ```bash
   kubectl describe deployment <deployment-name>
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

2. **Pods not starting**:
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   ```

3. **Services not accessible**:
   ```bash
   kubectl get services
   kubectl get endpoints
   kubectl describe service <service-name>
   ```

4. **Database connection issues**:
   - Check if PostgreSQL deployment is running: `kubectl get deployment db`
   - Verify database service selector matches deployment labels
   - Check pod logs: `kubectl logs -l app=db`

5. **Redis connection issues**:
   - Ensure Redis deployment is running: `kubectl get deployment redis`
   - Check Redis service configuration
   - Verify pod connectivity: `kubectl exec -it <redis-pod> -- redis-cli ping`

### Debug Commands:
```bash
# View all resources
kubectl get all

# Check deployment status
kubectl get deployments
kubectl describe deployment <deployment-name>

# Check pod logs
kubectl logs -f <pod-name>
kubectl logs -f deployment/<deployment-name>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/sh

# Port forward for testing
kubectl port-forward deployment/<deployment-name> 8080:80

# Check rollout history
kubectl rollout history deployment/<deployment-name>
```

## 🔒 Security Considerations

### Current Security Issues:
- **Database credentials** are hardcoded in plain text
- **No network policies** implemented
- **No resource limits** set on pods
- **No secrets management** for sensitive data

### Production Recommendations:
1. Use Kubernetes Secrets for database credentials
2. Implement Network Policies for pod-to-pod communication
3. Set resource requests and limits
4. Use ConfigMaps for configuration management
5. Enable RBAC (Role-Based Access Control)
6. Consider using Ingress instead of NodePort for external access

## 📊 Monitoring and Observability

### Basic Monitoring:
```bash
# Check resource usage
kubectl top pods
kubectl top nodes

# View events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Recommended Additions:
- Prometheus for metrics collection
- Grafana for visualization
- ELK stack for log aggregation
- Jaeger for distributed tracing

## 🧪 Testing the Application

1. **Access voting interface**: `http://<node-ip>:31000`
2. **Cast some votes** by clicking on different options
3. **View results**: `http://<node-ip>:31001`
4. **Verify data persistence** by refreshing the results page

## 🔄 Deployment Management

### Scaling Applications
```bash
# Scale voting app to 3 replicas
kubectl scale deployment vote --replicas=3

# Scale result app to 2 replicas
kubectl scale deployment result --replicas=2

# Check scaling status
kubectl get deployments
kubectl get pods -l app=vote
```

### Rolling Updates
```bash
# Update voting app image
kubectl set image deployment/vote vote=dockersamples/examplevotingapp_vote:latest

# Check rollout status
kubectl rollout status deployment/vote

# Rollback if needed
kubectl rollout undo deployment/vote

# View rollout history
kubectl rollout history deployment/vote
```

### Health Checks
```bash
# Check deployment health
kubectl get deployments
kubectl describe deployment <deployment-name>

# Check pod health
kubectl get pods
kubectl describe pod <pod-name>

# View logs
kubectl logs -f deployment/<deployment-name>
```

## 🗑️ Cleanup

To remove all resources:
```bash
# Delete all deployments and services
kubectl delete -f deployments/

# Or delete everything at once (if you have both directories)
kubectl delete -f .

# Verify cleanup
kubectl get all
```

### Individual Resource Cleanup:
```bash
# Delete specific deployment
kubectl delete deployment <deployment-name>

# Delete specific service
kubectl delete service <service-name>

# Scale down deployment to 0 replicas
kubectl scale deployment <deployment-name> --replicas=0
```

## 📚 Learning Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Service Types](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Docker Sample Voting App](https://github.com/dockersamples/example-voting-app)
- [Kubernetes Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)

## 🤝 Contributing

This is a learning project demonstrating Kubernetes Deployments and microservices concepts. Feel free to:
- Add more comprehensive error handling
- Implement health checks and readiness probes
- Add resource limits and requests
- Implement proper secrets management
- Add monitoring and logging
- Implement horizontal pod autoscaling (HPA)
- Add persistent volumes for database storage
- Implement network policies for security

## 📄 License

This project is for educational purposes and uses the Docker sample voting application as a base.

---

## 🎯 Key Benefits of Using Deployments

### Why Deployments over Pods?
- **Reliability**: Automatic pod restart on failure
- **Scalability**: Easy horizontal scaling with `kubectl scale`
- **Zero-downtime updates**: Rolling updates with `kubectl rollout`
- **Rollback capability**: Quick rollback with `kubectl rollout undo`
- **Self-healing**: Kubernetes automatically replaces failed pods
- **Production-ready**: Industry standard for managing stateless applications

### Migration from Pods to Deployments
The `pods/` and `services/` directories contain the original pod-based configuration for reference. The `deployments/` directory contains the improved deployment-based configuration that should be used for production workloads.

---

**Note**: This is a demonstration project. For production use, implement proper security measures, monitoring, and follow Kubernetes best practices.
