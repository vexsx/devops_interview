
---

### 1. **Say Some Experience with a Load Balancer**

#### 🔄 Common Load Balancing Algorithms

---

| Algorithm         | Description                                                     |
| ----------------- | --------------------------------------------------------------- |
| Round Robin       | Distributes requests evenly in a circular order                 |
| Least Connections | Sends requests to the server with the fewest active connections |
| IP Hash           | Hashes client IP to consistently route to same backend          |
| Weighted          | Gives some servers more traffic based on capacity               |

---

#### 🧰 Types of Load Balancing

| Type             | Protocol   | Example Tools           |
| ---------------- | ---------- | ----------------------- |
| L4 (Transport)   | TCP/UDP    | HAProxy, LVS, Envoy     |
| L7 (Application) | HTTP/HTTPS | HAProxy, NGINX, Traefik |

---

❌ **HAProxy is not a web server.**

---

#### ⚙️ What HAProxy does:

* Load balances HTTP, HTTPS, TCP traffic
* Terminates TLS (SSL offloading)
* Performs health checks
* Applies routing, rate limiting, and stickiness
* Acts as a gateway or edge proxy

---

#### ❓Then what’s a web server?

A **web server** (like NGINX, Apache, or Caddy) serves:

* Static content (HTML, JS, images)
* Or runs dynamic content (via PHP, Python, etc.)

---

### 2. **Difference between passthrough and terminate**

| Mode            | TLS Handled By | Load Balancer Sees HTTP? | Can Inspect Traffic? | Use Case                       |
| --------------- | -------------- | ------------------------ | -------------------- | ------------------------------ |
| **Passthrough** | Backend server | ❌ No                     | ❌ No                 | End-to-end encryption needed   |
| **Termination** | Load balancer  | ✅ Yes                    | ✅ Yes                | L7 routing, WAF, observability |

---

##### 🔄 1. **TLS Passthrough**

* The load balancer **does NOT decrypt** TLS.
* It just forwards encrypted traffic to the backend.
* Backend server terminates TLS.


##### 🔍 2. **TLS Termination**

* The load balancer **decrypts TLS** and handles HTTPS.
* It can inspect HTTP headers, paths, etc.
* Optionally re-encrypts traffic to backend (called **TLS re-encryption**)

---

### 3. **How can we reduce ssl costs?**

#### ✅ Use Free Certificate Authorities:

* **Let’s Encrypt** (most popular)

  * 100% free TLS certificates
  * Supports automatic renewal (via certbot or acme.sh)
* **ZeroSSL**, **Buypass**, **ACME CA**: Other free providers
* **Wildcard certificates** via DNS-01 challenge (e.g., `*.example.com`)

#### 🧠 Optimize TLS Settings:

* Prefer **ECDSA** over RSA (faster and smaller)
* Use modern TLS versions: **TLS 1.3** is faster (fewer round-trips)
* Use **session resumption** (session tickets or session IDs)
* Enable **OCSP stapling** to reduce client-side verification time

#### ⚙️ Use SSL Offloading:

* Terminate TLS at a **load balancer** (e.g., HAProxy, NGINX, AWS ELB)
* Offload to a dedicated **TLS proxy** or **hardware device** (HSM or SSL accelerator)

#### 🚀 Use a CDN:

* Offload TLS to a provider like **Cloudflare**, **Fastly**, or **AWS CloudFront**
* Reduces TLS handshake overhead on your origin server
* Often includes free certs + DDoS protection


#### 🔐 Use Wildcard Certificates:

* One cert to cover many subdomains = easier management

  * e.g., `*.yourdomain.com` covers `api.`, `admin.`, `www.`

#### 📦 Centralize TLS:

* Terminate TLS in **one layer** (e.g., HAProxy, Envoy)
* Internally, communicate over HTTP to reduce encryption overhead (if security allows)

---

### 4. **How TLS/SSL Works — Step-by-Step (Based on the Video)**

#### 🧱 Step 0: TCP Connection

Before TLS even starts, the client and server perform a **TCP 3-way handshake**:

```
Client → SYN
Server → SYN-ACK
Client → ACK
```

✅ This establishes a **reliable connection** over which TLS can operate.

---

#### 🔐 Step 1: Client Hello

The client (e.g., browser) sends a **Client Hello** message with:

* TLS version it supports
* Supported cipher suites
* A random number (used later in key generation)
* SNI (Server Name Indication — the domain it wants)

---

### 📜 Step 2: Server Hello + Certificate

The server replies with:

* Chosen TLS version and cipher suite
* Its **digital certificate** (public key signed by a CA)
* A server-generated random number

---

### 🔍 Step 3: Certificate Validation

The client verifies the certificate:

* Is it issued by a **trusted CA**?
* Is it **not expired**?
* Does the **domain match**?

✅ If any of these checks fail → ⚠️ browser warning
✅ If they pass → continue to key exchange

---

### 🔑 Step 4: Key Exchange

* In modern TLS (e.g., TLS 1.3), client and server use **Elliptic Curve Diffie-Hellman (ECDHE)** to securely generate a **shared secret**.
* No one else (even a sniffer) can see this secret.

---

### 🔒 Step 5: Encrypted Session Starts

Both parties now:

* Use the shared secret to derive **symmetric encryption keys**
* All further communication is **encrypted** and **authenticated**

---

### 🧠 Summary

| Step              | What Happens                              |
| ----------------- | ----------------------------------------- |
| 0. TCP Handshake  | Reliable connection is established        |
| 1. Client Hello   | Browser offers encryption methods         |
| 2. Server Hello   | Server chooses method + sends certificate |
| 3. Cert Check     | Client verifies certificate               |
| 4. Key Exchange   | Securely agree on a shared secret         |
| 5. Encrypted Data | Switch to fast symmetric encryption       |

---

