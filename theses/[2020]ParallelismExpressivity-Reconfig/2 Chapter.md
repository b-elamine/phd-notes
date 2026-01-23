# Chapter 2 : Distributed Systems and their Modeling


### Chapter Role :

- This chapter aim to define different concepts in distributed systems (background), how to model them the discussed concepts have to be understood so we can follow the next chapters where contributions are made

### Main Goal :

- Understanding and common vocabulary for distributed systems

### Introduced Concepts :

##### * Distributed Infras : 
Computing resources interconnected by networks.
###### Sub-concepts :

> We have multiple types of resources ( Physical Machine, VM and Container)

> Provisioning computing resources (Direct access, Grid and clusters, cloud computing)

> Remote access ( shell access and batch access)


##### *  Distributed Software :

###### Sub-concepts :

> Architecture of distributed apps : A software that runs in a distributed infra, multiple module interact each others, a distributed app is a piece of distributed software to achieve a goal; distributed app may be static or dynamic; Different types of distributed apps : Client-server, master-worker, map-reduce, peer-to-peer

> Component-based representation : Describe an app with a set of component and interfaces (ports) to use or provide a service


##### *  Life-cycle of a distributed App : 
life cycle of a piece of software is the set of configurations of this app as well as configurations changing. Each module have a life-cycle → the life cycle of different modules in one distributed app are dependents; We model life cycles using state machine ( in this work basic state machine, Petri and UML are introduced) 

  
##### *  Reconfig of Distributed Apps : 
Changing the config of an app ( interacting its life-cycle )
###### Sub-concepts :

> Common types of reconfigs : Deployment, Scaling, Update, Migration

> Autonomic computing : Giving a system the capability to change and adapt to an environment.
It includes Monitoring, decision making and the execution of changes ( known autonomic computing model MAPE-K )

  
##### *  Parallelism in Reconfig :

######  Sub-concepts :

> Overview : the traditional way of executing a reconfig plan was taking an order of actions that satisfies the dependencies of each action and execute them sequently which is not optimal. Parallelism was introduced, key idea is executing independent tasks in parallel

> Modeling reconfig actions using dependencies graphs : in short we use dependency graphs to to model a set of actions to go from a config to another. And note that two independent task can be executed in parallel

> Types of of parallelism in reconfig of distributed systems : Three types of parallelism will be presented (there exist others)

###### Sub-concepts :

> At the host level : multiple host doing reconfig actions in parallel

> At the module level : each module executing reconfig actions in parallel

> Within modules : Executing reconfig actions concerning the same module in the same time


### Model / mechanism :

* Nothing formalized , background chapter

### Assumptions :

* Focusing on one cause of deployment complexity : dependencies between modules

* Assuming that the problem : where to deploy each module, is already solved
  
### Enables :

* Gives deep understanding of every needed for contributions understanding

### Limitations :

* Nothing to mention

### Short Summary :

* This chapter presents the background on distributed infrastructures and software systems and introduces modeling concepts related to application life-cycles, reconfiguration, and parallel execution.
