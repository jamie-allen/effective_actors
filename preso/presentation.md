!SLIDE title-page
## Effective Actors
.notes 

Jamie Allen

jamie.allen@typesafe.com

@jamie_allen

github.com/jamie-allen

<img src="typesafe-logo-081111.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Groundwork - What Are Actors?

* Concurrent processes that communicate by exchanging messages

<img src="actors.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Akka Actor Code

	case class Start(thePonger: ActorRef)
	object PingPong extends App {
	  val system = ActorSystem()
	  val ponger = system.actorOf(Props(new Actor { 
	  	def receive = { case x => println("Ponger: " + x); sender ! "Pong!" } 
	  }))
	  val pinger = system.actorOf(Props(new Actor {
	    def receive = {
	      case Start(x) => x ! "Ping!"
	      case x => println("Pinger: " + x); sender ! "Ping!"
	    }
	  }))
	  pinger ! Start(ponger)
	}

!SLIDE transition=blindY
# Groundwork - Supervisor Hierarchy



<img src="supervisor_hierarchy.png" class="illustration" note="final slash needed"/>

!SLIDE transition=blindY
# Akka Supervision Code


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
# Groundwork - What Are Actors?

!SLIDE transition=blindY
# Groundwork - What Are Actors?
