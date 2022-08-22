Table of Contents

**JavaScript must be enabled in your browser to display the table of
contents.**

## The Future

Probabilistic methods are putting our current algorithms in a new
light — rather than constantly trying to improve their speed by adding
more computing power, we can simply allow for a bounded amount of error
and get immense gains. This will allow us to move our analyses from the
large computing clusters in which they currently reside into the hands
of individuals.

![The future](figures/28.svg)

##### Figure 1. Faster and lower resource analysis will bring complex computation to user devices

One potentially very exciting application would be the ability to take
in all the sensor data we can get hold of and do very cheap calculations
with that data. For example, a fitness tracker could record all
available data and provide the user with insights into that data on the
device, without the need for a cloud-backed cluster. This is in stark
comparison to current approaches, where data is heavily truncated before
being sent to a centralized server in the cloud for processing and
analysis. The move toward fog computing, where processing, storage, and
analysis are done on consumer devices closer to the network edge brings
analysis and insights closer to where they should be — with the people
asking the questions — so the information is in the right place at the
right time. In addition, since the data being analyzed is hashed, the
original information is lost and the users can rest assured that their
information remains private.

Another point of interest is the ability of these algorithms to deal
with changes in the underlying dataset. This is incredibly important as
we collect more data, but still care about actionable information that
relies on what is happening *right now*. Incoming data may change
completely, and we can accommodate this by using time-windowed versions
of the algorithms or, if the change is fundamental to the data type, a
change in hash function. Windowed algorithms are not only beneficial in
allowing us to bound resource usage, but also provide enhanced
understanding of the data by "forgetting" older information and always
providing us with the most recent insights.

This ability to window data and, in general, to be able to peek inside
of the data structure’s internal state at any point to get the current
results is incredibly important in today’s dynamic world. Getting
results too late or ones that potentially incorporate data that is of no
use anymore lessens the potential impact our results can make. For
example, if we were analyzing traffic patterns on our website in
relation to changes we have made to content (through A/B testing, or if
new content has recently been made available), we would want to know
immediately what effect the changes have made. Knowing that most of the
readers of the new content also liked another piece of content, or are
also users of a particular site feature, or how they compare to other
users is extremely actionable information that can be used to greatly
enhance user participation and the user experience. Furthermore, in a
world where the vast majority of interaction with a new piece of content
happens in the first 15 minutes, <span class="footnote">  
\[<http://tinyurl.com/3o7zeds>\]  
</span> we do not have time to do anything but a very fast realtime
analysis — by the time a batch analysis is done it is already too late!

Finally, probabilistic streaming methods provide a cultural paradigm
shift for data analysts. Currently, their job is over once they have an
algorithm that is able to output the analysis that they wanted. These
methods, however, show themselves as the additional step of cementing
the insights they learned in their exploration into systems to make
people smarter by answering their questions as they are being asked.

### Ethics

One major advantage of probabilistic methods is that they discard the
original data in favor of an internal summary (also known as a "sketch"
of the original data, or a "synopsis"). This summarization is generally
created using a one-way hashing function that makes it almost impossible
to reproduce the original data. As a result, almost perfect privacy
regarding the original data is guaranteed.

Data breaches are becoming more and more common. Howev- er, if
aggregated hashed data is stolen, it’s not currently possi- ble to
reverse engineer the source data from the values stored by probabilistic
systems (and it’s unlikely to ever be possible with any sort of
precision).

For example, we can create a graph that compares cookies of people who
go to various websites. We can easily determine how similar the
audiences of various websites are, and even analyze the paths by which
users navigate through the various resources. However, we cannot give an
enumerated list of the cookies that are following these trends (only
information regarding how many of them each website has seen, how many
are shared between sites, and how many are different). This is the holy
grail of bulk analysis — while we can still perform the analysis we
require and give in-depth insights regarding user patterns, we cannot
identify the users, and we maintain their privacy. This is particularly
important when dealing with medical or student data, where laws mandate
a level of privacy that often makes analysis more cumbersome.

![Probabilistic data structures offer inherit privacy while still
allowing for interesting insights in the aggregate](figures/29.svg)

##### Figure 2. Probabilistic data structures offer inherit privacy while still allowing for interesting insights in the aggregate

This added anonymity is not only important for the trust of users but it
is also becoming more and more legally required as more and more nations
legislate for personal data privacy. In the European Union, the Data
Protection Directive and the General Data Protection Regulation limit
what sort of personally identifiable information may be collected, how
it can be used, and how it can cross borders. This has become such a
problem that new datacenters are being created in Lithuania, simply as a
place within EU borders to perform user analysis. Similarly, in the US
user data is protected under HIPAA and various state laws. This has also
led to special datacenters (most notably the Amazon AWS HIPAA-compliant
servers) specifically designed to easily create a space for compliant
analysis.

As a result, there is a fantastic compromise between the needs of
service users and organizations interested in deriving insights. Users
can provide completely anonymized data with mathematical assurances that
their original data cannot be recovered. However, the organizations are
still able to derive insights that they are interested in by using the
appropriate probabilistic algorithms.

------------------------------------------------------------------------

Last updated 2022-08-22 08:56:03 PDT
