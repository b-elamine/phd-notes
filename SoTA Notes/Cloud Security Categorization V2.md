
### Layer-Based Categorization
##### Why this categorization? 
- Works for attacks, vulnerabilities and general security problems
- Countermeasures can naturally be organized by layer
- Every attack or problem targets easily a specific layer

### Schema : 
#### Layer 1 : Compute & Virtualization : 
***Concerns the hardware, data centers and physical and virtual resources.***
Threats that target the physical or virtual foundation of the cloud, including servers, storage, and networking hardware
- Hardware failures
- Physical unauthorized access
- VM escape attacks
- Hypervisor vulnerabilities
- Malware targeting infrastructure

#### Layer 2 : Network & Communication Security
***Concerns data in transit and communication***
Threats that exploit weaknesses in how data moves between users, services, and cloud components.
- DoS / DDoS attacks
- Jamming attacks
- Man-in-the-Middle attacks
- Wormhole attacks
- Black hole attacks
- Packet sniffing
#### Layer 3 : Data Protection & Privacy
***Concerns data integrity and data privacy***
Threats that target the confidentiality, integrity, or availability of stored or processed data.
- Data breaches
- Key exposure
- Data tampering
- Loss of data integrity
- Privacy violations
- Unauthorized data access
#### Layer 4 : Identity & Access Management
***Concerns who can access what and how identity is verified***
Threats that exploit weaknesses in authentication, authorization, or identity management systems.
- Account hijacking
- Identity theft
- Sybil attacks
- Phishing
- Credential theft
- Weak authentication
#### Layer 5 : Application & Services Security
***Concerns the software and services running on top of the cloud***
Threats that target cloud applications, APIs, and service interfaces directly.
- Insecure APIs
- Cloud service abuse
- Software vulnerabilities
- Injection attacks
- Insecure interfaces
- SaaS-level threats
#### Layer 6 : Governance & Compliance
***Concern people, policies and regulations***
Threats arising from human behavior, poor management, lack of policies, or regulatory non-compliance.
- Insider threats
- Inadequate diligence
- Audit failures
- Compliance violations
- Social engineering
- Poor security policies
### Comparative view of the different layers (threat/attack surface)

# Cloud Security Layered Model

A 6-layer approach to securing cloud environments:

| Layer                        | Protects                          | Example Threat/ Attack                 |
| ---------------------------- | --------------------------------- | -------------------------------------- |
| Compute & Virtualization     | Compute foundation (servers, VMs) | Hypervisor exploits, hardware failure  |
| Network and Communication    | Communication channels            | MITM, DDoS, packet sniffing            |
| Data protection and privacy  | Stored & processed information    | Data breach, tampering, key exposure   |
| Identity & Access management | Who can access what               | Stolen tokens, weak authentication     |
| Application & Service        | Service logic & APIs              | SQL injection, insecure APIs           |
| Governance & compliance      | Policies & compliance             | Insider threat, poor security policies |
#### Notes : explaining the threats/attacks examples above : 

# Cloud Security Layered Model – Compact Threat Summary

***Infra** :
- **Hypervisor exploit:** Attacker escapes a VM to access others.  
- **Hardware failure:** Server or storage fails, causing downtime or data loss.  
***Network*** :
- **MITM:** Intercepts packets to read or modify data in transit.  
- **DDoS:** Floods network/servers to disrupt service.  
- **Packet sniffing:** Captures packets to steal sensitive info like credentials.  
***Data*** :
- **Data breach:** Unauthorized access to stored data.  
- **Tampering:** Unauthorized modification of data.  
- **Key exposure:** Leaked encryption keys compromise confidentiality.  
***Identity & access*** :
- **Stolen token:** Attacker uses stolen JWT/OAuth to access resources.  
- **Weak authentication:** Poor passwords or missing MFA allow account takeover.  
***App & services*** : 
- **SQL injection:** Malicious input manipulates queries via HTTP/API.  
- **Insecure API:** Improper validation/auth allows API abuse.  
***Governance & Human*** : 
- **Insider threat:** Employee or contractor intentionally/accidentally compromises security.  
- **Poor policies:** Weak procedures or lack of compliance increase risk
