# Security for Virtualized Distributed Systems : from Modelization to Deployment

---

## Why This Matters

This thesis tackles a core problem in cloud security: the gap between what a user
wants in terms of security ("my data must stay confidential") and how the infrastructure
actually enforces it (SELinux rules, firewall policies, VM placement constraints).

The proposed answer is an end-to-end automated framework that takes a high-level user
specification as input and produces a secured, deployed application as output, without
requiring the user to understand any underlying security mechanism.

This is directly foundational for building any framework that aims to automate secure
deployment of virtualized infrastructures.

---

## The Core Problem

Securing a virtualized distributed system requires decisions at multiple levels:

- Where to place virtual machines on physical hardware (placement)
- How to configure security tools inside and around those VMs (configuration)
- How to formally verify that the deployed system actually satisfies the required properties

Each of these levels uses different tools, different languages, and different expertise.
No existing solution at the time unified them into a single automated workflow starting
from a user-level specification.

---

## Contribution 1 - A Unified Model for Applications and Security Requirements

### Problem

To automate anything, you need a formal representation of what you are deploying and
what you want to protect. The challenge is making this representation expressive enough
to capture real security requirements, yet simple enough for a non-expert user to fill in.

### Technical Approach

Lefray builds a metamodel using the Eclipse Modeling Framework (EMF). A metamodel is a
formal definition of the concepts and relations that can appear in a model. Think of it
as a schema: once you define the metamodel, you can create instances of it (actual models)
that describe specific applications.

The metamodel covers three things in a unified way:

**The application structure**

The user describes virtual machines, virtual networks, and how they are connected.
A VM has attributes: disk image, number of virtual CPUs, RAM, storage, geographic location.
Networks are modeled explicitly as nodes (not just as links between VMs), which allows
expressing different security policies on different network segments.
Inside VMs, the user can describe data entities (files, databases) and service entities
(processes), and the interactions between them.

**The security requirements**

Security properties are expressed in a textual domain-specific language. The user writes
things like:

```
#property Isolation(MyDomain, grade=medium);
#property Confidentiality(Data="Logs", Service="SSH");
```

The user does not specify how these properties are implemented. That is the job of the
deployment engine. This separation between specification and enforcement is a key design
choice.

**The infrastructure**

The physical infrastructure (servers, networks) is modeled separately and is not exposed
to the user. It is generated automatically from the real hardware using the Hwloc tool.
This includes not just server inventory but also the internal hardware topology: CPU cores,
cache levels (L1, L2, L3), NUMA nodes and their memory latencies. This level of detail
is necessary for the security-aware placement decisions described later.

### Why This Matters for Automated Secure Deployment

Any framework for automated secure deployment needs this kind of unified representation.
You cannot automate what you cannot describe formally. The key insight here is that the
application model, the security requirements, and the infrastructure model must be kept
aligned and processed together, not in separate silos.

---

## Contribution 2 - A Quantitative Risk Metric for Hardware-Level Information Leakage

### Problem

Even with perfect software-level isolation (firewalls, SELinux, hypervisor separation),
two virtual machines sharing the same physical server can leak information to each other
through shared hardware components. This is called a covert timing channel.

The attack works as follows: two processes on the same machine, even in different VMs,
share CPU caches. One process can modulate cache state (filling or evicting cache lines)
and another can observe the resulting latency changes to infer information. Bitrates of
hundreds of bits per second have been demonstrated on commercial cloud infrastructure.

No existing placement algorithm at the time could account for this threat in a
quantitative way.

### Technical Approach

Lefray proposes an information leakage metric measured in bits per second. It represents
the maximum theoretical bandwidth achievable through a covert timing channel between two
VMs sharing a given hardware component.

The formula for a cache-based covert channel is:

```
bitrate = 0.5 * (latency_hit + latency_miss) * cache_size / 4KB
```

Where latency values are measured in nanoseconds using the Lmbench benchmark suite
(specifically the lat_mem_rd tool), and cache sizes are read from the hardware topology
via Hwloc.

