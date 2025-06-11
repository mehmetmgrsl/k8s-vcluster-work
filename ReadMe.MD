### Why vCluster?

- To efficiently run multiple isolated Kubernetes environments on a single cluster, reducing cost and operational complexity


### What is vCluster?

- vCluster is a tool that creates virtual Kubernetes clusters—each with its own API server—inside a single physical Kubernetes cluster, offering better isolation than namespaces and lower cost than separate clusters.

### Architecture of vCluster

- ![Architecture of vCluster)](vcluster-01.png) (Diagram from 1*) 

- A user uses **kubectl** to interact with the **vCluster control plane**, **not the physical host cluster** directly.

- The **vCluster control plane** is deployed **as a pod** in the host cluster.

- Each vCluster operates inside a namespace (e.g., ns-1, ns-2) of the physical cluster.

- Inside the vCluster, the user sees high-level Kubernetes resources like Deployments, CRDs, StatefulSets, and Jobs, just like in a regular cluster.

- A syncer component translates these resources into low-level resources (pods, services, etc.) in the host cluster's namespace, **adding suffixes** to avoid conflicts.

- **Networking is shared**, but **each vCluster is isolated by default**, ensuring strong multi-tenancy.

### Key Concepts

1. **vCluster Control Plane**
- A full Kubernetes control plane that runs inside a pod in the host cluster.
- Includes:
  - Kubernetes API server
  - Controller manager
  - Data store (like etcd, SQLite, etc.)
  - Syncer component (connects virtual and host cluster)

2. **Syncing Resources**
- vCluster runs your workloads by syncing pods from the virtual cluster to the host cluster

3. **Pod Scheduling**
- By default, vCluster reuses the host cluster scheduler to schedule workloads, but you can enable the virtual cluster's scheduler.

4. **Storage**
- vCluster allows using the host cluster’s StorageClasses directly.

- You don't need to define them in the virtual cluster if syncing from host to vCluster is enabled.

5. **Networking**
- Resources like Services and Ingresses are synced from virtual cluster to host cluster.

- This ensures that workloads in the vCluster are reachable via the host’s network infrastructure.

6. **Nodes**
- vCluster creates virtual/pseudo nodes based on pod .spec.nodeName values.

- These are not real nodes, but virtual representations within the vCluster to simulate scheduling and topology.


### References

(1*) https://www.vcluster.com/