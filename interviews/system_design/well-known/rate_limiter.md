# Rate Limiter System Design

### What is RateLimiter and what are the use cases
Rate Limiter limits the request to a particular resource (system/service/nodes etc.). This is to prevent overwhelming the system and ensure there is no performance degradation. 
- Prevent DDOS i.e. prevent resource exhaustion because of distributed denial of service attacks
- Reduce Cost: Rate limiting keeps the 3rd party api calls in check and also Limiting excess requests means fewer servers and allocating more resources to high priority APIs. 

## System Requirements
1. **Rate Limiting System:** Implement a rate-limiting system for directing traffic to backend nodes, preventing overloading.
2. **Priority-Based Routing:** Requests should be routed to nodes based on priority (higher-priority nodes should get traffic first).
3. **Per-Node Rate Limiting:** Each node has an independent rate limit (with different capacities).
4. **Global Rate Limiting:** There is an overall system-wide limit on the total number of requests allowed.
5. **Distributed Rate Limiting:** The system is distributed across multiple API Gateway instances and nodes.
6. **Dynamic Node Management:** Nodes can be added, removed, or reprioritized dynamically.

### Non Functional
1. **High Available:** The system must be highly available, fault-tolerant, and resilient to failures.  
2. **Low Latency**: Rate Limiter should not increase the time of actual requests significantly
3. **Fault Tolerant:** If there are any problems with the rate limiter (for example, a cache server goes offline), it does not affect the entire system.

## High Level System Design

## Monitoring

## Scalability

## Fault Tolerance
- Service going down
- DB going down
- Connection going down

Tradeoffs
- Redis vs Memcache


