# Part 2 Akka Actor!

## Actor
- traditional object:
    - store their state as data
    - call thier method
- Actors
    - store state as data
    - send msg to them async

## Actor Intro
- actor is uniquely identify in actor system
- msg are async
- each actor may replied differently
- can't access other actor
```$xslt
class Test extends Actor {
    val a = 0
    def receive: PartialFunction[Any,Unit] = { // or : Receive (type)
        case message: String => a + message.length
        case msg => println("HUH")
    }
}
```
- instantiate created actor
    - need actorSystem -> pass our acts as a prop `actorOf(...)`
    - actor name cannot be the same
    - if have param -> pass as a `Props(new Actor2("ssss))`
    - or use companion object and def a method to create a new props for us
```$xslt
val actorSystem = ActorSystem("firstActorSystem")
val wordCounter = actorSystem.actorOf(Props[WordCountActor], "wordCounter")
 val anotherWordCounter = actorSystem.actorOf(Props[WordCountActor], "anotherWordCounter")
```
```$xslt
object Person {
    def props(name: String) = Props(new Person(name))
  }
...
val person = actorSystem.actorOf(Person.props("Bob"))
```
- communicate by sending msg
    - asynchronus! (NO ORDERRRRR)
```$xslt
wordCounter ! "I am learning Akka and it's pretty damn cool!" // "tell"
anotherWordCounter ! "A different message"
```

## Actor Capabilities
- message can be any type!
    - message must be immutable and serializable
    - use pattern matching
    - can send a case class (best practice)
- actor have information bout their context and itself
    - context ~ env of that actor
        - (context.self ~ this) === self (self is implicit)
- actor can reply to msg
    - using their context
    - use other ref as msg -> receive -> ref ! "...."
    - get last sender ref by `context.sender()` => sender ref to send msg to `! ...`
- deadLetters === garbage collector if send to null
- forwarding msg
    - D -> A -> B (forwarding = sending a message with the ORIGINAL sender)
    ```
    def receive: ... case WirelessPhoneMessage(content, ref) => ref forward (content + "s")
    ...
    case class WirelessPhoneMessage(content: String, ref: ActorRef)
    alice ! WirelessPhoneMessage("Hi", bob) // noSender.
    ```
- Put messages into the companion objects (best practice)

## How actor works
- message ordering? race conditions? aync for actors?
- Akka has a thread pool that shares with actor
    - only one thread operates on an actor at any time
    - actor ~ single threads
- Actor has mailbox -> run on a threads spawn by akka
- Sending msg
    - put msg in mailbox
- Processing msg
    - a thread is schedule to run the actor
    - msg deque
    - thread invoke handler on each msg
    - at some point actor is unscheduled
- Msg
    - at most one delivery
    - for any sender-receiver pair msg order is maintained
    
## Change actor behavior at different time
- traditional ways
    - we have var of state in actor and change after receiving msg
    - matching -> change state -> reply ...
    - *BAD!!*
- stateless
    - multiple custom `Receive` method , one `receive: Receive` = `handler`
        - change handler if msg is for other handler method `context.become(another handler)`

- changing actor behavior
    - anotherHandler type Receive
    - true = replace curent handler / false stack new handler on top
```$xslt
context.become(anotherHandler, true)
```
- reverting to prev behavior
    - pops current handler on top of the stack
    - if stack is empty => call `receive`
```$xslt
context.unbecome() 
```

## Parent-Child Actor
- actor can create other actor
    - using `context.actorOf(Props[..], ..)`
```$xslt
  object Parent {
    case class CreateChild(name: String)
    case class TellChild(message: String)
  }
  class Parent extends Actor {
    import Parent._

    override def receive: Receive = {
      case CreateChild(name) =>
        println(s"${self.path} creating child")
        // create a new actor right HERE
        val childRef = context.actorOf(Props[Child], name)
        context.become(withChild(childRef))
    }

    def withChild(childRef: ActorRef): Receive = {
      case TellChild(message) => childRef forward message
    }
  }
```
- actor hierachies 
    - tree like : parent -> child, child2
- Guardian Actor (manage parent)
    - /system = system guardian
    - /user = user-level guardian
    - / = root guardian (god)
- actor selection
    - `val childSelection = actorSystem.actorSelection("PATH")`
        - if path is wrong -> dead letter
- *DANGER*
    - NEVER PASS MUTABLE ACTOR STATE, OR THE `THIS` REFERENCE, TO CHILD ACTORS.
    
## Logging
- 4 level
    - Debug Info Warn Error
- use Logging val
    - `val logger = Logging(context.system, this)` in the actor
- use ActorLogging trait
    - extends/ with .... =>  log.info...
    
## Configuration
- config file is like name-value series
- akka config start with `akka {...}`
- `ConfigFactory` to create config object 
    - parseString
- `ConfigFactory.load(config object)` to load config
- default actor config in src/main/resources/application.conf
- seperate information in same config file
    - `ConfigFactory.load().getConfig("another name")`
- seperate config in another file (in resources)
    - `ConfigFactory.load("relative path of file to the resources folder")`
- other file 
    - `ConfigFactory.load("json/jsonConfig.json")`
    - `val propsConfig = ConfigFactory.load("props/propsConfiguration.properties")`
    

