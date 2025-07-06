
---

# 📁 `snapp_senior_devops_interview-2025_may`

## Kubernetes

---

### 1. **What is Kubernetes?**

Kubernetes is an open-source **container orchestration platform** designed to automate deployment, scaling, and operation of application containers across clusters of hosts.

> 📌 You use Kubernetes when you need **automation**, **scalability**, **high availability**, and **self-healing** for containerized applications.

It shines in environments where:

* You need to run microservices at scale.
* You want to decouple workloads from infrastructure.
* You’re moving toward GitOps or declarative infrastructure.

However, Kubernetes isn’t always the best choice:

* Small workloads may be overkill.
* It introduces operational complexity.
* It requires a learning curve and infrastructure investment.

> **TL;DR**: Kubernetes is ideal when your system's **scale, complexity, or uptime** requirements justify its operational cost.

---

### 2. **What are the Core Components of Kubernetes?**

| Component                 | Description                                                                              |
| ------------------------- | ---------------------------------------------------------------------------------------- |
| `etcd`                    | A consistent and highly available **key-value store** for all cluster data.              |
| `kube-apiserver`          | The **entry point** for all API calls. Manages communication inside the cluster.         |
| `kube-scheduler`          | Decides **which node** a Pod should run on, based on resource requirements and policies. |
| `kube-controller-manager` | Manages **controllers** that regulate the cluster state (e.g., node, replication, job).  |
| `kubelet`                 | An agent that runs on each node to ensure containers are running in a Pod.               |
| `kube-proxy`              | Manages **network routing** for services on each node using iptables or IPVS.            |
| `Container Runtime`       | Executes and manages containers (e.g., containerd, CRI-O, Docker – deprecated).          |

> All these components work together to provide a **resilient, self-healing, and declarative infrastructure** for containers.

---

### 3. **Is `kubelet` started on all nodes?**

