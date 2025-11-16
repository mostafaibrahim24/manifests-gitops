# K8s Manifests repo
<img width="120" height="703" alt="image" src="https://github.com/user-attachments/assets/b83a1b87-24a3-45c4-8ff4-19f61e455399" />
<img width="100" height="2160" alt="image" src="https://github.com/user-attachments/assets/f0d81c49-ea56-4e27-9d88-30e5441073eb" />
<img width="120" height="767" alt="image" src="https://github.com/user-attachments/assets/bf69d38a-961d-4960-afec-69cff66b52ed" />
<img width="120" height="236" alt="image-removebg-preview (21)" src="https://github.com/user-attachments/assets/8dafccd5-cf07-4d85-994a-3dc8ecb783c1" />

Kubernetes manifests repository for GitOps-managed resources, including gateway resource, routes, application deployments, stateful databases, horizontal pod-autoscaling configurations, prometheus target discovery, configs, secrets (sealed) and network policies.
</br>
## K8s Cluster
<img width="1665" height="775" alt="image" src="https://github.com/user-attachments/assets/be6cc0c0-512e-4fac-967b-fee6ff032c07" />


## Architecture Highlights

### Gateway API Implementation
<img width="60" height="243" alt="image" src="https://github.com/user-attachments/assets/b3153ca1-3873-4556-8581-f4216ca4df07" />

- Using Gateway API for App's Ingress traffic management
- Better **separation of concerns** between infrastructure and application routing

### Stateful Workloads Handling
<img width="60" height="512" alt="image" src="https://github.com/user-attachments/assets/283d0e3e-dde2-4a77-b445-baae97d21765" />

- _StatefulSets_ for databases (stable network identities along with persistent storage via auto-created pvcs via templates)
- _InitContainers_ for database initialization and replication setup along with _Sidecars_ for continuous replication sync and health monitoring
- MySQL primary-replica architecture with automated orchestration - Differentiated configurations per pod role (primary for writes, replicas for reads), init containers handle data cloning via XtraBackup streaming and replication initialization,
  dynamic pod labeling enables traffic routing to appropriate endpoints, continuous sidecar backup server maintains replica synchronization and facilitates new member onboarding.
- MongoDB automated replica set orchestration - Sidecar container dynamically discovers and configures replica set members, handling initialization, membership changes, and failover coordination automatically.

### Service Discovery 
<img width="60" height="800" alt="image" src="https://github.com/user-attachments/assets/360b738e-1f78-4ef9-9207-a994fce270d9" />
<img width="60" height="300" alt="rds-db-instance" src="https://github.com/user-attachments/assets/4ec6fefd-b34d-4976-a492-c101f87c0895" />
<img width="60" height="300" alt="image" src="https://github.com/user-attachments/assets/490f60ec-3968-443c-a442-04bcdecbdd32" />

- MySQL dual-service pattern - Headless mysql service for primary discovery and direct pod access, separate mysql-read headless service with mysql-role: replica selector for read-only traffic distribution across replicas
- Headless services for databases - Direct pod discovery via DNS, enabling StatefulSet stable network identities, client-side load balancing, and peer-to-peer replication coordination

### Metrics and Logs Collection
<img width="60" height="1200" alt="image" src="https://github.com/user-attachments/assets/4bdc3847-779e-4eb5-a127-0f1104e9e397" />
<img width="60" height="500" alt="image" src="https://github.com/user-attachments/assets/4069cdec-ad62-4778-9979-c6c13da787ed" />
<img width="60" height="512" alt="image" src="https://github.com/user-attachments/assets/4ae4a00e-b10c-46ba-b479-e8fcb911194b" />


- Prometheus metrics exposure - endpoints on application services, dedicated exporter sidecars for database observability providing performance and resource utilization metrics
- ServiceMonitors for declarative scraping - Custom resources automatically configure Prometheus target discovery and scrape intervals per service, eliminating manual Prometheus configuration
- Centralized log aggregation with Loki stack - Promtail agents deployed as DaemonSet on each node scrape container logs and ship to Loki for efficient storage and querying, unified with metrics in Grafana for correlated observability
  
### Multi-dimensional horizontal pod scaling strategy
<img width="60" height="125" alt="image" src="https://github.com/user-attachments/assets/b9af2812-ecb0-41ac-a886-d3e0e1d00dbd" /> </br>
HPAs consume both standard metrics (pod CPU/memory) and application-aware custom metrics via Prometheus Adapter, enabling scaling decisions based on actual user experience and system behavior rather than infrastructure utilization alone.</br>
So for example, Application latencies (user experience-driven scaling), Requests per second (traffic-based scaling), Process-level CPU (more accurate than container CPU) and Database query latencies (backend performance-driven scaling).

### Security
<img width="60" height="512" alt="image" src="https://github.com/user-attachments/assets/c5267aa9-7c8a-42bc-b2d4-61bf244664e4" />

- **Network policies** enforcing microsegmentation with explicit ingress/egress rules - default-deny posture ensuring only required service-to-service communication paths are permitted
- **Sealed Secrets** enabling secrets to be encrypted with cluster public key stored in Git, decrypted at runtime by controller with private key inaccessible outside the cluster.
- **Falco** monitors container runtime behavior for security anomalies and detects suspicious activities like privilege escalations and unauthorized file access, configured to alert on slack channel

