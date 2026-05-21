---
layout: post
title: "Dirty Frag (CVE-2026-43284): Linux Kernel Local Privilege Escalation via IPsec and RxRPC"
date: 2026-05-11
categories:
  - Security
  - Linux
  - CVE
tags:
  - Linux
  - Kernel
  - CVE
  - Privilege Escalation
  - Cybersecurity
  - IPsec
---

**Dirty Frag (CVE-2026-43284 + CVE-2026-43500): Another Week, Another Linux Root**

Hey Guys! The ink on the Copy Fail patches had barely dried when a second Linux kernel local privilege escalation landed: this time named **Dirty Frag**, tracked as CVE-2026-43284 and CVE-2026-43500. Disclosed on May 7, 2026 by independent researcher Hyunwoo Kim, it builds directly on the same bug class as Copy Fail and: critically: bypasses the Copy Fail mitigation entirely.

If you patched Copy Fail last week, you are not done.


**What is Dirty Frag?**

Dirty Frag is a chained vulnerability combining two separate page-cache write primitives in the Linux kernel. CVE-2026-43284 lives in the xfrm-ESP subsystem (IPsec), and CVE-2026-43500 lives in the RxRPC subsystem. Chained together, they give any unprivileged local user a reliable, deterministic path to root on virtually every major Linux distribution.

The root cause is structurally identical to Copy Fail: an in-place processing optimization added to the kernel allows attacker-controlled data to land in the page cache of files the attacker does not own. CVE-2026-43284 was introduced in January 2017 by a commit that moved IPsec ESP receive into a fast-path in-place decryption mode. CVE-2026-43500 followed in June 2023 when the same pattern was applied to RxRPC.

Where Copy Fail gave attackers a 4-byte write primitive, Dirty Frag provides full attacker-controlled plaintext at any chosen offset: a significantly more powerful primitive that requires no race condition and carries minimal risk of crashing the system.


**Why does the Copy Fail mitigation not help?**

The Copy Fail mitigation worked by blacklisting the `algif_aead` module. Dirty Frag goes through entirely different modules: `esp4`, `esp6`, and `rxrpc`: which are enabled by default in the standard kernel packages of every major enterprise distribution. The `algif_aead` blacklist does not touch these modules at all.

This means: if you applied the `modprobe.d` workaround for Copy Fail and have not yet rebooted into a patched kernel, your system is still vulnerable to Dirty Frag.


**The disclosure went wrong**

Hyunwoo Kim reported the vulnerabilities through responsible disclosure channels. Before distributions had time to coordinate and prepare patches, an unrelated third party broke the embargo. Kim was forced to publish ahead of schedule: with a working proof-of-concept exploit already public before a single distribution had shipped a fix.

This is a worst-case scenario for coordinated disclosure: the community had essentially zero lead time between public exploit availability and patch availability. AlmaLinux and CloudLinux moved to ship patched kernels within roughly 24 hours of disclosure. Ubuntu followed shortly after. RHEL and Debian stable releases are still working on patches as of May 11.

A second public exploit named **"Copy Fail 2: Electric Boogaloo"** targets the same vulnerability under a different name and reaches root through the same code paths: it is blocked by the same fix.


**Am I affected?**

If you are running a Linux kernel between roughly 4.x and current that has not received the Dirty Frag patch, and the `esp4`, `esp6`, or `rxrpc` modules are available, the answer is yes. That covers virtually every mainstream distribution in use today, including Ubuntu, Debian, RHEL, AlmaLinux, Rocky, Fedora, SUSE, Amazon Linux, and Arch.

Container workloads are also exposed. The kernel page cache is shared between containers and the host. Note however that registering an XFRM SA requires `CAP_NET_ADMIN`, which means hardened Kubernetes environments with default seccomp profiles are less directly exposed. The RxRPC half (CVE-2026-43500) exists precisely to cover that gap: `add_key("rxrpc")` and `socket(AF_RXRPC)` are unprivileged APIs. Unconstrained Docker and most Kubernetes pods without seccomp hardening remain fully exposed.


**Mitigation**

**Patch status as of May 11, 2026**

