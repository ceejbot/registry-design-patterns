## [fit] __design patterns__
## [fit] in the npm registry

---

## [fit] human brains are
## [fit] __pattern-detection__ machines

^ There are seven layers in our brains, more or less, that match successively more abstract patterns against things we've experienced before. In comes edges of retina stimulation; out comes an image of our mother's face. Or of a tortoise on its back in the desert. Or of an 80s movie adapation of a Philip K Dick novel.

---

## [fit] the process of writing software
## [fit] is __abstraction__ & pattern __extraction__

^ So of course we're obsessed with finding patterns in the software itself.

---

## [fit] __A Pattern Language__
## [fit] Christopher Alexander

^ Patterns in architecture & city design. Influential on a generation of programmers.

---

## [fit] __Design Patterns__
### [fit] Elements of Reusable Object-Oriented Software
### [fit] aka the Gang of Four book

^  Published in 1994. Hugely influential. Gamma, Helm, Johnson, Vlissides. When you hear "model/view/controller", it came from this book.

---

## [fit] __Don't read it.__
### [fit] At least not right now.

When you do, remember it's *descriptive* not *prescriptive*.

^ Also it's about OO software, which is not the be-all & end-all. Also they forgot that the GOF recommended composition over inheritance, but I digress.

---

## [fit] patterns in __code__
## [fit] patterns in __systems__

^ The Design Patterns book focused on patterns in the code itself. There are also patterns in the design of distributed systems.

---

## [fit] npm's package registry
## [fit] has typical __system__ patterns
## [fit] \(some good, some bad)

---

# [fit] what is the __registry__?

^ back up a second

---

## [fit] registry, noun:
## [fit] the services that manage
## [fit] __package__ metadata & payloads

---

# [fit] Okay, so what's a __package__?

---

# [fit] __module:__ a javascript code unit
# [fit] __package:__ directory with a package.json

^ This is your point of view. package.json is the single most overloaded concept in the app

---

# [fit] my point of view:
# [fit] __package:__ tar archive + metadata

^ The thing I have to keep track of for you & serve as fast as possible to you when you ask for it.

---

## [fit] __doc.json:__ the json metadata
## [fit] __tarball:__ the tar archive

^ More jargon! The first one comes from the couchdb roots of the registry. The second is unix slang.

---

## [fit] back to those design patterns…

---

## [fit] monolith
## [fit] microservices
## [fit] transaction logs
## [fit] message queues

^ Let's talk about them in the context of the registry.

---

## [fit] monoliths:
## [fit] registry 1.0
## [fit] our website

^ The registry was entirely implemented inside couchdb. (It still sort of is! The couch app still does work today.)

---

# monolith advantages

They are easy to write & change. This is fantastic when you are still figuring out the problem you're solving.

Their performance is more than good enough for services measuring their usage in requests/minute.

---

# monolith disadvantages

Hard to scale: expensive & unwieldy.
Hard to work on for large teams.
Often highly coupled inside: is it decomposed into modules?

---

# [fit] our 2.0 rewrite switched to
# [fit] many interconnected __microservices__

^ They all have fairly names like "frontdoor", auth, validate & store, etc.

---

# microservice advantages

- Forces you to define interfaces.
- Implementations are (mostly) hidden.
- Lots of scaling dials to turn.
- Easier to distribute the work.

---

# microservice disadvantages

- more complexity
- how do you handle failure & retries?
- lots of mass to move if you sliced it wrongly
- it's still possible to not be modular

---

## [fit] __modularity:__ a digression

---

## [fit] Q: what is __modularity__ anyway?
## [fit] A: __information hiding__

---

# [fit] "On the Criteria To Be Used in
# [fit] Decomposing Systems into Modules"
# [fit] — D. L. Parnas, 1972

^ This is a pretty cool paper. The example he works through is very 1972, but you get the idea from it.

---

## [fit] __hide__ information
## [fit] __hide__ implementation

---

## [fit] hide behind an interface
## [fit] so you can change things

---

## [fit] this is one of the great secrets of programming

^ I am not even kidding.

---

![fit](images/cli-publish-couch.png)

^ Block diagram!

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

# [fit] registry followers:
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

## [fit] pointless trivia!
## [fit] look at the __Reston__ package

^ the dustiest package in the registry right now is Reston, which is a REST service thing. node's themes were visible early

---

# [fit] publication is time-senstive
# [fit] we have __tens of seconds__ afterward

^ The clock is ticking on a publication: we want it to go, as fast as possible as robustly as possible, we get only one shot

---

## [fit] All kids love __log__.

^ This is a great pattern & we'll keep using it.

---

## [fit] I am __tired__ of
## [fit] microservices.

---

## [fit] __Estragon:__ Let's fix publication.
## [fit] __Vladimir:__ Fine. But how?

---

## [fit] __Message queues__

---

## [fit] message queues
## [fit] inversion of control

^ Turn the pattern around: instead of imperative code shoving a bunch of mutable data through services, you have an immutable message -- a request for work -- sitting in a message queue.

---

## functional, sorta kinda

* immutable messages
* workers doing what the message requests
* what needs to be done next
* what needs to be unwound if it fails
* retries are easy

---

## [fit] you __scale__ by adding more workers
## [fit] it's all about the fan-out

---

# http://queues.io

just to get an idea of how popular they are: look at them all! (note also Kafka, LinkedIn's entry)

---

![right](/Users/ceej/Dropbox/fun/peter-capaldi/PCapReactionGifs/pointing.gif)

## re-imagine publishing a package

Publication is a series of steps, each of which can either suceed or fail. Failure triggers a rollback & report to the requesting client.

---

- store the payload as a message: package, publisher, current state
- worker looks at the message
- authz (reject or move to next step)
- validation
- update registry metadata
- store tarball
- success!

---

# queue advantages

- retries are easy
- can scale by scaling workers
- rollback is easier
- the message queue needs to be reliable, but workers can crash

---

# queue disadvantages

- complexity
- complete overhaul of the way things are often structured
- they're often really slow (the fast one is Kafka & it's the JVM)

---

The registry is moving toward message queues behind the scenes.

Slowly.

---

![](/Users/ceej/Dropbox/fun/peter-capaldi/PCapReactionGifs/doctor_bye.gif)