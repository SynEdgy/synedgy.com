# RFC Configuration

This repository is an exploration to find out what a modern and multi-purpose
configuration management tool should look like, so we can define the overall architecture and interactions between the involved components.

## __Problem definition__

Setting up a system usually starts from a base configuration, sometimes seen as the default configuration or the **current state** of the configuration.
We start from that state, and progressively change it to the state we want: the **Desired State**.

### Changes = Transformative actions

Each change transforms the system into a new state.  
Reverting a change is applying a new transformative action, effectively
creating a new state.

<img src="./assets/transform.png" />

For this reason, the configuration of a system can be seen as a Directed Acyclic Graph, where each transformation (i.e. change to the system) is a vertex, and each resulting state an edge (or node).

<img src="./assets/DAG.png">

### Managing Complexity with abstraction

To change a system from one state to another, it may take a
great many number of low level changes. To simplify the
management of the changes, we abstract low level changes into higher level ones.  
One of the reason is to let a person who does not understand
those low level changes to apply the high level change.


Typically, we group low level commands/calls into higher level
functions, so that  

<img src="./assets/low_level_cmds.png">

Becomes:

<img src="./assets/high_level_function.png" />

These functions are still imperative transforms, and only
apply a (set of) change(s), **and may fail to do so, even
partially**.

This is what a script does and its purpose.

Changing a system to a desired state with imperative
transforms requires a rather **complex sequence**
(do _a_ then _b_) and **logic** (if, then) to ensure the transforms
towards the desired state succeed.

There is no guarantee that we will reach the desired state
(beyond what's written within the script),
and that the prerequisities are met before we can apply those
changes (again, except what's within the script).
Once an imperative transform has been applied, we don't know
what happens if we try to apply the same transform again.  
The point here is that a good script won't have those problems,
but it's not safe to make any assumptions.

### Imperative transforms do not scale well

While abstracting with higher level function helps, it does not scale
well when managing complex systems managed by several people, and constantly
changing over time.

Even with higher level functions, we focus on the transformation they bring, and the parameters they need for a given transformation.  

<img src="./assets/chaining_imperative_transforms.png" />

By thinking about the transforms, we force the administrators to think in terms
of changes at a given time, but they look the bigger picture.

We're making the configuration transactional, and the administrator focuses
on a single transaction.


### Configuration Drift

Traditionally, we then rely on monitoring to get an idea of the current 
state of that system. By doing so, we tend to consider the compliance with the 
desired state only after all the transforms have been applied.  
It is also often the case where we will create that monitoring, and eventually
write the documentation about that state (first corner to be cut when
constrained by time), after the transforms have been applied.

When the system to configure is complex, we have to chain those imperative
transforms together (run script 1, then script 2), maybe over a long time or
across different people or teams.  
As time passes, it's easy for the system to change away from that state
we wanted, without being caught by monitoring and remediated. This is called
**configuration drift**, and often accumulates over time.

### Elevating abstractions with **idempotent** resources

To improve the abstraction offered by standalone scripts or higher level
commands, we can enforce some contracts with the 