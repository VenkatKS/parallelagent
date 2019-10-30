# Parallel Agent Simulator Proposal

## Summary

The goal of this project is to provide an efficient parallel implementation for multi-agent, single-environment simulation which interfaces with OpenAI Gym[6] and supports parallelized agent trajectories, while still allowing rich interactions between the agents. In addition to analyzing the trade-offs between our implementation and the current state of the art simulators, we intend to explore efficient ways of parallelization in the presence of data dependencies introduced by the agent interactions with the environment.

## Background

Multi-agent learning scenarios for reinforcement learning (RL) are becoming increasingly common[1] due to their applicability to domains such as collective intelligence and population studies[2]. Popular frameworks for dealing with RL environments have been restricted to single-agent frameworks[6], or have limited support for multi-agent environments[3], or require setting up (and understanding) non-trivial parallel architectures[4]. The aim of this project is to provide an efficient implementation for agent actions and environment updates, exposed via a simple API for multi-agent game environments, for scenarios in which agents and environments can be collocated.

The OpenAI Gym provides an API which allows RL algorithms to interface with an “environment” during training and inference. This environment is a simulator (akin to the n-body simulator that we worked with in class) within which one or more independent entities (“agents”) interact to cause changes. Since this environment is invoked millions of times per episode (during both training and inference), implementing such environments efficiently is important for improving throughput of the RL algorithm and for reducing developer turnaround time.

There are 2 axes of parallelism which are exposed by multi-agent environments: a) parallelization of agent simulation and interactions, and b) environment state updates on the basis of agent actions. The correctness of applying parallelization is dependent on the nature of the environment and agent interactions - for the purposes of this project, we shall restrict ourselves to games in which agents act simultaneously based on environment state information (Markovian games[5]). For such games, we can think of each agent as an "instruction stream" or a "thread", as each agent will be able to “undertake” independent actions that directly impact the shared space. Furthermore, we intend to target environments and agent populations which can be collocated on the same server. Further discussion of parallelism and the challenges associated with it are discussed in Section 3.

From a systems perspective, the goal of this project is to: a) analyze the nature of agents and environments which allow for parallelization along the different axes, and b) provide support for executing agent interactions on a GPU (via CUDA) and analyze the different communication/computation tradeoffs involved. A discussion of concrete goals and deliverables is provided in Section 4.

## The Challenge

This problem is not trivially parallelizable because we are allowing rich interactions between actors within the environment. This fundamentally introduces data and control dependencies between the agents, and thus the agents cannot be parallelized completely independently (there has to be a way to resolve these dependencies in an efficient manner). We hope to design an intelligent middleware that can allow for a large percentage of the overall workload involved in the rollout to be parallelized, while only sequentializing the parts of the workload that have direct dependencies and cannot be directly parallelized.

There will be little execution divergence as the problem leads itself towards SIMD instruction. Each agent will have the same code executing on each core, while having different starting states. Therefore, for a majority of the execution, we will be executing the same instructions on different data, thus allowing for good parallelization. We will only need to break this model whenever an agent builds a dependence on another agent due to an interaction between them. This is where the data and control dependencies arise. 

Further, updating the environment after all the agents finish their updates also requires added complexity. Since each of the agent actions were parallelized and resolved, we now have a vector of actions that we need to reduce to update the environment with. As the environment is a shared construct, making the necessary updates in parallel is a challenging problem. 

## Goals and Deliverables

### Baseline goals:

- Design an API for supporting parallelization of agents and environments. Clarify assumptions which need to be satisfied, and dependence on underlying parallel implementation (for instance, which assumptions are necessitated by CUDA and could be obviated by OpenMP).
- Implement a gridworld environment[3] with support for parallel agent updates.
- Compare performance against sequential and OpenMP implementation (latter provided by MAgent[3] framework) for one game within gridworld (such as a battle scenario between 2 armies of particles).

### Project goals:
- Implement an automatically generated tile-based game (this is suggested as benchmark environment by [4]).
- Implement parallelism for agents, and for environment updates.
- Analyze requirements and bottlenecks for both kinds of parallelism. Compare against sequential implementation (or possibly a simple MPI-based implementation).

### Challenge goals:
- Compare CUDA implementation to a tree-based OpenMP implementation (for instance, cluster agents by some interaction metric, compute updates in parallel, and reduce for the environment).

## Platform Choice

We will largely focus on CUDA for our implementation. As each agent is essentially the same class of code, but with different instance variables, this problem leads itself to SIMD execution. With CUDA, we can easily scale the problem’s agent simulation aspect to many millions by assigning each agent to a single thread. Also, as the second axis of parallelism in our problem requires us to reduce the changes into a single vector and apply them to the environment, the reduction-paradigm would work well with the underlying CUDA architecture. 

We will largely utilize Python and PyCUDA, as our contributions will be to OpenAI’s gym interface.

## Schedule

| Week Of      | Goal |
| ----------- | ----------- |
| October 28, 2019      | Finish meeting with professor, pre-proposal submissions, finalize problem statement       |
| November 4, 2019  | Finish design of basic API that will be external facing, implement initial sequential simple gridworld examples, decide what baseline metrics will be used to compare the sequential gridworld example with the parallel reimplementations, replicate gridworld examples from MAgent and collect metrics        |
| November 11, 2019  | Finish initial parallel gridworld examples using the new API, collect equivalent metrics for the new gridworld implementation, begin on midterm report.   |


## References

1. Hernandez-Leal, Pablo, Bilal Kartal, and Matthew E. Taylor. _"A survey and critique of multiagent deep reinforcement learning."_ Autonomous Agents and Multi-Agent Systems (2019): 1-48.
2. Yang, Yaodong, et al. _"A Study of AI Population Dynamics with Million-agent Reinforcement Learning."_ Proceedings of the 17th International Conference on Autonomous Agents and MultiAgent Systems. International Foundation for Autonomous Agents and Multiagent Systems, 2018.
3. Zheng, Lianmin, et al. _"MAgent: A many-agent reinforcement learning platform for artificial collective intelligence."_ Thirty-Second AAAI Conference on Artificial Intelligence. 2018.
4. Suarez, Joseph, et al. _"Neural MMO: A Massively Multiagent Game Environment for Training and Evaluating Intelligent Agents."_ arXiv preprint arXiv:1903.00784 (2019).
5. Littman, Michael L. _"Markov games as a framework for multi-agent reinforcement learning."_ Machine learning proceedings 1994. Morgan Kaufmann, 1994. 157-163.
6. Brockman, Greg, et al. _"Openai gym."_ arXiv preprint arXiv:1606.01540 (2016).

