## CliqueStream: Prototype

The major proliferation of social networks into our communities offers a
fantastic opportunity to study the ways in which people interact on a
large scale. However, doing so can be quite a computational
challenge — many things on the social web are constantly in flux, and
there is always new data to consider. In addition, the calculations we
wish to perform can often be computationally intensive, to the point
where we can no longer easily incorporate all of the newest information
into our results. This is what makes analyzing social networks a perfect
application for streaming algorithms.

In our prototype, CliqueStream, we wished to create a connected graph
including subreddits and Twitter hashtags where the connections are
formed by the similarities in the rhetoric used. Furthermore, we wished
to be able to identify trends for specific keywords and the similarities
between users interacting in these various groups.

This prototype presents a realtime visualization of the relationships
between billions of words being used on two major social networks,
reddit and Twitter. It allows us to explore current language use and to
understand how people are discussing and interacting with new ideas, and
the relationships between those ideas. This prototype is both a showcase
for the probabilistic methods discussed in this report and a novel
visualization of a fascinating data set.

![The Cliquestream prototype](figures/15.png)

##### Figure 1. The Cliquestream prototype

All of these calculations can be quite taxing — analyzing the similarity
of word use between 50 subreddits can easily result in trillions of
comparisons, depending on the number of words used in each one. A
classic algorithm for comparing the similarity between two sets results
in (N) operations, where <span class="monospaced">N</span> is the number
of items the larger of the two sets (this is assuming the data was
heavily preprocessed and sorted). If each of the 50 subreddits used
1,000,000 words, then creating this graph would take more than
1,225,000,000 operations! Even if every operation happened in 0.5
microseconds, this would still amount to 10.2 minutes of computation.

On the other hand, if we used a probabilistic data structure (such as a
K-Min Values structure with 1.75% error), we could reduce the number of
operations to 2,508,800. This would represent 1.25 seconds of work, as
compared to 10.2 minutes for the classical computation. The ability to
do this calculation quickly allows us to create an informative and
interactive frontend where users can play with data and get results
immediately. As a result, instead of us pre-describing what we want our
users to see, they can form their own complicated queries and see the
results right away.

Don’t underestimate the power of responsive data analysis. If an analyst
must wait a 10 minutes for a result instead of seconds, there’s a
psychological cost to the context switch that slows down the human part
of the creative data analysis process.

Probabilistic algorithms sped up our analysis by 490x, making it
possible for results to be computed in real time.

In addition to the savings in computation, writing our algorithms
probabilistically gave us an enormous savings in memory used. As
described in <a href="#algorithms" class="doc_link">algorithms</a>,
analyzing data probabilistically allows us to store only a small
synopsis of the data as opposed to the full dataset. This is
particularly useful when dealing with a stream of data, where storing
the full dataset would constantly require more and more storage space.
Our probabilistic implementation has bounded memory use — once enough
subreddit and hashtag data comes in to fill the data structures with the
minimum amount of data necessary (see
<a href="#things-to-consider" class="doc_link">Things to Consider</a> for more discussion),
the memory use stabilizes. In the case of our prototype this
stabilization happens at 296.3 MB of memory and 147 MB of storage
(including data snapshots for reloading the in-memory data). This is
quite amazing considering we are ingesting data at a rate of 1.59 Mb/s
(with a modest 40 messages per second) and see a gigabyte of data every
hour and a half!

The ability to bound our memory use and our computational complexity
also saves us maintenance time and decreases system complexity. Our
prototype easily runs on any standard laptop, yet it delivers the kind
of results that are generally associated with a cluster solution
(Hadoop, for example) that requires extraordinary computing power and
significant manpower to maintain.

### Implementation

An important aspect when designing a good data analysis pipeline with
streams is to decouple as many of the components from each other as
possible. That is to say, the various elements should communicate
through a simple API and not be dependent on their particular
implementation. This is a strong principle of modern system architecture
and is particularly relevant for stream-based systems.

![Data analysis pipeline](figures/16.svg)

##### Figure 2. General streaming data analysis pipeline

