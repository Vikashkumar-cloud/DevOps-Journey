
## What is Monitoring?
- Continuously checks system health and issuing alerts if a problem arises.
- Monitoring = Is something wrong?
- Monitoring is usually strong at detecting known problems.
  For example: You already know that:
  - High CPU usage is a problem.
  - A full disk is a problem.
  - An increase in 5xx errors is a problem.
  - Increased latency is a problem
 
## What is Observability?
- The capability to understand the system's internal behavior and identify the root cause of unknown problems using available system data.
- **Monitoring** indicates that there is a problem.
- **Observability** helps understand why there is a problem.

**Example:** 

- You received an alert: ```🚨 API latency > 2 seconds```. This is the **monitoring**.
- Now you investigate:
  ```
  API latency
     ↓
  Service A
     ↓
  Service B
     ↓
  Database
     ↓
  Slow SQL query
  ```
  And you discover: ```Database query = 4.8 seconds```. This is the **Observability**.

## Monitoring vs Observability

| Monitoring                       | Observability                                                  |
| -------------------------------- | -------------------------------------------------------------- |
| System health track karta hai    | System behavior understand karne mein help karta hai           |
| Known problems detect karta hai  | Known + unknown problems investigate karne mein help karta hai |
| Mostly predefined metrics/alerts | Metrics + Logs + Traces + context                              |
| "What is wrong?"                 | "Why is it wrong?"                                             |
| Alerting focused                 | Investigation + troubleshooting focused                        |
| Usually dashboards + alerts      | Telemetry + correlation + exploration                          |

#### Example
- Suppose users complain: "Website is very slow."
- Monitoring dashboard:
  ```
  CPU       = 40%
  Memory    = 50%
  HTTP 5xx  = 1%
  Latency   = 5 sec
  ```
- Monitoring indicated: Latency is high.
- You investigate further using Observability tools:
  ```
  Request
   ↓
  API
   ↓
  Payment Service
   ↓
  Database
   ↓
  Slow query
  ```
- And the trace revealed: ```DB query: 4.7 sec```
- Logs: ```Query timeout/retry```
- Now you have evidence to identify the root cause.

## The 3 Pillars of Observability
In traditional Observability there are 3 major telemetry signals:
```
              Observability
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Metrics      Logs      Traces
        |          |          |
     "How much?"  "What?"   "Where?"
```

### Metrics
- Numerical data.
```
CPU = 70%
Memory = 80%
Request rate = 500 req/sec
Error rate = 2%
Latency = 300 ms
```
- Common tool: Prometheus

### Logs
- Events of application or system.
```
2026-09-10 10:30:21 ERROR
Payment request failed
order_id=12345
timeout=5s
```
- Common tools:
  - Loki
  - Elasticsearch / OpenSearch
  - Fluent Bit
  - Logstash
 
### Traces
- A trace is the journey of a single request from start to finish as it travels through your entire system.
```
  User Request
    |
    ├── API Gateway       20ms
    |
    ├── User Service      50ms
    |
    ├── Payment Service   800ms
    |
    └── Database          700ms
```
- It appears that the bottleneck could be with the payment service or on the DB side.
- Common tools:
  - OpenTelemetry
  - Jaeger
  - Tempo

