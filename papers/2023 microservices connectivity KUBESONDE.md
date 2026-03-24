### What is Kubesonde?
- ***K8s network security auditing tool***
- ***Probe pod to pod, pod to service and pod to internet network path inside a k8s cluster and audit the connectivity comparing what is actually reachable against what the operator expects to be allowed or denied*** 

### How it works?
- ***Inject ephemeral containers inside the pods and run TCP/UDP scans from there***
- ***Architecture :***
	- Kubernetes cluster : 
		- ***Kubesonde Controller Manager Pod*** :  control-plane container component
			- K8s API EventListener : extracts Pods declarative specification including
				the declared ports, injects the probe and monitor containers into
				them, and initiates the actual probing process
			- Kubesonde API Server : Extends k8s server it allows Kubesonde to communicate with the external world (provides endpoints to to start and customize probing and getting the probing results --specified in YAML file--)
			- Probe Dispatcher : Maintain list of probing sources and targets + the queue of probs to be executed (the dispatcher gives commands to the probe containers and collect results as well as scheduling periodic re-probing)
			- Monitor Listener : receives additional ports info from monitor containers and generates new probing targets
		- ***Pod :*** 
			- Probe Container : Initiate connections to other containers inside the same cluster and selected services, externel networks and the internet (probe containers is in the same network namespace as the source container)
			- Monitor Container : observes the network in the pod it belongs accessing the interface offered by the os (proc filesystem in linux) to get information about opened ports of the container of that pod

___
### KUBESONDE source code important notes : 

***- crd/ : controller, language GO, it runs as a deployment inside the k8s cluster***
***- frontend/ :  Frontend, developed in Typescript / React, can be executed locally "npm run dev"***

#### Back-end Deep Dive Notes :

**CRD Api Types :**
- crd/api/v1/kubesonde_types.go : Defines how to create a kubesonde resource in the cluster
- crd/api/v1/probe_server_types.go : defines output structer
**Entry Point & Controller Manager :**
- main.go : Process start here, two things are done (register the kubesondeReconsiler and start HTTP server)
**The Reconciler :**
- crd/ineternal/controller/kubesonde_controller.go : the entry point into all probing logic and it is called by controller-runtime whenever a kubesonde crb object is created or updated ..
**Event Listner :**
- crd/controllers/events/listener.go, pod.event.go, service.event.go : uses k8s shared informer for watching pod and service changes in realtime
**Event Storage :**
- crd/controller/event-storage/probe.go, recorder.go : globval-in memory database as a package it holds, commands, active pods, deleted pods, services (a dedublication key is used to avoid doing the same probing more than once)
**Probe Commands :**
- crd/controllers/probe_command/probe_commands.go, probe.command.go : shell commands are constructed here
**Dispatcher (Scheduling engine) :**
- crd/controllers/dispatcher/probe_dispatcher.go, priority_queue.go : probing queue running in an infinite loop 
**Inner -- Remote commands Exec :** 
- crd/controllers/inner/probe_controller.go : communication with k8s API to run command inside a container
**Monitor controller :**
- crd/controllers/monitor/monitor_controller.go : runtime port discovery (defined in the arcitecture above as monitor listener)
**Debug Container :** 
- crd/controllers.debug-container/ephemeralContainer_controller.go: check and manages the ephemeral container injected in each app Pod
**Recursive Probing -- the heartbeat loop : 
- crd/controllers/recusrsive-probing/recursive-probing.go : if queue is empty it re-queues all the probes in low priority (every 20 sec it checks if the queue is empty)
**API Server :**
- crd/rest_apis/apis.go, crd/rest_apis/get_probes.go : Go http server signle endpoint GET /probes --> returns ProbeOutput as json 
**State Package :** 
- crd/controllers/state/probe_state_controller.go : Hold the final results
####  Data Flow Step by Step Notes : 
