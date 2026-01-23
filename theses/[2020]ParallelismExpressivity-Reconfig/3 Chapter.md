## Draft Notes :

* Exclusion : 
* software packages management (no distributed sys support and remote dependecies), 
* devops (Do not adresse the execution of reconfig)
* Dyn Soft Update ()
* standards APIs, autonomous frameworks addressing specific needs, adressing other parts of MAPE-K than E and K, Software component models.

* Analysis Criteria : Reconfig elements, types of reconfig operation supported, life cycle handling, parallelism of reconfig tasks, separation of concerns, formal modeling.

### Config Management tools :
Allow admins to deploy and configure virtual infras or softwares on physical infras or cloud.
Categorized using 2 criterias : 
1 - Types of elements to reconfigure :
* Software only (SCM tools)
* Containers only
* VMs only
* Combinations of all three above
2 - Declarative or imperative nature of CM tools : 
Declarative --> define target system, actions will be done based on the difference between start and target 
Imperative --> Declare the set of reconfig actions to be performed directly

###### Imperative SCM tools : 
Ansible, Chef
##### Declarative CM tools : 
TOSCA, Brooklyn, Juju, Terraform, PaaSage, AWS CloudFormation, OpenStack Heat, Kubernetes, Docker Swarm, MoDEMO, Puppet, SaltStack, Joli Redeployment Optimiser. 


### Control Component models : 
Using components-based approach to detail the life cycle of modules, components here are used here to model and handle the life cycle of the modules not their functioning 

Jade, Tune, DeployWare, Adage, SmartFrog, Engage, Aeolus. 


# Chapter 3 : Reconfig of Distributed Systems : State of the Art


### Chapter Role :

- This chapter presented two types of  existing solutions addressing reconfiguration : CM tools and Control components models, then compared them to see strength and weaknesses using different analysis criteria 

### Main Goal :

- Analyse the existing , find gaps and areas of development 

### Introduced Concepts :

Draft notes above contains key concepts presented in this state of the art chapter.

### Model / mechanism :


### Assumptions :

Exclusion of some concepts argued in the paper, defining criteria for comparison 
  
### Enables :

* Having a broad view and understand of the literature and existing solutions 

### Limitations :


### Short Summary :

* After deep dive into existing solution the chapter show us that we didn't achieve a good level of a general solution reconciling parallelism expressivity and separation of concerns in reconfig of distributed systems



