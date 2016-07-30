## [fit] __design patterns__
## [fit] in the npm registry

---

![right](images/malcolm-files.jpg)

## [fit] C J Silverio, CTO
## [fit] @ceejbot

^ I have npm engineering under my care. Cat Technology Officer.

---

## [fit] human brains are
## [fit] __pattern-detection__ machines

^ There are seven layers in our brains, more or less, that match successively more abstract patterns against things we've experienced before. In comes edges of retina stimulation; out comes an image of our mother's face. Or of a tortoise on its back in the desert. Or of an 80s movie adapation of a Philip K Dick novel.

---

## [fit] the process of writing software
## [fit] is __abstraction__ & pattern __extraction__

^ So of course we're obsessed with finding patterns in the software itself.

---

## [fit] patterns in __code__
## [fit] patterns in __systems__

^ The Design Patterns book focused on patterns in the code itself. There are also patterns in the design of distributed systems.

---

## [fit] npm's package registry
## [fit] has typical __system__ patterns
## [fit] \(some good, some bad)

^ We are going to descriptively analyze some patterns I see in the npm registry.

---

![](images/chalkboard.gif)

^ What is the registry? Let's back up & define some terms.

---

## [fit] registry, noun:
## [fit] the services that manage
## [fit] __package__ metadata & payloads

^ Okay, so what's a package?

---

# [fit] __module:__ a javascript code unit
# [fit] __package:__ directory with a package.json

^ This is your point of view.

---

# [fit] my point of view:
# [fit] __package:__ tar archive + metadata

^ The thing I have to keep track of for you & serve as fast as possible to you when you ask for it.

---

# [fit] 318,466 packages
# [fit] 1.4 million tarballs
# [fit] medium data (fits on 1 disk)

----

![](images/winking.gif)

^ okay! now we know what we're discussing

---

## [fit] the systems that manage all this data
## [fit] have some emergent __patterns__

^ Let's go into them

---

## [fit] monoliths
## [fit] microservices
## [fit] transaction logs
## [fit] message queues

^ Here's what we're going to talk about

---

## [fit] monoliths:
## [fit] everything in one big process

^ The registry was entirely implemented inside couchdb. (It still sort of is! The couch app still does work today.)

---

## [fit] monoliths are __okay__
## [fit] easy to write & change
## [fit] perf more than good enough

^ They are easy to write & change. This is fantastic when you are still figuring out the problem you're solving. Their performance is more than good enough for services measuring their usage in requests/minute.

---

![](images/ollie-double-takes.gif)

^ time to scale it

---

## [fit] time to __scale__ perf & team size
## [fit] monoliths are less okay

---

![](images/malcolm-two-phones.jpg)

^ This is how you scale a monolith.

---

## [fit] it's easy to write highly-coupled code
## [fit] inside a __non-modular__ monolith

^ Often highly coupled inside: is it decomposed into modules?

---

## [fit] __modularity:__ a digression

---

## [fit] Q: where does __modularity__ come from?
## [fit] A: __information hiding__

---

# [fit] "On the Criteria To Be Used in
# [fit] Decomposing Systems into __Modules__"
# [fit] — D. L. Parnas, 1972

^ This is a pretty cool paper. The example he works through is very 1972, but you get the idea from it.

---

## [fit] __hide__ information
## [fit] __hide__ implementation

---

## [fit] __hide__ behind an interface
## [fit] so you can __change__ things

---

![fit](images/great_interest.gif)

^ One of the great secrets of programming.  If you master this, tell me all about how you did it, please.


---

# [fit] our 2.0 rewrite switched to
# [fit] many interconnected __microservices__

^ They all have fairly names like "frontdoor", auth, validate & store, etc.

---

# microservice advantages

- forces you to define interfaces
- implementations are hidden inside services
- lots of scaling dials to turn

---

# microservice disadvantages

- more complexity
- how do you handle failure & retries?
- lots of mass to move if you sliced it wrongly
- it's still possible to not be modular

---

![left](images/obviously.gif)

# [fit] reads are __boring__:
# [fit] auth ➜ nginx serving __files__

---

# [fit] mutating data is interesting:
# [fit] let's look at __publishing__

