## Introduction

Since the days of analog computers built on cams and gears,^[<https://www.youtube.com/watch?v=_8aH-M3PzM0>]
we’ve been engineering systems around the flow of data and the
critical calculations we must perform.

While the philosophy of our designs has remained consistent, our
engineering constraints are constantly evolving. In the past five years
we’ve seen the emergence of "big data," or the ability to use commodity
infrastructure to analyze very large data sets in a batch. We’re
currently in the midst of a significant step forward in the tools,
methods, and technologies available for working with realtime streams of
data.

![The sheer amount of data in realtime streams can overwhelm
conventional batch analysis architecture](figures/01.svg)

##### Figure 1. The sheer amount of data in realtime streams can overwhelm conventional batch analysis architecture

The amount of time, memory, and computation required to do even simple
operations on very large data sets can be extraordinary. In this report,
we explore *probabilistic methods for realtime stream analysis*. These
methods allow for quick and efficient calculations that approximate the
results that form a batch analysis.

![The Tricorder — powered by probabilistic methods?](figures/02.svg)

##### Figure 2. The Tricorder — powered by probabilistic methods?

It may sound like science fiction to be able to perform a task like
calculating the percentage overlap between two sets of billions of items
not only in milliseconds, but also with megabytes of memory. This
ability will not only reduce computers' resource needs while solving
problems on big data, but it will also allow smaller devices to use more
advanced algorithms. Tricorders from *Star Trek*, for example, work with
complex algorithms running directly on a small device with hundreds of
sensors — all this in order to answer people’s questions, in real time,
to make them smarter. The key to this is understanding not only the
computational and memory bounds of our algorithms, but also the bounds
to their accuracy that are needed.

A probabilistic approach to algorithms realizes that it is not scalable
to simply continually add more computational power to a system — at some
point, adding more power will give diminished returns. Instead, we must
find a way to make our algorithms better, which in many cases means
rephrasing the questions we are asking and accepting approximations. In
return for this compromise, we can answer incredibly complex questions
and only use a fraction of the resources we would have otherwise needed.

![Probabilistic methods create a summary of the data as it comes in,
allowing them to run faster and more efficiently](figures/03.svg)

##### Figure 3. Probabilistic methods create a summary of the data as it comes in, allowing them to run faster and more efficiently

Why would we accept a calculation that isn’t entirely precise? Because
it’s much, much faster! In our prototype, *CliqueStream*, we are able to
compare many very large sets in seconds when classically this
calculation would take dozens of minutes. Furthermore, although there is
*some* error to our calculations, it is bounded to 1.7%, which is more
than enough accuracy to understand the trends and signals and make
actionable decisions.

Furthermore, a problem like finding the similarity between all pairs of
items in a large set is not well adapted for many batch frameworks that
are currently in use. For example, Hadoop takes advantage of the
map-reduce paradigm of computing, where computation is brought to data
in order to split up and speed up tasks. However, given a long list of
items, when asking how all pairs of items compare to each other we no
longer have data locality, and any Hadoop-based solution will be working
hard against its core philosophy in order to proceed with the
calculation. It is simply the wrong tool for the job.

Realtime systems require a different approach to engineering than static
systems. Generally, designers start with offline research on data to
develop and validate the model that they wish to use. Once the engineers
have a good understanding of the model, they build the realtime system.
Finally, it is deployed and quality is monitored. Probabilistic
algorithms are an added set of tools to aid in the construction of these
realtime systems for when complex calculations must be done but time and
resources are a constraint.

These systems offer a competitive edge. In essence, they bring you
knowledge about the future faster than anyone else could possibly have
computed it. Furthermore, they allow for these calculations to be done
closer to the user. Because of their low resource and computational
needs, calculations can be done client-side (even on a user’s phone), as
opposed to first having to transmit potentially bulky data back and
forth to a cluster or cloud service before it is analyzed.

This changes our interaction with data and computation — instead of
interacting with an all-knowing cloud that is predesigned to solve a
specific set of questions, we can own and understand the data ourselves
with algorithms powerful enough to answer a wider range of questions in
a more timely manner. This is more of a "fog" of computing, where
potential insights surround us and can be gleaned at any time. In this
paradigm, instead of data being moved to servers powerful enough to run
an algorithm, the algorithm itself is moved to the data source to answer
questions as they are asked.

------------------------------------------------------------------------