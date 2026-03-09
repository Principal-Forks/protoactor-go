# Proto.Actor Go Architecture

Proto.Actor is an actor model framework for Go that provides lightweight, high-performance concurrent programming with built-in support for distribution, fault tolerance, and persistence.

## Core Concepts

### Actor System
The root container that manages all actors, extensions, and system-wide configuration. Every Proto.Actor application starts by creating an ActorSystem.

### Actors & Contexts
- **Root Context** - Entry point for spawning top-level (root) actors that have no parent
- **Actor Context** - Runtime context provided to each actor for handling messages, spawning children, and accessing system services

### Process Identification
- **PID** (Process Identifier) - Unique address for any actor, whether local or remote
- **Process Registry** - Maps PIDs to their underlying process handlers

## Message Processing Pipeline

### Props
Configuration object that defines how an actor should be created:
- Producer function (actor factory)
- Mailbox type (bounded/unbounded)
- Dispatcher configuration
- Middleware chains
- Supervision strategy

### Mailbox
Message queue that buffers incoming messages. Types include:
- **Unbounded** - No limit on pending messages
- **Bounded** - Drops or blocks when full

### Dispatcher
Schedules actor execution on goroutines. Controls concurrency and throughput.

## Fault Tolerance

### Supervision Strategies
When an actor fails, its supervisor decides the recovery action:
- **One-For-One** - Only restart the failed actor
- **All-For-One** - Restart all sibling actors
- **Exponential Backoff** - Restart with increasing delays

## Extensions

### Remote
gRPC-based remote actor communication:
- Transparent location: send messages to remote actors using PIDs
- Endpoint management and connection pooling
- Protobuf serialization

### Cluster
Distributed virtual actors (grains):
- Automatic actor placement across cluster members
- Gossip-based membership and state dissemination
- Identity lookup with consistent hashing
- Support for Consul, etcd, Kubernetes

### Persistence
Event sourcing and snapshotting:
- Replay events to rebuild actor state
- Periodic snapshots for faster recovery
- Pluggable storage backends