Concrete example on a NUMA server (Grid'5000 Taurus machine):

```
L3 cache: 15 MB shared, hit latency 17 ns, miss latency 108 ns
Bitrate = 0.5 * (17 + 108) * 15MB / 4KB = 4.167 Kbps
```

The US Department of Defense (TCSEC guidelines) considers any channel above 1 bps a
security concern. The proposed metric allows expressing this threshold directly in
security properties as a grade value. If two VMs have an isolation property and the
leakage between their assigned hardware resources exceeds the grade threshold, they
cannot be placed together.

### Why This Matters for Automated Secure Deployment

A framework that ignores hardware-level leakage provides weaker guarantees than it
claims. This metric is the first step toward placement decisions that are genuinely
security-aware at the microarchitectural level. Any serious automated deployment
framework for multi-tenant cloud environments should incorporate this kind of reasoning.

---

## Contribution 3 - A Formal Language for Security Properties (IF-PLTL)

### Problem

Security properties need to be formal enough to be automatically processed, split,
and mapped to enforcement mechanisms. Natural language is too ambiguous. Existing
formal logics (LTL, HyperLTL, MFOTL) do not handle indirect information flows at
runtime.

The key difficulty is indirect flows: if A sends information to B, and B sends it to C,
then there is a flow from A to C even without any direct A-to-C communication. Expressing
"C must never receive information that originated from A, no matter how many intermediaries
are involved" requires a transitive closure operator that most existing logics lack.

### Technical Approach

Lefray defines a new logic called IF-PLTL (Information Flow Past Linear Time Logic).
It is based on the past fragment of Linear Temporal Logic, extended with a transitive
closure relation over information flows.

**Extracting information flows from a running system**

The logic operates on information flow traces, not on raw system calls. The extraction
pipeline has three steps:

Step 1: The Linux Security Module (LSM) intercepts system calls at the kernel level
and produces observable events of the form (source, operation, destination). For example:
`(alice, begin_read, logs)` followed by `(alice, end_read, logs)`.

Step 2: Begin and end events are merged into continuous functional events.
`(alice, read, logs)` occurring during interval [t1, t2].
Two classifier functions determine the direction of information flow:
- is_read_like: reading means information flows from the source to the reader
- is_write_like: writing means information flows from the writer to the destination

Step 3: All operations are reduced to a single relation: `a > b` meaning information
flows from a to b. A read from b by a becomes b > a. A write from a to b becomes a > b.

**The runtime monitor**

A runtime monitor receives each new information flow event, checks it against the
stored history of past flows, evaluates the IF-PLTL formula, and produces a binary
decision: Accept or Refuse. The event is allowed or blocked in real time.

**Core property definitions**

```
Confidentiality(S, A):
  for all x in S, for all y: if x > y then y must be in (S union A)

Integrity(S, A):
  for all x in S, for all y: if y > x then y must be in (S union A)

Isolation(S, A):
  Confidentiality(S, A) AND Integrity(S, A)
```

Where S is the secured set and A is the authorized set.

**Key splitting theorem**

For S = S1 union S2 with S1 and S2 disjoint:

```
Confidentiality(S, A) = Confidentiality(S1, A union S2)
                    AND Confidentiality(S2, A union S1)
```

The same equivalence holds for Integrity and Isolation. This theorem is what makes
automated preprocessing possible.

### Why This Matters for Automated Secure Deployment

A deployment framework needs to go from a global user-level property ("isolate this
group of VMs") to local, enforceable properties on individual components. Without a
formal language with proved splitting equivalences, this transformation cannot be
automated reliably. IF-PLTL provides that foundation.

---

## Contribution 4 - The Deployment Engine

This is the operational core of the framework. It takes formal security properties as
input and produces a deployed, secured application as output. It works in three
sequential phases.

### Phase A - Preprocessing: From User Properties to Enforceable Properties

The user writes properties at a high level. Security mechanisms operate at a low level.
The preprocessing bridges this gap through three automated transformations.

**Transformation 1: Implicit to Explicit**

The user writes `Isolation(MyDomain)` on a group of entities. The algorithm analyzes
the application model as a graph and determines the authorized set for each entity
automatically, based on its connectivity.

Entities in the group are classified as:
- Border entities: have at least one connection to something outside the group
- Internal entities: all connections stay within the group

The authorized set for an internal entity is the rest of the group. The authorized set
for a border entity is the rest of the group plus its external neighbors.

This produces explicit properties with fully specified secured and authorized sets.

**Transformation 2: Global to Singleton**

A property covering a group of entities is split into one property per secured entity,
using the splitting theorem from IF-PLTL.

Input: `Isolation({VM1, VM2, VM3}, {External})`

Output:
```
Isolation({VM1}, {VM2, VM3, External})
Isolation({VM2}, {VM1, VM3, External})
Isolation({VM3}, {VM1, VM2, External})
```

**Transformation 3: Typed Split**

Security mechanisms are type-specific. A network firewall protects network flows. SELinux
protects OS-level access between processes and files. They cannot be used interchangeably.

Each singleton property is split by the type of entities in the authorized set:

```
IsolationVM({VM1},   {VM2, VM3})         -> enforced by placement or SELinux
IsolationVNet({VM1}, {Intranet, Public}) -> enforced by network configuration
```

### Phase B - Placement

**The optimization problem**

VM placement under security constraints is formally a Bin Packing Problem with Conflicts
(BPPC): pack items (VMs) into bins (cores) minimizing the number of bins used, where
some items cannot share a bin (isolation constraints). This problem is NP-hard.

Adding the memory dimension (NUMA nodes) makes it strictly harder. Lefray uses a
First-Fit heuristic for practical feasibility.

**CPU pinning**

VMs are statically bound to specific cores using libvirt. This is mandatory: if a VM
can migrate to different cores at runtime, an isolation property satisfied at deployment
time might be violated later when a conflicting VM lands on the same core.

**NUMA memory management**

Physical servers have NUMA (Non-Uniform Memory Access) architectures with multiple
memory banks. Lefray uses the strict allocation policy via libvirt: memory is only
allocated from specified NUMA nodes.

When a VM needs more memory than one NUMA node has available, multiple NUMA nodes
are merged into a virtual ComposedNUMA object. The information leakage between two
VMs is then computed based on all the hardware resources (cores and NUMA nodes) they
share, including resources shared through a ComposedNUMA.

The composition and decomposition of NUMA groups is tracked dynamically: a ComposedNUMA
is dissolved when no VM is allocated on it anymore, restoring the individual NUMA nodes
to the availability pool.

**Security check during placement**

Before assigning a VM to a set of cores and NUMA nodes, the algorithm checks that for
every isolation property involving that VM, either:
1. The other VM is in the authorized set (isolation is not required), or
2. The information leakage bitrate between the two configurations is below the grade threshold

### Phase C - Automatic Configuration via Security Agents

**The agent model**

A security agent is a process that translates a typed security property into a concrete
configuration of one or more low-level security tools. Agents can live:
- Inside a VM (guest-level): configures tools like SELinux, application firewalls
- In the hypervisor (host-level): more trustworthy, less visibility inside VMs
- At the network level: configures VLANs, SDN rules

Each agent declares its capabilities: the types of properties it can enforce, and the
maximum protection grade it can provide. For example:

```
Confidentiality(Data | Service, Data | Service, grade=80)
```

This capability matches any confidentiality property between data or service entities
where the required grade is at most 80.

**Matching and solving**

The deployment engine matches the typed local properties (output of preprocessing)
to agent capabilities:

1. Guest-level pass: match properties to agents embedded in the VM images
2. Host-level pass: for unresolved properties, filter candidate hosts to those
   with capable host-level agents
3. Placement pass: apply the BPPC algorithm on the filtered host list
4. Final binding: map each remaining property to the capability of the assigned host agent

The scheduling engine is implemented in Java and integrated with OpenStack via its API.

### Full Pipeline Summary

```
User writes application model + security properties
                  |
                  v
     [Preprocessing - 3 transformations]
     implicit -> explicit (graph analysis)
     global -> singleton (splitting theorem)
     singleton -> typed (by entity type)
                  |
                  v
     [Typed local properties]
                  |
          +-------+-------+
          |               |
          v               v
    [Placement]    [Configuration]
    BPPC + NUMA    Agent capability
    First-Fit      matching + triggering
    libvirt        SELinux, iptables...
          |               |
          +-------+-------+
                  |
                  v
     [Secured application on OpenStack]
```

---

## Validation

The complete pipeline was validated on a real industrial case: the Musik advertising
content manager system used by the company Ikusi to manage airport display systems across
multiple airports simultaneously. The system is multi-tenant with strict data separation
requirements between airports. The security policy contained over 70 properties. The
application model was built directly by Ikusi engineers with minimal guidance from the
research team, validating the usability claim of the modeling interface for non-expert users.

---

## Key Limitations and Open Problems

These are gaps explicitly acknowledged by Lefray, and directly relevant to the next
generation of work on automated secure deployment.

**Static deployment**

The entire pipeline runs once. After deployment, there is no mechanism to update the
security configuration. If a new CVE is published affecting a component in a deployed VM,
nothing happens automatically. Adding a new tenant or a new software dependency requires
restarting the full workflow manually.

**No runtime feedback loop**

There is no monitoring layer after deployment. The system cannot detect whether security
properties are being violated at runtime, and cannot trigger reconfiguration in response
to detected threats or changed conditions.

**Coverage limited to IaaS**

The metamodel covers virtual machines and virtual networks. It does not handle containers,
serverless functions, PaaS services, or edge and IoT infrastructure. The framework
assumes a classical cloud virtualization stack.

**No natural language interface**

All modeling is done through a graphical tool and a structured textual language. The user
must learn the metamodel concepts. There is no way to express requirements in natural
language and have them automatically interpreted.

**Network property enforcement incomplete**

The preprocessing correctly computes typed properties on virtual networks, but the
deployment engine does not yet enforce them. Network security is identified as future work.

**Grading system underdeveloped**

The mapping between abstract grade values and concrete information leakage thresholds
is not fully defined. A proper grading system with user-facing semantics is left as
future work.

---

## Summary: What a Next-Generation Framework Needs

Building on Lefray's work, a framework for automated secure deployment of virtualized
infrastructures that goes beyond these limitations would need to address the following:

1. A way to express security requirements without requiring expert knowledge of any
   metamodel, ideally from natural language or from existing compliance standards
   (CIS, NIST, PCI-DSS, GDPR), processed automatically.

2. A mechanism to update the security configuration dynamically when the environment
   changes: new software versions, newly published vulnerabilities, new tenants,
   or evolving compliance requirements.

3. A monitoring layer that detects security events or policy violations at runtime
   and feeds them back into the configuration pipeline to trigger reconfiguration.

4. Coverage beyond IaaS, toward heterogeneous infrastructures including edge nodes
   and IoT devices alongside classical cloud deployments.

5. Integration of the full lifecycle: from initial specification, through deployment,
   through monitoring, to dynamic reconfiguration, as a single continuous automated
   workflow rather than a one-shot process.
