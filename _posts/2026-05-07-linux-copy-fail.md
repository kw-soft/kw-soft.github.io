---
layout: post
title: "CVE-2026-31431: Copy Fail: Linux Kernel Local Privilege Escalation"
date: 2026-05-07
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
  - Open Source
---

**CVE-2026-31431: "Copy Fail": Root on Every Major Linux Distribution**

Hey Guys! On April 29, 2026, one of the most significant Linux kernel vulnerabilities in recent years was publicly disclosed. Nicknamed **Copy Fail** and tracked as **CVE-2026-31431**, it allows any unprivileged local user to escalate to root: reliably, deterministically, and with a 732-byte Python script that works unmodified across virtually every major Linux distribution built since 2017.

I wrote a safe detection script for it, which you can find on [GitHub](https://github.com/kw-soft/copyfail). But first, let me break down what this vulnerability actually is.


**What is Copy Fail?**

Copy Fail is a logic flaw in the Linux kernel's cryptographic subsystem: specifically in the `algif_aead` module, which is part of the AF\_ALG userspace crypto API. The bug was introduced in 2017 via commit `72548b093ee3`, which switched AEAD (Authenticated Encryption with Associated Data) operations to in-place processing as an optimization.

The problem: this optimization allows a page cache page to end up in the kernel's writable destination scatterlist for a crypto operation. An unprivileged process can then use the `splice()` system call into an AF\_ALG socket to perform a controlled **4-byte write into the page cache** of any readable file on the system: without ever touching the file on disk.

By targeting a setuid binary like `/usr/bin/su`, an attacker can patch its in-memory copy and gain a root shell the next time that binary is executed. The entire attack is in-memory, leaves no traces on disk, and bypasses SELinux and AppArmor in default configurations.


**Why is this particularly dangerous?**

Most kernel privilege escalation exploits require at least one of the following: a race condition, precise kernel offsets per distribution, compiled payloads, or specific kernel versions. Copy Fail requires none of these.

- **Deterministic**: no race window, no retry loop, no crash risk
- **Universal**: the same unmodified script works on Ubuntu, RHEL, Amazon Linux, SUSE, Debian, Arch, and more
- **Tiny**: the published proof-of-concept is a 732-byte Python script using only the standard library
- **Stealthy**: the write bypasses the normal VFS write path and leaves no on-disk artifact
- **Container-breaking**: the page cache is shared across containers and the host, enabling cross-tenant attacks in Kubernetes and shared cloud environments

The vulnerability affects every Linux kernel from version **4.14 (2017) through 6.19.11**. The upstream fix, committed on April 1, 2026, simply reverts the 2017 optimization.


**Am I affected?**

If you are running a Linux system with a kernel between 4.14 and 6.19.11, the answer is most likely yes: unless your distribution has already shipped a patched kernel or a module-disabling mitigation.

A few notable cases:

- **Ubuntu** shipped a mitigation on April 30 via the `kmod` package (USN-8226-1), which drops a `modprobe.d` rule that prevents `algif_aead` from loading. This was distributed as a security update through `unattended-upgrades`, so most up-to-date Ubuntu systems received it automatically within 24 hours.
- **Ubuntu 26.04+** ships an unaffected kernel and is not vulnerable at all.
- **RHEL-family distributions** (RHEL, CentOS, AlmaLinux, Rocky) have `algif_aead` built directly into the kernel, meaning `modprobe.d` blacklists have **no effect**. The correct mitigation there requires adding `initcall_blacklist=algif_aead_init` as a kernel boot parameter via `grubby`.
- **Arch Linux and Fedora** had patches available at the time of disclosure.


**The Detection Script**

To make it easy to verify whether a system is protected, I wrote a safe detection script: **[kw-soft/copyfail](https://github.com/kw-soft/copyfail)**

It performs three checks without touching the AF\_ALG interface or executing any exploit code:

1. **Kernel version**: checks whether the running kernel falls within the vulnerable upstream range
2. **Module state**: checks whether `algif_aead` is loaded or blocked
3. **Distro patch status**: for Debian/Ubuntu/Parrot, verifies the `kmod` package version against the USN-8226-1 fix; for other distributions, points to the correct update command

The Ubuntu check deserves a special mention: because Ubuntu backports security patches without changing the upstream kernel version number, checking `uname -r` alone is misleading. A system running `6.8.0-71-generic` may or may not be mitigated depending on the `kmod` package version: so the script checks that directly.

```bash
curl -fsSL https://raw.githubusercontent.com/kw-soft/copyfail/main/copyfail.sh -o copyfail.sh
chmod +x copyfail.sh
sudo ./copyfail.sh
```


**Applying the mitigation**

If your system is not yet protected, here is what to do depending on your distribution.

**Debian / Ubuntu / Parrot / Linux Mint:**
```bash
sudo apt update && sudo apt upgrade && sudo reboot
```
This installs the patched `kmod` package which blocks `algif_aead` via `modprobe.d`.

**RHEL / CentOS / AlmaLinux / Rocky** (built-in module: `modprobe.d` has no effect):
```bash
sudo grubby --update-kernel=ALL --args='initcall_blacklist=algif_aead_init'
sudo reboot
```

**Arch Linux:**
```bash
sudo pacman -Syu linux && sudo reboot
```

The mitigation does not affect SSH, dm-crypt/LUKS, IPsec, OpenSSL, GnuTLS, or kTLS.


**Checking your automatic update status**

Ubuntu's `unattended-upgrades` service handles security updates automatically: but only if it is enabled and configured. You can verify this with:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Both values should read `"1"`. Also worth knowing: even with automatic upgrades active, **automatic reboots are off by default**. A new kernel only becomes active after a manual reboot, so keep that in mind when patching kernel-level vulnerabilities.


**References**

- Theori / Xint Code: [xint.io/blog/copy-fail-linux-distributions](https://xint.io/blog/copy-fail-linux-distributions)
- NVD: [nvd.nist.gov/vuln/detail/CVE-2026-31431](https://nvd.nist.gov/vuln/detail/CVE-2026-31431)
- Ubuntu advisory: [ubuntu.com/blog/copy-fail-vulnerability-fixes-available](https://ubuntu.com/blog/copy-fail-vulnerability-fixes-available)
- USN-8226-1: [ubuntu.com/security/notices/USN-8226-1](https://ubuntu.com/security/notices/USN-8226-1)
- CERT-EU: [cert.europa.eu/publications/security-advisories/2026-005](https://cert.europa.eu/publications/security-advisories/2026-005/)
- Microsoft Security Blog: [microsoft.com/en-us/security/blog/2026/05/01/cve-2026-31431-copy-fail-vulnerability-enables-linux-root-privilege-escalation](https://www.microsoft.com/en-us/security/blog/2026/05/01/cve-2026-31431-copy-fail-vulnerability-enables-linux-root-privilege-escalation/)
- Detection script: [github.com/kw-soft/copyfail](https://github.com/kw-soft/copyfail)