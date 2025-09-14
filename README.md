# Voting Application on Kubernetes

A complete microservices-based voting application deployed on Kubernetes, demonstrating a modern distributed architecture with separate services for voting, result display, background processing, and data storage.

## 🏗️ Architecture Overview

This application consists of 5 microservices working together to provide a complete voting system:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Vote App      │    │   Result App    │    │   Worker        │
│   (Frontend)    │    │   (Results)     │    │   (Processor)   │
│   Port: 31000   │    │   Port: 31001   │    │   (Background)  │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Redis       │    │   PostgreSQL    │    │   Kubernetes    │
│   (Message      │    │   (Database)    │    │   Services      │
│    Queue)       │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📁 Project Structure

```
voting-app-k8s/
├── pods/                    # Pod definitions
│   ├── db-pod.yaml         # PostgreSQL database pod
│   ├── redis-pod.yaml      # Redis cache/message queue pod
│   ├── result-app-pod.yaml # Result display application pod
│   ├── voting-app-pod.yaml # Main voting application pod
│   └── worker-pod.yaml     # Background worker pod
├── services/               # Service definitions
│   ├── db-service.yaml     # Database service (ClusterIP)
│   ├── redis-service.yaml  # Redis service (ClusterIP)
│   ├── result-service.yaml # Result app service (NodePort)
│   └── voting-service.yaml # Voting app service (NodePort)
└── README.md              # This documentation
```

## 🚀 Services Description

### 1. **Voting Application** (`vote`)
- **Purpose**: Main user interface for casting votes
- **Image**: `dockersamples/examplevotingapp_vote`
- **Port**: 80 (internal), 31000 (external NodePort)
- **Access**: External users can vote via `http://<node-ip>:31000`

### 2. **Result Application** (`result`)
- **Purpose**: Displays real-time voting results and statistics
- **Image**: `dockersamples/examplevotingapp_result`
- **Port**: 80 (internal), 31001 (external NodePort)
- **Access**: External users can view results via `http://<node-ip>:31001`

### 3. **Worker Service** (`worker`)
- **Purpose**: Background processor that handles vote processing
- **Image**: `dockersamples/examplevotingapp_worker`
- **Function**: Consumes vote messages from Redis and stores them in PostgreSQL
- **Access**: Internal only (no external ports)

### 4. **Redis Cache** (`redis`)
- **Purpose**: Message queue and caching layer
- **Image**: `redis:alpine`
- **Port**: 6379 (internal ClusterIP only)
- **Function**: Acts as message broker between voting app and worker

### 5. **PostgreSQL Database** (`db`)
- **Purpose**: Persistent storage for voting data
- **Image**: `postgres:15-alpine`
- **Port**: 5432 (internal ClusterIP only)
- **Credentials**: postgres/postgres (⚠️ Change in production!)

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

### 1. Deploy Pods
```bash
# Deploy all pods
kubectl apply -f pods/

# Verify pods are running
kubectl get pods
```

### 2. Deploy Services
```bash
# Deploy all services
kubectl apply -f services/

# Verify services are created
kubectl get services
```

### 3. Check Deployment Status
```bash
# Check pod status
kubectl get pods -o wide

# Check service endpoints
kubectl get endpoints

# View pod logs (if needed)
kubectl logs <pod-name>
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

## 🔧 Service Types Explained

### ClusterIP Services
- **Database & Redis**: Internal cluster communication only
- **Security**: Not accessible from outside the cluster
- **Use Case**: Backend services that don't need external access

### NodePort Services
- **Voting & Result Apps**: External access via node IP
- **Ports**: 31000 (voting), 31001 (results)
- **Use Case**: Frontend applications that need user access

## 🐛 Troubleshooting

### Common Issues:

1. **Pods not starting**:
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   ```

2. **Services not accessible**:
   ```bash
   kubectl get services
   kubectl get endpoints
   ```

3. **Database connection issues**:
   - Check if PostgreSQL pod is running
   - Verify database service selector matches pod labels

4. **Redis connection issues**:
   - Ensure Redis pod is running
   - Check Redis service configuration

### Debug Commands:
```bash
# View all resources
kubectl get all

# Check pod logs
kubectl logs -f <pod-name>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/sh

# Port forward for testing
kubectl port-forward pod/<pod-name> 8080:80
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

## 🗑️ Cleanup

To remove all resources:
```bash
# Delete services
kubectl delete -f services/

# Delete pods
kubectl delete -f pods/

# Or delete everything at once
kubectl delete -f .
```

## 📚 Learning Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes Service Types](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Docker Sample Voting App](https://github.com/dockersamples/example-voting-app)
- [Kubernetes Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

## 🤝 Contributing

This is a learning project demonstrating Kubernetes concepts. Feel free to:
- Add more comprehensive error handling
- Implement health checks and readiness probes
- Add resource limits and requests
- Implement proper secrets management
- Add monitoring and logging

## 📄 License

This project is for educational purposes and uses the Docker sample voting application as a base.

---

**Note**: This is a demonstration project. For production use, implement proper security measures, monitoring, and follow Kubernetes best practices.
