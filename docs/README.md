# Primer on Configuration Management

This article explains the concepts behind configuration management from code (policy-driven configuration),
why it's the natural evolution from scripting, and its characteristics.  
In this context, Configuration Management (CM) refers to the practice initiated with CFEngine,
and later evolved with Puppet (2005), OpsCode later Chef software (2008), Desired State Configuration
(2012), and more recently Kubernetes (2014) with its object system (and its own particularities).

Many other Configuration Management tools exist and are not considered here. They either don't use the same
principle (i.e. GUI-driven) or they may follow or implement similar approaches (or partially) but implement loose contracts (offering more flexibility, but risking to suffer from the scaling problems CM tries to
address).

## Problem definition

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
of changes at a given time, but they lose the ability to understand the bigger picture.

We're making the configuration transactional, and the administrator focuses
on a single transaction at a given time.

### Configuration Drift

Traditionally, we then rely on monitoring to get an idea of the current 
state of that system. By doing so, we tend to consider the compliance with the 
desired state only after all the transforms have been applied.  
It is also often the case where we will create that monitoring, and eventually
write the documentation about that state (first corner to be cut when
constrained by time), after the transforms have been applied.

When the system to configure is complex, we have to chain those imperative
transforms together (run script 1, then script 2), maybe over a long time or
across different people or teams, increasing change management complexity.  
As time passes, it's easy for the system to change away from that state
we wanted, without being caught by monitoring and remediated. This is called
**configuration drift**, and often accumulates over time.

### Elevating abstractions with **idempotent** resources

To improve the abstraction offered by standalone scripts or higher level
commands, we can apply some practices for those scripts.  
The first thing to change is to make sure we only apply an imperative when
needed, which means we test if the change is needed before we apply the transform.  

By doing so, no matter how many times you apply the transform, it will only effectively
change something when it detects it's needed.  
Even with the change applied several times, it will always give the same result.  
This is **idempotence**.

If a transform was designed to:  
`Append the content '[something]' to the end of the file xyz.txt.`  
We now change the thinking to:  
`Make sure the file xyz.txt ends with '[something]'.`

The difference is that we want the file to end with that content, not to bluntly
add the content if it's already there.

Now this also shifts how we think about changes to the system. We now want to
reach a desired state, not just apply a transformation.
We're now thinking about the state(s), not so much about what imperatives changes
are required to get there.

<img src="./assets/DAG_of_states.png" />

But the thing that effects the change is still some kind of "high level function" or code.  
What changes is that we have different expectations from that:
- It will test before making a change to see if it's that change is needed
- It will converge to the same state no matter how many times it's executed

This piece of code has to take the desired state as parameters, and make sure
the systems **converges** to it. In our case it's called a **resource**, but it has
other names in other ecosystems.

We declare the desired state, and that "function" has to **converge** to that state.  
To do so, the resource always implement the following methods:
- Test: Find out whether the current state is the same as the desired state.
- Set: Apply the Transform to converge to the desired state.

The test has to, in some ways, call another method:
- Get: Retrieve the current state