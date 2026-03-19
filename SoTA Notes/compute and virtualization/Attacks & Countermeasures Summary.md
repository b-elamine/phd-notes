
### Attack 1 — VM Escape

**Key example:** CVE-2015-3456 "VENOM" — QEMU Floppy Disk Controller overflow

**What it is:** An attacker running code inside a guest VM exploits a bug in a hypervisor device emulator to execute arbitrary code on the host, breaking out of the VM boundary entirely.

**How VENOM works:**

1. Attacker has guest OS-level access (unprivileged user or compromised application).
2. Writes to the FDC I/O port (`0x3F5`), accessible even to unprivileged guest code.
3. The FDC command FIFO buffer in QEMU is never reset between certain commands — repeated writes overflow it.
4. The overflow corrupts QEMU's heap on the host (QEMU is a userspace process).
5. A crafted payload redirects QEMU's execution to shellcode.
6. The attacker now runs code as the QEMU process, with host-level privileges: full access to other VMs, host filesystem, and host network.

**Defenses:**

- **Patching** — bounds check added to FDC FIFO; reactive, requires a known CVE.
- **seccomp sandboxing** — restricts which syscalls QEMU can make, limiting blast radius.
- **Virtual Machine Introspection (VMI)** — tools like DRAKVUF observe guest I/O at the hypervisor level and can flag anomalous FDC access patterns.
- **Microkernel VMM architecture** — seL4/disaggregated Xen isolate each device driver in a separate domain so a compromise of one emulator doesn't reach the hypervisor core.

**Key gap:** VMI detects but doesn't respond. DRAKVUF can generate an alert; the decision to live-migrate or terminate the VM is still made manually. No standard mechanism exists for a VM's "normal" behavioral contract against which deviations trigger automated action.

**Primary references:** [1] VENOM disclosure (CrowdStrike, 2015) · [2] DRAKVUF (Lengyel, WOOT 2014) · [3] Ren et al., arXiv 2025 · [4] HyperHammer (ASPLOS 2025) · [5] Garfinkel & Rosenblum, NDSS 2003

---

### Attack 2 — Cross-VM Side-Channel

**Key examples:** Prime+Probe · Flush+Reload · CacheBleed (2016) · Spectre/Meltdown (2018)

**What it is:** Two VMs co-located on the same physical CPU share the L3 (last-level) cache. An attacker VM can observe how the victim VM uses that shared cache and reconstruct the victim's memory access patterns — extracting cryptographic keys without any hypervisor bug.

**How Prime+Probe works:**

1. Attacker and victim are co-located on the same physical CPU (normal cloud placement).
2. **Prime:** Attacker fills specific cache sets with its own data.
3. Victim runs a sensitive operation (e.g., RSA decryption) whose memory accesses depend on secret key bits.
4. Victim's accesses evict the attacker's cache lines from the sets it used.
5. **Probe:** Attacker re-reads its cache lines and measures latency. Slow = victim evicted it = victim accessed that cache set.
6. The access-time pattern reveals which memory addresses the victim used, which reveals key bit patterns.
7. Repeat thousands of times. CacheBleed recovered a full 2048-bit RSA key in under an hour.

**Defenses:**

- **Intel CAT (Cache Allocation Technology)** — partitions the L3 cache so VMs use non-overlapping regions; static, configured at placement time.
- **Cache noise injection** — adds random timing delays; performance penalty too high for commercial cloud.
- **CPU pinning / strict co-location policy** — never place sensitive VMs on the same physical CPU as untrusted tenants; scheduler has no runtime knowledge of which tenants are "sensitive."
- **Constant-time cryptographic implementations** — makes memory access patterns independent of secret data; adopted in modern OpenSSL/BoringSSL.

**Key gap:** CAT and CPU pinning are static placement decisions. No mechanism continuously reassesses co-location risk at runtime and dynamically triggers re-partitioning or live migration. Hardware performance counters (PMU: `LLC_MISSES`, `LLC_LOADS`) could in principle detect the characteristic fill-probe pattern, but no system monitors these per-VM at hypervisor level and acts on anomalies.

**Primary references:** [6] Liu et al., IEEE S&P 2015 · [7] Yarom & Falkner, USENIX Security 2014 · [8] CacheBleed (JCE 2017) · [9] Spectre (IEEE S&P 2019) · [10] Intel CAT Specification

---

### Attack 3 — VM Rollback Attack

**Key mechanism:** Hypervisor snapshot restoration abused to replay revoked credentials and bypass patches.

**What it is:** A VM's state (RAM, CPU registers, disk, network config) can be captured as a snapshot. Restoring a snapshot makes the VM believe it is living in the past — its cryptographic keys, session tokens, and patch level all revert to the snapshot time, undoing any security event that occurred after the snapshot was taken.

**Concrete scenario:**

