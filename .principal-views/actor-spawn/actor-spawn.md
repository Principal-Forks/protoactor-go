# Actor Spawn Workflow

The Actor Spawn workflow documents the complete lifecycle of spawning actors in Proto.Actor Go.

## Overview

Actor spawning is a core operation that creates new actor instances, registers them in the process registry, and starts their mailbox to begin processing messages.

## Entry Points

### Root Spawning (`root-spawn`)
- `RootContext.Spawn(props)` - Auto-generated ID
- `RootContext.SpawnPrefix(props, prefix)` - Prefixed auto-generated ID
- `RootContext.SpawnNamed(props, name)` - Explicit name

Used when spawning actors without a parent context (top-level actors).

### Child Spawning (`child-spawn`)
- `actorContext.Spawn(props)` - Auto-generated ID
- `actorContext.SpawnPrefix(props, prefix)` - Prefixed auto-generated ID
- `actorContext.SpawnNamed(props, name)` - Explicit name

Used when spawning child actors from within an actor's message handler.

## Spawn Flow

### Common Flow (Root & Child)
1. **Validate** - Check for guardian strategy conflicts
2. **Apply Middleware** - Run spawn middleware chain if configured
3. **Create Context** - Create new `actorContext` for the actor
4. **Produce Mailbox** - Create bounded/unbounded mailbox
5. **Register** - Add to ProcessRegistry (can fail with `ErrNameExists`)
6. **Initialize** - Run `onInit` callbacks
7. **Start Mailbox** - Begin message processing with `Started` message

### Child-Specific Step
8. **Link to Parent** - Call `addChild(pid)` to track child in parent's context (child spawn only)

## Error Cases

- `ErrNameExists` - Actor ID already registered in ProcessRegistry

## Source Files

- `actor/props.go` - Default spawner implementation
- `actor/root_context.go` - Root-level spawn functions
- `actor/actor_context.go` - Child spawn functions, `incarnateActor()`

## Related

- Actor Restart Workflow - Fault recovery
- Actor Stop Workflow - Graceful shutdown
