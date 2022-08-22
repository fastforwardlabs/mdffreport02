Table of Contents

**JavaScript must be enabled in your browser to display the table of
contents.**

## Applications

Stream processing with probabilistic algorithms has a strong and quiet
history in industry. The main advantage of probabilistic algorithms is
their ability to do classic calculations with much less computational
and memory resources than traditional methods. As a result, many of the
applications listed below are indeed things that we could previously
have calculated. However, the difference now is the speed at which we
can do these calculations.

Further, these possible speed increases are orders of magnitude faster
than current standard techniques. This isn’t just a small improvement,
but a massive innovation.

Imagine we have a thermometer that issues a temperature reading every
.05 seconds. To calculate the average temperature in a batch system, we
would take an hour’s worth of data, or 72,000 data points. We would add
them all up and then divide by the number of data points. We might run
this calculation each minute.

![A batch method would store each temperature reading in a database that
calcuations can be performed upon](figures/04.svg)

##### Figure 1. A batch system would store each temperature reading in a database that calcuations can be performed upon

In a probabilistic realtime system, we can only examine each data item
in the stream once. As a result, we could choose one of many possible
algorithms that either take a subsample of the data or find a way to
create a synopsis of the data that can be used later for better
analysis.

![A probabilistic system performs the calculations as the data comes in,
storing only a synopsis](figures/05.svg)

##### Figure 2. A probabilistic system performs the calculations as the data comes in, storing only a synopsis of the data

One possible realtime approach is to choose a number of sample values,
say 10, and then replace those values with decreasing likelihood over
time. An average can be calculated from those 10 values at any given
time and is very likely to reflect the average temperature over the time
the calculation has been running (this is called reservoir sampling).
Another possible solution is to use a decaying distributional database
(such as forgettable, described in
<a href="#keywordusage" class="doc_link">keywordusage</a>) to maintain a
small representation of the current distribution of temperatures. In
this way we can infer not only a view of the average, but also other
higher-order statistics.

In the batch example, we’d be using significant computational resources
on an ongoing basis, not to mention storing a complete hour’s worth of
data. In the realtime example, we store only 10 data points, and the
computation is orders of magnitude cheaper.

In addition, being online algorithms, there is a fundamental difference
in how the algorithms generate results — while batch algorithms only
have a result when they are done processing, online algorithms maintain
an internal state that is constantly updating with the most recent
results. This provides excellent mechanics for these algorithms to be
incorporated into data streams and any sort of realtime processing. We
can have a process that is responsible for reading the stream and
updating the internal state, while any other process can freely read the
internal state and see what the current results are. Moreover, in terms
of distributed data analysis, this separation between reading and
writing is incredibly important to building performant systems.

Practically, these orders-of-magnitude improvements in performance allow
us to run 10 or 100 times more computations for the same cost for
high-performance applications; or to run many experimental calculations;
or to even push computation onto cheap devices, which has wide
implications for mobile development and the emerging Internet of Things
(IoT).

In the following few sections we walk through specific examples of where
probabilistic techniques are being used today.

### Trending Topics

