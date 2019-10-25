# Observability

## USE/RED

USE = Utilization, Saturation, and Errors [Resource Scoped]

- utilization: the average time that the resource was busy servicing work (%)
- saturation: the degree to which the resource has extra work which it can't
  service, often queued
- errors: the count of error events

RED = Rate, Errors, Duration [Request scoped]

- R = Request Throughput, in requests per second
- E = Request Error Rate, as either a throughput metric or a fraction of
  overall throughput
- D = Latency, Residence Time, or Response Time; all three are widely used

- [Monitoring and Observability with USE and
  RED](https://www.vividcortex.com/blog/monitoring-and-observability-with-use-and-red)
- [Monitoring
  Microservices](https://www.slideshare.net/weaveworks/monitoring-microservices)
