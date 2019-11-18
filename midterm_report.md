# Midterm Report

## Resources

Group Members:

* Karthik Duddu (sduddu@cs.cmu.edu)
* Venkat K Srinivasan (venkatks@cs.cmu.edu)

URL: [Midterm Report Hyperlink](https://venkatks.github.io/parallelagent/midterm_report)

## Summary Of Work Completed So Far

The work we have finished so far can be summaried into three different
categories. They include a finalization of the preliminary outline of the 
parallel API we want to provide the users of our package, a sequential 
implementation of a simple GridWorld example that have created using our own
environment package, as well as a clarification of what our project deliverables
will likely be at the end of the project timeline.

### Finalization Of The Preliminary Outline

One of the deliverables we wanted to complete by the midterm point is a 
API design that will allow us to offer a full-featured interface for 
GridWorld examples that utilize the package we're offering to build ontop of.

We have completed such an interface, and have provided abstract classes within
the core of the package that developers can extend to implement the environment
that they want. Ultimately, the backend implementations within this abstract
class hierarchy will actually implement the parallelism framework that will 
allow the efficient parallelization of the developer-specified agent's 
interactions, including the rich data and control dependendencies that might 
arise due to the rich interplay and interactions between each of the agents
among themselves and with the environment.

In brief, we have devised a 5-stage pipeline which deals with data dependencies
between agent actions while exploiting both available axes of parallelism -
agent actions and environment updates. That is,

* _State dependencies between agents_: Actions of a particular agent can influence
another agent. For instance, an action such as `A1 -> Kill A2` affects A2 but has
been issued to A1. In order to deal with the agent state dependencies created 
by actions, we decouple actions and action updates in the interface. This implies
that our framework has to perform the per-agent action update mapping on the
basis of the output of the agents.

* _Data dependencies between agent actions and map updates_: In order to efficiently
deal with map updates by agents (which could potentially have ordering constraints)
we provide a means of aggregation for related updates via `tags`, thereby
enabling unrelated map updates to be performed efficiently.

At a high level, our pipeline consists of the following steps:

1. Perform agent state checks and agent action effect generation (_parallelized
over agents_). Agent-specific logic is supplied by the environment designer.
2. Aggregate action-effects for each agent (_parallelized over agents_). This is
provided by our framework, and allows decoupling of agents affecting each other.
3. Perform agent state update and generate map updates based on all the supplied
actions effects (_parallelized over agents_). The specific update is a part of
the API implemented by the designer.
4. Aggregate map updates for each 'unit' of the map (_parallelized over updates_).
This is provided by our framework, and allows unrelated updates to be performed
in parallel.
5. Perform map updates (_paralellized over updates_).
6. Calculate reward received by each agent (_parallelized over agents_).

The sequential implementation has all the above steps implemented. The core 
packages that we have completed so far can be found under the following link [2].

### A Sequential Implementation

One of the other deliverables that we wanted to complete by the midterm point
was a sequential implemetnation of a simple GridWorld example using entirely
our own packages and core abstractions. This has been completed, as well.

In this example, we create a simple GridWorld that has "food" pellets on the 
outskirts of the table and allow the multitude of agents that are spawned
randomly on the outskirts of the grid to find it. No two agents should
occupy the same gridtile at any point; this addition was to create control
and data dependencies as a way of testing our (soon to be created) parallel
version of the program.

### A Clarification Of The Project Deliverables

While initially we wanted to deliver a series of kernels that a user can use
to trivially parallelize their environment (on their end), akin to the
Spark98 kernels that were talked about in class, our goals have mutated a bit
to change the project to be a bit more exploratory.

We now want to reframe the paper by providing an exploratory analysis of the
different technologies that we've talked about in class, and compare and
contrast the performance metrics and performance improvements that we get from
each one of them.

In other words, we want to be implement the full API, and provide the users
with different underlying technologies that can used. For instance, the user
should be able to specify that they want the middleware to utilize OpenMP as a
way of parallelizing their environment, or MPI in the case of clusters (or even
things like ISPC). If time permits, we also want to provide the CUDA kernels, 
as well.

Ultimately, our goal is to utilize these implementations of different parallel
paradigms, ultiize many examples with different topologies of dependencies -- 
both control and data -- as examples that get built on top of our API, and 
compare which technologies perform the best on each category. We hope that this
would allow us to gain a deeper understanding of the fundamental differences
between these different parallel framework abstractions and the best practices
for applying them to different types of problems with different categories and
topologies of dependencies.


## Modifications of Deliverables

We hope to change what we're delivering a little bit, as well. In our 
preproposal report, we mentioned that we wanted to deliver a series of kernels,
similar to the Spark98 kernels, that will allow us offer developers a quick way
of parallelizing their multi-agent environments. While this remains one of our
stretch goals, we are shifting our focus into a more analytical approach rather
than purely focusing on CUDA. 

We want to utilize all of the technologies discussed in class this semester,
including OpenMP, OpenMPI, Pthreads (or fork-join frameworks) and possibly CUDA
(if time permits) into our middleware framework and allow the user to specify a
technology they want our middleware to use to parallelize their workload. 

In our report, we want to implement multiple examples, both GridWorld-based and
non-GridWorld based, that will have varying dependency toplogies that will allow
us to examine how efficient each of the aforementioned technologies are at 
parallelizing that particular format of workload and compre and constrast these
things in the report.

## Delivery At The Poster Session

At the poster session, we plan on discussing the topology of dependencies
of the various examples, as well as what technology works best for each one.
A discussion of why will be included, as well. We also plan on having a 
demo walkthrough of code that uses our technology, and how it makes parallelizing
multi-agent scenes simpler.

## Largest Concerns

One of the main concerns for us is regarding our stretch goals. We want to 
utilize CUDA kernels in our deliverables, as well, but due to the memory
hierarchy of CUDA, it is far more difficult to be able to parallelize any
arbitrary agent action under this paradigm.
