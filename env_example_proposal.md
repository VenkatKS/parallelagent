# Environment Proposals
This memo's purpose is to propose a series of environments that can be implemented on the ParallelAgent platform. Each environment is designed to have a different data or control dependency, to illustrate the limitatations of the various parallelization paradigms.

---

## Direct-No-Depend

**Category:** No Dependencies

#### Introduction
This environment consists of a series of agents that are spawned in the center of the environment and run to the edges as soon as the simulation is started.

#### Dependencies

This environment is designed to have no dependencies between the agents and is meant to illustrate the raw performance improvements possible due to parallelization. 

#### Goals 

The metrics collected here will largely be the amount of time taken for the entire environment to enter stable state (i.e. all the agents have reached the end of the "world"). A sequential version of this environment will be launched, as well as exmaples parallelized using OpenMP, OpenMPI, and Cilk. The same metrics will be collected in every run.

##### Reach Goals

The reach goal here would be to also simulate this using AgentM's environment, and to compare the metrics gathered there with ours.

---

## Direct-Baton-Pass
**Category**: Pair-wise dependencies 

#### Introduction

The goal here is to design an environment where there is pair-wise dependencies among agents. An agent will be paired with another at the start of the simulation, and when the first agent reaches a certain point (for example, the current location of the second agent), the second agent will start to run to the end of the map. An important point to note here is that some agents will move very fast, while others will move very slow, and thus it allows for a bit more complex simulation requirements as opposed to when every agent moves at the same speed (i.e. the latter case will just end up being two "turns" of simulation). The variable speed requirement avoids this.

#### Dependencies

The second agent will largely depend on the first agent's position. 

#### Goals

Simulation time will also be one of the goals here. It would be interesting to see how various paradigms deal with this, as the performance differentials might manifest due to the overhead of context switching between different agents as we scale up the number of agents (as some paradigms are able to keep a dedicated execution context per agent, while others are unable to do so).

---

## InDirect-Global

**Category**: Global Dependencies

#### Introduction

In this example, certain agents will not be activated until the state of the "world" enters a specific state. These agents will wait until the world enters a "golden" state due to the actions of the other agents, and then quickly start acting up.

#### Dependencies

Every agent could potentially have a dependency on the state (i.e. health, position, etc.) of another agent, as well as on the world itself.

### Goals

TBD.