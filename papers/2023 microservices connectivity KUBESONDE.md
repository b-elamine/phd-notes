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