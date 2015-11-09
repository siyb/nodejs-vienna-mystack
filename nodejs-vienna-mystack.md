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

## Microservices - Motivation 1

* I am only going to talk about rudimentary microservice challenges, not all of them
* Why microservices?

## Microservices - Motivation 2 - Scalability 1

![Scalability (source: http://microservices.io/articles/scalecube.html)](../DecomposingApplications.jpg)

## Microservices - Motivation 2 - Scalability 2

* X
	* Adding more nodes (i.e. starting multiple instances of you monolithic software) and using a load balancer to distribute requests
	* Not specialized: we are duplicating everything, while we might only need a certain part of our software

* Y
	* Splitting up the software according to different aspects

// TODO:MORE

## Microservice - Motivation 3 - Flexibility

* In theory, each microservice could ...
	* ... be programmed using a different programming language
	* ... use different databases
* Microservices 

## Microservices - Challenges 1

* Long running tasks
	* HTTP - Timeouts -> solutions: polling? long polling? Alternatives?
	* burning question: sync vs. async, or hybrid?
	* async:
		* choreography vs. orchestration (SPOF)?
		* multiple consumer problem?
* Service Discovery
	* Centralized, Dedicated (clustered) -> e.g. consul.io? or Decentralized?

## Microservices - Challenges 2

* Configuration Management
	* Centralized, Decentralized
* Error Handling
	* Local / Global / Both?
	* Rollbacks?
	* Journal?
* Internal vs. External API? Facades?