This decoupling is beneficial because it allows developers to work
concurrently without affecting each other’s systems. In addition, if the
interface is sufficiently general (for example, using an HTTP-based
API), then individual developers can use any language in order to
implement a particular segment of the overall system.

Furthermore, scaling becomes much easier when the components are
modular. If we start by running the entire system on one server and soon
realize we need more computing power or better fault tolerance, we can
move *some* of the system to new servers that can function as backup
systems or provide additional computational capacity.

In general, the components of a data processing pipeline involve
gathering the data, moving the data to the right places, analyzing it,
and providing a method to gather the results. In classic batch
infrastructures (such as Hadoop), the results are provided in a text
file that contains the output of the analysis. For a streaming
architecture, we need a way to query the system whenever we want to see
what the current state of the calculation is. To accommodate this, we
chose to create a REST API such that any external system can issue
queries to the system and see what the current results are.
Alternatively, some analysis schemes can provide new, modified versions,
of the original data stream for other processes to take advantage of.
Generally this is used to either filter the stream (by removing
non-pertinent messages) or to augment it (by adding new fields).

For example, in
<a href="#figure-2.-general-streaming-data-analysis-pipeline" class="doc_link">Figure 2. General streaming data analysis pipeline</a> the <span
class="monospaced">cat\_extractor</span> analysis routine finds tweets
relating to cats and filters them into their own stream (named the <span
class="monospaced">cat\_stream</span>). Any other analysis routine can
now read this filtered stream and do further analysis to it (such at the
<span class="monospaced">all\_cat\_tweets</span> routine).

##### Components of a Processing Pipeline

-   Data Sources

    -   Data Source-Specific Libraries

-   Data Routing

    -   NSQ

    -   RabbitMQ

    -   Kafka

    -   Amazon Kinesis

-   Analysis Plugins

    -   Any Language

    -   Simple and Uniform API

-   Query Interface

    -   REST API

#### Collection and Analysis Pipeline

The first hurdle when working with streaming data is setting up an
infrastructure that can route messages to the correct place. This can be
made even more challenging when dealing with multiple sources of data,
some of which support streaming and some of which require polling.

For the prototype, we identified two main data sources of interest:
reddit and Twitter. The Twitter API supports streaming results. For
this, we regularly check which hashtags are currently trending and set
up a stream of tweets that mention these entities. This provides our
backend with a constant supply of data to parse.

Reddit, on the other hand, requires that our backend go and fetch new
data at regular intervals. This was set up as an asynchronous process
that gets new data and fills a buffer with it. Once the buffer is full,
we can process this data.