## Golden Signals
- The "4 Golden Signals" are commonly used in the Google SRE methodology.
  - Latency
  - Traffic
  - Errors
  - Saturation

  ### Latency
  - How long is it taking for the request to be completed?
  - Example: ```API response latency = 200 ms```
  - Looking only at average latency in production is not enough.
  - Better:
    ```
    P50 = 100 ms
    P95 = 400 ms
    P99 = 1.2 sec
    ```
  - Interview point: Latency should ideally be measured separately for successful and failed requests because errors can sometimes return very quickly.
    ```
    Successful request = 500 ms
    Failed request     = 20 ms
    ```
  - If you look only at average latency, the picture can be misleading.
   
  ### Traffic
  - How much load or how many requests is the system receiving?
  - Example: ```Request/sec = 1,000```
  - Other Examples:
    ```
    HTTP requests/sec
    Messages/sec
    Transactions/sec
    Active users
    Network packets/sec
    ```
  - Suppose normal traffic ```1,000 req/sec```. Suddenly ```10,000 req/sec```
  - So, The system might be overloaded.

  ### Errors
  - How many requests are failing?
  - Example:
    ```
    Total requests = 100,000
    Failed requests = 2,000

    Error rate = 2%
    ```
  - Errors include:
    ```
    HTTP 5xx
    HTTP 4xx (depending on use case)
    Timeouts
    Connection failures
    Application exceptions
    ```
    > Important: Not every 4xx necessarily system failure. Ex: ```404 Not Found``` ERROR could be wrong URL. But ```500 Internal Server Error``` usually indicate application/server-side issue.

  ### Saturation
  - How close are the system resources to their limits OR How busy your resources are.
  - Example:
    ```
    CPU utilization = 95%
    Memory = 90%
    Disk = 95%
    Connection pool = 98%
    Thread pool = 95%
    ```
  - Saturation basically asks "System ki capacity kitni consume ho chuki hai?"
    Example: ```DB connection pool: Maximum = 100 and current used = 98``` ```Saturation 98%```
  - If traffic increases further, requests may delays or fail.
      
### Golden Signals - quick summary
| Signal         | Simple question                   | Example     |
| -------------- | --------------------------------- | ----------- |
| **Latency**    | Request kitni slow hai?           | P99 = 2 sec |
| **Traffic**    | Kitna load aa raha hai?           | 5k req/sec  |
| **Errors**     | Kitni requests fail ho rahi hain? | 3% 5xx      |
| **Saturation** | Capacity kitni consume ho gayi?   | CPU 95%     |

> Ques: What are the four golden signals?
> 
> The four golden signals are latency, traffic, errors, and saturation. Latency tells us how long requests take, traffic tells us the demand on the system, errors tell us how many requests are failing, and saturation tells us how close the system is to its resource limits.

## RED Method
- Purpose of RED: "Meri service users ke perspective se kaisi perform kar rahi hai?"
- The RED method is useful for monitoring services/microservices.
  ```
  R -> Rate
  E -> Erros
  D -> Duration
  ```
  **R — Rate**
  - How many requests are coming in? ```Requests/sec = 2,000```
    Example:
    ```
    GET /users = 1,000 req/sec
    POST /orders = 500 req/sec
    GET /products = 500 req/sec
    ```
  **E — Errors**
  - How many requests are fails? ```Error rate = 2.5%```
    Example:
    ```
    HTTP 500 = 2%
    Timeouts = 0.5%
    ```

  **D — Duration**
  - How long are the requests taking to complete?
    ```
    P50 = 100ms
    P95 = 400ms
    P99 = 1s
    ```

## USE Method
- This method is useful for analyzing infrastructure and resources.
- This method is commonly used in infrastructure performance analysis.
  ```
  U → Utilization
  S → Saturation
  E → Errors
  ```
  **U** — Utilization
  - How busy is the resource?
  - Example:
    ```
    CPU utilization = 85%
    Disk utilization = 70%
    ```

  **S** — Saturation
  - How much pending work does the resource have, or how close are they to their resource limit?
  - Example:
    ```
    CPU run queue high
    Disk I/O queue high
    Thread pool exhausted
    DB connection pool nearly full
    ```
  - Important: Utilization and saturation are not the same.
  - Example:
    ```
    CPU utilization = 70%
    CPU run queue = very high
    ```
  - CPU is 70% busy, but the request is waiting for the CPU. That's saturation.

  **E** — Errors
  - Resource-related errors.
  - Example:
    ```
    Disk I/O errors
    Network packet errors
    NIC errors
    Filesystem errors
    Hardware errors
    ```

