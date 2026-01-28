### [***From Security to Assurance in the Cloud: A Survey***](https://dl.acm.org/doi/10.1145/2767005)

A survey that addresses the SoTA of ***cloud security*** ( properties, techniques..) and ***cloud security assurance***, and gives recommendations to develop ***next generation cloud security/assurance*** (identify existing trends, highlighting gaps)

- ***Cloud sec assurance*** : be able to prove that security is correctly implemented and actually working 
	- ***Sec*** = mechanisms that protects  | ***Assurance*** = Evidence that these mechanisms works
- *Selection Criteria* : ***Coverage*** (sec of all levels of cloud, all security requirements), ***Actionability*** (what can be really produced), ***Timeliness*** (Up to date nearly a decade), ***Quality*** (Favoring papers in ACM, IEEE and Elsevier) 
- *Taxonomy* :  three Categories in fig below and the 4 dimensions technique (***when*** (timeline of the solution), ***what*** (which property addressed), ***where*** (attack surface) and ***how*** (how they increase sec) coordinates)
 
![Taxonomy figure](2015cloudSecAssurance.png)

- **Vulnerabilities, threats and attacks*** : 
	- *App-Level : target mainly interaction user-services focus on SaaS*
	- *Tenant-on-Tenant : Typical for virtualized Envs, Diff tenant share infra same HW (PaaS, IaaS)*
	- *Provider-on-Tenant (<->) : Mainly when cloud providers are malicious, alt, 1 or more  tenant attack cloud infra (IaaS)*
	- *Conclusion : Provider-on-Tenant(<->) attack surface is less explored*
- **Cloud Security** : cloud sec challenging, why ? heterogeneous cloud stack, lack of formal sec requirements, lack of stable categorization of techniques, trade offs sec flexibility and high perf, lack of transparency of events happening in cloud back-end --> Ad Hoc research works
	- *Encryption : For achieving confidentiality (for most papers)*
	- *Signature : Encryption based digital signature for integrity, privacy or both*
	- *Access control : Existing not directly applicable to the cloud, new approaches for cloud defined by research community.*
	- *Auth : Authentication and identity management*
	- *Trusted computing : TPMs and similar hardware, require to be adapted to virtualized Envs *
	- IDS/IPS "Intrusion (Detection/Prevention) System" 
	- Conclusion : Big focus on encryption and access ctrl, recently IDS/IPS and trusted computing is an increasing research area.   
- ***Cloud Assurance*** : Poor security <=> prevent proving good security and privacy. However, we can have sometimes good security and poor assurance (security mechanisms are not made visible to the user)
	- Testing : Applied on all cloud layers, give the assurance to users*
	- *Monitoring : Increases the level of transparency*
	- *Certification : Usually human readable format <-> Static monolithic software. Not well suited to the service scenarios (it needs to accomplish the dynamic nature of clouds)*
	- *Cloud Audit/Compliance : "Making cloud auditable" --> Trustworthiness*
    - *Service Level Agreement (SLA) : Contract between clients and service providers, regulating interactions*
    - *Conclusion : Mainly general purpose assurance techniques rather than focusing on specific security aspects*
- **Next gen cloud sec and assurance :** 
    - *Good solution = balancing control of sec and assurance of cloud between providers and customers(tenants in general)*
    - *Increased cloud transparency = better security management*
    - *standardized access to cloud activities data = customer have evidence on security*
    - *Conclusion : using cloud back-end (currently underused) we can build new services and improve cloud security and assurance*

