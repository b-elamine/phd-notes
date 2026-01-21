
## ** Towards AI-driven Framework of Automating Security Orchestration in Clouds **

### Goal of the Paper : 
Creating a framework that automatically and dynamically secure deployment in clouds in two ways : infra hardening, and post-deployment threat detection 

### Introduced Concepts : 
* Subject-Action-Object : "Who did what to which object", this will be extracted using SpaCy's model (en_core_web_trf) dependencies parsing and DeBERTa for semantic role labeling
* Used in this work as a semantic bridge between NL and and IaC policy 
* Context definition using RAG systems, and code generation using fine tuned LLMs
* Post Deployment Rules generation Validated and integrated with SIEM env based on ELK

### Model / mechanism :
5 Models suggested in this work : 
Security    User                 Model,
		 Platform      
		 Application 
		 Rules
		 Detection 
		 
### Assumptions :
* Target IaC framework Terraform 
  
### Enables :
###### SAO Enables : 
* Eliminates ambiguity through semantic grounding
* Enhances LLMs results with structured context 
* Prevents hallucination and policy violation by applying code with intent
###### Deployment and Quality checking
*** Deployment ***
* The architecture integrates 4 sequential actions : NLInterpreter, RAG, Code generation and validation & Self healing  
* This ensure operational correctness and may fail security policies.

*** Checking ***
Generated Checkov code (same generation architecture used before), then we integrate Checkov rule before "Terraform_apply"

###### Threat detection : 
Sigma rules generation, threat intelligence collection, build context, rules generation, ELK validation, deploy for prod
### Limitations :
The work is ongoing not set all together yet.

### Short Summary :
This work aim to automate and make dynamic deployment and security orchestration in cloud infras, starting with defining different security models. First of all, the work address deployment where we we generate IaC (Terraform code) then a loop of self-healing before applying the Terraform code. Second part is post deployment threat detection that consists of sigma rules generation. 

