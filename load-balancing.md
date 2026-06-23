# Load Balancing

## Overview

Load balancing is a technique for distributing incoming network traffic and requests across multiple servers or resources. It ensures:
- **High availability** - No single point of failure
- **Scalability** - Handle more traffic by adding servers
- **Performance** - Optimal resource utilization
- **Reliability** - Fault tolerance and redundancy

---

## Load Balancing Algorithms

### Round Robin
- **How it works**: Distributes requests sequentially to each server in order
- **Best for**: Stateless applications with uniform server capacity
- **Pros**: Simple, fair distribution
- **Cons**: Ignores server load, doesn't consider active connections

### Least Connection
- **How it works**: Routes requests to the server with the fewest active connections
- **Best for**: Long-lived connections, file uploads, streaming services
- **Pros**: Better load distribution for variable request durations
- **Cons**: More computational overhead to track connections

### IP Hash (Source IP Affinity)
- **How it works**: Routes requests from the same client IP to the same server
- **Best for**: Sessions requiring client-server affinity, without shared session storage
- **Pros**: Consistent routing, no session lookup needed
- **Cons**: Uneven load if many requests come from same IP; server failures break affinity

### Weighted Round Robin
- **How it works**: Distributes based on server capacity/weight
- **Best for**: Heterogeneous server infrastructure with different capabilities
- **Pros**: Accounts for server differences
- **Cons**: Manual weight configuration needed

---

## OSI Layer Considerations

### L4 - Network Load Balancing (TCP/UDP)
- Operates at the transport layer
- Routes based on IP protocol data
- **Examples**: NLB (AWS), HAProxy
- **Use case**: Extreme performance, millions of RPS, non-HTTP protocols

### L7 - Application Load Balancing (HTTP/HTTPS)
- Operates at the application layer
- Can inspect HTTP headers, URLs, hostnames
- **Examples**: ALB (AWS), Nginx, Traefik
- **Use case**: Most web applications, microservices, content-based routing
- **Popular tool**: Nginx

---

## Session Management

### Problem: Stateful Sessions
When a server maintains user session data, a load balancer must ensure the same user reaches that server consistently.

### Sticky Sessions (Session Affinity)
- Load balancer assigns a session to a server and always routes that user there
- **Problem**: If the server crashes, user session is lost and routed to another unaware server
- **Trade-off**: Reduces load balancer flexibility and creates bottlenecks

### Better Solution: Distributed Session Storage
- Store sessions in a shared cache layer (Redis, Memcached)
- Any server can handle any request for any user
- **Pros**: True horizontal scalability, fault tolerance
- **Best practice**: Use Redis for session storage across multiple servers

---

## Platform-Specific Load Balancers

### AWS
- **ALB (Application Load Balancer)** - L7, HTTP/HTTPS, content-based routing
- **NLB (Network Load Balancer)** - L4, extreme performance, TCP/UDP
- **Use when**: Building on AWS, need managed service, automatic scaling

### Kubernetes
- **Nginx Ingress Controller** - Popular, feature-rich, community support
- **Traefik** - Cloud-native, auto-discovery, dynamic configuration
- **Use when**: Container orchestration, microservices, self-managed infrastructure

### Bare Metal / On-Premises
- **Nginx** - Lightweight, flexible, reverse proxy + load balancer
- **HAProxy** - High-performance, enterprise-grade, powerful configuration
- **Use when**: Custom hardware, no cloud platform, maximum control

### Microservices Architecture (Service Mesh)
- **Envoy + Istio** - Advanced traffic management, observability, service discovery
- **Use when**: Complex microservices, need advanced routing policies, observability

### Enterprise On-Premises
- **F5** - Feature-complete, DDoS protection, SSL offloading, premium support
- **Use when**: Fortune 500 budget, compliance requirements, 24/7 support needed

---

## Additional Considerations

### Health Checks
- Load balancers periodically check if servers are healthy
- Unhealthy servers are removed from rotation
- Types: HTTP health checks, TCP ping, custom endpoints

### Failover & Redundancy
- Multiple load balancers in active-passive or active-active setup
- Ensures load balancer itself doesn't become a bottleneck

### Rate Limiting & Circuit Breaking
- Protect backend servers from overwhelming traffic
- Circuit breaker pattern: trip after threshold, fail fast

### SSL/TLS Termination
- Offload encryption/decryption to load balancer
- Reduces server CPU usage, simplifies backend certificate management