Social networks see tons of data. This data is flowing in as a constant
stream of timestamped updates. For example, there are approximately six
thousand tweets posted to Twitter each second, <span class="footnote">  
\[<http://bit.ly/1yKfYAT>\]  
</span> and that number will probably continue to increase. Despite this
volume of data, most networks offer a realtime view of the top
discussion topics that are currently popular with users of the service.
This is a tool for users to explore and discover the most interesting
and timely content.

![Twitter and Facebook trending topics for a random day in January,
2015](figures/06.png)

##### Figure 3. Twitter and Facebook trending topics for a random day in January, 2015

These calculations are not simply counts of words used or clicks on
links. If that were the case, Justin Bieber and Kim Kardashian would top
the lists all the time. <span class="footnote">  
\[In 2010, updates about pop star Justin Bieber were believed to consume
about 3% of Twitter’s server infrastructure
(<http://mashable.com/2010/09/07/justin-bieber-twitter/>).\]  
</span> Instead, as messages enter the system, they are broken up into
phrases of various lengths, called *n-grams*. These phrases are then
monitored with a probabilistic calculation that represents the rate of
mentions of the phrases.

Machine learning loves odd vocabulary. Single words, or 1-grams, are
called "unigrams"; pairs of two words, or 2-grams, are called "bigrams";
sets of three words, or 3-grams, are called "trigrams"; and larger sets,
such as 4-grams, are just pronounced as written ("four grams," etc.).

Once we have a current rate and a history of all n-grams' usage over
time, we can quantify how anomalous the current traffic is. When the
system sees a disproportionate rate of mentions of a phrase, it can
elevate that n-gram to be "trending."

The remarkable thing about this system is that for any phrase in any
language, it can effectively answer the question, what is the current
rate of discussion? This massive calculation would be prohibitively
expensive and much too slow if done in a batch system.

![A bursting phrase on social media](figures/07.png)

##### Figure 4. A burst of activity around a phrase in social media

This method allows for a system that can monitor attention to tens of
millions of phrases efficiently and in real time.

Finally, some networks have human review before topics are presented
publicly to check that they are appropriate. This is a good idea
whenever data directly from the Internet is used in a consumer-facing
product.

### User Segmentation

![User segmentation](figures/08.svg)

##### Figure 5. User are segmented into different groups to understand their behaviours

When operating large websites, it is often important to understand the
various types of people that are visiting your pages. This can become
quite a task when there are millions of people visiting the website
daily and your content is constantly changing. You may be able to do the
required analysis overnight, but the result may be of less use the next
day; you want to know the types of people visiting the page right now.

This problem highlights the difference between *online* and *offline*
clustering methods. User segmentation is often done by selecting a group
of features from the users' sessions (for example, the titles or tags of
the pages they have gone to and the paths through the website they have
taken) and seeing if there are any groups of users who share similar
actions. Each of these groups will be considered one cluster and
represent one type of user behavior.

One problem, however, is that as the content on your pages changes, so
too will the actions of your users. In fact, you could put up a new page
that attracts a completely different group of people you have never seen
before! It is very useful to be able to understand this right away and
see how the changes you make to the content of your website change user
behavior in real time.

In a realtime system, we can run an online clustering algorithm which is
able to cluster users into distinct populations. As more users perform
actions, the characteristics of these populations will change and so
will the characterisations given by the clustering algorithm. This
allows us to query the algorithm at any point to see if two users are in
the same population and how many users are within each group.

### Database Precaching

Databases are storing more and more data, which at some point must be
stored on a disk or a cluster of disks. As these clusters grow in size,
the potential cost for reading from them will invariably increase as
well. Some databases try to mitigate this latency through various
indexing schemes. An index is a compressed representation of the data
with pointers to where you can find the full values.

![Database precaching](figures/09.svg)

##### Figure 6. A quick precaching mechanism reduces the number of requests to the bulky database

For example, riak <span class="footnote">  
\[<http://basho.com/riak/>\]  
</span> will store a copy of the index in memory so that we know whether
the data even exists and where it is stored. This type of scheme is
necessary when we are dealing with potentially very high latency reads
from the actual database on disk — if we have a very quick way of first
figuring out whether the data exists we can make sure we only do the
very expensive disk reads when absolutely necessary.

The riak approach rapidly encounters a problem when the index requires
too much memory, however. This can happen very quickly when we want to
store many small items. In this case we will quickly fill up memory with
all of the keys we are trying to store, even though the actual values do
not come close to the storage capacity of our database cluster.

This sort of problem is perfectly suited for a probabilistic data
structure. Bloom filters are data structures with a fixed, small memory
footprint that store a set of objects and can be queried to see if
things are in that set of objects. This allows us to answer the
question: for any object, have we seen this object in the past? This
question can be answered with no false negatives and only false
positives. What this means is that, in the worst-case scenario, a bloom
filter will return an incorrect result and say that it has seen an
object before when it actually hasn’t (the converse can never happen — a
bloom filter will never say it hasn’t seen an object when it really
has). In addition, this error can be made incredibly low (in
<a href="#memory_pd_wiki_comparison"
class="doc_link">memory_pd_wiki_comparison</a> we’ll see that we can
store 4,956,262 unique keys with 0.14% error using only 11.5 MB).

By using a bloom filter, we can very easily create a precache that
stores all of the keys in the database that we have seen before. If a
lookup is requested, we can quickly verify whether we have seen the data
before and, if we have, proceed with the expensive disk read. Since we
are using a probabilistic data structure for this problem, as opposed to
storing the full keyspace in memory, we can store an incredible amount
of keys in memory for incredibly fast access while not practically
limiting our database capacity to the amount of RAM available. For
example, if we wanted our bloom filter to only have a 0.1% error rate,
then we could store a representation of 556,421,600 items per gigabyte
of RAM (regardless of the actual item size). This is starkly more than
the 41,666,666 keys/GB we could store if we stored the raw keys
(assuming the keys were 24 characters long and there was no overhead).

This specific problem appears in many places even outside of databases.
In biology this approach is frequently used in protein folding problems
where backtracking algorithms are used. A small representation of some
work that was previously attempted and may have failed can be stored in
a probabilistic system so that in future iterations of the algorithm we
do not waste time attempting the calculation again. This is particularly
important since the actual models that are being worked on are very
complex — storing a small representation saves memory and the
probabilistic system saves computation so that the algorithm can focus
on the folding. This allows the system to perform very large, complex
operations and then store a compact representation of the results, which
is an efficient way to search through a large, complex space.

In general, whenever calculations are expensive but can be avoided if we
can identify whether they’ve been done before, these sorts of precaching
algorithms are indispensable.

### Graph Databases

The growing interdependencies between systems and groups of people have
led to a major focus on graphs as an essential way of understanding the
dynamics of social and human networks. Graph techniques have been used
in a wide range of applications, from recommendation algorithms
(collaborative filtering and SPEAR) to search indexing (PageRank),
social networks (frontrunners, influence tracking), and
traffic/networking problems for both computer and physical
infrastructure (path detection).

In all of these cases, the amount of data we are tracking is large and
is only increasing. For example, in the past social networks used to
only change by thousands of connections per day, but now the changes can
easily be on the order of thousands per second! These systems are taking
in more and more data, and we still want to be able to do the same kinds
of complex analysis.

For graphs, one major hurdle in this shift to more data is how to
efficiently store the data on a cluster of servers. Not only are we
storing some value representing a node on the graph, but we must also
store connections to other nodes and hopefully be able to retrieve
neighboring nodes quickly when necessary. It is this need to constantly
be traversing the graph that makes storing graphs difficult. If we have
a distributed database and request a user’s data, as well as their
friends' data, the more the information is spread across our cluster the
slower our request will be! This is because the most expensive factor in
retrieving data from a graph database is the latency in accessing the
different nodes in the cluster. It’s much better to make one query to
one node than incur large amounts of network overhead getting data from
many nodes.

This problem of how to place data in a distributed graph database is
called the *balanced graph partitioning problem*. We want to spread the
data onto as many machines and as equally as possible, while minimizing
the number of edges that span across machines. A realtime solution to
this problem will allow us to start storing truly massive datasets and
doing thorough analyses of them where it is currently simply impractical
to do so. Currently, most graph databases become impractical once the
dataset reaches billions of nodes with a comparable number of
relationships. The only remedy that is available is to use considerable
resources building customized solutions or using an off-the-shelf
solution which has far from optimal performance.

The balanced graph partitioning problem is an open and difficult
research problem. A solution to this problem will revolutionize the
state of data science.

While there have been many attempts to solve this problem, and there are
many promising methods out there, a current leading approach is based on
subtree kernels and being able to compare graphs to each other. This
problem is quite complex when calculated fully; however, probabilistic
methods have been devised that make a fingerprint of the structure of
the graph and allow for quick comparisons.

![Comparing graph similarity](figures/10.svg)

##### Figure 7. A quick and robust comparison method for graphs opens up completely new analysis possibilities

This method is not only useful as a potential solution to graph
partitioning, but also allows us to use graphs for many other
applications that were not possible before. We can run classifiers that
can classify graphs without having to hand-code features from them. For
example, we can compare the types of friend groups (represented as
graphs of people) of different people and classify the different
communities without having to manually decide what important features in
the graph should be considered in the classification. This makes any
resulting model much more robust and meaningful, since we can later
deduce what features the algorithm thought were important in the
classification.

### Anomaly Detection

Anomaly detection is another common application. Anomaly detection is
akin to finding the needle in the haystack, while other applications
that we’ve discussed are more like understanding the nature of the
haystack <span class="footnote">  
\[Thanks to Gary King for this analogy (<http://bit.ly/1zxaQit>)\]  
</span> Furthermore, we want to be able to not only find the needle, but
predict when we will next see it and what other haystacks will contain
similar needles. In finance, for example, it is critical to be able to
find, track, and predict any sort of possible anomalous situation.

![Anomaly detection](figures/11.svg)

##### Figure 8. Anomaly detection can alert when specific trends start to appear and help correlate multiple signals to eachother

This can become quite taxing computationally as the number of different
series of data you are tracking increases. Moreover, when it becomes
necessary to correlate different series with one another, the complexity
of the system goes through the roof.

Probabilistic data structures and general streaming data analysis can
help by filtering out which series possibly have anomalies in them and
doing a first-pass correlation in order to reduce the total workload.
Various hashing schemes like locality-sensitive hashing provide a quick
way to summarize data and find similar instances of it. Furthermore,
various methods using HyperLogLog as a first-pass filter have been
studied that are able to automatically find correlations within an
arbitrary stream of data.

### Security and Exploitation

In computer security, it is often hard to be able to identify a threat
before it becomes a danger. This is what makes maintaining the security
of a network difficult — while it is relatively easy to protect against
threats that have been seen before, you never know whether you have
already been targeted by new threats. In addition, even if you know the
fingerprint of a potential attack, it is often hard to sift through all
the requests happening on the network in order to identify it before it
is too late.

New measures in computer security focus on identifying trends and
patterns in normal service usage and being able to quickly identify when
users deviate from the norm. This helps security teams recognize traffic
that may represent attack vectors into their systems.

Probabilistic algorithms that are able to summarize the complex traffic
pattern of a user — potentially spanning months of service usage — are
incredibly important in this process. By being able to summarize these
usage patterns and perform similarity measurements on these
fingerprints, we are able to quickly spot when something anomalous is
happening. In addition, because we’re operating on summaries we can do
the calculation quickly and store much less data. This transforms a
complex analysis requiring special-purpose hardware into one involving
small algorithms that can practically run on routers or load balancers,
or concurrently with other services on a system. By being able to do
these calculations quickly, we can recognize and prevent intrusions.
This will help thwart attacks before they complete instead of
identifying them after the fact.

For example, a port scan is when a malicious (or curious) machine
attempts many connections to a target server over the network.
Algorithms such as thresholded random walks have shown promise at
detecting port scans, which have always been hard to monitor because of
the low signal-to-noise ratio (for each malicious connection by a port
scan there could be thousands of normal connections) and the sheer
sophistication of the methods used. With the use of online and
probabilistic methods, it is possible to quickly (both computationally
and in terms of the number of samples needed) detect port scans with a
very low incidence of false negatives.

------------------------------------------------------------------------

Last updated 2022-08-22 08:56:03 PDT