For this particular prototype, we chose to have the code generating the
data speak directly to the analysis plugins. However, for more
complicated situations (e.g., when multiple different systems need
access to the same stream of data), a messaging protocol should be used.
NSQ ^[<http://nsq.io/>]
is a simple solution that does just that — data can be published
to different topics and consumers can subscribe to those topics in order
to read the stream. These sorts of systems also help deal with fault
tolerance by having guarantees on the delivery of each message.
Furthermore, they allow for easy scaling of a system by decoupling
systems from each other (see "NSQ for Robust Production Clustering" in
Chapter 10 of *High Performance Python* for a more in-depth discussion).

As data is coming in, we queue it into small batches. This idea of
mini-batches (approximately 50-100 messages per batch) is incredibly
important when doing performance analysis. While our algorithms are
online and can handle one piece of data at a time, we can greatly
optimize the I/O performance by handling small batches. It is quite
important to tune any I/O, whether it is an API request or a database
operation or simply updating an in-memory data structure, such that the
overhead of initializing such an operation is offset by the amount of
work that is actually done (see Chapter 8 of *High Performance Python*
for a more in-depth treatment of this issue).

An example of using mini-batches is with making API requests through an
HTTP interface. If we were requesting data for 50 different users, we
could either initiate 50 different requests or use an endpoint that can
satisfy all requests for us at once. While the total amount of data
being transferred back and forth is the same, we expect the batched
request to complete faster since we only need to initiate one HTTP
connection. This speedup is most evident when the time to transfer the
actual response data becomes comparable to the time to initialize a
connection.

The batches of data are sent to a collection of processing plugins.
These plugins do a variety of things, from maintaining the probabilistic
data structures to checking the health of the system. The only
requirement is that the plugin inherits from the <span
class="monospaced">PluginBase</span> object to ensure we can properly
maintain the state and integrity of the pipeline.

    class PluginBase(object):
        def save(self, location):
            """
            Save the current state of the plugin to the directory location
            """
            logger.info("Save not implemented")

        def load(self, location):
            """
            Load state from directory location or raise exception
            """
            logger.info("Load not implemented")

        def __call__(self, messages):
            """
            Process messages in the messages list. Raise exception on fatal error
            that should stop processing.
            """
            logger.info("Call not implemented")

        def routes(self):
            """
            Return description of the HTTP routes in order to get plugin state. For
            example, we could return the following list of tuples to register
            "plugin/get" and "plugin/status":
                [
                    ('/plugin/get'    , GetHandler)    ,
                    ('/plugin/status' , StatusHandler) ,
                ]
            The handler objects should inherit from tornado.web.BaseHandler.
            """
            logger.info("Routes not implemented")

These plugins are all responsible for their own external APIs (as
defined by their <span class="monospaced">routes</span> functionality).
Therefore, each plugin can register as many interfaces into its state as
are necessary. This allows us to create multiple routes for each plugin
in order to get different insights into the analyses they are
performing. For example, our plugin that handles tracking the words used
in a subreddit/hashtag registers one route to find the number of unique
words used in that community and another route to compare the similarity
of multiple communities.

#### Comparing Words

To calculate the similarity between word choices, we used a K-Min Values
data structure with <span class="monospaced">k</span> set to <span
class="monospaced">1024</span> (see
<a href="#kminvalues" class="doc_link">kminvalues</a>). This choice was
made because this structure can efficiently calculate the Jaccard
distance between two sets, which encodes how similar they are normalized
by how big they are. This is the perfect score for quantifying how much
two subreddits or hashtags share word usage while normalizing for
particularly verbose communities.

The implementation was done in GoLang and is called <span
class="monospaced">gocountme</span> ^[<http://github.com/mynameisfiber/gocountme>]
it provides a simple HTTP interface to manipulate probabilistic
sets. This allows us to very simply add elements to a collection and
then do various set operations on them. For example, we can easily add
all users of a website and compare them in a language-agnostic way.

    $ for user in $( cat site_A_users.txt ); do
        curl "http://gocountme/add?key=siteA&value=${user}";
    done;

    $ for user in $( cat site_B_users.txt ); do
        curl "http://gocountme/add?key=siteB&value=${user}";
    done;

    $ curl "http://gocountme/cardinality?key=siteA"
    {
      "status_code": 200,
      "status_txt": "",
      "data": 9943.261918592398 # Actual cardinality is 10,000 (an error of 0.56%)
    }

    $ curl "http://gocountme/correlation?key=siteA&key=siteB"
    {
      "status_code": 200,
      "status_txt": "",
      "data": [
        {
          "keys": [
            "siteA",
            "siteB"
          ],
          "jaccard": 0.234375 # Actual Jaccard distance is 0.2305 (an error of 1.65%)
        }
      ]
    }

This system was engineered to optimize for throughput and availability
by using Google’s <span class="monospaced">leveldb</span> ^[<https://github.com/google/leveldb>]
library. In addition, it supports writing customized queries
through a simple querying language in order to build up complicated
statements that can easily be evaluated server-side.

Lastly, we can examine the actual state of the internal K-Min Values
structure. This allows us to easily accumulate data from multiple <span
class="monospaced">gocountme</span> instances and perform calculations
on them — so, we can have small <span
class="monospaced">gocountme</span> instances on many computers in order
to maintain local statistics with the ability to accumulate the states
to gather insights into global statistics.

#### Keyword Usage

Keyword monitoring requires solving two problems: first we must be able
to efficiently extract valuable keywords from a given text, and then we
must be able to maintain relevant statistics for them. Extracting
keywords can quickly become a difficult problem as the number of
keywords being tracked increases. Likewise, maintaining statistics for
that set can become difficult as we start wanting to store statistics
for hundreds of thousands of items.

In order to extract the keywords from a text, we use the hierarchical
Bloom filter described in
<a href="#composite-structures" class="doc_link">Composite Structures</a>. This allows us to
find any of 10,000 keywords using only 56.7 KB of memory. Simply storing
the keywords in a hash table for lookup would require 2192.8 KB;
however, considerably more space would be required to efficiently search
for the keywords within a text, since a lot of preprocessing must be
done. Furthermore, the Bloom filter’s memory use is agnostic to the
actual size of the keywords and only scales with the number of them.

Once we have extracted the keyword data, we insert an association of
subreddit/hashtag to keyword into a system called *forgettable*.
^[<http://github.com/mynameisfiber/forgettable>]
At first look, forgettable seems to simply be a database for
categorical distributions — it can store the counts and probabilities of
keyword mentions for each subreddit or hashtag. However, it uses a
scheme borrowed from radioactive decay using Poisson processes in order
to discard old data and slowly "forget" old statistics.

![Expiring data](figures/17.svg)

##### Figure 3. Different results with different data expiration policies

The ability to forget old data in this way gives us many advantages.
Firstly, we can bound the size of the database so that we aren’t
constantly required to upgrade the server’s capacity. Also, we are able
to discard old data without having to store information regarding when
the data first came in or having discontinuities in our data as a result
of data expiration policies.

Storing timestamps for when data was inserted can be quite a burden on a
data store. In fact, storing data insertion times for the purpose of
deleting old data can easily grow the size of your data by many orders
of magnitude. In addition to the extra storage comes the extra
computation necessary to sift through the metadata in order to find the
correct elements to delete. This method has also been implemented in the
past by inserting data into a queue (thus avoiding the need to store
additional timestamps); however, this organization of the data greatly
reduces the speed of the system by forcing a complete recalculation of
results at every query.

Alternatively, another common scheme for forgetting data is to "expire"
it (i.e., delete it from the database completely after a certain
period). However, this causes problems with the statistics: old data
does not fade away into the background, but rather *all* the data simply
disappears after a certain time, causing discontinuities in any insights
given from the data.

With the forgettable approach, not only are we able to maintain smooth
statistics on our data, but we are also able to reasonably say that the
statistics provided favor recent data. One of the major advantages is
that as trends of usage change, so will the distribution that we are
storing. As a result, we can more easily identify what the current state
of a distribution is. This translates, in our prototype, to being able
to easily talk about the *recent* importance of a particular keyword in
a community. Below we see a direct call to the <span
class="monospaced">forgettable</span> API requesting the top 5
subreddits and hashtags that have recently mentioned the word "whiskey".
In addition to the direct counts in each of these groups, we see the
proportion relative to all mentions of the word (for example, the
subreddit "bourbon" accounts for 10.5% of all mentions of "whiskey").

    $ curl "http://forgettable/nmostprobable?distribution=whiskey&N=5"
    {
        "data": {
            "T": 1422464547,
            "Z": 1223,
            "data": [
                {
                    "bin": "bourbon",
                    "count": 129,
                    "p": 0.1054783319705642
                },
                {
                    "bin": "funny",
                    "count": 77,
                    "p": 0.06295993458708095
                },
                {
                    "bin": "whiskey",
                    "count": 63,
                    "p": 0.05151267375306623
                },
                {
                    "bin": "AskReddit",
                    "count": 587,
                    "p": 0.47996729354047424
                },
                {
                    "bin": "twitter",
                    "count": 260,
                    "p": 0.21259198691741618
                }
            ],
            "distribution": "whiskey",
            "last_sync_time": 1422464538,
            "prune": true,
            "rate": 4.629629e-05
        },
        "status_code": 200,
        "status_txt": ""
    }

### Design

Using probabilistic methods, we are able to efficiently process large
amounts of subreddit and Twitter trending topic data. Of course, with
great amounts of data comes the responsibility to display it without
completely overwhelming the viewer. After examining several methods of
representation, we decided to use the JavaScript library D3.js 
^[<http://d3js.org/>]
to create an interactive, force-directed graph that modeled the
subreddits and trending topics as nodes linked to each other by word use
similarity.

#### Visualizing Similarity

Displaying similarity relationships between multiple objects can be
tricky. To get a truly comprehensive view of the relationships between
the 40 nodes we display by default in the prototype, we would need
40-dimensional vision. We may someday get to that point (cybernetic
implants?), but for now we’re forced to embed those relationships in the
two dimensions of a computer screen.
^[Virtual reality systems may soon offer new opportunities for more
immersive data visualizations, but we’ll still be short by 37
dimensions.]


![Subreddit word use similarity visualized using an adjacency
matrix](figures/18.png)

##### Figure 4. Subreddit word use similarity visualized using an adjacency matrix

One of the most comprehensive methods of display is the adjacency
matrix.
^[<http://bost.ocks.org/mike/miserables/>]
This method satisfies the condition of showing the relationship
of each node to every other one, but it does so at the cost of easy or
intuitive decipherability.

![Force-directed graph: the final product](figures/19.png)

##### Figure 5. Force-directed graph: the final product

A force-directed graph displays the same information, but it uses
humankind’s inherent understanding of physical forces to build a more
intuitive model of relationships.
^[<http://bl.ocks.org/mbostock/4062045>]
The D3.js force-directed layout includes a simplified but robust
set of simulated forces based on charged particles and springs. Nodes
are pulled toward each other by links according to the strength of their
relationship. At the same time, nodes repel one another by a set force
(otherwise, the result would be a clump of nodes stacked on top of each
other — not a particularly useful visualization). The final
visualization is a result of these forces settling into an equilibrium.

![Force-directed graph: the hairball](figures/20.png)

##### Figure 6. Force-directed graph: the hairball

Force-directed graphs carry their own disadvantages, however. Plugging
our data directly into the visualization without any adjustments
resulted in a "hairball" effect, in which the number of crossed links
overwhelmed any ability to discern meaningful relationships. We first
attempted to control the hairballness by creating a link value
threshold, removing links whose similarity values were below a certain
number. This helped, but because of the variation in size between the
subreddits we were examining, the global threshold created a technically
correct but ultimately uninformative split between the top and bottom
ends of the subreddit size spectrum, where the top nodes still
hairballed and the bottom nodes were set adrift.

![Subreddit word use similarity visualized using a force-directed graph
with a global link threshold](figures/21.png)

##### Figure 7. Subreddit word use similarity visualized using a force-directed graph with a global link threshold

What we really needed was a localized threshold, which highlighted each
particular node’s strongest links. After a bit of exploration, we
settled on the following code, which ensures that each node’s two
strongest links are on the graph.

##### Example 1. Localized threshold for links

    // Sort all links from server by strength
    raw_links.sort(compareJaccard);
    // Loop through those links
    for (var i=0; i<raw_links.length; i++) {
      var link = raw_links[i];
      // Get source and target nodes from link
      var source_node = link.source_node;
      var target_node = link.target_node;
      // Set link limit per node
      var limit = 2;
      // If either the source node OR the target node has less than the link limit, then add the link to the graph. This means that each node will have at least the link limit, and many will have more.
      if (source_node.related_nodes.length < limit || target_node.related_nodes.length < limit) {
        source_node.related_nodes.push(link);
        target_node.related_nodes.push(link);
        links.push(link);
      }
    }

Because one node’s strongest link could be another’s fifth-strongest,
this does not mean each node has *only* two links, but rather guarantees
that each one meets that minimum. We found that this filter, by
significantly trimming down the hairball, did a much better job of
making the significant links visible and the overall network structure
understandable. The type of filter you may want to apply depends, like
your data structure, on what questions you are interested in.

![Force-directed graph, with the filter applied](figures/22.png)

##### Figure 8. Force-directed graph, with the filter applied

Even with the filter applied, the graph of keyword similarity is still a
lot of information to take in. Fortunately, since we are working in an
interactive medium, we can let the user clarify their picture of the
data through exploration. Using conventions from other visualizations,
such as the pan and zoom of Google Maps, makes this process feel
intuitive and approachable. In our prototype, hovering over a node
highlights its connections while fading the rest of the graph into the
background. Clicking on that node pins it to the center and reveals
additional information.

![Force-directed graph, node detail](figures/23.png)

##### Figure 9. Force-directed graph, node detail

Allowing the user to move through different levels of detail and
abstraction helps make the large amount of data in CliqueStream
digestible. As our ability to analyze huge amounts of data increases, we
must make sure to place an equal focus on making that analysis
understandable. Visualizations like force-directed layouts can be a
great aid in this process, provided we use them thoughtfully and always
as a means to communicate specific information.

### Things to Consider

There are many lessons to be learned when creating a streaming data
infrastructure and analysis routines. Making mistakes with the initial
setup can negatively impact the adoption of these systems if there is a
steep learning curve, or create many hidden costs if the infrastructure
is inefficient. However, when properly created, an easy data
infrastructure can make prototyping new projects and accessing necessary
data incredibly easy and reduce the possible complexity (and cost) of
the resulting systems.

Special care must also be taken when implementing a streaming analysis
application. Streaming data consumers are different from other analysis
applications in that they do not stop — as opposed to batch analysis
programs that run over a given chunk of data and then stop, streaming
programs should be able to keep running as long as there is data to be
processed. As a result, special care must be given to architecting in
these solutions and understanding how they will operate over time.

#### Streaming Infrastructure

-   Data should be organized and easy to find.

    -   Naming is everything. It should be easy to find and connect to a
        stream that contains information you will need.

    -   Having a directory of all available data is critical to
        promoting data usage and adoption. NSQ’s <span
        class="monospaced">nsqadmin</span> application is fantastic for
        this.

-   Data schema should be regular and documented.

    -   Keys for data should be regular and easily decipherable. All
        data in the same stream should have the same key values.

-   Decouple data production and consumption.

    -   Make sure your streams are queued, fault tolerant, and robust.

    -   Upstream services should not be affected if a downstream
        application fails.

    -   If the amount of data created increases past the consumption
        rate, stream boxes should queue messages (potentially to disk to
        avoid running out of memory) and alert so that more consumption
        capacity can be added.

    -   Data should be requested from the consumer as opposed to pushed
        from the data queue. This will keep the consumer insulated and
        avoid adding more burden to an application that may be
        experiencing problems.

    -   Make sure to have exponential backoffs when experiencing
        problems and adequate backpressure so that faults do not
        propagate through the system.

-   Think of data availability as guarantees.

    -   How much latency can you guarantee between data production and
        consumption?

    -   Can you guarantee that every message will get delivered? What
        are the bounds on how many times a message can be delivered?

    -   Does a consumer get messages it missed if it went offline? What
        if it went offline for days?

    -   Can you guarantee that data will maintain the same schema? Will
        changes to the data be in a new stream or must clients be able
        to accept changes?

#### Data Analysis

-   Enumerate the questions being asked by a project and the smaller
    questions that must be answered along the way.

    -   Are these questions providing the insights that are necessary?

    -   Can the questions be simplified? Are there other questions that
        would provide similar insights?

    -   How accurate must the answers be to still be useful? Most of the
        time approximate and order-of-magnitude results are sufficient!

-   What are the memory bounds of this solution?

    -   How much memory will be needed to do the calculation currently?

    -   How much memory will be needed to do the calculation in a year?

    -   If the amount of data doubles, what will happen to memory use?
        How can this be mitigated as resource usage exceeds server
        capacity? (Increasing server capacity should be a last resort!)

-   What are the computational bounds of this solution?

    -   How much computation must be done per piece of data?

    -   How does computation scale if the data doubles?

    -   Can the computation be distributed to many servers? If
        inter-server communication is necessary for this, how can it be
        minimized?

-   Protect against data outages.

    -   What happens if the data stream temporarily stops as a result of
        upstream problems?

    -   Will it take time for the system to recover from problems with
        upstream data? How long?

-   How can you provide the results of your analysis to users or other
    applications for further processing?

    -   Is it worthwhile to output a new stream resulting from your
        application’s analysis?

    -   What is the simplest way to make an API for users to query the
        current results? What are the most requested results, and how
        can those be provided easily?

    -   What request rate can your application maintain to its API? What
        happens when this is doubled? Can you use caching to improve
        this?

------------------------------------------------------------------------