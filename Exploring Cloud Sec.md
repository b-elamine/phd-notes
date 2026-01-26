
# Extracted from [[2015 survey-sec-assurance-cloud]]

## 1) Use the paper’s taxonomy as your “reading lens” (your categorization backbone)

The paper gives you a ready-made classification that can be reused directly in your state-of-the-art notes:

### A. Security properties (“What”)

Security structured around five properties: **confidentiality, integrity, availability, authenticity, privacy**.

### B. Contribution type 

1. **Vulnerabilities / threats / attacks**
    
2. **Security techniques / mechanisms**
    
3. **Assurance techniques** (evidence + validation that security claims are true)


### C. Attack surface (“Where”)

- **Application-level** (SaaS-level services/data)
- **Tenant-on-tenant** (multi-tenancy; PaaS/IaaS resources, isolation)
- **Provider-on-tenant / tenant-on-provider** (IaaS-level trust boundary, malicious provider/tenant)


### D. Mechanism class (“How”) - Security techniques

Security mechanisms  classified into: **encryption, signatures, IDS/IPS, access control, authentication, trusted computing**.


### E. Mechanism class (“How”) - Assurance techniques

They classify assurance into: **testing, monitoring, certification, audit/compliance, SLA**.


**Using this for every paper ?** 
For every paper, the following have to be filled :

- **When** (year)
- **Where** (attack surface)
- **What** (property)
- **How** (technique class: security or assurance)

This is explicitly the survey’s methodology (“when/where/what/how”).


---

## 2) Fast “map of cloud security is vast”

1. **Multiple service layers & actors** (SaaS vs PaaS/IaaS; tenant vs provider) captured via their attack-surface taxonomy.
2. **Multiple security properties** (CIA + authenticity + privacy).
3. **Multiple mechanism families** (crypto, IAM, IDS/IPS, trusted hardware, etc.).
4. **Plus assurance**, which is _not the same as security_: “good security, poor assurance” when users cannot _see/verify_ security operation.
---

## 3) Categorize cloud security using the paper’s tables 
### A. Attacks/threats categorized (Table I)

Table I maps **attack surface × property** = instant classification for “cloud-specific” security issues.

- Application-level: e.g., signature wrapping against cloud control interfaces; DoS variants.
- Tenant-on-tenant: side-channels, co-residency leakage, covert channels.
- Provider-on-tenant: malicious insiders/provider visibility, key exposure constraints.

### B. Security solutions categorized (Table II)

Table II maps **security technique class × property** and shows where research concentrated 

### C. Assurance solutions categorized (Table III)

Table III gives assurance classification 

---

## 4) Identify “problems in cloud security” 

The survey surfaces multiple gaps and imbalance patterns
### Problem 1 - Lack of transparency / evidence (security vs assurance gap)

They argue assurance is about _justifiable confidence_ and depends on **transparency**, including both:

- **introspection** (provider observing its own internals)
- **outrospection** (customers able to observe provider internals affecting their apps/data)

This becomes a strong research direction: standardized interfaces for evidence, metrics, and continuous assurance.

### Problem 2 - Research skew: confidentiality & integrity dominate; availability lags

Their quantitative summary shows security techniques target **confidentiality most** and **availability least**, and they explain availability is often treated as bordering security/reliability/performance (so it gets under-covered).

### Problem 3 - Provider-on-tenant / tenant-on-provider is less explored

They report more work on application-level and tenant-on-tenant; provider-related attack surface is less covered and often overlaps with other surfaces.


PhD angle: stronger threat models for malicious/honest-but-curious providers; verifiable computing, key management constraints, confidential computing assurance.

### Problem 4 - Technique skew: encryption + access control dominate; trusted computing and IDS/IPS have adoption friction

They quantify technique distribution: encryption and access control are the largest shares; trusted computing and IDS/IPS are smaller partly due to recency and deployment/overhead/cost.


PhD angle: reduce operational burden; assurance-friendly IDS; cost-aware trusted hardware + evidence pipelines.

### Problem 5 - Assurance skew: audit/monitoring/testing dominate; certification and SLA are less mature

They show assurance is split: audit/monitoring/testing are most common; **SLA and certification** are the lowest shares and are relatively newer in cloud contexts.


PhD angle: continuous certification for dynamic clouds; SLA that is measurable/technical (not just contractual); bridging monitoring evidence to certification claims.

### Problem 6 - Cloud dynamics break static security/assurance (migration, federation, elasticity)

They explicitly state that cloud features (migration, federation, scalability, elasticity) complicate enforcement and verification of security/assurance controls.

---

## 5) What you should produce by end of week (suggested outputs)

To keep your SoTA structured and reusable, generate these three artifacts from this single survey:

1. **A one-page taxonomy diagram in your own words**
    
	- Properties (5), attack surfaces (3), security techniques (6), assurance techniques (5).
    
2. **A “gap list” (10 bullets max) with citations to the survey**  
    
    Use the “problems” section above; each bullet should map to a potential research question.
    
3. **A candidate research-direction shortlist (3 directions)**  
    Example shortlist that is strongly aligned with the paper:
    
    - Standardized transparency/evidence interfaces enabling outrospection (assurance-by-design).
    - Continuous certification/SLA enforcement under cloud dynamism.
    - Under-explored provider-on-tenant threat model + practical key/compute assurance constraints