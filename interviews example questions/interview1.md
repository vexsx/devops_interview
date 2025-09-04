
## Security Knowledge 🔒

* **Good to know:**

  * C2S/CIS
  * STIG
* **Desirable Knowledge:**

  * Firewalls

### Questions on Security

1. **Why do we disable kernel modules for security? 🛡️**

   * Disabling unnecessary kernel modules reduces the potential attack surface. By disabling unused modules, we can prevent attackers from exploiting vulnerabilities in these modules, improving overall system security. It’s a good practice to only load essential modules required for the system’s operation.

2. **What does "stateful firewall" mean? 🔥**

   * A stateful firewall tracks the state of active connections and makes decisions based on the state of the connection (e.g., new, established, or related). It provides a higher level of security than stateless firewalls by ensuring that each incoming packet is part of a valid, established connection.

3. **What is `dhparams` in NGINX? 🔑**

   * `dhparams` refers to Diffie-Hellman parameters used in NGINX for securely exchanging cryptographic keys. These parameters are essential for the strength of the TLS (SSL) encryption. Generating strong `dhparams` helps mitigate attacks like the Logjam vulnerability.

---

## Troubleshooting Skills 🛠️

* **Solid step-by-step troubleshooting approach:**

  * **Internet Issues 🌐:**

    * Troubleshooting internet issues often involves checking the network connection (e.g., `ping`, `traceroute`), DNS resolution, and ensuring proper routing configurations. Tools like `curl` and `nslookup` help identify issues with external connectivity.

  * **OS Issues 🖥️:**

    * Troubleshooting OS issues typically involves examining system logs, using commands like `dmesg`, `top`, and `ps` to identify resource utilization issues, and investigating configurations for system-level problems.

---

## Content Delivery Networks (CDN) & Rules 🌍

* **CDN Knowledge:**

  * CDNs are used to deliver content quickly to users by caching content at geographically distributed servers. They help reduce latency, enhance website speed, and decrease server load.

### Questions on CDN

1. **What is a Page Rule in a CDN? 📜**

   * A page rule in a CDN allows you to configure specific behaviors for different URLs or domains. You can define settings like cache control, redirection, SSL/TLS configurations, and more to optimize content delivery and security for particular pages.

2. **How do you validate caching by a CDN? 🔄**

   * To validate caching by a CDN, you can check the `Cache-Control` headers in the response, use tools like `curl` or browser developer tools to inspect cache status, or monitor cache hit/miss metrics in the CDN’s dashboard. Ensuring cache headers are properly set and verifying that content is served from cache are key steps.

---

## Tool Expertise ⚙️

* **Key tools and technologies:**

  * **NGINX Headers:**

    * `X-Cache-Miss` indicates that a requested resource was not served from the cache and had to be fetched from the origin server. This is useful for monitoring cache efficiency and troubleshooting cache issues.
  * **NGINX vs. HAProxy:**

    * **NGINX** is often used as a web server and reverse proxy. It’s efficient for handling HTTP traffic and offers easy configuration for SSL/TLS, load balancing, and caching.
    * **HAProxy** is a high-performance TCP/HTTP load balancer and proxy server. It’s more focused on load balancing and provides more granular control over traffic distribution. HAProxy is generally considered more robust for heavy traffic and complex load balancing scenarios.

---

## Networking 🌐

* **Traffic Forwarding with iptables:**

  * `iptables` is a powerful tool for configuring network traffic rules in Linux. You can use it to forward traffic between interfaces, set up NAT (Network Address Translation), block or allow specific IPs or ports, and more. It’s commonly used for network security and routing configurations.

* **Http Codes:**

  * read from [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) 

  * **diffrence between ssl termination and offloading**

  * read from [here](https://www.quotaguard.com/blog/ssl-passthrough-vs-ssl-offloading-a-quick-primer/#:~:text=the%20decrypted%20traffic.-,In%20SSL%20Termination%2C%20the%20load%20balancer%20establishes%20a%20new%20SSL,traffic%20as%20unencrypted%20HTTP%20traffic.) 



  * docker compose best practice ? 

  * diff between docker container and docker image 

  RabbitMQ vs Kafka? RabbitMQ = broker/routers with acks, flexible routing, per-message semantics; Kafka = distributed log, partitioned, pull-based, huge throughput & retention.

Clustering vs Federation/Shovel? Cluster = single logical broker in one LAN; Federation/Shovel = link brokers across WAN/DCs.

Quorum queues advantages? Raft-based replication, predictable failover, no “split-brain” mirrored-queue issues.

How to avoid message loss? Durable queues + persistent messages + publisher confirms + mirrored/quorum replication + correct acks.

Back-pressure handling? Prefetch tuning, max-length policies, lazy/quorum queues, rate limits, DLQs, and scale consumers.

kuber loadbalancers ?

