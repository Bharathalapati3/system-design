# Scalability

Verticle scaling - Increase more cpu cores, RAM of a single machine.
 - More down time, good for poc server or servers with low/no traffic.


Horizantal Scaling - Increase the number of tiny machines which provides zero redundacy. (99.99% availblity)
- Apps become stateless, need to maintain the load balancer to ditribute the requests to multiple servers.



Commands:
 - htop - live cpu and memory usage
 - vmstat - show memory, swap, cpu activity persecond
 - perf - profiling the htop to see which code is taking time

Tools:
- Prometheus
- Data dag
- Grafana

