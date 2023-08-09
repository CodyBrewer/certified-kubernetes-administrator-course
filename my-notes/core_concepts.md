# Core Concepts

- Kubernetes Cluster Architecture
- Docker, Containerd, and CLI Tools Overview
- Introduction to ETCD
- ETCD in Kubernetes
- ETCD & ETCDCTL Utility
- Kube-Apiserver in Kubernetes
- Controller Managers
- Kube Scheduler
- Kubelet
- Kube-proxy
- Kubernetes Pods
- Creating Kubernetes Pods using a YAML Configuration File

## Kubernetes Cluster Architecture

- Understand Kubernetes by comparing its components to a shipping analogy.
- Components can be broadly divided into:
  - Master Node (control ships)
  - Worker Node (cargo ships)

### Master Node Components (Control Ships)

- Role: Orchestrate and manage the Kubernetes cluster.

#### 1. **etcd**

- A `highly available key-value store`.
- Retains cluster configuration and state.
- Analogous to a database maintaining ship and container details.

#### 2. **Schedulers**

- Decides which node an incoming container should be placed on.
- Considers factors like:
  - Node capacity.
  - Resource requirements of the container.
  - Policies and constraints (e.g., taints, tolerations, node affinity rules).

#### 3. **Controllers**

- Multiple types available:
  - **Node Controller**: Manages nodes (e.g., onboarding, handling unavailability).
  - **Replication Controller**: Ensures a specified number of containers are always running.

#### 4. **Kube API Server**

- Primary management component.
- Orchestrates all operations within the cluster.
- Exposes Kubernetes API for external users, controllers, and worker nodes.

### Worker Node Components (Cargo Ships)

- Role: Host and manage application containers.

#### 1. **Container Runtime Engine (e.g., Docker)**

- Software required to run containers.
- Not restricted to Docker. Other engines: `ContainerD`, `Rocket`.

#### 2. **Kubelet**

- Acts as the agent on each node.
- Communicates with the Master Node.
- Manages containers based on instructions from Kube API Server.

#### 3. **Kube Proxy Service**

- Enables inter-container communication across nodes.
- Sets up rules to allow containers to discover and communicate with each other.

### Kubernetes Architecture Summary

- **Master Node** hosts:
  - `etcd` for data storage.
  - Schedulers for container placement.
  - Controllers for managing various cluster aspects.
  - Kube API Server for orchestration.
- **Worker Node** hosts:
  - Container Runtime to run containers.
  - Kubelet for node-level operations.
  - Kube Proxy for inter-container communication.

> **Up Next**: In-depth exploration of each component in upcoming lectures.

## Docker, Containerd, and CLI Tools Overview

### Docker vs. Containerd

- **Docker**:
  - Prevalent in older blogs and documentation.
  - Dominant due to its user-friendly interface.
  - Initially, Kubernetes was built to orchestrate Docker only.
  - Docker is more than just a container runtime (includes CLI, API, build tools, volume support, etc.)
- **Containerd**:
  - Part of Docker, but can function as its standalone runtime.
  - CRI (Container Runtime Interface) compatible.
  - As of Kubernetes 1.24, direct support for Docker was removed. However, images built by Docker still work with Containerd due to OCI standards.
  - Now a separate project and CNCF member.

### CLI Tools

1. **ctr**:
   - CLI tool that comes with Containerd.
   - Mainly for debugging with limited features.
   - E.g., `ctr images pull` to pull images.

2. **nerdctl (nerd control)**:
   - Docker-like CLI for Containerd.
   - More user-friendly compared to ctr.
   - Supports many Docker features and some advanced features like encrypted images and lazy image pulling.
   - Usage similar to Docker, e.g., `nerdctl run`.

3. **crictl (cri control)**:
   - Developed by Kubernetes community.
   - Interacts with CRI compatible container runtimes.
   - Primarily for debugging.
   - Can interact with pods, unlike Docker. E.g., `crictl pods` to list pods.
   - Syntax is very similar to Docker commands, making the transition smoother.

### Comparison Table

| Function | Docker | crictl |
|----------|--------|--------|
| Attach   | ✓      | ✓      |
| Exec     | ✓      | ✓      |
| Images   | ✓      | ✓      |
|...       |...     |...     |

