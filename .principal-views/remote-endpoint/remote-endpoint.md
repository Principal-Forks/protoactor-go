# Remote Endpoint Connection

Lazy endpoint establishment for remote actors with gRPC-based bidirectional message streaming.

## Overview

When sending a message to a remote actor, Proto.Actor establishes a connection to the remote node lazily on first use. The connection is managed by an endpoint supervisor that spawns writer and watcher actors for each remote address.

## Architecture

### Endpoint Manager
Central coordinator that:
- Tracks all remote connections in a sync.Map
- Subscribes to endpoint events (connected, terminated)
- Creates lazy endpoints on demand
- Routes messages to appropriate endpoint writers

### Endpoint Lazy
Lazy connection wrapper using `sync.Once`:
- Connection established on first `Get()` call
- Requests endpoint from supervisor
- Caches endpoint for reuse

### Endpoint Supervisor
Actor that manages endpoint lifecycle:
- Receives address string as message
- Spawns EndpointWriter for outbound messages
- Spawns EndpointWatcher for remote actor monitoring
- Returns endpoint struct with writer/watcher PIDs

### Endpoint Writer
Actor that handles outbound message batching:
- Establishes gRPC connection to remote address
- Sends ConnectRequest to establish stream
- Batches messages for efficient transmission
- Handles connection failures and retry

### Endpoint Watcher
Actor that monitors remote actors:
- Handles Watch/Unwatch requests
- Tracks remote actor termination
- Sends Terminated messages to watchers

## Connection Flow

1. **Message Sent** - `Send()` to remote PID
2. **Ensure Connected** - Check endpoint cache
3. **Lazy Connect** - If not cached, trigger connection
4. **Supervisor Request** - Ask supervisor to create endpoint
5. **Spawn Writer/Watcher** - Create endpoint actors
6. **gRPC Connect** - Writer establishes gRPC stream
7. **Ready** - Endpoint cached and ready for messages

## Error Handling

- **Connection Failed** - Retry with backoff, publish EndpointTerminatedEvent
- **Stream Error** - Terminate endpoint, reconnect on next message
- **Remote Unreachable** - Send to dead letter

## Source Files

- `remote/endpoint_manager.go` - Endpoint management and routing
- `remote/endpoint_writer.go` - gRPC connection and message batching
- `remote/endpoint_watcher.go` - Remote actor monitoring
- `remote/activator_actor.go` - Remote actor activation

## Related

- Actor Spawn Workflow - Local actor creation
- Cluster Grain Request - Virtual actor resolution