Yes — **`kubelet` must run on every node** (including control plane nodes if they're also running workloads, but it is not best practice).

> 🧠 The `kubelet` is the **primary node agent** that connects each node to the Kubernetes control plane.

#### 🔧 Why is it required?

* It **registers the node** with the cluster via the API server.
* It **monitors Pod specs** assigned to that node.
* It ensures that containers are **running and healthy**.
* It **communicates with the container runtime** (e.g., containerd) to manage containers.

> If a node doesn’t have `kubelet` running, it **cannot join** the cluster or run workloads.

---

### 4. **Who checks the probes of the Pod?**

The **`kubelet`** is responsible for checking both **liveness** and **readiness probes** of Pods running on its node.

#### 🔍 Why `kubelet`?

Because:

* `kubelet` manages the **lifecycle** of all Pods on the node.
* It watches the PodSpec (via the API server) and performs health checks according to the probe configuration.
* It uses HTTP, TCP, or command execution to run the probe logic defined in the Pod YAML.

#### 🔄 What happens when a probe fails?

* **Liveness probe fails**: `kubelet` kills the container → it will be restarted by the controller (e.g., Deployment).
* **Readiness probe fails**: `kubelet` marks the Pod as **not ready** → it is removed from Service load balancing.

---

### 5. **What is the Kubernetes Control Plane?**

The **control plane** is the **brain** of the Kubernetes cluster.

| Component                               | What It Does                                                            |
| --------------------------------------- | ----------------------------------------------------------------------- |
| `kube-apiserver`                        | The **front door**. All communication goes through here.                |
| `etcd`                                  | The **database** that stores everything (cluster state).                |
| `kube-scheduler`                        | Decides **which node** will run a new Pod.                              |
| `controller-manager`                    | Keeps the cluster in the **desired state** (e.g., scaling, restarting). |
| (Optionally) `cloud-controller-manager` | Talks to the cloud provider (e.g., creates load balancers).             |

> All of these run on **control plane nodes** (also called **master nodes**).

---

Great intuition again — you're right that the `kube-apiserver` is the central communication point. Let’s rephrase and expand your answer into a clear and professional response, suitable for a senior DevOps interview:

---

### 6. **How does the `kubelet` communicate with the controller?**

The **`kubelet` communicates with the Kubernetes control plane via the `kube-apiserver`** — the central and only access point to all cluster state and controller logic.

#### 🔗 Communication Flow:

* `kubelet` watches the **Pod specifications** assigned to its node via the `kube-apiserver`.
* It **reports node status**, **Pod status**, and **probe results** back to the API server.
* It doesn’t talk **directly** to the scheduler or controllers — **only through the API server**.

---

### 🧠 Why do we say "controller"?

In Kubernetes, **controllers** are control loops that **watch resources** via the API server and **reconcile** the actual state with the desired state.

#### 🤖 Common controllers (running inside `kube-controller-manager`):

* **ReplicaSet Controller**: Ensures correct number of Pod replicas.
* **Node Controller**: Monitors node health.
* **Job Controller**: Handles completion of batch jobs.
* **Deployment Controller**: Manages rollouts and updates.

So when we say "controller," we refer to any of these logic components that act upon the state reported to the `kube-apiserver`.

---

### 7. **How Does Kubernetes Process a Request?**

📥 Step-by-Step:

1. **Client Sends Request**

   * Could be an API call, HTTP request, etc.

2. **Ingress Point (depends on Service type)**

   * **`LoadBalancer`**: If exposed via a cloud LB, traffic hits the external LB → forwarded to a Node.
   * **`Ingress`**: For HTTP/S traffic, goes through an Ingress Controller (like NGINX, Traefik).
   * **`NodePort`**: Accesses a high port on any node → forwarded internally.
   * **`ExternalName`**: Simply resolves to an external DNS name (no traffic routing in cluster).

3. **kube-proxy (on the target node)**

   * Receives traffic for the **ClusterIP** of the Service.
   * Uses **iptables**, **IPVS**, or **eBPF** to route traffic to a healthy backend Pod.

4. **ClusterIP Service**

   * The virtual IP assigned to the Service.
   * kube-proxy balances traffic across one of the selected Pods.

5. **Pod**

   * Finally, the request reaches the container inside a Pod via the `targetPort`.

---

### 8. **How Does Kubernetes Handle Authentication and Authorization?**

Kubernetes separates **auth** into two key phases:

---

## 🔐 1. **Authentication (Who are you?)**

Kubernetes first checks *who* is making the request. It supports several **authentication methods**, such as:

| Method             | Example Use Case                           |
| ------------------ | ------------------------------------------ |
| TLS Client Certs   | Admin users / kubelets                     |
| Bearer Tokens      | Service accounts / static tokens           |
| OIDC               | Login via Google, Azure AD, Keycloak, etc. |
| Webhook Token Auth | Custom external auth service               |
| Anonymous Access   | Disabled by default (for security reasons) |

The **`kube-apiserver`** performs this check.

---

## ✅ 2. **Authorization (Are you allowed to do this?)**

Once the user is authenticated, Kubernetes checks whether they’re **authorized** to perform the requested action.

This is handled using:

### 🔑 **RBAC (Role-Based Access Control)**

RBAC uses:

* **Roles / ClusterRoles**: Define permissions (verbs like get, list, delete on resources).
* **RoleBindings / ClusterRoleBindings**: Attach those roles to users or service accounts.

---

## 📍 Optional: Admission Control

After passing authentication and authorization, **admission controllers** can perform **validations, mutations, or policy enforcement** (e.g., deny creating a privileged Pod).

---

### 9. **How Does Communication Happen in Kubernetes?**

Kubernetes networking enables **Pod-to-Pod**, **Pod-to-Service**, and **intra-/inter-node** communication through a combination of:

---

## 🔧 Key Components of K8s Networking

| Component           | Role in Communication                                  |
| ------------------- | ------------------------------------------------------ |
| **CNI Plugin**      | Manages Pod IPs and Pod-to-Pod communication           |
| **kube-proxy**      | Routes Service traffic to Pods using ClusterIP         |
| **ClusterIP**       | Virtual IP for Services that routes traffic internally |
| **Node networking** | Ensures traffic can move between nodes                 |

---

### 📡 1. **Pod-to-Pod Communication (CNI)**

* Kubernetes expects **all Pods to have unique IPs** in a flat network.
* This is handled by the **CNI (Container Network Interface)** plugin.

  * Examples: Calico, Flannel, Cilium, Weave.
* The CNI ensures that:

  * Every Pod gets a routable IP.
  * Any Pod can talk to any other Pod — on the **same or different nodes**.

> ✅ No NAT is required for Pod-to-Pod traffic (in ideal CNI setups).

---

### 🔀 2. **Pod-to-Service Communication (kube-proxy + ClusterIP)**

* Services (like `ClusterIP`) use **virtual IPs**.
* `kube-proxy` intercepts traffic to that IP and routes it to a healthy backend Pod using:

  * `iptables`
  * `IPVS`
  * `eBPF` (in advanced setups like Cilium)

> 📌 The request is load-balanced across Pods selected by the Service’s label selector.

---

### 🌐 3. **Cross-Node Communication**

* When Pods on different nodes talk, the traffic flows over the node's network.
* The CNI must support **inter-node routing**.
* kube-proxy ensures the request lands on the correct node and Pod.

---

### 10. **How does the Kubernetes network work?**