1. Day 0 — routine snapshot taken; VM holds TLS key K1 and session token S1.
2. Day 7 — compromise detected; keys rotated (K1 → K2), tokens revoked.
3. Day 7+1h — attacker (still at hypervisor level) restores the Day 0 snapshot.
4. VM is back to K1/S1; it is un-patched; the key rotation never happened from its perspective.
5. Attacker uses K1 to decrypt previously captured TLS traffic. Recorded sessions are now decryptable.

**Defenses:**

- **TPM sealed storage** — keys sealed to TPM state are only accessible when the system is in the expected measured configuration; a rolled-back TPM can't unseal new keys.
- **Monotonic counters** — tamper-resistant hardware counters that only increment; a VM can detect it was rolled back if the stored counter value is lower than the hardware counter.
- **Snapshot integrity logs** — cryptographically timestamped chain of snapshot events; visible to auditors after the fact, not in real time.
- **Remote attestation at boot/join time** — rolled-back VM's measured state won't match expected state; attestation fails at join. Only checked at join time, not continuously.

**Key gap:** All defenses are point-in-time. The hypervisor already generates an event for every snapshot restoration — no current system treats unexpected restoration as a security signal requiring automated response. The orchestration question is distinguishing "legitimate backup restore" from "post-incident rollback."

**Primary references:** [11] TCG TPM 2.0 Specification · [12] Szefer & Lee, ASPLOS 2012 · [13] AMD SEV-SNP White Paper, 2020

---

### Attack 4 — Malicious VM Image Injection

**Key examples:** Poisoned AMIs in AWS Marketplace · Backdoored OpenStack Glance images · QCOW2 supply chain

> **Layer distinction:** This attack operates on full VM disk images (`.qcow2`, `.vmdk`, `.vhd`, AWS AMI) — a Layer 1 attack. The analogous container image supply chain problem (Docker/OCI) is a Layer 5 problem and is not covered here.

**What it is:** An attacker publishes a malicious VM image that appears to be a legitimate OS. Organizations sourcing base images from public marketplaces instantiate it, running the backdoor from first boot — below the application layer, invisible to applications inside the VM.

**How it works:**

