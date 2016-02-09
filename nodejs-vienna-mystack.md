% My Stack AKA. node.js Microservices In Production
% Patrick Sturm
% 09.11.2015

# Introduction

## Contact

* E-Mail: p.sturm@spherical-elephant.com

## Motivation

* Christoph asked me to hold a presentation on my stack ... (he is very persuasive)
* ... the problem is that my stack is most likely not all that different from yours :D
* Therefore I decided to go a bit off topic and explain how different components of "my" (actually "our") stack are interacting

## Disclaimer

* Please note that everything that I say here tonight is specific to our model and might not apply to your project
* I am not here to tell you what is right and what is wrong or how you are supposed to design your software, I am merely describing how we implemented things :)
* If you did it differently, that's awesome as well
* There is no silver bullet
* I am going to cover our backend stack only

## Agenda

* Talk about the most vital software / frameworks used
	* Databases (very quick overview, pretty basic stuff)
	* node.js - Frameworks / Libraries
	* I am not going to speak about the Ops (DevOps) perspective though, I am trying to get someone else to do that, if you are interested ;)
* Challenges of microservice design (actually: challenges we needed to tackle)
* (Our) Solution

# Part 1: Stack

## Stack - Databases - Redis

* Key / Value store
* We use it as a (job/task) queue ...
	* More on that later
* ... as well as a token storage

## Stack - Databases - mongoDB

* You probably all know mongoDB anyways
* Document oriented database:
	* We only use mongoDB if we do _not_ require relations
		* This is an internal contract we agreed on, it does not mean that we consider using relationships in mongoDB to be wrong!
		* Yes, we know that they are supported
		* Yes, we know that you can use the ```$isolated``` operator and "sarcify" performance by employing a lock
	* We embed documents as needed
	* We declare a (loose) schema (more on that later)

## Stack - Databases - PostgreSQL

* Relational Database
* We choose PostgreSQL over all other relational databases because their logo is kinda similar to ours 
	* JK, PostgreSQL is stable, battle prooven and fast
* We use postgreSQL whenever we are confronted with relational data

## Stack - Broker - Mosquitto

* Mosquitto is a MQTT (Message Queue Telemetry Transport) message broker
* We use MQTT as means of transporting information in between our services
	* Alternative option: AMQP (e.g. RabbitMQ - can also speak MQTT), Socket (e.g. ØMQ), etc (there are many other protocol and implementation options ...)
	* PUB/SUB
* Why we choose MQTT:
	* TCP/IP
	* Very small footprint, is therefore predestinated used for mobile devices, sensors and other embedded systems (buzzword: IoT)
	* QoS

## Stack - node.js - Overview

* I think that most of you will be familiar with most of the node.js frameworks / libraries that I mention here, but I want to provide an overall impression

## Stack - node.js - Express

* Web Application Framework
	* We use Express to build our API layer
	* Some of the middlewares used:
		* body-parser
		* compression
		* accept-language (not actually an express middleware)
		* multer
		* express-useragent

## Stack - node.js - Bluebird

* Promise framework (we started before node.js had native promise support)
* Promise.promisify / Promise.promisifyAll

## Stack - node.js - Lodash

* Utilities - a must have (imho)
	* Use lodash-deep for some "deep" features on older versions of lodash (pre 3.7)
	* Especially useful for data manipulation, filtering and mapping outside a database context

## Stack - node.js - mongoose

* mongoDB ODM (Object Data Manager)
	* define schema
	* validation

## Stack - node.js - Sequelize

* ORM (Object Relational Mapper) for node.js
	* define schema
	* validation (stuff that goes beyond SQL data types)

## Stack - node.js - MQTT / Mosca

* mqtt - mqtt client bindings
* mosca - alternative (node.js) broker, that supports multiple backends:
	* mongoDB
	* Redis
	* RabbitMQ
	* Moquitto
* Nice to play around with (and for testing) but does not handle heavy load unless a proper broker is used. Backend limitations apply!

## Stack - node.js - bull

* Job queue for node.js
* Uses redis as backend
* node.js cluster support
* could be used as a message queue as well ;)
* We use bull to queue long running tasks and share queues between services of the same kind

## Stack - node.js - MISC 1

* uuid - uuid generator
* validator - multi purpose validator
* extend - prototype inheritance made easy
* moment - JavaScript time format handling
* tmp - temp file creation
* tar-fs / tar-stream - tar support (file and stream based)
* nodemailer - for sending E-Mails
* fstream - advanced file streams
* cld - compact language detector bindings


## Stack - node.js - MISC 2

* mkdirp - mkdir -p (duh Oo)
* pg - postgreSQL client
* geoip-lite - IP lookup
* redis - redis client
* redis-sessions - sessions for redis
* mmmagic - libmagic bindings
* rimraf - rm -rf 
* request - http client
* cron - cron like functionality for node

## Outlook

* part two will offer detailed explanations on how our stack is used
* we will cover some microservice fundamentals and some of the challenges that we faced
* we will also cover some of the solutions that we implemented

# Part 2: Microservices

## Microservices - Sources

* Building Microservices 1st Edition - Sam Newman
* Wikipedia
* Martin Flowler
* microservices.io

## Microservices - Agenda