---

![fit](images/cli-publish-couch.png)

^ Block diagram! This looks modular, right? Lots of microservices, arrows. There's a secret horror in there.

---

# [fit] auth sets up package access on a publish
# [fit] as a side effect

---

![fit](images/cant-be-good.gif)

^ No, it can't. And we'll come back to this.

---

# [fit] after publication, it's a different pattern:
# [fit] the transaction __log__

^ The microservices pattern of distributed system is how everything works up to the moment a publication is commited. Afterward we switch to a different design pattern.

---

## [fit] transaction logs
## [fit] write-ahead logs (WAL)
## [fit] commit logs

^ This is a great pattern! Solid, reliable, the heart of many systems.

---

# [fit] [The Log](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying): What every software engineer should know
## [fit] about real-time data's unifying abstraction

^ The blog post to read, from a LinkedIn engineer.

---

## couchdb's super power
## the __changes__ feed

^ One of couchdb's superpowers is that it has a changes feed, which is sort of a commit log. You can use this to get data about the current state of every package in the registry, in the order of least-recently-modified to most-recently-modified

---

## [fit] registry __followers__:
## [fit] consumers of couchdb's commit logs

^ We use the log pattern to fan data out from a central, standard format into structures more suitable for specific tasks, or to act on the news that a package changed.

---

![fit](images/post-pub.png)

^ we fan out like mad after package metadata hits couchdb

---

- distribute __tarballs__
- invalidate our __CDN__'s cache
- populate __postgresdb__ to drive the website
- index data in __ElasticSearch__
- scan packages for __security leaks__
- populate our registry __mirror__
- fire __webhooks__

^ less time-critical

---

![](images/axe-tank.gif)

^ This is obviously awesome.

---

## [fit] pointless trivia!
## [fit] look at the __Reston__ package

^ the dustiest package in the registry right now is Reston, which is a REST service thing. node's themes were visible early

---

# [fit] publication is time-sensitive
# [fit] we have __tens of seconds__ afterward

^ The clock is ticking on a publication: we want it to go, as fast as possible as robustly as possible, we get only one shot

---

![](images/malcolm-head-hands.gif)

^ I am tired of microservices. We messed up the modularization. Unwinding failure is a PITA.

---

## [fit] __Estragon:__ Let's fix publication.
## [fit] __Vladimir:__ Fine. But how?

---

## [fit] __Message queues__

^ This enters the speculative portion of the talk.

---

## [fit] message queues
## [fit] inversion of __control__

^ Turn the pattern around: instead of imperative code shoving a bunch of mutable data through services, you have an immutable message -- a request for work -- sitting in a message queue.

---

## [fit] __workers__ consume messages
## [fit] & retry or unwind on failure

---

## [fit] you __scale__ by adding more workers
## [fit] \(sounds a lot like the log handlers, huh?)

---

## [fit] http://queues.io

^ just to get an idea of how popular they are: look at them all! (note also Kafka, LinkedIn's entry)

---

![](/Users/ceej/Dropbox/fun/peter-capaldi/PCapReactionGifs/pointing.gif)


^ re-imagine publishing a package: Publication is a series of steps, each of which can either suceed or fail. Failure triggers a rollback & report to the requesting client.

---

# queue advantages

- retries are easier
- scale by scaling workers
- rollback is easier
- the queue must be reliable, but workers can crash

---

# queue disadvantages

- complexity
- complete overhaul of the way things are often structured
- they're sometimes slow

^ The fast one is Kafka & it's weird & requires the JVM.

---

# [fit] we're moving toward queues
# [fit] slowly, __invisibly__

---

## [fit] monoliths
## [fit] microservices
## [fit] transaction logs
## [fit] message queues

---

# [fit] none of these patterns are __wrong__
# [fit] none of these patterns are __right__

---

# [fit] it's __tradeoffs__
# [fit] all the way down

---

# [fit] what __problem__ are you solving?
# [fit] what __tools__ do you have to hand?
# [fit] what is your __team__ experienced with?

---

![](/Users/ceej/Dropbox/fun/peter-capaldi/PCapReactionGifs/doctor_bye.gif)