> **Note**: Full list of differences available at https:kubernetes.io/docs/reference/tools/map-crictl-dockercli/

### Key Takeaways

- **Kubernetes 1.24 Changes**: Introduced `cri-dockerd.sock` replacing the older `dockershim.socketpoint`.
- **ctr & crictl**: Mainly for debugging. While ctr is Containerd-specific, crictl works with all CRI compatible runtimes.
- **nerdctl**: Docker-like CLI for general purposes.

---

**References**:

- [Kubernetes cri tools GitHub repository](https://github.com/kubernetes-sigs/cri-tools) - [PR 869](https://github.com/kubernetes-sigs/cri-tools/pull/869) & [Issue 868](https://github.com/kubernetes-sigs/cri-tools/issues/868)

## Introduction to ETCD

### Basics of ETCD

- **What is ETCD?**:
  - A `distributed reliable key-value store`.
  - Described as `simple`, `secure`, and `fast`.

### Key-Value Store Vs Traditional Databases

- **Traditional Databases**:
  - Uses `rows` and `columns` (tabular format).
  - Adjusting the schema (e.g., adding columns) affects all records.
  - Can result in lots of `empty cells`.
- **Key-Value Store**:
  - Information stored as `documents` or `pages`.
  - Each entity gets its document.
  - Flexible structure: Documents for working individuals might have `salary` fields, students might have `grades`, etc.
  - Changes to one document don't affect others.
  - Complex data usually in `JSON` or `YANO`.

### Getting Started with ETCD

- **Installation**:
  - Simply download the binary, extract, and run.
  - Grab it from the GitHub releases page.
  - ETCD service listens on default port `2379`.

- **Using the ETCD Control Client (`etcdctl`)**:
  - CLI tool for ETCD.
  - Use `etcdctl set key1 value1` to store data.
  - Use `etcdctl get key1` to retrieve data.
  - Run `etcdctl` with no arguments for more options.

### ETCD Release History & Commands

- **Release Highlights**:
  - **0.1**: Initial release in August 2013.
  - **2.0**: February 2015. RAFT consensus algorithm redesign. 10k+ writes/sec.
  - **3.0**: January 2017. Optimizations and improvements.
  - **CNCF Incubation**: November 2018.
- **Version Differences**:
  - Major change between `version 2` and `version 3`.
  - Command variations between the two versions.
  - Check version with `etcdctl version`.
- **Version vs API Version**:
  - `etcdctl` has its version.
  - It also specifies which API version it's set up for (`V2` or `V3`).
  - Remember: API version determines the command set.
- **Toggling Between API Versions**:
  - Use the environment variable `ETCDCTL_API`.
  - Set it to `2` or `3` as needed.
  - With `V3`, commands have changed: `etcdctl put` to set and `etcdctl get` to retrieve.

> **Up Next**: Exploring how ETCD integrates within Kubernetes and how to ensure high availability.

## ETCD in Kubernetes

### Role of ETCD in K8s

- **Central Storage in Kubernetes**:
  - Holds all crucial cluster info:
    - `Nodes`, `pods`, `configs`, `secrets`, `accounts`, `roles`, `role bindings`, etc.
  - **Note**: When using `kubectl get`, the info fetched is from ETCD.
  - **Critical Point**: Updates are finalized ONLY once reflected in ETCD.

### Different K8s Deployment Methods

- Two main methods highlighted:
  1. **From Scratch**: In-depth method where you handle ETCD setup.
  2. **kubeadm**: Automated tool for quicker setup. Used in our practice tests.

### Setting Up ETCD from Scratch

- Steps:
  1. Download and install `ETCD binaries`.
  2. Setup ETCD as a service on the master node.
- **Config Options**:
  - Lots related to `TLS certificates`:
    - Deep dive coming up in the TLS section.
  - Cluster setup options discussed in the HA section.
- 🚀 **Key Takeaway**: The `Advertised client URL`.
  - ETCD's listening point.
  - Usually on server IP at port `2379`.
  - **Action Item**: Remember to configure this in the Kubernetes API server.

#### Deploying with `kubeadm`

- `kubeadm` handles ETCD setup, positioning it as a pod inside `kube-system` namespace.
- **Pro Tip**: Utilize `etcdctl` within the pod to explore ETCD database.
- To view all Kubernetes keys: `etcdctl get`.
- **K8s Data Structure**:
  - Base directory = `registry`.
  - Contains entities like `nodes`, `pods`, `replica sets`, `deployments`, etc.

### High Availability (HA) Considerations

- In HA environments:
  - Multiple master nodes ➡️ Multiple ETCD instances.
  - **Action Item**: Ensure ETCD instances communicate by tweaking the ETCD service config.
    - Check the `initial cluster` option to define ETCD instances.
- More on HA is on the horizon in this course.

> **Final Thought**: High availability is crucial for robust K8s setups. Keen to dive deeper into it in upcoming lessons.

## ETCD & ETCDCTL Utility

- **ETCDCTL** is the command-line interface (CLI) for interacting with ETCD.
- Supports two API versions: **Version 2** and **Version 3**.
- Setting the correct API version is crucial, as commands vary between versions.

### API Versions & Commands

#### Version 2 (Default)

Commands for ETCDCTL version 2 include:

- `etcdctl backup`
- `etcdctl cluster-health`
- `etcdctl mk`
- `etcdctl mkdir`
- `etcdctl set`

#### Version 3

Commands for ETCDCTL version 3 include:

- `etcdctl snapshot save`
- `etcdctl endpoint health`
- `etcdctl get`
- `etcdctl put`

### Setting the API Version

- To specify which API version to use, set the `ETCDCTL_API` environment variable.

    ```bash
    export ETCDCTL_API=3
    ```

- Default behavior:
  - When **not** set: Defaults to version 2.
  - When set to version 3: Only version 3 commands are functional.

### Authentication to ETCD API Server

- It's essential to provide certificate files for ETCDCTL to authenticate with the ETCD API Server.
- **Certificate Paths** (on `etcd-master`):
  - CA Certificate: `/etc/kubernetes/pki/etcd/ca.crt`
  - Server Certificate: `/etc/kubernetes/pki/etcd/server.crt`
  - Server Key: `/etc/kubernetes/pki/etcd/server.key`

### Example Command

To run commands with the proper API version and certificates, use the following format:

```bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```

## Kube-Apiserver in Kubernetes

### Introduction

- In this lecture, the primary focus is on `kube-apiserver`, the central management component in Kubernetes.

### How kubectl Interacts with kube-apiserver

1. Running a `kubectl` command communicates with the `kube-apiserver`.
2. `kube-apiserver` authenticates and validates the request.
3. Data is retrieved from the `etcd` cluster and sent back to the requester.
4. Alternatively, APIs can be invoked directly using a POST request, bypassing `kubectl`.

### Workflow of Creating a Pod

1. The request to create a pod is authenticated and validated.
2. A pod object is created, but no node is assigned yet.
3. `kube-apiserver` updates the information in `etcd` and informs the user.
4. The scheduler monitors the API server and assigns a node for the new pod.
5. This node information is passed to the `kubelet` on the appropriate worker node.
6. `kubelet` instructs the container runtime engine to deploy the application.
7. Upon success, `kubelet` sends status updates back to the API server, which then updates `etcd`.

### Role of Kube-Apiserver

- Central hub for tasks to make changes in the cluster.
- Authenticates and validates requests.
- Only component that interacts directly with the `etcd` data store.
- Other components (e.g., scheduler, kube-controller-manager, kubelet) use the API server for cluster updates.

### Configuring Kube-Apiserver

1. Available as a binary on the Kubernetes release page.
2. To be run as a service on the Kubernetes master node.
3. Comes with numerous parameters for configuration.
4. Understanding key configurations can aid in future cluster setups.

### Kubernetes Architecture

- Comprises various components that interact and communicate.
- Has multiple authentication, authorization, encryption, and security modes, leading to many configuration options.
- For this lecture, focus is on a few vital configurations:
  - **Certificates**: For securing component interactions. More details in the SSL/TLS section.
  - **etcd servers option**: To specify the location of `etcd` servers, enabling the `kube-apiserver` to connect.

### Viewing Kube-Apiserver Options

- Depends on the cluster setup method:
  - **Using kubeadmin**: The `kube-apiserver` is deployed as a pod in the `kube-system` namespace. Options are in the pod definition file at `etc/kubernetes/manifest`.
  - **Non-kubeadmin setup**: Options can be seen in the service file at `etc/systemd/system/kube-apiserver.service`. You can also see the running process on the master node by searching for `kube-apiserver`.

## Controller Managers

### **Introduction**

- Discussing the role and importance of the **Kube Controller Manager** in Kubernetes.

### **What is a Controller?**

- **Analogical Explanation**:

  - A controller in Kubernetes is likened to an office or department within a ship.
  - These offices monitor ships' statuses and take necessary actions, e.g., when a ship arrives, leaves, or gets destroyed.
- **Technical Definition**:

  - In Kubernetes, a controller is a process that:
        1. Continuously monitors the state of various components.
        2. Aims to bring the entire system to a desired state.

### **Examples of Controllers**

1. **Node Controller**:

    - Role: Monitors the status of nodes and ensures applications keep running.
    - Interaction: Works through the Kube API server.
    - Behavior:
        - Checks node status every 5 seconds.
        - Marks a node as unreachable if no heartbeat is received for 40 seconds.
        - Gives the node 5 minutes to recover.
        - If not recovered, moves its PODs to healthy nodes (if they're part of a replica set).
2. **Replication Controller**:

    - Role: Monitors the status of replica sets.
    - Behavior:
        - Ensures the desired number of PODs are available within a replica set.
        - If a POD dies, it creates a new one.

- Note: Multiple other controllers exist within Kubernetes, implementing intelligence for constructs like deployments, services, namespaces, and persistent volumes.

### **Locating and Viewing Controllers**

- Controllers are bundled into one process: **Kubernetes Controller Manager**.
- On installation of Kubernetes Controller Manager, various controllers get installed.

### **Installation and Configuration**

- **Installation Steps**:

    1. Download Kube Controller Manager from the Kubernetes release page.
    2. Extract and run it as a service.
- **Configuration Options**:

  - A variety of customization options are provided.
  - Specifics to remember:
    - Default settings for node controller, e.g., node monitor period, grace period, and eviction timeout.
    - `controllers` option allows specifying which controllers to enable (all are enabled by default).

### **Inspecting Kube Controller Manager**

- **With Kube admin tool**:

  - Kube Controller Manager is deployed as a POD in the `kube-system` namespace on the master node.
  - Options can be found in the POD definition file at `/etc/kubernetes/manifest`.
- **Without Kube admin tool**:

  - Inspect options by checking the Kube Controller Manager service in the services directory.
  - View running processes on the master node to see effective options for the Kube Controller Manager.

## Kube Scheduler

### **Role of Kube-Scheduler**

- **Main Responsibility**: Deciding which pod should be placed on which node.
- **Misconception**: The scheduler doesn't place the pod on nodes. That's the **kubelet's** job.

### **Why Do We Need a Scheduler?**

- **Analogical Explanation**:
  - Just like ensuring the right container ends up on the right ship based on ship size and destination, the scheduler ensures pods are placed correctly.
- **Technical Reasons**:
  - Match pod resource requirements with node capacities.
  - Certain nodes might be dedicated to specific applications.

### **How Does the Scheduler Work?**

1. **Pod Analysis**: For each pod, the scheduler identifies its resource needs.
2. **Filtering Phase**: Nodes that do not meet the pod's requirements are filtered out.
    - Nodes lacking sufficient CPU/memory are excluded.
3. **Ranking Phase**: Remaining nodes are ranked based on best fit.
    - A priority function assigns scores to nodes (0-10 scale).
    - The node with the best score (e.g., most free resources after pod placement) is selected.

### **Customization**

- The scheduler can be tailored to specific needs.
- Users can even write their own scheduler.
- Further details will be covered in the subsequent course section.

### **Installing the Kube-Scheduler**

1. Download the kube-scheduler binary from the Kubernetes release page.
2. Extract and run it as a service.
3. Specify the scheduler configuration file when running as a service.

### **Viewing Kube-Scheduler Options**

- **With kubeadm tool**:
  - The kube-scheduler is deployed as a pod in the `kube-system` namespace on the master node.
  - Options can be viewed in the pod definition file located at `/etc/kubernetes/manifest/` folder.
- **Without kubeadm tool**:
  - Inspect the kube-scheduler process on the master node to see its options.

## Kubelet

### **Role of Kubelet**

- **Analogy**: Kubelet is likened to the captain on a ship.
  - **Responsibilities**:
    - Leading all ship activities.
    - Handling necessary paperwork for joining the cluster.
    - Being the singular point of contact with the master (or the "mastership").
    - Loading and unloading containers based on instructions from the master's scheduler.
    - Periodically reporting on the ship's (node's) status and the status of its containers.

### **Kubelet in Practice**

- **On a Kubernetes Worker Node**:
  - Kubelet registers the node with the Kubernetes cluster.
  - Upon receiving pod/container loading instructions, kubelet communicates with the container runtime engine (e.g., Docker).
    - It requests the engine to pull the necessary image and run an instance.
  - Kubelet continuously monitors the pod and its containers.
    - Regularly reports the state to the kube API server.

### **Installation of Kubelet**

- **Using `kubeadm`**:
  - Although the `kubeadm` tool can deploy a cluster, it **does not** automatically deploy the kubelet.
    - This behavior contrasts with other components.
  - Kubelet installation is a **manual process** on worker nodes:
        1. Download the installer.
        2. Extract the installer.
        3. Run kubelet as a service.
- **Viewing Kubelet in Action**:
  - To observe the kubelet process and its options:
    - List processes on the worker node and search specifically for "kubelet."

### **Further Studies**

- Later in the course:
  - Deeper dive into kubelets.
  - Configuration of kubelets.
  - Certificate generation processes.
  - How to utilize TLS Bootstrapping for kubelets.

## Kube-proxy

### **Pod Networking in Kubernetes**

- **Inter-pod Communication**: Within a Kubernetes cluster, every pod can communicate with every other pod.
  - Achieved via a **pod networking solution**.
  - **Pod Network**: A virtual network spanning across all nodes. All pods connect to this network, allowing them to communicate with each other.
  - Multiple solutions available for setting up this network.

### **Services and their Role**

- **Scenario**:

  - A web application on Node 1.
  - A database application on Node 2.
  - Web app can communicate with the database using the pod's IP.
  - **Note**: The IP of the database pod might change over time.
- **Utilizing Services**:

  - A better approach: Use a **service**.
    - Exposes the database application across the cluster.
    - The web application accesses the database using the service name, e.g., "DB".
  - A service has its own assigned IP.
  - When a pod attempts to connect to a service via its IP or name, it routes the traffic to the appropriate backend pod.
- **Nature of Services**:

  - Not a real entity like pods.
  - Virtual component existing in Kubernetes memory.
  - It doesn’t join the pod network as it doesn't have interfaces or a listening process.

### **Role of Kube-proxy**

- **Definition**: A process that runs on every node in the cluster.
- **Responsibility**:
  - Monitor for new services.
  - Upon the creation of a new service, it formulates rules on each node to route traffic from those services to the backend pods.
  - One mechanism for this: Using **iptables rules**.
    - Example: Traffic targeting the service IP (10.96.0.12) is directed to the actual pod IP (10.32.0.15).

### **Installing Kube-proxy**

- **Installation Steps**:

    1. Download kube-proxy binary from the Kubernetes release page.
    2. Extract the binary.
    3. Run kube-proxy as a service.
- **Deployment via `kubeadm`**:

  - `kubeadm` deploys kube-proxy as pods on every node.
  - Deployed using **DaemonSet** ensuring a single pod is always active on each node.
  - **Note**: If you're unfamiliar with DaemonSet, it will be covered in an upcoming lecture.

### **Conclusion and Future Direction**

- The lecture provides a high-level overview of components in the Kubernetes control plane.
- Comprehensive exploration of these components will occur in subsequent sections of the course.

## Kubernetes Pods

- **Main Topic**: **Kubernetes Pods**.

### **Prerequisites**

- **Application Deployment**:

  - Application is developed and transformed into Docker images.
  - Docker images are available in a repository like Docker Hub so Kubernetes can retrieve it.
- **Kubernetes Cluster**:

  - Cluster should be operational.
  - Configuration can be either single-node or multi-node.
  - All services should be active.

### **Understanding Pods**

- **Containers and Pods**:

  - Goal: Deploy applications as containers on machines in a Kubernetes cluster.
  - Containers are not directly deployed. They are encapsulated within **pods**.
  - **Pod**: A single instance of an application, smallest object in Kubernetes.
- **Scaling with Pods**:

  - With increased application demand, more instances are needed.
  - To scale up, create additional pods, not containers within an existing pod.
  - Each pod maintains a 1:1 relationship with a container.
  - If physical capacity is met on a node, new pods are deployed to other nodes.

### **Container Relations within a Pod**

- **Multiple Containers in a Pod**:
  - Typically, a pod has a 1:1 relationship with a container.
  - However, a pod can house multiple containers, but usually not of the same type.
  - **Helper Containers**: Containers that perform supporting tasks for the main application.
    - They can exist alongside the main container in the same pod.
    - They are created and destroyed together since they share the same pod.
  - Containers within a pod can communicate directly as they share the same network space. They can also share storage.

### **Analogy with Docker Containers**

- **Simple Docker Container Deployment**:

  - Initially, deploy an application using a Docker run command.
  - For increased demand, run the command multiple times for more instances.
- **Complex Docker Container Deployment**:

  - With application growth, helper containers emerge.
  - Need for manual mapping of app and helper containers.
  - Challenges: Establishing network connectivity, shared volumes, monitoring state, and manual lifecycle management.
- **Kubernetes Pods Simplify Deployment**:

  - Pods provide automatic management of containers.
  - They ensure containers in a pod share storage, network namespace, and lifecycle.
  - Pods are necessary even for simple applications to anticipate future growth.
  - In this course, focus is on one container per pod, although multi-container pods are possible.

### **Deploying Pods**

- **Command**: `kubectl run`

  - Deploys a Docker container by creating a pod.
  - Image name is specified with `--image` parameter.
  - Image is fetched from Docker Hub or another repository.
- **Viewing Deployed Pods**:

  - Use `kubectl get pods` to view a list of active pods.
- **Access to Deployed Pods**:

  - Pods in this state aren't accessible externally, only internally from the node.
  - Future lectures will cover networking and services to expose services to users.

## Creating Kubernetes Pods using a YAML Configuration File

### Kubernetes and YAML

- Kubernetes uses YAML files as inputs.
- Objects created: pods, replicas, deployments, services, etc.
- All objects have similar structure.

---

### Four Top-Level Fields in Kubernetes YAML

Every Kubernetes YAML file contains:

1. **API version**: Version of Kubernetes API used.
2. **Kind**: Type of object being created.
3. **Metadata**: Information about the object (name, labels, etc.).
4. **Spec**: Details and specifications about the object.

---

### Deep Dive: Fields

#### 1. API version

- For creating pods: `v1`.
- Other versions: `apps/V1beta`, `extensions/V1beta`.

#### 2. Kind

- For our case: `Pod`.
- Other possible values: `ReplicaSet`, `Deployment`, `Service`.

#### 3. Metadata

- Name: Identifier for the pod (e.g., `My App Pod`).
- Labels: Key-Value pairs.
  - Use to group and identify objects later (e.g., frontend, backend, database).
  - E.g., `app: my app`.

> **Note**: Only specific properties are allowed under `metadata`, but any key-value pairs can be added under `labels`.

#### 4. Spec

- Contains detailed specifications of the object.
- E.g., for a pod with a single container:
  - Container details (name, image, etc.).
  - Containers can be an array, as a pod can have multiple containers.
  - For this lecture: single container with image `NGINX`.

### Practical Application

- Once YAML file is ready, use command to create the pod:

    `kubectl create -f pod-definition.yaml`

- To view pods:

    `kubectl get pods`

- For detailed pod info:

    `kubectl describe pod <pod>`

### Pods with YAML summary

1. Remember the four top-level properties: API version, kind, metadata, spec.
2. Structure and values depend on the object type.
3. Use kubectl commands for interactions and querying pods.