## RED vs USE
| RED                      | USE                              |
| ------------------------ | -------------------------------- |
| Services ke liye         | Infrastructure/resources ke liye |
| Rate                     | Utilization                      |
| Errors                   | Saturation                       |
| Duration                 | Errors                           |
| User/request perspective | Resource perspective             |

**RED** = Request/Service side

**USE** = Underlying infrastructure side

Example:
```
              Application
                  |
             RED Method
          /      |       \
       Rate    Errors   Duration
                  |
                  ↓
            Infrastructure
                  |
             USE Method
          /      |       \
 Utilization  Saturation  Errors
```
    
### Ques: How would you monitor a production service?
- I would start with four golden signals — latency, traffic, errors, and saturation. For service-level monitoring, I would use the RED method to track request rate, errors, and duration. For infrastructure, I would use the USE method to monitor utilization, saturation, and errors. Along with metrics, I would collect logs and distributed traces so that when an alert fires, we can investigate the root cause rather than just knowing that something is wrong.

```
MONITORING
    ↓
Something is wrong?

OBSERVABILITY
    ↓
Why is it wrong?

GOLDEN SIGNALS
    ↓
Latency
Traffic
Errors
Saturation

RED
    ↓
Rate
Errors
Duration

USE
    ↓
Utilization
Saturation
Errors
```

## P50, P90, P95, P99

- A percentile basically indicates: "What percentage of requests were completed within this latency?"

Suppose you received 100 API requests.
```
Fastest                                      Slowest
|-----------------------------------------------|
1ms  2ms  5ms  10ms ... 100ms ... 500ms ... 5sec
```

**P50 — Median**
- P50 = 50% of requests were completed within this latency.
- Example: ``` P50 = 100 ms```.
- Meaning, 50% requests 100 ms ya usse kam mein complete hui.
- And roughly 50% 100 ms requests 100 ms se zyada le sakti hain.

**P90**
- ```P90 = 200 ms```
- Meaning: 90% requests 200 ms ya usse kam mein complete hui.
- Sirf 10% requests 200 ms se zyada slow thi.
  
**P95**
- ```P95 = 500 ms```
- Meaning: 95% requests 500 ms ya usse kam mein complete hui.
- Sirf 5% requests 500 ms se zyada slow thi.
 
**P99**
- ```P99 = 2 seconds```
- Meaning: 99% requests 2 seconds ya usse kam mein complete hui.
- Sirf 1% requests 2 seconds se zyada slow thi.    
         
| Percentile | Simple meaning                   |
| ---------- | -------------------------------- |
| **P50**    | 50% requests this fast or faster |
| **P90**    | 90% requests this fast or faster |
| **P95**    | 95% requests this fast or faster |
| **P99**    | 99% requests this fast or faster |

## Why P99 is important in SRE?
- Average latency: ```Average = 150ms```
- Looking at average latency you can say "The application is fast"
- But the actual picture could be:
  ```
  P50 = 100ms
  P95 = 500ms
  P99 = 5 seconds
  ```
- This means the majority of users find the application fast, but a small percentage are having a very poor experience. That's why in production monitoring P95/P99 are very useful.

**Interview one-liner:** P99 latency means 99% of requests complete within that latency, while the slowest 1% take longer.

---

**Metric** = Data representing the system's behavior in numerical form.
```
Examples:

CPU usage        = 75%
Memory usage     = 8 GB
Requests/sec     = 500
Error rate       = 2%
Latency          = 200 ms
Active users     = 1,000
```

Tools like Prometheus primarily collect, store, and query these metrics.

## 1. Counter
- A counter is a metric that normally only increases.
- Example:
  ```
  HTTP requests = 100
  HTTP requests = 200
  HTTP requests = 350
  ```
- If the application restarts or It might reset when the counter restarts: ```350 -> 0```
- Real Examples:
  ```
  Total HTTP requests
  Total errors
  Total login attempts
  Total orders
  Total bytes processed
  ```
