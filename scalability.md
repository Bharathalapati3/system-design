# Scalability

## 1. Vertical Scaling (Scale Up)

**Definition:** Increase CPU cores and RAM of a single machine.

**Pros:**
- Simple to implement
- Single machine to manage

**Cons:**
- ❌ Limited by hardware ceiling
- ❌ More downtime during upgrades
- ❌ Single point of failure
- ⚠️ More expensive at higher tiers

**Best for:** POC servers or low/no-traffic applications

---

## 2. Horizontal Scaling (Scale Out)

**Definition:** Increase the number of machines to distribute the load.

**Architecture:**
```
                    ┌─────────────┐
                    │Load Balancer│
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
    ┌───▼───┐          ┌───▼───┐          ┌──▼───┐
    │Server1│          │Server2│          │Server3│
    └───────┘          └───────┘          └──────┘
```

**Requirements:**
- ✅ Stateless application design
- ✅ Load balancer to distribute requests
- ✅ Database replication/sharding

**Benefits:**
- ✅ High availability (99.99% uptime)
- ✅ No single point of failure
- ✅ Better resource utilization
- ✅ Cost-effective scaling

---

## Monitoring & Performance Tools

### Commands

| Command | Purpose |
|---------|---------|
| `htop` | Live CPU and memory usage monitoring |
| `vmstat` | Memory, swap, and CPU activity per second |
| `perf` | Code profiling to identify bottlenecks |

### Monitoring Platforms

- **Prometheus** - Metrics collection and alerting
- **DataDog** - Full-stack monitoring and analytics
- **Grafana** - Metrics visualization and dashboards

---

## Key Takeaways

| Approach | Best Use Case | Availability |
|----------|---------------|--------------|
| **Vertical** | Small apps, POC | Lower |
| **Horizontal** | Production systems | 99.99%+ |
