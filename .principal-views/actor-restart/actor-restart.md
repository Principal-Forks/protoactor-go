# Actor Restart & Supervision

Fault recovery workflow handling actor failures through supervision strategies, child termination, and actor restart with stashed message recovery.

## Overview

When an actor fails (panics or returns an error), Proto.Actor's supervision system takes over. The failing actor's supervisor decides whether to restart, stop, resume, or escalate the failure based on its configured strategy.

## Supervision Strategies

### One-For-One
Applies the directive only to the failing child. Other siblings continue unaffected.
- Default strategy with configurable retry limits
- Tracks restart statistics within time window

### All-For-One
Applies the directive to all children when one fails. Used when siblings have interdependencies.

### Restarting
Always restarts failed actors. Simple strategy for stateless actors.

### Exponential Backoff
Restarts with increasing delays between attempts. Prevents thundering herd on transient failures.

## Directives

| Directive | Action |
|-----------|--------|
| Resume | Continue processing, ignore the failure |
| Restart | Stop and restart the actor |
| Stop | Permanently stop the actor |
| Escalate | Pass failure to parent supervisor |

## Restart Flow

1. **Failure Detection** - Actor panics or calls `EscalateFailure()`
2. **Mailbox Suspended** - Prevent new message processing
3. **Escalate to Supervisor** - Send `Failure` message to parent
4. **Strategy Decision** - Supervisor applies strategy to get directive
5. **Handle Directive**:
   - Resume: Resume mailbox
   - Restart: Begin restart sequence
   - Stop: Stop actor permanently
   - Escalate: Pass to grandparent
6. **Restart Sequence** (if restarting):
   - Set state to `restarting`
   - Send `Restarting` message to actor
   - Stop all children recursively
   - Wait for children to terminate
   - Incarnate new actor instance
   - Resume mailbox
   - Send `Started` message
   - Replay stashed messages

## Source Files

- `actor/actor_context.go` - handleRestart(), handleFailure(), restart()
- `actor/supervision.go` - SupervisorStrategy interface
- `actor/strategy_one_for_one.go` - One-for-one strategy implementation
- `actor/strategy_all_for_one.go` - All-for-one strategy implementation
- `actor/strategy_exponential_backoff.go` - Exponential backoff strategy

## Related

- Actor Spawn Workflow - Initial actor creation
- Actor Stop Workflow - Graceful shutdown (vs restart)