- Prometheus example: ```http_requests_total 12500```
- Often, just by looking at the _total, you can tell that it's a counter.

**Important interview point**
- We usually calculate the rate at the counter: ```rate(http_requests_total[5m])```
- Meaning, What was the average rate of requests per second over the last 5 minutes?

## 2. Gauge
- The gauge value can either increase or decrease.
- Example:
  ```
  CPU usage:
  40% → 70% → 55% → 90% → 45%

  Memory usage:
  4 GB → 6 GB → 5 GB → 8 GB
  ```
- Real examples:
  ```
  CPU usage
  Memory usage
  Temperature
  Number of active users
  Number of running processes
  Queue size
  ```
- Prometheus: ```node_memory_available_bytes 4294967296```
- Gauge = abhi ki current value kya hai?
  
## 3. Histogram
- The histogram is important because it is widely used in latency monitoring.
- A histogram divides values ​​into buckets.
- Histogram = values ko ranges/buckets mein divide karna.
- Suppose API latency is:
  ```
  50ms
  100ms
  150ms
  300ms
  700ms
  2sec
  ```
- We can create buckets:
  ```
  ≤ 100ms
  ≤ 500ms
  ≤ 1sec
  ≤ 5sec
  ```
- A Prometheus histogram maintains data something like this:
  ```
  request_duration_seconds_bucket{le="0.1"} 500
  request_duration_seconds_bucket{le="0.5"} 900
  request_duration_seconds_bucket{le="1"}   980
  request_duration_seconds_bucket{le="5"}   1000
  ```
  Simple meaning:
  ```
  ≤ 100ms → 500 requests
  ≤ 500ms → 900 requests
  ≤ 1sec  → 980 requests
  ≤ 5sec  → 1000 requests
  ```
  **Main use of Histogram.**
  Understand Latency distribution and in Prometheus, P50, P95, and P99 values ​​can be calculated from the histogram.
  ```
  # This will calculate approximately P99 latency.
  
  histogram_quantile(
    0.99,
    rate(http_request_duration_seconds_bucket[5m])
  )
  ```

## 4. Summary
- Summary is also primarily used for distributions/quantiles especially latency.
- Example:
  ```
  P50 = 100ms
  P90 = 300ms
  P99 = 800ms
  ```
- Summary can calculate quantiles on the client/application side.

### Histogram vs Summary
| Histogram                                  | Summary                                         |
| ------------------------------------------ | ----------------------------------------------- |
| Values ko buckets mein store karta hai     | Quantiles calculate karta hai                   |
| Server-side quantile calculation possible  | Quantile usually client-side calculate hota hai |
| Prometheus mein aggregation ke liye better | Multiple instances mein aggregation difficult   |
| Bucket configuration required              | Quantile configuration required                 |
| Prometheus mein commonly preferred         | Specific use cases mein useful                  |

Histogram observes values in configurable buckets and allows aggregation across instances. Summary calculates quantiles on the client side, so its quantiles are generally harder to aggregate across multiple instances.

---

### CPU Metric = Server ka processor kitna busy hai.
- CPU indicates: How busy the server's processor is.
- Example: ```CPU usage = 75%```
- With Prometheus/node-exporter, you get raw CPU time metrics, such as: ```node_cpu_seconds_total```

### Memory Metrics = RAM kitni available/used hai.
- Memory indicates: How much RAM is available/used.
- Example:
  ```
  Total RAM = 16 GB
  Used       = 12 GB
  Available  = 4 GB
  ```
- Important metrics:
  ```
  Total memory
  Available memory
  Used memory
  Swap
  ```
- Prometheus/node-exporter examples:
  ```
  node_memory_MemTotal_bytes
  node_memory_MemAvailable_bytes
  node_memory_SwapFree_bytes
  ```