1. Organizations source base VM images from public marketplaces rather than building their own.
2. Attacker publishes a malicious image appearing to be Ubuntu/CentOS/Windows Server with a hidden payload: backdoored init script, kernel module rootkit, credential harvester, or delayed cryptominer.
3. The image passes basic hash checks (the hash matches the attacker's published hash) and may pass signature scans if obfuscated.
4. On instantiation the hypervisor boots the image; malicious components run from first boot.
5. The payload operates at kernel/init level — applications inside the VM are unaware.
6. Malicious behavior only surfaces through behavioral signals: unusual network connections, unexpected process spawning, anomalous syscall patterns.

**Defenses:**

- **Hash-based attestation** — SHA-256 of the image compared against a trusted-source hash; detects post-publication tampering, not malicious content baked in at publication.
- **Pre-instantiation scanning** — Trivy, OpenSCAP, commercial tools scan for known CVEs and malware signatures; novel/obfuscated payloads and logic bombs are typically missed.
- **Measured boot / UEFI Secure Boot** — boot stages are hashed into TPM PCR registers; a backdoored init script produces a different measurement.
- **Private image repositories with governance** — internal registries with admission scanning and signing; shifts trust anchor to the build pipeline.

**Key gap:** All defenses are pre-boot. A signed, scanned image can contain a logic bomb activating 72 hours after first boot or on detection of specific environment variables. No standard behavioral specification exists for what a "normal" first-boot looks like for a given image class. The hypervisor intercepts every syscall and I/O operation but no current system uses this visibility to automatically profile and classify newly booted VM behavior against an expected model.

**Primary references:** [14] HyperPill, USENIX Security 2024 · [15] NIST SP 800-190 · [16] Pfoh et al., "Nitro," ACNS 2011

---

### Attack 5 — Blue Pill Hypervisor Rootkit

**Key examples:** Blue Pill (Rutkowska, 2006) · SubVirt (King & Chen, 2006) · LoJax UEFI implant (APT28, 2018)

**What it is:** A malicious hypervisor is injected below the running OS. The entire OS is demoted into a VM running inside the attacker's hypervisor, which operates in VMX root mode — completely outside OS visibility. Every security tool in the OS is now inside the attacker's VM and can be intercepted, modified, or silenced.

**How Blue Pill works:**

1. Attacker gains kernel-level access (any escalation vector).
2. Loads a malicious kernel module containing a minimal hypervisor.
3. The module captures current OS state (registers, memory map, processes) and constructs a VMCS (Intel VM control structure) representing it as a guest.
4. Executes `VMLAUNCH` — CPU transitions to VMX non-root mode; OS resumes as a guest with an imperceptible pause.
5. Malicious hypervisor now intercepts all hardware access: disk I/O, network, memory-mapped hardware, BIOS/UEFI.
6. CPUID queries for hypervisor presence (leaf 1, bit 31) are intercepted and return 0 — OS believes it has bare-metal access.
7. **Persistence (LoJax variant):** payload embedded in UEFI firmware; survives OS reinstall and hard drive replacement.

**Defenses:**

- **Secure Boot + UEFI measured boot** — requires signed boot components; effective against boot-time injection, not runtime injection after a valid boot.
- **DRTM (Intel TXT / AMD SKINIT)** — re-establishes hardware-rooted trust measurement at runtime, not just at boot; a Blue Pill inserted after boot is detected. Point-in-time check only, not continuous.
- **AMD SEV-SNP / Intel TDX** — hardware-enforced VM isolation; even the hypervisor cannot read or modify guest VM memory; limits what a malicious hypervisor can do even if it controls the hardware layer.
- **Timing-based hypervisor detection** — measures latency of `CPUID`/`RDTSC`/`RDTSCP` instructions; hypervisor interception adds measurable overhead. High false-positive rate in cloud environments where legitimate VMs are already virtualized.

**Key gap:** TPM PCR registers record state at measurement events — there is no continuous stream of hardware-rooted attestation data. The window between "measured at boot" and "continuously verified" is exactly what Blue Pill exploits. AMD SEV-SNP measurement registers could in principle be sampled periodically; an orchestration layer maintaining a time-series of attestation measurements per VM could detect the discontinuity caused by runtime injection.

**Primary references:** [17] Blue Pill (Rutkowska, Black Hat 2006) · [18] SubVirt (King & Chen, IEEE S&P 2006) · [19] LoJax (ESET, 2018) · [20] AMD SEV-SNP White Paper, 2020 · [21] Wojtczuk & Rutkowska, Black Hat DC 2009

---

## References

| # | Title | Authors | Venue / Date |
|---|---|---|---|
| [1] | VENOM: Virtualized Environment Neglected Operations Manipulation | Jason Geffner, CrowdStrike | CrowdStrike Research Blog, May 2015 |
| [2] | DRAKVUF: Stealthy Observation System | Tamas K Lengyel | IEEE Security & Privacy Workshop on Offensive Technologies (WOOT), 2014 |
| [3] | Breaking Isolation: A New Perspective on Hypervisor Exploitation via Cross-Domain Attacks | Ren et al. | arXiv, 2025 |
| [4] | HyperHammer: Breaking Free from KVM-Enforced Isolation | Chen et al. | ASPLOS, 2025 |
| [5] | A Virtual Machine Introspection Based Architecture for Intrusion Detection | Tal Garfinkel, Mendel Rosenblum | NDSS, 2003 |
| [6] | Last-Level Cache Side-Channel Attacks are Practical | Fangfei Liu, Yuval Yarom, Qian Ge, Gernot Heiser, Ruby Lee | IEEE S&P (Oakland), 2015 |
| [7] | FLUSH+RELOAD: A High Resolution, Low Noise, L3 Cache Side-Channel Attack | Yuval Yarom, Katrina Falkner | USENIX Security, 2014 |
| [8] | CacheBleed: A Timing Attack on OpenSSL Constant-Time RSA | Yuval Yarom, Daniel Genkin, Nadia Heninger | Journal of Cryptographic Engineering, 2017 |
| [9] | Spectre Attacks: Exploiting Speculative Execution | Paul Kocher et al. | IEEE S&P, 2019 |
| [10] | Cache Allocation Technology — Specification | Intel Corporation | Intel Architecture Software Developer Manual, Vol. 3B, 2016 |
| [11] | Trusted Platform Module Library Specification, Family "2.0" | Trusted Computing Group (TCG) | TCG Specification, 2016 (rev. 1.59, 2023) |
| [12] | Architectural Support for Hypervisor-Secure Virtualization | Jakub Szefer, Ruby B. Lee | ASPLOS, 2012 |
| [13] | AMD SEV-SNP: Strengthening VM Isolation with Integrity Protection and More | AMD Corporation | AMD White Paper, 2020 |
| [14] | HyperPill: Fuzzing for Hypervisor-bugs by Leveraging the Hardware Virtualization Interface | Alexander Bulekov et al. | USENIX Security, 2024 |
| [15] | NIST SP 800-190: Application Container Security Guide | NIST | NIST Special Publication, 2017 |
| [16] | Nitro: Hardware-based System Call Tracing for Virtual Machines | Jonas Pfoh, Christian Schneider, Claudia Eckert | ACNS, 2011 |
| [17] | Subverting Vista Kernel for Fun and Profit (Blue Pill) | Joanna Rutkowska | Black Hat USA, 2006 |
| [18] | SubVirt: Implementing Malware with Virtual Machines | Samuel T. King, Peter M. Chen | IEEE S&P (Oakland), 2006 |
| [19] | LoJax: First UEFI Rootkit Found in the Wild, Courtesy of the Sednit Group | ESET Research | ESET White Paper, 2018 |
| [20] | AMD SEV-SNP: Strengthening VM Isolation with Integrity Protection | AMD Corporation | AMD White Paper, 2020 |
| [21] | Attacking Intel Trusted Execution Technology | Rafal Wojtczuk, Joanna Rutkowska | Black Hat DC, 2009 |