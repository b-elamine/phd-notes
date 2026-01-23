
# [Artificial Intelligence based Security Orchestration, Automation and Response System](https://ieeexplore.ieee.org/document/9418109)

**This work is not specifically for cloud**

**AI-enhanced Security Orchestration, Automation, and Response (SOAR)** system designed to reduce the manual workload of security teams by automating alert analysis, threat intelligence enrichment, and incident response.

The system collects security events from multiple sources (firewalls, IDS/IPS, honeypots) and **standardizes heterogeneous logs** into a common format. It enriches alerts using a **Threat Intelligence Repository** and, when needed, external open-source intelligence. Multilingual threat intelligence is translated using **NLP-based models**, and all intelligence is converted into **privacy-preserving STIX format**.

An **AI-based SIEM** component applies deep learning (FCNN, CNN, LSTM) to create event profiles and **classify alerts as true or false positives**. For confirmed threats, the system generates **Indicators of Compromise (IoCs)**, gathers evidence, assigns risk levels, and triggers predefined **SOAR playbooks** to accelerate response.

Overall, the work presents an **integrated architecture** combining AI, SIEM, threat intelligence, and SOAR to improve detection accuracy, response speed, and scalability in modern cyber-security operations.
