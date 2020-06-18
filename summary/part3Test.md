# Part 3 testing

## Basic testing
- class ... extends `TestKit(ActorSystem("BasicSpec))`
    - with `ImplicitSender` =>> (testActor as a sender for msg)
    - with `WordSpecLike`
    - with `BeforeAndAfterAll` (hook)
- use companion object to store information of tests
- `expectMsg(message)
    - test kit wait for message for some amount of time (timeout)
    - `akka.test.single-expect-default` (timeout for expectMsg...)
- `expectNoMessage(1 seconds)`
- `expectMsgAnyOg("hi", "hello")` (hi or hello string response)
- seding multiples msg test
    - expectMsgAllOf("...", "..." ,)    
- receiveN(2) (wiat for the return msg til N first before do next)
    - Seq[Any]
- `expectMsgPf() {}` //partial function 

## Test probe
- used to probe actor inside actor ....
    - TestProbe("name") -> special kind of actor with assert capability
    - basically like a stub
- probe can (mock)
    - assert (expectMsg .....)
    - reply(....) 
    - receiveWhile(PF) // can set duration etc
        - PF is usually a matching etc.

## Time assertion
- when actor takes long time to compute...
- time box it
- `within(500 millis, 1 second`) [.5 - 1]
- receiveWhile[T](max, idle, message) {PF to process}
    - idle is like gap between msg
    - can get Seq (by extraction => ...)
- timeout for assert => not conform to within
    - `akka.test.single-expect-default` is configured like in .conf

## Logging Interception
- if some actor don't send out msg -> hard to know whats going on
- look at the log message intead
- `Eventfilter`.`info(pattern = String that match the log, occurences = ...)` `intercept` {}
    - `intercpet` default wait for 3 secs => can config in .conf
        - akka.test.filter-leeway = ...
    - intercept the logging at info level that has this logging msg and occur only ... times 
- `Eventfilter[RuntimeException](occurences = ...)` used to intercept `RuntimeException`
    - can intercept any throwable type
- need to configure to use in .conf file
```$xslt
interceptingLogMessages {
  akka {
    loggers = ["akka.testkit.TestEventListener"]
    test {
      filter-leeway = 5s
    }
  }
}
```

## Synchronous Testing
- no need to extend TestKit
- `TestActorRef[Actor](Props[Actor])` with Actorsystem implicit
- sending msg to TestActorRef -> insame thread -> sync (work only in the calling thread)
    - can assert assert(Actor.underlyingactor.VAL == ???)
- actorOf(..)`.wtihDispatcher(CallingThreadDispatcher.ID)` will run in the same calling thread

