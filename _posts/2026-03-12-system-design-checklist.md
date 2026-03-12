---
title: "System Design Checklist"
date: 2026-03-12
layout: post
categories:
- design
tags:
- design
- systems
- interviews
- template
- questions
---

# Table of Contents

1.  [System Design Checklist](#org64c8782)
    1.  [Use Cases and Problem Framing](#org946964f)
        1.  [Why / Who](#orgde7f2dd)
        2.  [Users and Traffic Volume](#org66711e3)
        3.  [Success Metrics / KPIs](#orga7f9f4d)
        4.  [Constraints and Trade-offs](#org03d3d85)
        5.  [Data Consistency](#org8a71ef6)
        6.  [Security and Compliance](#orgbd75584)
        7.  [Back-of-the-Envelope Estimation](#org4318abb)
    2.  [High-Level Design](#org04d514e)
        1.  [Core System Components](#org69d7d5b)
        2.  [Architecture Diagram](#org3e386ec)
        3.  [Integration Points](#orga33274f)
        4.  [APIs and Service Communication](#org85c46d0)
    3.  [Detailed Design](#orgeba3472)
        1.  [Data Storage](#orgaaa3de3)
        2.  [Algorithms and Mental Models](#org54ab725)
        3.  [Caching Static data](#org4bd752c)
        4.  [Fault Tolerance](#org37084dc)
    4.  [Infrastructure & Operations](#org172bb78)
        1.  [Deployment Options](#orgaca8118)
        2.  [Cost Optimization](#org4519d4d)
        3.  [Backpressure](#org4806961)
        4.  [Disaster Recovery](#org018c82d)
        5.  [Deployment Strategy](#org49cdd38)
        6.  [Testing](#orgeac331c)
        7.  [Data Lifecycle](#org160b59e)
    5.  [Platform Considerations](#org77ce688)
        1.  [Developer Experience and Onboarding](#org4a02b09)
    6.  [Observability (M,A,L,T,E)](#orgce11650)
    7.  [Scalability](#org8acd9b2)
    8.  [Desired System Characteristics / Non functionals](#orga82122d)
    9.  [Trade-offs](#org05b0126)
    10. [Security and Compliance](#orgdb361fd)


<a id="org64c8782"></a>

# System Design Checklist

Engineers spend a lot of time searching for answers. However, in system design,
the quality of the **questions** often determines the quality of the solution.

When building a system, the goal is not just to **build the system right**, but to
**build the right system**. This distinction becomes critical in large-scale
distributed systems. Before diving into architecture diagrams, technologies, or
algorithms, it is important to clarify the problem space and constraints.

Over time, I collected and curated the following checklist from multiple sources. It has
helped me structure my thinking when designing production systems and when mentoring
junior architects to avoid common design blind spots. The checklist is not meant to be
rigid. Instead, think of it as a structured guide to ensure the important questions are not overlooked.


<a id="org946964f"></a>

## Use Cases and Problem Framing

Before discussing architecture, we must understand the problem.


<a id="orgde7f2dd"></a>

### Why / Who

-   Why does this system need to exist?
-   What problem does it solve?
-   Who are the primary users of the system?

Examples of users could include:

-   End users
-   Internal services
-   External partners
-   Developers using an API platform

Understanding the **why** and **who** shapes almost every design decision that
follows.


<a id="org66711e3"></a>

### Users and Traffic Volume

Once we understand the users, we need to estimate how the system will be used.

1.  User Distribution

    -   Are users globally distributed?
    -   Are there geographic usage patterns?
    -   Does traffic vary across time zones?

2.  MAU / DAU

    -   Monthly Active Users (MAU)
    -   Daily Active Users (DAU)

    These numbers help estimate system scale.

3.  Peak Traffic

    -   What are the peak hours for the system?
    -   Are there seasonal spikes (e.g., sales events)?

4.  Read / Write Patterns

    -   What is the expected read/write ratio?
    -   Are there burst workloads or sudden traffic spikes?

    Understanding traffic patterns is critical for designing storage, caching, and
    scaling strategies.


<a id="orga7f9f4d"></a>

### Success Metrics / KPIs

Define what success looks like.

Examples include:

-   Latency targets
-   Throughput
-   Error rates
-   User engagement
-   Business metrics (conversion, retention, etc.)

Without clear metrics, it is difficult to determine whether the system is
actually delivering value.


<a id="org03d3d85"></a>

### Constraints and Trade-offs

Every system operates under constraints.

These could include:

-   Budget constraints
-   Latency requirements
-   Regulatory requirements
-   Infrastructure limitations
-   Customer SLA / SLO requirements

Understanding acceptable trade-offs is key. For example:

-   Consistency vs Availability
-   Cost vs Performance
-   Latency vs Throughput


<a id="org8a71ef6"></a>

### Data Consistency

Clarify the consistency requirements.

Questions to consider:

-   Do we need strong consistency?
-   Is eventual consistency acceptable?
-   Which parts of the system require stricter guarantees?

Not all data needs strong consistency, and relaxing constraints can simplify the
system significantly.


<a id="orgbd75584"></a>

### Security and Compliance

Identify security and regulatory requirements.

Examples:

-   Authentication and authorization
-   Encryption requirements
-   Data residency regulations
-   Compliance standards (GDPR, SOC2, HIPAA, etc.)

Security considerations should be integrated into the design from the beginning.


<a id="org4318abb"></a>

### Back-of-the-Envelope Estimation

Simple estimates help avoid under-designing or over-engineering the system.

Examples:

-   Requests per second
-   Storage requirements
-   Network bandwidth
-   Cache size

Rough calculations provide intuition about system scale.


<a id="org04d514e"></a>

## High-Level Design

Once the problem is well understood, we can outline the architecture.


<a id="org69d7d5b"></a>

### Core System Components

Identify the major components of the system. What existing components can we leverage

Examples:

-   API gateway
-   Application services
-   Databases
-   Message queues
-   Cache layers
-   Background workers


<a id="org3e386ec"></a>

### Architecture Diagram

Create a high-level diagram showing:

-   System components
-   Data flow
-   Interactions between services

The goal is to communicate the overall structure of the system clearly.


<a id="orga33274f"></a>

### Integration Points

Define how different parts of the system interact.

This includes:

-   Client interfaces
-   Internal service communication
-   Third-party integrations

Understanding the system's **surface area** helps identify potential bottlenecks
and failure points.


<a id="org85c46d0"></a>

### APIs and Service Communication

Define service interfaces and communication patterns.

Examples:

-   REST APIs
-   gRPC
-   Event-driven messaging
-   Webhooks

Consider:

-   API versioning
-   Idempotency
-   Rate limits
-   Timeout and retry policies


<a id="orgeba3472"></a>

## Detailed Design

After the high-level architecture is established, we can drill into the details.


<a id="orgaaa3de3"></a>

### Data Storage

Choose appropriate storage technologies.

Examples:

-   SQL databases
-   NoSQL stores
-   Object storage

Key considerations:

-   Data models and schemas
-   Indexing strategies
-   Query optimization

Also consider:

-   Replication
-   Sharding / partitioning


<a id="org54ab725"></a>

### Algorithms and Mental Models

Some systems require specific algorithms or data structures.

Examples:

-   Ranking algorithms
-   Deduplication strategies
-   Scheduling algorithms

Always consider alternative approaches and their trade-offs.


<a id="org4bd752c"></a>

### Caching Static data

Caching is essential for performance at scale.

Common caching layers include:

-   Distributed cache (Redis, Memcached)
-   Expiration Policy (TTLs)
-   Eviction Policy
-   Local in-memory cache
-   CDN caching

Key questions:

-   What data should be cached?
-   What is the cache invalidation strategy?


<a id="org37084dc"></a>

### Fault Tolerance

Distributed systems fail in unpredictable ways.

Consider:

-   Service failures
-   Network partitions
-   Partial outages

Mitigation strategies include:

-   Retries
-   Circuit breakers
-   Graceful degradation
-   Redundancy


<a id="org172bb78"></a>

## Infrastructure & Operations


<a id="orgaca8118"></a>

### Deployment Options

Where will the system run? Will it be a separate microservice or part of an existing service

Options include:

-   Cloud infrastructure
-   On-premise deployments
-   Hybrid environments

Consider scalability, operational complexity, and vendor lock-in.


<a id="org4519d4d"></a>

### Cost Optimization

Large systems can incur significant operational costs.

Opportunities for optimization include:

-   Autoscaling
-   Storage tiering
-   Efficient resource utilization
-   Automated lifecycle management


<a id="org4806961"></a>

### Backpressure

-   What happens when the system is overloaded?
-   Are requests queued, throttled, or dropped?


<a id="org018c82d"></a>

### Disaster Recovery

-   What happens during regional outages?
-   What are the RTO / RPO requirements?


<a id="org49cdd38"></a>

### Deployment Strategy

-   Blue / Green deployments
-   Canary releases
-   Rollback strategy


<a id="orgeac331c"></a>

### Testing

-   Load testing
-   Chaos testing
-   Failure injection


<a id="org160b59e"></a>

### Data Lifecycle

-   Data retention policies
-   Archival and cold storage
-   GDPR deletion requirements


<a id="org77ce688"></a>

## Platform Considerations


<a id="org4a02b09"></a>

### Developer Experience and Onboarding

If the system is a platform or API, developer experience becomes important.

Providing:

-   SDKs
-   Client libraries
-   Good documentation

can significantly improve adoption and reduce integration friction.


<a id="orgce11650"></a>

## Observability (M,A,L,T,E)

A production system must be observable.

MALTE is a useful mnemonic:

-   Metrics
-   Alerts
-   Logs
-   Traces
-   Events

These signals help diagnose issues and understand system behavior.

Additional practices include:

-   Dashboards for system health
-   Debugging tools
-   Distributed tracing


<a id="org8acd9b2"></a>

## Scalability

Design the system to handle growth.

Important considerations:

-   Horizontal scaling
-   Load balancing
-   Partitioning and sharding
-   Capacity planning

Also think about how the system behaves during sudden traffic spikes.


<a id="orga82122d"></a>

## Desired System Characteristics / Non functionals

A well-designed system typically aims for the following qualities:

-   Scalability
-   Availability
-   Reliability
-   Maintainability

Balancing these characteristics while meeting business goals is the essence of
system design.


<a id="org05b0126"></a>

## Trade-offs

No system is perfect.

Every design involves trade-offs such as:

-   Cost vs Performance
-   Consistency vs Availability
-   Latency vs Throughput
-   Simplicity vs Flexibility

Explicitly documenting these trade-offs makes the design easier to reason about.


<a id="orgdb361fd"></a>

## Security and Compliance

Security must be integrated throughout the system.

Key elements include:

-   Authentication (AuthN)
-   Authorization (AuthZ)
-   Rate limiting
-   Input validation
-   Encryption

Security should never be an afterthought.
