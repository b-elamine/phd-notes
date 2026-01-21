## Problem
Cloud–Edge–IoT (CEI) systems enable low-latency, distributed applications but make orchestration hard due to:
- Heterogeneous and distributed resources
- Mobility and volatility
- Dynamic workloads and QoS constraints

Goal: automate end-to-end management of services and resources across cloud, edge, and IoT.

---

## CEI Continuum
Three-layer architecture:
- **IoT**: data generation, constrained devices
- **Edge**: low-latency, proximity processing
- **Cloud**: scalable compute and global control

Applications are multi-layered and require careful component placement.

---

## Core Concepts
- **Resource**: raw infrastructure (CPU, storage, network)
- **Service**: abstraction built on resources (VMs, storage services)
- **Orchestration**: automated lifecycle management across CEI

---

## Orchestration Lifecycle
1. **Selection / Composition**
   - Choose services and placement (cloud / edge / IoT)
1. **Deployment**
   - Provision resources and deploy services
3. **Execution & Monitoring**
   - Observe latency, load, failures
4. **Runtime Control**
   - Scaling, migration, reconfiguration

Key CEI issue: placement decisions are central and dynamic.

---

## Main Challenges
- **Portability & interoperability** across providers
- **Resource heterogeneity & distribution**
- **Data locality, consistency, privacy**
- **QoS management** under dynamic conditions

---

## Orchestrator Evaluation Taxonomy
1. **Impact & Adoption**: scientific impact, community support
2. **Platform & Integration**: providers, interoperability, virtualization
3. **Architecture & Extensibility**: centralized vs decentralized, standards
4. **Supported Operations**: lifecycle coverage and automation

---

## Existing Solutions
### Conventional (Cloud-Centric)
- Kubernetes, Terraform, Cloudify, OpenTOSCA
- Mature and widely adopted
- Limited CEI awareness and runtime adaptation

### Emerging CEI-Oriented
- Oakestra, SODALITE, MiCADO-Edge, PrEstoCloud, GAIKube
- Edge-aware and often decentralized
- Low adoption and limited automation

PrEstoCloud uniquely supports automatic service composition.

---

## Research Gaps
- Declarative, intent-based application descriptions
- Automated service selection and placement
- End-to-end cloud–edge–IoT orchestration
- Runtime adaptation under uncertainty
- Stronger transition from research to industry