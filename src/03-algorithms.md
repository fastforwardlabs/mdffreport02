Table of Contents

**JavaScript must be enabled in your browser to display the table of
contents.**

## Algorithms

In this section we present a technical introduction to stream analysis
with online algorithms and probabilistic data structures. Then we dive
deep into a small selection of probabilistic data structures to give an
in-depth understanding of how they work. This understanding of the
fundamentals will give the reader a foothold for understanding most of
the other probabilistic algorithms in the wild. Finally, we will go
through two real-world examples in order to compare the selected data
structures.

The source code in this section is presented in the Python programming
language, which is commonly used for probabilistic applications and also
has the advantage of being easy to read.

This section provides a foundation for actionable decisions regarding
which structure to use in different applications.

### Realtime Algorithm Primer

Realtime algorithms are generally characterized by being "online" or
"one-pass." This means that the algorithm needs to see a particular
piece of data once, and only once, in order to maintain its calculation
and later give a response to whatever query the algorithm was designed
to answer. While these algorithms generally have a bit more mathematical
sophistication, the payoff is an enormous reduction in computation time
and memory footprint.

For example, let’s look at two similar algorithms for calculating the
variance of a series of numbers. The offline (batch) method would take
all of the data points, sum them, and then divide the sum by the total
number of data points in order to calculate the mean. Then, we would go
back and see how our data points deviated from this mean in order to
find the variance. If a new data point came in and we wanted to again
know the variance for our dataset, we would need to recalculate the mean
and then again go through and see how our data deviates from the new
mean.

On the other hand, an online algorithm that calculates the variance for
a stream of numbers always maintains the current sum of the data points,
how many data points have already been seen, and an auxiliary quantity,
<span class="monospaced">M2</span>, that relates to the variance. In
this case, whenever we want to know the variance of our dataset we
simply must divide the auxiliary quantity that we are maintaining by our
count of the number of items. These two quantities that the algorithm
keeps track of are called the *internal state* of the algorithm.

When looking at the complexity of an algorithm, we generally use Big-O
notation. In this notation we describe how many operations, in general,
the algorithm must perform for an input of length <span
class="monospaced">N</span>. For example, if our algorithm is <span
class="monospaced">O(N)</span>, if we double the amount of data then we
double the amount of work that must be done. For latexmath:<span
class="monospaced">O(N^2)</span>, a doubling of the data is a
quadrupling of the amount of work needed. Ideally an algorithm is <span
class="monospaced">O(1)</span>, which means the algorithm does not
perform differently depending on how much data you give it — it will
always have the same performance.

Using these internal states, we are able to shift the computational
complexity of a query into the insertion. With the batch algorithm we
must do <span class="monospaced">O(1)</span> calculations to insert a
new item and <span class="monospaced">O(N)</span> to calculate the
average. On the other hand, for the online algorithm we must again do
<span class="monospaced">O(1)</span> for an insertion (albeit with a
larger constant factor), but only <span class="monospaced">O(1)</span>
to calculate the variance! Furthermore, we save on memory as well. For
the batch algorithm, we must store every data point we wish to use in
our calculation (<span class="monospaced">O(N)</span>); however, the
online algorithm only needs to store the two internal state variables
(<span class="monospaced">O(1)</span>).

##### Example 1. Batch versus online variance

    class BatchVariance(object):
        # The internal state of this object grows linearly with the amount of data
        # fed into it
        self._dataset = []

        def insert(self, data):
            self._dataset.append(data)

        def mean(self):
            return sum(self._dataset) / float(len(self._dataset))

        def variance(self):
            mean = self.mean()
            exp2 = 0
            for data in self._dataset:
                exp2 += (data - mean)**2
            return exp2 / (n - 1)


    class OnlineVariance(object):
        # The internal state of this object is fixed at 3 numbers regardless of how
        # much (or how little) data it has seen
        _n = 0
        _mean = 0
        _M2 = 0

        def insert(self, data):
            self._n += 1
            delta = data - self._mean
            self._mean += delta / self._n
            self._M2 += delta * (data - self._mean)

        def mean(self):
            return self.mean

        def variance(self):
            if self._n < 2:
                return 0
            return self._M2/(self._n - 1)

#### Online Algorithms for Streams

When working with streaming data, having an online algorithm is
critical. It is *because* we can look at a piece of data once and then
throw it away that these algorithms are so attractive. As a result, the
computational complexity and memory use of these algorithms are very
strictly bounded and they are very well behaved. Batch algorithms, on
the other hand, have computational complexity and memory use that are
very dependent on the input dataset and can quickly make analysis
unfeasible with available hardware and time constraints.

Streams are characterized by a potentially unending sequence of data
that is constantly being fed to your algorithms. If your algorithm is
online, then the amount of memory required is simply the size of its
internal state. For most realtime algorithms (in particular for those
described below, except the scaling Bloom filter), the size of the
internal state is static and does not depend on how much data has been
seen before. That is to say, whether we have just turned on our
algorithm or it has been processing data for a month, it will still use
the same amount of resources and have the same performance
characteristics when processing a query.

Imagine a bucket that can hold any amount of water. That’s an online
algorithm. Now imagine a balloon, which expands to hold the amount of
water you put inside it. That’s an offline algorithm.

On the other hand, batch algorithms completely depend on how much data
they have seen. Many batch algorithms work perfectly well with small
amounts of test data but start to under perform when put into production
with realistic datasets. This is because as they see more data their
internal state grows, which means they not only need to store a larger
state but also must process more data every time they respond to a
query.