### Disk Metrics
- In Disk monitoring mainly two things are important.

  **1. Disk space**
  - Example: ```Disk = 90% full```
  - Metric:
    ```
    node_filesystem_avail_bytes
    node_filesystem_size_bytes
    ```
  - Problem: ```Disk 100% full```
  - Potential impact:
    - Application logs cannot be written
    - Database write operations may fail
    - Application may crash
    - System instability
   
  **2. Disk I/O**
  - Disk space and disk performance are different things.
  - Suppose: ```Disk space = 40%```
  - But:
    ```
    Disk I/O = very high
    I/O wait = high
    ```
  - Application can be slow. So Disk Capacity + Disk I/O both are important in monitoring.

### Network Metrics
- In network monitoring, we observe:
  ```
  Incoming traffic
  Outgoing traffic
  Packets
  Errors
  Dropped packets
  Network throughput
  ```
- Example:
  ```
  Incoming = 500 Mbps
  Outgoing = 300 Mbps
  Packet drops = increasing
  ```
- If network packet drops are increasing, application connectivity or performance could be impacted.

### Application Metrics
- Infrastructure metrics: ```CPU```, ```Memory```, ```Disk````, and ```Network```
- But for SRE, application-level metrics are also extremely important.
- Example e-commerce application:
  ```
  HTTP requests
  HTTP errors
  Request latency
  Active users
  Orders created
  Payment failures
  Database connections
  Queue length
  ```
- Example
  ```
  http_requests_total
  http_request_duration_seconds
  http_requests_errors_total
  ```
- Business metrics:
  ```
  orders_created_total
  payments_failed_total
  ```

#### Infrastructure vs Application Metrics
- Suppose:
  ```
    CPU       = 40%
    Memory    = 50%
    Disk      = 40%
    Network   = normal
  ```
- Everything looks healthy on infrastructure level but application metrics:
  ```
    HTTP 500 = 10%
    Payment failures = 8%
    Latency P99 = 5 sec
  ```
- At the application side, application is unhealthy. That's why both layers are important in SRE monitoring.
  ```
                    Monitoring
                        |
              ┌─────────┴─────────┐
              ↓                   ↓
         Infrastructure         Application
              |                   |
         CPU / Memory          Request rate
         Disk / Network        Error rate
                               Latency
                               Business metrics
  ```

  ---
Suppose you have Order API:

  **Counter** = Total kitne orders create hue?
  
  **Gauge** = Currently kitne active orders hain?
  
  **Histogram** = Requests ki latency kis range mein distributed hai?
  
  **Summary** = Request latency ke quantiles kya hain?

#### Ques. What is the difference between Counter and Gauge?
- Counter ek monotonically increasing metric hai jo kisi event ke total occurrences ko represent karta hai, jaise total HTTP requests ya errors. Ye process restart hone par reset ho sakta hai. Gauge current value represent karta hai aur increase ya decrease dono ho sakta hai, jaise CPU usage, memory usage ya active users.

#### Ques. Histogram vs Summary?
- Histogram observations ko buckets mein store karta hai aur Prometheus mein multiple instances ke data ko aggregate karke quantiles calculate kar sakte hain. Summary client side par quantiles calculate karta hai, isliye multiple instances ke across quantile aggregation difficult hoti hai. Prometheus environments mein latency monitoring ke liye histogram commonly preferred hota hai.

```
METRICS
   |
   ├── Counter
   |      └── Total events
   |          requests, errors
   |
   ├── Gauge
   |      └── Current value
   |          CPU, memory, active users
   |
   ├── Histogram
   |      └── Distribution / buckets
   |          latency
   |
   └── Summary
          └── Quantiles
              P50, P95, P99


INFRASTRUCTURE
   ├── CPU
   ├── Memory
   ├── Disk
   └── Network

APPLICATION
   ├── Request rate
   ├── Error rate
   ├── Latency
   ├── Active users
   ├── DB connections
   └── Business metrics
```
    
