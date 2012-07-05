!SLIDE title-page
## Effective Actors - JaxConf 2012

Jamie Allen

jamie.allen@typesafe.com

@jamie_allen

github.com/jamie-allen

<img src="typesafe-logo-081111.png" class="illustration" note="final slash needed"/>  <img src="jaxconf.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Groundwork - What Are Actors?

* Concurrent processes that communicate by exchanging messages
* Isolation of state, no internal concurrency

<img src="actors.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Akka Actor Code

	case class Start(thePonger: ActorRef)
	class Pinger extends Actor {
	    def receive = {
	      case Start(x) => x ! "Ping!"
	      case x => println("Pinger: " + x); sender ! "Ping!"
	    }
	}
	class Ponger extends Actor { 
	  def receive = { case x => println("Ponger: " + x); sender ! "Pong!" } 
	}

	object PingPong extends App {
	  val system = ActorSystem()
	  val ponger = system.actorOf(Props[Ponger])
	  val pinger = system.actorOf(Props[Pinger])
	  pinger ! Start(ponger)
	}

!SLIDE transition=blindY
# Groundwork - Supervisor Hierarchy
.notes Don't try to predict every possible thing that can go wrong.  Instead, understand how to reset your actor system and retry until it works.

* Specifies handling mechanisms for groupings of actors in parent/child relationship 

<img src="supervisor_hierarchy.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Akka Supervision Code

	class MySupervisor extends Actor {
      override val supervisorStrategy = OneForOneStrategy(
      	  maxNrOfRetries = 10, 
      	  withinTimeRange = 1 minute) {

        case ae: ArithmeticException => Resume
        case np: NullPointerException => Restart
      }

      context.actorOf(Props[MyActor])
    }

!SLIDE transition=blindY
# Groundwork - Parallelism

* Easily scale a task by creating multiple instances of an actor and applying work using various strategies
* Order is not guaranteed, nor should it be
* Focus on idempotent, "pure" functions

<img src="routing.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Akka Routing
.notes Of course, my example shows an impure function (writing to the console is a side effect)

	object Parallelizer extends App {
	  class MyActor extends Actor { 
	  	def receive = { case x => println(x) }
	  }

	  val system = ActorSystem()
	  val router: ActorRef = system.actorOf(Props[MyActor].
	  	withRouter(RoundRobinRouter(nrOfInstances = 5)))

	  for (i <- 1 to 10) router ! i
	}

!SLIDE transition=blindY
# Effective Actors

* Best practices based on several years of actor development
* Helpful hints for reasoning about actors in production

!SLIDE transition=blindY
# RULE: Single Responsibility Principle

* Do not conflate responsibilities in actors
* Becomes hard to define the boundaries of responsibility
* Leads to more difficult supervision as you handle more possibilities
* Debugging becomes very difficult

!SLIDE transition=blindY
# Supervision

* Supervisors should do nothing except manage their actors
* Actors handling messages should only handle messages
* Create a supervisor at every level of your hierarchy

!SLIDE transition=blindY
# Conflated Supervision

!SLIDE transition=blindY
# Explicit Supervision

!SLIDE transition=blindY
# RULE: Never Block in an Actor
.notes If you are blocking on IO/locks, then tying up a thread is a massive waste of memory resources

* Passively waiting when that thread can be doing other things
* Eventually results in actor starvation as thread pool dries up
* Horrible performance
* Massive waste of memory resources

!SLIDE transition=blindY
# Futures and Timeouts

* Futures are composable tasks to be performed asynchronously
* Timeout within a reasonable period of time
* Exponential backoff
* Handle failure

!SLIDE transition=blindY
# Futures

	case class SumSequence(ints: Seq[Int]) { def perform = ints.reduce(_ + _) }

	object Worker extends App {
	  val system = ActorSystem()
      val workActor = system.actorOf(Props[WorkActor])

	  val workFuture = workActor ? SumSequence(1 to 100)

      workFuture.onComplete {
        case Left(x: Throwable) => println("Exception: %s".format(x.getMessage))
        case Right(y) => println("Got a result: " + y)
      }
    }

!SLIDE transition=blindY
# RULE: Never Send Behavior in Messages

!SLIDE transition=blindY
# RULE: Never Send Mutable Objects

!SLIDE transition=blindY
# RULE: Name Your Actors

* Better semantic logging
* Allows for lookup

!SLIDE transition=blindY
# RULE: Do Not Optimize Prematurely

* Start with a simple configuration and profile
* Use mailbox sizes to determine where to reconfigure
* Do not parallelize until you know you need to and where

!SLIDE transition=blindY
# Start Simply
.notes In other words, start without using Actors!  Declarative is expressing the logic of a computation but not the control flow - no side effects, referentially transparent; Logic and Functional Programming are subsets.

* Deterministic
* Declarative
* Immutable

!SLIDE transition=blindY
# Layer in Complexity

* Add indeterminism (actors and agents)
* Add mutability in hot spots (CAS and STM)
* Add explicit locking and threads

!SLIDE transition=blindY
# RULE: Never Have Direct References to Other Actors
.notes This is specific to JVM implementations of actors, not Erlang.  Erlang processes cannot call methods on actors, where JVM-based actors are just instances of objects.

* Actors die
* Doesn't prevent someone from calling into an actor with another thread
* Akka solves this with the ActorRef abstraction

!SLIDE transition=blindY
# RULE: Push, not Pull
.notes You'll be surprised at how few places require guaranteed delivery in your system when you take this route.

* Start with no guarantees about delivery
* Add guarantees only where you need them
* Retry until you get the answer you expect
* Switch your actor to a "nominal" state at that point

!SLIDE transition=blindY
# RULE: Externalize Business Logic
.notes You just want to prove that your business logic works as expected, but actors are tricky this way.  Receiving a message and calculating a value can result in side effects such as creating new actors, which may be entirely unrelated from what you're trying to test.  Use integration tests to prove that actors are behaving as expected.

* Use external functions to encapsulate business logic
* Easier to unit test outside of actor context

!SLIDE transition=blindY
# RULE:

!SLIDE transition=blindY
# RULE:

!SLIDE transition=blindY
# RULE:

!SLIDE transition=blindY
# RULE:

!SLIDE transition=blindY
# RULE:

!SLIDE transition=blindY
# RULE:

!SLIDE transition=blindY
# Credits

* The Typesafe Team (Jonas Bonér, Viktor Klang, Roland Kuhn, Havoc Pennington)
!SLIDE transition=blindY
# RULE:

