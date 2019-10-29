# Parallel Agent Simulator Proposal

## Summary

We are going to implement a multi-agent, single-environment simulator that plugs into OpenAI gym to allow for parallelized agent trajectories, while still allowing for rich interactions between the agents, and provide a detailed analysis of the trade-offs between our implementation and the current state of the art simulators. 

## Background

OpenAI provides an API that allows for engineers to specify the environment in which their reinforcement learning agent will interact in. This environment is essentially a simulator, akin to the n-body simulator that we worked with during class. In most cases, the simulator will only support a single agent interacting with the environment (analogous to single player games); such problems need no parallelzation as there is only a single "thread" operating on the shared data ("the environment"). In a sense, we can think of each agent as an "instruction stream" or a "thread", as each agent will be able to make independent actions that directly impact the shared space. 

However, when we introduce multiple agents and allow for them to interact with each other, then we open up the possibility of multiple streams of execution, as each agent is autonomous and will likely make their own decisions independent of the others. This leads to multiple parallel instruction streams, and requires multiple "threads" to interact with the environment, leading to a need for parallelism. This project seeks to add CUDA support for parallelizing these interactions.

## The Challenge

TBD.
