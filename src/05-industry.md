## State of the Industry

There’s been significant growth in recent years in the number of
software systems available for working with streaming data, both in the
open source and commercial markets. In this section, we review the
options currently available for working with data streams, with a focus
on implementations of probabilistic algorithms.

Realistically, however, we have found that while there are commodity
systems for working with streams of data, very few are designed for
probabilistic modeling. Most existing systems are designed to feed
realtime data into a traditional SQL database or data warehouse.

Also, the vocabulary commonly used in the market is still evolving. You
may see streaming systems referred to as "event processing systems," or
"complex event processing" or even "event correlation" systems. We are
starting to see the rich history of legacy event processing systems
collide with the modern reality of commoditized Hadoop and other
map-reduce infrastructures.

It’s an exciting time for streaming infrastructure, as these tools are
beginning to hit maturity and become stable platforms.

### Open Source

Much of the impressive development of streaming infrastructure has come
from the open source community. There are several projects that are
commonly found in production on "web-scale" products. On the business
side, several of these projects have large commercial efforts behind
them and are great options for enterprise systems.

A product is considered to be "web scale" when it must manage data from
very large numbers of Internet users or web pages. The obvious examples
are Google and Facebook, but there are many smaller companies that have
built impressive infrastructures for managing data.

We have chosen to write about only those open source projects that are
proven to work well in production for large-scale systems, and that
offer value for probabilistic algorithms. This list is by no means
exhaustive; keep in mind also that this is a rapidly evolving field.

#### Apache Storm

Storm is designed to do for realtime stream processing what Hadoop did
for batch processing — that is, create a stable and robust commodity
open source solution for a problem area that usually requires messy and
complex homebrewed architecture, or very expensive proprietary software.

Storm originated at BackType, a startup founded in 2008 with a focus on
providing analytics for businesses across social and web platforms. Work
on Storm began in 2010, and the system was open-sourced after Twitter
acquired BackType in 2011. It became an official Apache project in 2013.

Practically, Storm is considered by many engineers to be a heavyweight
solution. It must also be deployed alongside other infrastructure (Storm
only provides the stream management system), so it is not, by itself, a
complete solution.

Storm famously powers Twitter’s realtime analytics,^[<http://analytics.twitter.com>]
and YieldBot has used Storm, alongside Apache Kafka (a
distributed messaging system) and Apache Cassandra (a distributed
database), for probabilistic ranking and trend analysis.

#### NSQ and Messaging Protocols

NSQ is a realtime distributed messaging platform that is horizontally
scalable and handles high-throughput streams. NSQ is written in Go, a
programming language created by Google with strong support for
concurrency that is starting to be adopted by many distributed system
engineers.

NSQ was developed in 2012 at bitly, a social media analytics company, to
handle a messaging architecture that powers many downstream services
with varying requirements. It is now running in production at many
companies, including Stripe, BuzzFeed, Digg, and Hailo.

![NSQ System Architecture](figures/25.svg)

##### Figure 1. NSQ System Architecture

We include NSQ in this review because it is an excellent package for
handling incoming data streams that feed into probabilistic systems.

#### Streamtools

Streamtools^[<http://nytlabs.github.io/streamtools/>]
is a graphical tool for visually working with streams of data.
It was developed in 2013 by the New York Times R&D Lab with the goal of
enabling developers to easily work with streams of data.

Streamtools offers a visual vocabulary of operations that can be applied
to realtime streams of data without programming. It’s significant
because it democratizes the ability to work with realtime streams,
vastly increasing the number of people in a typical organization who can
create products or find insights on the available data.

#### RethinkDB

RethinkDB^[<http://rethinkdb.com/blog/realtime-web/>]
is an emerging project we find fascinating enough to include in
this report. It’s an open source distributed database with a built-in
administrative interface and intuitive query language.

The RethinkDB team recently added a streaming API that supports working
with, querying, and storing realtime streams of data. This feature
allows application developers to invert the typical model of
development, so that rather than constantly polling a datastore, they
can subscribe to database updates and have RethinkDB push those updates
into the application. This can be a strong advantage for applications
built on realtime data.

### Commercial Vendors

Commercial complex event processing systems have been in the market
since the 1990s, and all major database vendors offer a streaming data
processing solution. These solutions are available at high cost (typical
pricing is approximately US$40,000 annually) and are generally only a
recommended option if you already run data solutions from a particular
vendor in your environment.

The largest difference between the open source options in this space and
the commercial ones is the developer experience. All of the commercial
options have a recommended or required Integrated Development
Environment (IDE). This simplifies the process of integrating streaming
analytics into an existing infrastrcture, but forces compliance to a
specific metaphor of data flow design.

Few of the provided components use probabilistic models, though
implementation is straightforward with any of them.

In this section, we give a brief overview of some of the dominant
commercial solutions.

#### SAP Event Stream Processor (ESP)

The SAP ESP solution includes an Integrated Development Environment
(IDE) with a graphical interface to visually connect and program the
data sources and outputs. It supports visual analytics, allowing for
both realtime and batch analytics to be designed in the same dashboard
view.

![SAP ESP](figures/26.png)

##### Figure 2. A screenshot of the SAP ESP analytics design view

#### IBM InfoSphere Streams

IBM’s offering retails at $43,700.00 for a production annual license. It
also offers a graphical view of how data flows through the application
(one of their training manuals insists this is the "natural way to view
an application"), and allows programmers to use various analysis
metaphors for working with and analyzing their data.

Many applications of IBM’s offering are found in the finance industry.

#### Oracle Event Processing

Oracle’s Event Processing solution is a stand-alone stream processor
that also integrates with Oracle’s suite of solutions. It offers a
visual data flow editor in the IDE, as well as a web view.

![Oracle Event Processing](figures/27.png)

##### Figure 3. A screenshot of the Oracle Event Processing interface

------------------------------------------------------------------------