A common remedy for this is to only look at data from a fixed time
window (for example, only using the past week’s worth of data). However,
this is simply a band-aid for the problem: the algorithm will still do
just as poorly if the amount of data seen per day or the complexity of
the data increases. <span class="footnote">  
\[The complexity might increase, for example, if the number of edges in
the graph being analyzed increases.\]  
</span> Furthermore, these sorts of constraints to the input dataset are
often motivated simply by the resource usage of the algorithm and not
the actual desired insights from the results, which can result in
confusion or simply making the data useless. Most importantly, however,
this sort of grouping by time can easily skew statistics. For example,
if you were to group your data into hour long groupings, what would
happen if you shifted the groups by 30min? How would your insights
change if the grouping were changed to 1min instead? This problem stems
from the fact that these sorts of models have no optimal value for the
number or size of the groupings <span class="footnote">  
\[<http://en.wikipedia.org/wiki/Histogram#Number_of_bins_and_width>\]  
</span> yet the results are completely dependant on this choice.

### Probabilistic Data Structures

Section adapted from *High Performance Python* by Micha Gorelick and Ian
Ozsvald (O’Reilly). Copyright 2014 Micha Gorelick and Ian Ozsvald,
978-1-4493-6159-4.

Probabilistic data structures allow us to make trade-offs in accuracy
for immense decreases in memory usage. They are a form of online
algorithm that performs a small calculation when data first arrives in
order to store a "synopsis" of what it has seen, so it is able to answer
queries. As a result, the number of operations we can do on these data
structures is much more restricted than when we have the full dataset in
a <span class="monospaced">set</span> or a trie. However, with a single
HyperLogLog++ structure using 2.56 KB of memory we can count the number
of unique items up to approximately 7,900,000,000 items with 1.625%
error.

This means that if we were trying to count the number of unique license
plate numbers for cars, if our HyperLogLog++ counter said there were
654,192,028, we would be confident that the actual number lies between
664,822,648 and 643,561,407. Furthermore, if this accuracy is not
sufficient, we can simply add more memory to the structure and it will
perform better. Giving it 40.96 KB of resources will decrease the error
from 1.625% to 0.4%. If we had stored the full dataset this would have
taken 3.925 GB (and that is assuming no overhead!).

On the other hand, the HyperLogLog++ structure would only be able to
count license plates or compare with another collection to see how many
license plates the two structures had in common or how many were
different. So, for example, we could have one structure for every state
in order to find how many unique license plates are in those states. We
could then merge them all to get a count for the whole country. However,
given a license plate we couldn’t tell you if we’ve seen it before with
very good accuracy, and we couldn’t give you a sample of the actual
license plate numbers we have already seen. If those were the important
questions to ask, we would choose another probabilistic data structure
(for example, a Bloom filter) instead.

Probabilistic data structures are fantastic when you have taken the time
to understand the problem you are trying to solve and need to put
something into production that can answer a very small set of questions
about a very large set of data. Each different structure has different
questions it can answer at different accuracies, so finding the right
one is just a matter of understanding your requirements.

In almost all cases, probabilistic data structures work by finding an
alternative representation for the data that is more compact and
contains the relevant information for answering a certain set of
questions. This can be thought of as a type of lossy compression, where
we may lose some aspects of the data but we retain the necessary
components. Since we are allowing the loss of data that isn’t
necessarily relevant for the particular set of questions we care about,
we can still answer the questions at hand while storing the minimal
amount of information. It is because of this that the choice of which
probabilistic data structure you will use is quite important — you want
to pick one that retains the right information for your use case!

Before we dive in, it should be made clear that all the "error rates"
here are defined in terms of standard deviations. This term comes from
describing Gaussian distributions and describes how spread out a
function is around a center value. When the standard deviation grows, so
do the number of values further away from the center point. Error rates
for probabilistic data structures are framed this way because all the
analyses around them are probabilistic. So, for example, when we say
that the HyperLogLog algorithm has an error of err =
\frac{1.04}{\sqrt{m}} we mean that 66% of the time the error will be
smaller than <span class="monospaced">err</span>, 95% of the time it
will be below <span class="monospaced">2\*err</span>, and 99.7% of the
time it will be below <span class="monospaced">3\*err</span>. <span
class="footnote">  
\[These numbers come from the 66-95-99 rule of Gaussian distributions.
More information can be found at
<http://en.wikipedia.org/wiki/68%E2%80%9395%E2%80%9399.7_rule>.\]  
</span>

#### Very Approximate Counting with a 1-byte Morris Counter

We’ll introduce the topic of probabilistic counting with one of the
earliest probabilistic counters, the Morris counter (by Robert Morris of
the NSA and Bell Labs). Applications include counting millions of
objects in a restricted-RAM environment (e.g., on an embedded computer),
understanding large data streams, and problems in AI like image and
speech recognition.

The Morris counter keeps track of an exponent and models the counted
state as 2^{exponent} (rather than a correct count) — it provides an
order of magnitude estimate. This estimate is updated using a
probabilistic rule.

We start with our state, an exponent, set to 0. If we ask for the
"value" of the counter, we’ll be given <span
class="monospaced">pow(2,exponent) = 1</span> (the keen reader will note
that this is off by one — we did say this was an *approximate*
counter!). If we ask the counter to increment itself it will generate a
random number (using the uniform distribution) and test if <span
class="monospaced">random(0, 1) &lt;= 1/pow(2,exponent)</span>. For
<span class="monospaced">exponent == 0</span> this will always be true
and the counter will increment, setting the exponent to 1.

The second time we ask the counter to increment itself it will test if
<span class="monospaced">random(0, 1) &lt;= 1/pow(2,1)</span>. This will
be true 50% of the time. If the test passes, then the exponent is
incremented. If not, then the exponent is not incremented for this
increment request.

The table below shows the likelihoods of an increment occurring for each
of the first exponents.

##### Table 1. Morris counter details

exponent

pow(2,exponent)

P(increment)

0

1

1

1

2

0.5

2

4

0.25

3

8

0.125

4

16

0.0625

…

…

…

254

2.894802e+76

3.454467e-77

The maximum we could approximately count where we use a single unsigned
byte for the exponent is <span class="monospaced">math.pow(2,255) ==
5e76</span>. The error relative to the actual count will be fairly large
as the counts increase, but the RAM saving is tremendous as we only use
1 byte rather than the 32 unsigned bytes we’d otherwise have to use. In
the 1970s, when this scheme was devised, this saving of 31 bytes was
tremendous and allowed for previously unimaginable calculations to be
done.

##### Example 2. Simple Morris counter implementation

    # A more fully fleshed out implementation which uses an `array` of bytes
    # to make many counters is available at:
    # https://github.com/ianozsvald/morris_counter

    from random import random

    class MorrisCounter(object):
        counter = 0
        def add(self, *args):
            if random() < 1.0 / (2 ** self.counter):
                self.counter += 1

        def __len__(self):
            return 2**self.counter

Using the example implementation in
<a href="#memory_morris_example_code"
class="doc_link">memory_morris_example_code</a>, we can see that the
first request to increment the counter succeeds and the second fails.
<span class="footnote">  
\[A more fully fleshed out implementation that uses an <span
class="monospaced">array</span> of bytes to make many counters is
available at <https://github.com/ianozsvald/morris_counter>."\]  
</span>

##### Example 3. Morris counter library example

    In [2]: mc = MorrisCounter()
    In [3]: print len(mc)
    1.0
    In [4]: mc.add()  # P(1) of doing an add
    In [5]: print len(mc)
    2.0
    In [6]: mc.add()  # P(0.5) of doing an add
    In [7]: print len(mc)  # the add does not occur on this attempt
    2.0

In
<a href="#FIG-morris-counter" class="doc_link">FIG-morris-counter</a>,
the thick black line shows a normal integer incrementing on each
iteration. On a 64-bit computer this is an 8-byte integer. <span
class="footnote">  
\[More information about the error behavior can be seen at
<http://geomblog.blogspot.co.uk/2011/06/bob-morris-and-stream-algorithms.html>.\]  
</span> The evolution of three 1-byte Morris counters is shown as dotted
lines: the y-axis shows their values, which approximately represent the
true count for each iteration. Three counters are shown to give you an
idea about their different trajectories and the overall trend; the three
counters are entirely independent of each other.

![Three 1-byte Morris counters](figures/12.svg)

##### Figure 1. Three 1-byte Morris counters vs. an 8-byte integer

#### K-Min Values

When comparing sets is critical — for example, when creating similarity
graphs showing connectivity between different communities (such as in
the prototype, CliqueStream) or comparing user trends — K-Min Values is
the perfect data structure to use.

In the Morris counter, we lose any sort of information about the items
we insert. That is to say, the counter’s internal state is the same
whether we do <span class="monospaced">.add("alice")</span> or <span
class="monospaced">.add("bob")</span>. This extra information is useful
and, if used properly, could help us have our counters only count unique
items. In this way, calling <span
class="monospaced">.add("alice")</span> thousands of times would only
increase the counter once.

To enable this, we will exploit properties of hashing functions. These
functions can be any arbitrary method that takes in an input and assigns
a numerical value to it. These values do not need to be unique (in fact,
these "collisions" account for most of the error in probabilistic data
structures), however they should be repeatable. That is to say, I should
get the same result for the same input. However, the main property we
would like to take advantage of is the fact that the hash function takes
input and *uniformly* distributes it. For example, let’s assume we have
a hash function that takes in a string and outputs a number between 0
and 1. For that function to be uniform means that when we feed it in a
string we are equally likely to get a value of 0.5 as a value of 0.2, or
any other value. This also means that if we feed it in many string
values, we would expect the values to be relatively evenly spaced.
Remember, this is a probabilistic argument: they won’t always be evenly
spaced, but if we have many strings and try this experiment many times,
they will tend to be evenly spaced.

Suppose we took 100 items and stored the hashes of those values (the
hashes being numbers from 0-1). Knowing the spacing is even means that
instead of saying, "We have 100 items," we could say, "We have a
distance of 0.01 between every item." This is where the K-Min Values
algorithm <span class="footnote">  
\[Beyer, K., Haas, P. J., Reinwald, B., Sismanis, Y., and Gemulla, R.
"On synopses for distinct-value estimation under multiset operations."
*Proceedings of the 2007 ACM SIGMOD International Conference on
Management of Data - SIGMOD ’07* (2007): 199-210.
doi:10.1145/1247480.1247504.\]  
</span> finally comes in — if we keep the <span
class="monospaced">k</span> smallest unique hash values we have seen, we
can approximate the overall spacing between hash values and infer what
the total number of items is.

In <a href="#FIG-kmv-hash-density"
class="doc_link">FIG-kmv-hash-density</a> we can see the state of a
K-Min Values structure (also called a KMV) as more and more items are
added. At first, since we don’t have many hash values, the largest hash
we have kept is quite large. As we add more and more, the largest of the
<span class="monospaced">k</span> hash values we have kept gets smaller
and smaller. Using this method, we can get error rates of
O\left(\sqrt{\frac{2}{\pi(k-2)}}\right).

![Density of hash space for K-Min Values structures](figures/13.svg)

##### Figure 2. The values stored in a K-Min Values structure as more elements are added

This is similar to estimating the number of people in a room by looking
at how much space a smaller portion of the group takes up. If we assume
that people want to be as spaced out as possible (the property we get
from the uniformity of our hash function), then the amount of space
<span class="monospaced">k</span> people use indicates the total number
of people. For example, if we have a room with a size of 1,000 square
feet (sf) and we know that a group of 10 people in the room are taking
up 20 sf, then we can estimate there are 500 people in the room!

The larger <span class="monospaced">k</span> is, the more we can account
for the hashing function we are using not being completely uniform for
our particular input, and for unfortunate hash values. An example of
unfortunate hash values would be hashing <span class="monospaced">\["A",
"B", "C"\]</span> and getting the values <span
class="monospaced">\[0.01, 0.02, 0.03\]</span>. If we start hashing more
and more values, it is less and less probable that they will clump up.

Furthermore, since we are only keeping the smallest *unique* hash
values, the data structure only considers unique inputs. We can see this
easily because if we are in a state where we only store the smallest
three hashes and currently <span class="monospaced">\[0.1, 0.2,
0.3\]</span> are the smallest hash values, if we add in something with
the hash value of <span class="monospaced">0.4</span> our state will not
change. Similarly, if we add more items with a hash value of <span
class="monospaced">0.3</span>, our state will also not change. This is a
property called *idempotence*; it means that if we do the same operation
with the same inputs multiple times, the state will not be changed. This
is in contrast to, for example, an <span
class="monospaced">append</span> on a list, which will always change its
value. This concept of idempotence carries on to all of the data
structures in this section except for the Morris counter.

##### Example 4. Simple KMinValues implementation

    import mmh3
    from blist import sortedset

    class KMinValues(object):
        def __init__(self, num_hashes):
            self.num_hashes = num_hashes
            self.data = sortedset()

        def add(self, item):
            item_hash = mmh3.hash(item)
            self.data.add(item_hash)
            if len(self.data) > self.num_hashes:
                self.data.pop()

        def __len__(self):
            if len(self.data) <= 2:
                return 0
            # 2**32-1 normalizes the hashes we store since our hash function gives
            # us a number from 0 to 2**21-1 instead of 0 to 1
            return (self.num_hashes - 1) * (2**32-1) / float(self.data[-2] + 2**31 - 1)

Using the <span class="monospaced">KMinValues</span> implementation in
the Python package <span class="monospaced">CountMeMaybe</span>, <span
class="footnote">  
\[<https://github.com/mynameisfiber/countmemaybe>\]  
</span> we can begin to see the utility of this data structure. This
implementation is very similar to the one in
<a href="#memory_simple_kmv" class="doc_link">memory_simple_kmv</a>, but
it fully implements the other set operations, such as union and
intersection. Also note that "size" and "cardinality" are used
interchangeably (the word "cardinality" is from set theory and is used
more in the analysis of probabilistic data structures). Here, we can see
that even with a reasonably small value for <span
class="monospaced">k</span>, we can store 50,000 items and calculate the
cardinality of many set operations with relatively low error:

    >>> from countmemaybe import KMinValues

    >>> kmv1 = KMinValues(k=1024)

    >>> kmv2 = KMinValues(k=1024)

    >>> for i in xrange(0,50000): # We put 50,000 elements into kmv1.
        kmv1.add(str(i))
       ...:

    >>> for i in xrange(25000, 75000): # kmv2 also gets 50,000 elements, 25,000 of which are the same as those in kmv1
        kmv2.add(str(i))
       ...:

    >>> print len(kmv1)
    50416

    >>> print len(kmv2)
    52439

    >>> print kmv1.cardinality_intersection(kmv2)
    25900.2862992

    >>> print kmv1.cardinality_union(kmv2)
    75346.2874158

With these sorts of algorithms, the choice of hash function can have a
drastic effect on the quality of the estimates. Both of these
implementations use <span class="monospaced">mmh3</span>, a Python
implementation of <span class="monospaced">mumurhash3</span> that has
nice properties for hashing strings. However, different hash functions
could be used if they are more convenient for your particular dataset.

#### Bloom Filters

![Bloom filter](figures/14.svg)

##### Figure 3. Bloom filter

Bloom filters excel at easily distinguishing if a particular item has
been seen before. In addition, advanced versions of the algorithms give
extra features like scalability and time windowing. This can be useful
when making a caching layer or when simply asking whether you have seen
a particular user within a certain amount of time.

Sometimes we need to be able to do other types of set operations, for
which we need to introduce new types of probabilistic data structures.
Bloom filters <span class="footnote">  
\[Bloom, B. H. "Space/time trade-offs in hash coding with allowable
errors." *Communications of the ACM* 13:7 (1970): 422-426.
doi:10.1145/362686.362692.\]  
</span> were created to answer the question of whether we’ve seen an
item before.

Bloom filters work by having multiple hash values in order to represent
a value as multiple integers. If we later see something with the same
set of integers, we can be reasonably confident that it is the same
value.

In order to do this in a way that efficiently utilizes available
resources, we implicitly encode the integers as the indexes of a list.
This could be thought of as a list of <span
class="monospaced">bool</span> values that are initially set to <span
class="monospaced">False</span>. If we are asked to add an object with
hash values <span class="monospaced">\[10, 4, 7\]</span>, then we set
the tenth, fourth, and seventh indexes of the list to <span
class="monospaced">True</span>. In the future, if we are asked if we
have seen a particular item before, we simply find its hash values and
check if all the corresponding spots in the <span
class="monospaced">bool</span> list are set to <span
class="monospaced">True</span>.

This method gives us no false negatives and a controllable rate of false
positives. What this means is that if the Bloom filter says we have not
seen an item before, then we can be 100% sure that we haven’t seen the
item before. On the other hand, if the Bloom filter states that we
*have* seen an item before, then there is a probability that we actually
have not and we are simply seeing an erroneous result. This erroneous
result comes from the fact that we will have hash collisions, and
sometimes the hash values for two objects will be the same even if the
objects themselves are not the same. However, in practice Bloom filters
are set to have error rates below 0.5%, so this error can be acceptable.

We can simulate having as many hash functions as we want simply by
having two hash functions that are independent of each other. This
method is called "double hashing." If we have a hash function that gives
us two independent hashes, we can do:

    def multi_hash(key, num_hashes):
        hash1, hash2 = hashfunction(key)
        for i in xrange(num_hashes):
            yield (hash1 + i * hash2) % (2^32 - 1)

The modulo ensures that the resulting hash values are 32 bit (we would
modulo by <span class="monospaced">2^64 - 1</span> for 64-bit hash
functions).

The exact length of the <span class="monospaced">bool</span> list and
the number of hash values per item we need will be fixed based on the
capacity and the error rate we require. With some reasonably simple
statistical arguments <span class="footnote">  
\[The Wikipedia page on Bloom filters has a very simple proof for the
properties of a Bloom filter; see
<http://en.wikipedia.org/wiki/Bloom_filter#Probability_of_false_positives>.\]  
</span> we see that the ideal values are:

\begin{array}{rcl} num\\\_bits & = & -capacity \cdot
\frac{log(error)}{log(2)^2} \\\\ num\\\_hashes & = & num\_bits \cdot
\frac{log(2)}{capacity} \end{array}

That is to say, if we wish to store 50,000 objects (no matter how big
the objects themselves are) at a false positive rate of 0.05% (that is
to say, 0.05% of the times we say we have seen an object before, we
actually have not), it would require 791,015 bits (0.7Mb) of storage and
11 hash functions.

To further improve our efficiency in terms of memory use, we can use
single bits to represent the <span class="monospaced">bool</span> values
(a native <span class="monospaced">bool</span> actually takes 4 bits).
We can do this easily by using the <span
class="monospaced">bitarray</span> module.

##### Example 5. Simple Bloom filter implementation

    import bitarray
    import math
    import mmh3

    class BloomFilter(object):
        def __init__(self, capacity, error=0.005):
            """
            Initialize a bloom filter with given capacity and false positive rate
            """
            self.capacity = capacity
            self.error = error
            self.num_bits = int(-capacity * math.log(error) / math.log(2)**2) + 1
            self.num_hashes = int(self.num_bits * math.log(2) / float(capacity)) + 1
            self.data = bitarray.bitarray(self.num_bits)

        def _indexes(self, key):
            h1, h2 = mmh3.hash64(key)
            for i in xrange(self.num_hashes):
                yield (h1 + i * h2) % self.num_bits

        def add(self, key):
            for index in self._indexes(key):
                self.data[index] = True

        def __contains__(self, key):
            return all(self.data[index] for index in self._indexes(key))

        def __len__(self):
            num_bits_on = self.data.count(True)
            return -1.0 * self.num_bits * math.log(1.0 - num_bits_on / float(self.num_bits)) / float(self.num_hashes)

        @staticmethod
        def union(bloom_a, bloom_b):
            assert bloom_a.capacity == bloom_b.capacity, "Capacities must be equal"
            assert bloom_a.error == bloom_b.error, "Error rates must be equal"

            bloom_union = BloomFilter(bloom_a.capacity, bloom_a.error)
            bloom_union.data = bloom_a.data | bloom_b.data
            return bloom_union

What happens if we insert more items than we specified for the capacity
of the Bloom filter? At the extreme end, all the items in the <span
class="monospaced">bool</span> list will be set to <span
class="monospaced">True</span>, in which case we say that we have seen
every item. This means that Bloom filters are very sensitive to what
their initial capacity was set to, which can be quite aggravating if we
are dealing with a set of data whose size is unknown (for example, a
stream of data).

One way of dealing with this is to use a variant of Bloom filters called
*scalable Bloom filters*. <span class="footnote">  
\[Almeida, P. S., Baquero, C., Preguiça, N., and Hutchison, D. "Scalable
Bloom Filters." *Information Processing Letters* 101 (2007): 255–261.
doi:10.1016/j.ipl.2006.10.007.\]  
</span> They work by chaining together multiple Bloom filters whose
error rates vary in a specific way. <span class="footnote">  
\[The error values actually decrease like the geometric series. This
way, when you take the product of all the error rates it approaches the
desired error rate.\]  
</span> By doing this, we can guarantee an overall error rate and simply
add a new Bloom filter when we need more capacity. In order to check if
we’ve seen an item before, we simply iterate over all of the sub-Blooms
until either we find the object or we exhaust the list. A sample
implementation of this structure can be seen in
<a href="#memory_scaling_bloom"
class="doc_link">memory_scaling_bloom</a>, where we use the previous
Bloom filter implementation for the underlying functionality and have a
counter to simplify knowing when to add a new Bloom.

Another way of dealing with this is using a method called timing Bloom
filters. <span class="footnote">  
\[<http://github.com/mynameisfiber/fuggetaboutit>\]  
</span> This variant allows elements to be expired out of the data
structure, thus freeing up space for more elements. This is especially
nice for dealing with streams, since we can have elements expire after,
say, an hour and have the capacity be large enough to deal with the
amount of data we see per hour. Using a Bloom filter this way would give
us a nice view into what has been happening in the last hour.

##### Example 6. Simple scaling Bloom filter implementation

    from bloomfilter import BloomFilter

    class ScalingBloomFilter(object):
        def __init__(self, capacity, error=0.005, max_fill=0.8, error_tightening_ratio=0.5):
            self.capacity = capacity
            self.base_error = error
            self.max_fill = max_fill
            self.items_until_scale = int(capacity * max_fill)
            self.error_tightening_ratio = error_tightening_ratio
            self.bloom_filters = []
            self.current_bloom = None
            self._add_bloom()

        def _add_bloom(self):
            new_error = self.base_error * self.error_tightening_ratio ** len(self.bloom_filters)
            new_bloom = BloomFilter(self.capacity, new_error)
            self.bloom_filters.append(new_bloom)
            self.current_bloom = new_bloom
            return new_bloom

        def add(self, key):
            if key in self:
                return True
            self.current_bloom.add(key)
            self.items_until_scale -= 1
            if self.items_until_scale == 0:
                bloom_size = len(self.current_bloom)
                bloom_max_capacity = int(self.current_bloom.capacity * self.max_fill)

                # We may have been adding many duplicate values into the bloom, so
                # we need to check if we actually need to scale or if we still have
                # space
                if bloom_size >= bloom_max_capacity:
                    self._add_bloom()
                    self.items_until_scale = bloom_max_capacity
                else:
                    self.items_until_scale = int(bloom_max_capacity - bloom_size)
            return False

        def __contains__(self, key):
            return any(key in bloom for bloom in self.bloom_filters)

        def __len__(self):
            return sum(len(bloom) for bloom in self.bloom_filters)

Using this data structure will feel much like using a <span
class="monospaced">set</span> object. Below we use a scalable Bloom
filter to add several objects, test if we’ve seen them before, and then
try to experimentally find the false positive rate:

    >>> bloom = BloomFilter(100)

    >>> for i in xrange(50):
       ....:     bloom.add(str(i))
       ....:

    >>> "20" in bloom
    True

    >>> "25" in bloom
    True

    >>> "51" in bloom
    False

    >>> num_false_positives = 0

    >>> num_true_negatives = 0

    >>> # None of the following numbers should be in the Bloom.
    >>> # If one is found in the Bloom, it is a false positive.
    >>> for i in xrange(51,10000):
       ....:     if str(i) in bloom:
       ....:         num_false_positives += 1
       ....:     else:
       ....:         num_true_negatives += 1
       ....:

    >>> num_false_positives
    54

    >>> num_true_negatives
    9895

    >>> false_positive_rate = num_false_positives / float(10000 - 51)

    >>> false_positive_rate
    0.005427681173987335

    >>> bloom.error
    0.005

We can also do unions with Bloom filters in order to join multiple sets
of items. One caveat with this is that you can only take the union of
two Blooms with the same capacity and error rate. Furthermore, the final
Bloom’s used capacity can be as high as the sum of the used capacities
of the two Blooms unioned to make it. What this means is that you could
start with two Bloom filters that are a little more than half full and,
when you union them together, get a new Bloom that is over capacity and
not reliable!

    >>> bloom_a = BloomFilter(200)

    >>> bloom_b = BloomFilter(200)

    >>> for i in xrange(50):
       ...:     bloom_a.add(str(i))
       ...:

    >>> for i in xrange(25,75):
       ...:     bloom_b.add(str(i))
       ...:

    >>> bloom = BloomFilter.union(bloom_a, bloom_b)

    >>> "51" in bloom_a # <1>
    Out[9]: False

    >>> "24" in bloom_b # <2>
    Out[10]: False

    >>> "55" in bloom # <3>
    Out[11]: True

    >>> "25" in bloom
    Out[12]: True

1.  The value of <span class="monospaced">"51"</span> is not in <span
    class="monospaced">bloom\_a</span>.

2.  Similarly, the value of <span class="monospaced">"24"</span> is not
    in <span class="monospaced">bloom\_b</span>.

3.  However, the <span class="monospaced">bloom</span> object contains
    all the objects in both <span class="monospaced">bloom\_a</span> and
    <span class="monospaced">bloom\_b</span>!

#### LogLog Counters

LogLog counters, particularly HyperLogLog++, offer the best efficiency
when counting the size of a set or taking the union of sets. They don’t,
however, do well at intersections. They can be useful, for example, when
counting unique users by geography and allowing for accumulations based
on different geographic combinations. In general, LogLog-type algorithms
are very versatile, and there are even adaptations that can temporally
window your data.

LogLog-type counters <span class="footnote">  
\[<http://algo.inria.fr/flajolet/Publications/DuFl03-LNCS.pdf>\]  
</span> are based on the realization that the individual bits of a hash
function can also be considered to be random. That is to say, the
probability of the first bit of a hash being <span
class="monospaced">1</span> is 50%, the probability of the first two
bits being <span class="monospaced">01</span> is 25%, and the
probability of the first three bits being <span
class="monospaced">001</span> is 12.5%. Knowing these probabilities, and
keeping the hash with the most \`0\`s at the beginning (i.e., the least
probable hash value), we can come up with an estimate of how many items
we’ve seen so far.

A good analogy for this method is flipping coins. Imagine we would like
to flip a coin 32 times and get heads every time. The number 32 comes
from the fact that we are using 32-bit hash functions. If we flip the
coin once and it comes up tails, then we will store the number <span
class="monospaced">0</span>, since our best attempt yielded 0 heads in a
row. Since we know the probabilities behind this coin flip, we can also
tell you that our longest series was <span class="monospaced">0</span>
long and you can estimate that we’ve tried this experiment <span
class="monospaced">2^0 = 1</span> time. If we keep flipping the coin and
we’re able to get 10 heads before getting a tail, then we would store
the number <span class="monospaced">10</span>. Using the same logic, you
could estimate that we’ve tried the experiment <span
class="monospaced">2^10 = 1024</span> times. With this system, the
highest we could count would be the maximum number of flips we consider
(for 32 flips, this is <span class="monospaced">2^32 =
4,294,967,296</span>).

In order to encode this logic with LogLog-type counters, we take the
binary representation of the hash value of our input and see how many
<span class="monospaced">0</span> s there are before we see our first
<span class="monospaced">1</span>. The hash value can be thought of as a
series of 32 coin flips, where <span class="monospaced">0</span> means a
flip for heads and <span class="monospaced">1</span> means a flip for
tails (i.e., <span class="monospaced">000010101101</span> means we
flipped 4 heads before our first tails and <span
class="monospaced">010101101</span> means we flipped 1 head before
flipping our first tail). This gives us an idea of how many attempts
happened before this hash value was gotten. The mathematics behind this
system are almost equivalent to those of the Morris counter, with one
major exception: the "random" values are acquired by looking at the
actual input instead of using a random number generator. This means that
if we keep adding the same value to a LogLog counter its internal state
will not change.

##### Example 7. Simple implementation of LogLog register

    import mmh3

    def trailing_zeros(number):
        """
        Returns the index of the first bit set to 1 from the right side of a 32-bit
        integer
        >>> trailing_zeros(0)
        32
        >>> trailing_zeros(0b1000)
        3
        >>> trailing_zeros(0b10000000)
        7
        """
        if not number:
            return 32
        index = 0
        while (number >> index) & 1 == 0:
            index += 1
        return index

    class LogLogRegister(object):
        counter = 0
        def add(self, item):
            item_hash = mmh3.hash(str(item))
            return self._add(item_hash)

        def _add(self, item_hash):
            bit_index = trailing_zeros(item_hash)
            if bit_index > self.counter:
                self.counter = bit_index

        def __len__(self):
            return 2**self.counter

The biggest drawback of this method is that we may get a hash value that
increases the counter right at the beginning and skews our estimates.
This would be similar to flipping 32 tails on the first try. In order to
remedy this, we should have many people flipping coins at the same time
and combine their results. The law of large numbers tells us that as we
add more and more flippers, the total statistics become less affected by
anomalous samples from individual flippers. The exact way that we
combine the results is the root of the difference between LogLog-type
methods (classic LogLog, SuperLogLog, HyperLogLog, HyperLogLog++, etc.).

This "multiple flipper" method can be accomplished by taking the first
couple of bits of a hash value and using that to designate which of our
flippers had that particular result. If we take the first 4 bits of the
hash, this means we have <span class="monospaced">2^4 = 16</span>
flippers. Since we used the first 4 bits for this selection, we only
have 28 bits left (corresponding to 28 individual coin flips per coin
flipper), meaning each counter can only count up to <span
class="monospaced">2^28 = 268,435,456</span>. In addition, there is a
constant (alpha) that depends on the number of flippers there are, which
normalizes the estimation. <span class="footnote">  
\[A full description of the basic LogLog and SuperLogLog algorithms can
be found at <http://algo.inria.fr/flajolet/Publications/DuFl03.pdf>.\]  
</span> All of this together gives us an algorithm with 1.05 / \sqrt(m)
accuracy, where *m* is the number of registers (or flippers) used.

##### Example 8. Simple implementation of LogLog

    from llregister import LLRegister
    import mmh3

    class LL(object):
        def __init__(self, p):
            self.p = p
            self.num_registers = 2**p
            self.registers = [LLRegister() for i in xrange(int(2**p))]
            self.alpha = 0.7213 / (1.0 + 1.079 / self.num_registers)

        def add(self, item):
            item_hash = mmh3.hash(str(item))
            register_index = item_hash & (self.num_registers - 1)
            register_hash = item_hash >> self.p
            self.registers[register_index]._add(register_hash)

        def __len__(self):
            register_sum = sum(h.counter for h in self.registers)
            return 2 ** (float(register_sum) / self.num_registers) * self.num_registers * self.alpha

In the <span class="monospaced">\_\_len\_\_</span> method, we are
averaging the estimates from all of the individual LogLog registers.
This, however, is not the most efficient way to combine the data! This
is because we may get some unfortunate hash values that make one
particular register spike up while the others are still at low values.
Because of this, we are only able to achieve an error rate of
O(\frac{1.30}{\sqrt{m}}), where *m* is the number of registers used.

SuperLogLog <span class="footnote">  
\[Durand, M., and Flajolet, P. "LogLog Counting of Large Cardinalities."
*Proceedings of ESA*, 2832 (2003): 605-617.
doi:10.1007/978-3-540-39658-1\_55.\]  
</span> was devised as a fix to this problem. With this algorithm, only
the lowest 70% of the registers were used for the size estimate, and
their value was limited by a maximum value given by a restriction rule.
This addition decreased the error rate to O(\frac{1.05}{\sqrt{m}}). This
was counterintuitive, since we got a better estimate by disregarding
information!

Finally, HyperLogLog <span class="footnote">  
\[Flajolet, P., Fusy, É, Gandouet, O., et al. "HyperLogLog: The analysis
of a near-optimal cardinality estimation algorithm." *Proceedings of the
International Conference on Analysis of Algorithms* (2007): 127–146.\]  
</span> came out in 2007 and gave us further accuracy gains. This was
done simply by changing the method of averaging the individual
registers: instead of simply averaging, we use a spherical averaging
scheme <span class="footnote">  
\[Spherical averaging is simply a more complicated statistical measure
that relates to the normal average.\]  
</span> that also has special considerations for different edge cases
the structure could be in. This brings us to the current best error rate
of O(\frac{1.04}{\sqrt{m}}). In addition, this formulation removes a
sorting operation that is necessary with SuperLogLog. This can greatly
speed up the performance of the data structure when trying to insert
items at a high volume.

##### Example 9. Simple implementation of HyperLogLog

    # the following import imports our previously defined LogLog register
    from ll import LL
    import math

    class HyperLogLog(LL):
        def __len__(self):
            indicator = sum(2**-m.counter for m in self.registers)
            E = self.alpha * (self.num_registers**2) / float(indicator)

            if E <= 5.0 / 2.0 * self.num_registers:
                V = sum(1 for m in self.registers if m.counter == 0)
                if V != 0:
                    Estar = self.num_registers * math.log(self.num_registers / (1.0 * V), 2)
                else:
                    Estar = E
            else:
                if E <= 2**32 / 30.0:
                    Estar = E
                else:
                    Estar = -2**32 * math.log(1 - E / 2**32, 2)
            return Estar

    if __name__ == "__main__":
        import mmh3
        hll = HyperLogLog(8)
        for i in xrange(100000):
            hll.add(mmh3.hash(str(i)))
        print len(hll)

The only further increase in accuracy was given by the HyperLogLog++
algorithm, which increased the accuracy of the data structure while it
is relatively empty. When more items are inserted, this scheme reverts
to standard HyperLogLog. This is actually quite useful, since the
statistics of the LogLog-type counters require a lot of data to be
accurate — having a scheme for allowing better accuracy with fewer items
greatly improves the usability of this method. This extra accuracy is
achieved by having a smaller but more accurate HyperLogLog structure
that can be later converted into the larger structure that was
originally requested. Also, there are some imperially derived constants
that are used in the size estimates that remove biases.

#### Composite Structures

It is important to note that the above structures are simply the
building blocks for more complicated algorithms and methods. We can put
these tools together in different ways in order to achieve various
results.

For example, one common problem we chose to solve in the prototype was
identifying which of several thousand phrases occurs within a given
piece of text. To make matters worse, this calculation needed to happen
very quickly, since we were parsing many messages per second.

The resulting solution was a hierarchy of Bloom filters arranged in such
a way as to efficiently implement a Rabin–Karp string search algorithm.
This gave us the low memory use of a Bloom filter and the efficiency of
an optimized string searching algorithm. In the end, we were able to
search through 1,000 words for one of 16,000 variable-length keywords in
under a millisecond on commodity hardware.

    from bloomfilter import BloomFilter
    from itertools import ifilter


    class MultigramSearch(object):
        def __init__( self, ngrams, delimiter='##', stop='$$',
                error=0.0001, error_tightening_ratio=0.5):
            self.blooms = []
            self.error = error
            self.error_tightening_ratio = error_tightening_ratio
            self.min_ngram = min(len(d) for d in ngrams) or 1
            self.max_ngram = max(len(d) for d in ngrams)
            self.delimiter = delimiter
            self.STOP = stop
            self._build_structure(ngrams)

        def _build_structure(self, ngrams):
            delimiter = self.delimiter
            STOP = self.STOP
            for i, n in enumerate(xrange(self.min_ngram, self.max_ngram+1)):
                num_items = sum(1 for x in ngrams if len(x) >= n)
                # we tighten the error so that the compounded error converges to
                # the desired error
                cur_error = self.error * (self.error_tightening_ratio ** i)
                bloom = BloomFilter(num_items, error=cur_error)
                for item in ifilter(None, ngrams):
                    if len(item) >= n:
                        bloom.add(delimiter.join(item[:n]))
                    elif len(item) + 1 == n:
                        bloom.add(delimiter.join(item) + STOP)
                self.blooms.append(bloom)

        def intersection(self, text):
            i = 0
            offset = self.min_ngram
            L = len(text) - offset
            delimiter = self.delimiter
            while i <= L:
                for N, bloom in enumerate(self.blooms):
                    # check if the current substring is in the bloom filter without
                    # the STOP sequence -- this would mean we have a partial ngram
                    # match
                    test = delimiter.join(text[i:i+N+offset])
                    if test not in bloom:
                        if N > 0:
                            # now we check if the current bloom has the substring
                            # with the STOP sequence appended to it
                            new_test = delimiter.join(text[i:i+N+offset-1])
                            if (new_test + self.STOP) in bloom:
                                yield text[i:i+N+offset-1]
                        break
                i += N + 1

#### Real-World Example

For a better understanding of the data structures, we first created a
dataset with many unique keys, and then one with duplicate entries.
<a href="#FIG-prob-ds-comparison"
class="doc_link">FIG-prob-ds-comparison</a> shows the results when we
feed these keys into the data structures we’ve just looked at and
periodically query, "How many unique entries have there been?"

Probabilistic data structures are about guarantees — once you know the
questions you’re asking and the computational constraints, you can pick
the structure that makes the right guarantees for your situation.

##### Example 10. Comparison between various probabilistic data structures for unique (above) and repeating (below) data

![PDS with unique data](figures/10_unique.png)

![PDS with repeating data](figures/10_rep.png)

We can see that the data structures that contain more stateful variables
(such as HyperLogLog and K-Min Values) do better, since they more
robustly handle bad statistics. On the other hand, the Morris counter
and the single LogLog register can quickly have very high error rates if
one unfortunate random number or hash value occurs. For most of the
algorithms, however, we know that the number of stateful variables is
directly correlated with the error guarantees, so this makes sense.

Looking just at the probabilistic data structures that have the best
performance (and really, the ones you will probably use), we can
summarize their utility and their approximate memory usage (see
<a href="#memory_pd_comparison"
class="doc_link">memory_pd_comparison</a>). We can see a huge change in
memory usage depending on the questions we care to ask. This simply
highlights the fact that when using a probabilistic data structure, you
must first consider what questions you really need to answer about the
dataset before proceeding. Also note that only the Bloom filter’s size
depends on the number of elements. The HyperLogLog and K-Min Values’s
sizes are *only* dependent on the error rate.

##### Example 11. Comparison of major probabilistic data structures

<table class="tableblock frame-topbot grid-all" style="
width:100%;
">
<colgroup>
<col style="width: 16%" />
<col style="width: 16%" />
<col style="width: 16%" />
<col style="width: 16%" />
<col style="width: 16%" />
<col style="width: 16%" />
</colgroup>
<thead>
<tr class="header">
<th class="tableblock halign-left valign-top"></th>
<th class="tableblock halign-left valign-top">Size</th>
<th class="tableblock halign-left valign-top">Union <span
class="footnote"><br />
[Union operations occur without increasing the error rate.]<br />
</span></th>
<th class="tableblock halign-left valign-top">Intersection</th>
<th class="tableblock halign-left valign-top">Contains</th>
<th class="tableblock halign-left valign-top">Size <span
class="footnote"><br />
[Size of data structure with 0.05% error rate, 100,000,000 unique
elements, and using a 64-bit hashing function.]<br />
</span></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td class="tableblock halign-left valign-top"><p>HyperLogLog</p></td>
<td class="tableblock halign-left valign-top"><p>Yes
(O(\frac{1.04}{\sqrt{m}}))</p></td>
<td class="tableblock halign-left valign-top"><p>Yes</p></td>
<td class="tableblock halign-left valign-top"><p>No <span
id="_footnote_memory_caveat" class="footnote"><br />
[These operations <em>can</em> be done]<br />
</span></p></td>
<td class="tableblock halign-left valign-top"><p>No</p></td>
<td class="tableblock halign-left valign-top"><p>2.704 MB</p></td>
</tr>
<tr class="even">
<td class="tableblock halign-left valign-top"><p>K-Min Values</p></td>
<td class="tableblock halign-left valign-top"><p>Yes
(O(\sqrt{\frac{2}{\pi(m-2)}}))</p></td>
<td class="tableblock halign-left valign-top"><p>Yes</p></td>
<td class="tableblock halign-left valign-top"><p>Yes</p></td>
<td class="tableblock halign-left valign-top"><p>No</p></td>
<td class="tableblock halign-left valign-top"><p>20.372 MB</p></td>
</tr>
<tr class="odd">
<td class="tableblock halign-left valign-top"><p>Bloom filter</p></td>
<td class="tableblock halign-left valign-top"><p>Yes
(O(\frac{0.78}{\sqrt{m}}))</p></td>
<td class="tableblock halign-left valign-top"><p>Yes</p></td>
<td class="tableblock halign-left valign-top"><p>No <span
class="footnoteref"><br />
<a href="#_footnote_memory_caveat">[memory_caveat]</a><br />
</span></p></td>
<td class="tableblock halign-left valign-top"><p>Yes</p></td>
<td class="tableblock halign-left valign-top"><p>197.8 MB</p></td>
</tr>
</tbody>
</table>

As another, more realistic test, we chose to use a dataset derived from
the text a partial dump of Wikipedia. This set contains 8,545,076 unique
tokens from a portion of the English Wikipedia site and takes up 111 MB
on disk. We ran a very simple script in order to extract all single-word
tokens with five or more characters from the dataset and store them in a
newline-separated file. The question then was, "How many unique tokens
are there?" The results can be seen in
<a href="#memory_pd_wiki_comparison"
class="doc_link">memory_pd_wiki_comparison</a>. In addition, we
attempted to answer the same question using a trie structure (this trie
was chosen as opposed to the others because it offers good compression
while still being robust enough to deal with the entire dataset).

##### Example 12. Size estimates for the number of unique words in Wikipedia

<table class="tableblock frame-topbot grid-all" style="
width:100%;
">
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th class="tableblock halign-left valign-top"></th>
<th class="tableblock halign-left valign-top">Elements</th>
<th class="tableblock halign-left valign-top">Relative error</th>
<th class="tableblock halign-left valign-top">Processing time <span
class="footnote"><br />
[Processing time has been adjusted to remove the time required to read
the dataset from disk. We also use the simple implementations provided
earlier for testing.]<br />
</span></th>
<th class="tableblock halign-left valign-top">Structure size <span
class="footnote"><br />
[Structure size is theoretical given the amount of data since the
implementations used were not optimized.]<br />
</span></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td class="tableblock halign-left valign-top"><p>Morris counter <span
class="footnote"><br />
[Since the Morris counter doesn’t deduplicate input, the size and
relative error are given with regard to the total number of
values.]<br />
</span></p></td>
<td class="tableblock halign-left valign-top"><p>1,073,741,824</p></td>
<td class="tableblock halign-left valign-top"><p>6.52%</p></td>
<td class="tableblock halign-left valign-top"><p>751s</p></td>
<td class="tableblock halign-left valign-top"><p>5 bits</p></td>
</tr>
<tr class="even">
<td class="tableblock halign-left valign-top"><p>LogLog
register</p></td>
<td class="tableblock halign-left valign-top"><p>1,048,576</p></td>
<td class="tableblock halign-left valign-top"><p>78.84%</p></td>
<td class="tableblock halign-left valign-top"><p>1,690 s</p></td>
<td class="tableblock halign-left valign-top"><p>5 bits</p></td>
</tr>
<tr class="odd">
<td class="tableblock halign-left valign-top"><p>LogLog</p></td>
<td class="tableblock halign-left valign-top"><p>4,522,232</p></td>
<td class="tableblock halign-left valign-top"><p>8.76%</p></td>
<td class="tableblock halign-left valign-top"><p>2,112 s</p></td>
<td class="tableblock halign-left valign-top"><p>41 KB</p></td>
</tr>
<tr class="even">
<td class="tableblock halign-left valign-top"><p>HyperLogLog</p></td>
<td class="tableblock halign-left valign-top"><p>4,983,171</p></td>
<td class="tableblock halign-left valign-top"><p>-0.54%</p></td>
<td class="tableblock halign-left valign-top"><p>2,907 s</p></td>
<td class="tableblock halign-left valign-top"><p>40 KB</p></td>
</tr>
<tr class="odd">
<td class="tableblock halign-left valign-top"><p>K-Min Values</p></td>
<td class="tableblock halign-left valign-top"><p>4,912,818</p></td>
<td class="tableblock halign-left valign-top"><p>0.88%</p></td>
<td class="tableblock halign-left valign-top"><p>3,503 s</p></td>
<td class="tableblock halign-left valign-top"><p>256 KB</p></td>
</tr>
<tr class="even">
<td class="tableblock halign-left valign-top"><p>Scaling Bloom</p></td>
<td class="tableblock halign-left valign-top"><p>4,949,358</p></td>
<td class="tableblock halign-left valign-top"><p>0.14%</p></td>
<td class="tableblock halign-left valign-top"><p>10,392 s</p></td>
<td class="tableblock halign-left valign-top"><p>11,509 KB</p></td>
</tr>
<tr class="odd">
<td class="tableblock halign-left valign-top"><p>Datrie</p></td>
<td class="tableblock halign-left valign-top"><p>4,505,514 <span
class="footnote"><br />
[Because of some encoding problems, the datrie could not load all the
keys.]<br />
</span></p></td>
<td class="tableblock halign-left valign-top"><p>0.00%</p></td>
<td class="tableblock halign-left valign-top"><p>14,620 s</p></td>
<td class="tableblock halign-left valign-top"><p>114,068 KB</p></td>
</tr>
</tbody><tfoot>
<tr class="even">
<td class="tableblock halign-left valign-top"><p>True value</p></td>
<td class="tableblock halign-left valign-top"><p>4,956,262</p></td>
<td class="tableblock halign-left valign-top"><p>0.00%</p></td>
<td class="tableblock halign-left valign-top"><p>-----</p></td>
<td class="tableblock halign-left valign-top"><p>49,558 KB <span
class="footnote"><br />
[The dataset is 49,558 KB considering only unique tokens, or 8.742 GB
with all tokens.]<br />
</span></p></td>
</tr>
</tfoot>

</table>

The major takeaway from this experiment is that if you are able to
specialize your code, you can get amazing speed and memory gains.
Probabilistic data structures are an algorithmic way of specializing
your code. We store only the data we need in order to answer specific
questions with given error bounds. By only having to deal with a subset
of the information given, not only can we make the memory footprint much
smaller, but we can also perform most operations over the structure
faster (as can be seen with the insertion time into the datrie in
<a href="#memory_pd_wiki_comparison"
class="doc_link">memory_pd_wiki_comparison</a> being larger than with
any of the probabilistic data structures).

As a result, whether or not you use probabilistic data structures, you
should always keep in mind what questions you are going to be asking of
your data and how you can most effectively store that data in order to
ask those specialized questions. This may come down to using one
particular type of list over another, using one particular type of
database index over another, or maybe even using a probabilistic data
structure to throw out all but the relevant data!

------------------------------------------------------------------------

Last updated 2022-08-22 08:56:03 PDT