CVE-2026-43284 is patched in mainline Linux (commit `f4c50a4034e6`). Ubuntu and AlmaLinux have pushed patched kernels to production repositories. RHEL and Debian stable releases are still pending. CVE-2026-43500 has no upstream patch yet across any distribution.

Until a patched kernel is installed, blacklist and unload the affected modules:

```bash
sudo sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' \
  > /etc/modprobe.d/dirtyfrag.conf"
sudo rmmod esp4 esp6 rxrpc 2>/dev/null || true
sudo update-initramfs -u
```

Then reboot as soon as a patched kernel is available for your distribution.

**Debian / Ubuntu / Parrot / Linux Mint:**
```bash
sudo apt update && sudo apt upgrade && sudo reboot
```

**RHEL / CentOS / AlmaLinux / Rocky:**
```bash
# AlmaLinux: patched kernel available
sudo dnf clean metadata && sudo dnf upgrade && sudo reboot
# RHEL/CentOS: patch pending — use module blacklist above until available
```

**Arch Linux:**
```bash
sudo pacman -Syu linux && sudo reboot
```

**Note for RHEL-family systems:** If `esp4` and `esp6` are compiled into your kernel rather than shipped as loadable modules, the `modprobe.d` blacklist will have no effect. Check with `grep -i esp /boot/config-$(uname -r)`: if the result is `=y` rather than `=m`, a kernel update is the only fix.


**The bigger picture**

Two severe Linux kernel LPEs in the same week from the same bug class. CVE-2026-43284 was introduced in 2017, CVE-2026-43500 in 2023: both introduced by in-place optimization commits that nobody audited for this class of flaw. Both found with AI-assisted tooling. Both deterministic and broadly exploitable with tiny Python scripts.

This is not a coincidence. AI-assisted vulnerability research is systematically scanning kernel subsystems for patterns that were previously too tedious to audit by hand. The Linux kernel's cryptographic and networking subsystems have decades of optimization commits that have never been reviewed with this kind of tooling. Expect more of this.



**References**

- Dirty Frag disclosure (oss-security): [openwall.com/lists/oss-security/2026/05/07/8](https://www.openwall.com/lists/oss-security/2026/05/07/8)
- Wiz writeup: [wiz.io/blog/dirty-frag-linux-kernel-local-privilege-escalation-via-esp-and-rxrpc](https://www.wiz.io/blog/dirty-frag-linux-kernel-local-privilege-escalation-via-esp-and-rxrpc)
- Sysdig detection guide: [sysdig.com/blog/dirty-frag-cve-2026-43284-and-cve-2026-43500](https://www.sysdig.com/blog/dirty-frag-cve-2026-43284-and-cve-2026-43500-detecting-unpatched-local-privilege-escalation-via-linux-kernel-esp-and-rxrpc)
- AlmaLinux advisory: [almalinux.org/blog/2026-05-07-dirty-frag](https://almalinux.org/blog/2026-05-07-dirty-frag/)
- Ubuntu advisory: [ubuntu.com/blog/dirty-frag-linux-vulnerability-fixes-available](https://ubuntu.com/blog/dirty-frag-linux-vulnerability-fixes-available)
- Red Hat advisory: [access.redhat.com/security/vulnerabilities/RHSB-2026-003](https://access.redhat.com/security/vulnerabilities/RHSB-2026-003)
- Debian tracker: [security-tracker.debian.org/tracker/CVE-2026-43284](https://security-tracker.debian.org/tracker/CVE-2026-43284)
- NVD CVE-2026-43284: [nvd.nist.gov/vuln/detail/CVE-2026-43284](https://nvd.nist.gov/vuln/detail/CVE-2026-43284)
- Tenable FAQ: [tenable.com/blog/dirty-frag-cve-2026-43284-cve-2026-43500-frequently-asked-questions-linux-kernel-lpe](https://www.tenable.com/blog/dirty-frag-cve-2026-43284-cve-2026-43500-frequently-asked-questions-linux-kernel-lpe)
- Copy Fail detection script: [github.com/kw-soft/copyfail](https://github.com/kwilck/copyfail)