* Very small intro into microservices
* Rudimentary microservice challenges
* Why microservices?
* Challenges
* Some possible/attempted solutions to the presented challenges

## Microservices - Introduction 1

* What is microservice architecture?
	* Microservice architecture is a specialization of SOA
		* Explicit boundaries
		* Autonomy of services
		* Contracts between services
		* Compatibility policy
	* They are meant to be "small" (what constitutes small is up for debate, which metric should we use?)
		* LOC? - lines of code
		* SOC? - seperation of concerns
		* etc?

## Microservices - Introduction 2

* Microservices should be deployable independently
	* No prerequisists (e.g. service X must run to be able to start service Y)
	* Loose coupling, i.e. lax dependencies (this is especially hard to do in some cases, because somewhere, data has to come together)
	* Automation is key (according to some people on the internet!)

## Microservices - Motivation 1 - Scalability 1

![Scalability (source: http://microservices.io/articles/scalecube.html)](../DecomposingApplications.jpg)

## Microservices - Motivation 2 - Scalability 2

* X - Horizontal Duplication
	* Adding more nodes (i.e. starting multiple instances of you monolithic software) and using a load balancer to distribute requests
	* Not specialized: we are duplicating everything, while we might only need a certain part of our software

* Y - Functional Decomposition (this is the essence of microservices)
	* Splitting up the software according to different aspects
	* Services:
		* Either take a single aspect, e.g. "converting profile pictures" (micro) ...
		* or a group of aspects, e.g. "User Profile Management" (macro?)

## Microservices - Motivation 3 - Scalability 3

* Z - Data Partitioning (sharding)
	* A bit like X axis scaling, each node runs the same copy of the software
	* Data is partitioned, meaning that data is not fully shared between nodes
		* db context: rows of a table are distributed among multiple servers
		* webservice context: a webservice only relies on a subset of data

## Microservices - Motivation 4 - Flexibility 1

* In theory, each microservice could ...
	* ... be programmed using a different programming language
	* ... use a different database
	* In practise, you may want to provide a stack of technologies, otherwise, software stack maintainance might become a problem (I found that when working with npm, some maintainers violate semver spec)

## Microservices - Motivation 5 - Flexibility 2

* Microservices are very small, thus, they can be replaced easily (in theory)
	* Throw code away if no one can maintain it (e.g. if someone decided to use an uncommon programming language)
	* Trial and error deployment possible, deploy working version on failure -> deploy often
	* Try out new frameworks / libraries / programming languages with little risk
		* What about database choices? Porting data may be problematic.
* Bigger pool of possible developers
	* Only if we are not sticking to one language!

## Microservices - Challenges 1 - Overview

* This section provides some insights on what to expect when starting your first microservice project
	* Mostly questions
	* I will point out which choices we made

## Microservices - Challenges 2 - Where do we start?

* Code decomposition vs. new project vs. microservice rewrite
	* Split up an existing system gradually?
	* Rewrite an existing project using microservice architecture?
	* Start a new project using microservice architecture?

* Does it make sense to plan ahead for microservice architecture, even if I don't want to adopt the architecture right away? How would I do that?

## Microservices - Challenges 3 - Communication?

* Long running tasks
	* HTTP - Timeouts -> solutions: polling? long polling? Alternatives?
	* burning question: sync vs. async, or hybrid?
	* async (e.g. via a message queue):
		* choreography vs. orchestration (SPOF)?
		* multiple consumer problem?
* Service Discovery
	* Centralized, Dedicated (clustered) -> e.g. consul.io?
	* ... or Decentralized -> algorithm?
* Internal vs. External API? Facades? Where to put up the facades?

## Microservices - Challenges 4 - Error Handling / Configuration?

* Configuration Management
	* Centralized, Decentralized?
* Error Handling
	* Local (service bound) / Global (for all services) / Both?
	* Rollbacks, retries or quick failures?
	* Reporting
		* Journal?
	* MOST IMPORTANT ASPECT OF MICROSERVICES!

## Mircoservices - Challenges 5 - Nano Services

* Service vs. library (Nano service anti pattern)
* When to use a service rather than a library?
* Some aspects of monolithic software architecture are hard to trasfer to microservices directly
	* Mostly systems that are dependencies of large parts of your software
	* Can cause copious amounts of overhead!
	* e.g. User management, access restriction, resource management


## Mircoservices - Challenges 6 - Automate and test all the things!

* Automated testing is a must have!
	* Integration tests are essential in order detect protocol misimplementations quickly
* CI and CD are essential as well
	* automated deployment with proper error reporting would be ideal

## Microservices - Pitfalls / Lessons Learned 1

* Additional complexity: messaging protocols, network protocols, load balancing, containers (single service per "host"?)
	* ops...
* Network latency / failures become a problem -> Error Handling?
	* Also see: the fallacies of distributed computing
* Testing becomes difficult, you need to mock other services or at least services communication (requests, replies)

## Microservices - Pitfalls / Lessons Learned 2

* Testing, integration testing in particular is essential ...
* ... so is error handling and reporting
* CI and CD are essential
* Don't waste too much time on fragmenting your services in the beginning, you can always split up a service if you feel that you need to ...
* ... likewise, don't hesitate to merge functionality if it turns out that logic is too fragmented
* A mix of sync and async is often required, since there are some tasks that take longer that you want an http call to be




