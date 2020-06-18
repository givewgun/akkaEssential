# Part 4 Fault Tolerance

## Start Stopping Watching Actor

- method 1 *stop signal*
    - stopping other `context.stop(Other Actor ref)`
        - just send a stop signal -> not stop immediately
    - stop self `context.stop(self)`
        - will stop child actor too (any actor that is `actorOf` this)
        - child will stop first
- method 2 *poison pill*
    - `actor ! PoisonPill` == Ded -> dead letter
    - cannot be catch like msg
- method 3 *kill*
    - `actor ! Kill` == Ded -> *THROW ActorKilledException*
    
- Death watch
    - `context.watch(actor)` 
    - watcher receive `Terminated(dead ref)` msg after watched actor is dead

## Life cycle
- start = create a new ActorRef with UUID at a given path
- Suspend = actor ref will enqueue but not process more message
- Resume = actor ref will continue processing more msg
- restart = swap current actor with new actor of the same type
    - if actor throw exception => auto restart
        - without putting the exception msg in the mail box (untouched)
- *too much info find on net or course*

## Supervision
- parent must decide what to do when child failed
- override the strategy => with proper relation (1-1) => match exception cases
    - tell child to Restart, Stop, Resume, Escalate (to supervisor's parent)
- `override val superVisorStrategy = ` STRATEGY `{ exception cases }`
    - OneForOneStrategy() -> apply strategy on one actor that fail
    - AllForOne() -> apply to all child actor -> every one do the same
- example syntax
```
override val supevisorStrategy = OneForOneStrategy(maxNrOfRetries = 10, withinTimeRange = 1 minute) {
    case e: IllegalArgumentException => Restart
}
```
