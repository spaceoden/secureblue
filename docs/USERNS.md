# User namespaces

[User namespaces](https://en.wikipedia.org/wiki/Linux_namespaces#User_ID_(user)) are a kernel feature introduced in kernel version 3.8. When an unprivileged user asks the kernel to create a namespace, the kernel needs to permit that user to do so. Whether this is permitted by the kernel is controlled via a sysctl flag.

There is a [long history](https://madaidans-insecurities.github.io/linux.html#kernel) of vulnerabilities made possible by allowing this functionality for unprivileged users ever since its [introduction](https://gitlab.com/apparmor/apparmor/-/wikis/unprivileged_userns_restriction). Given this history, you might think we should just disable this functionality altogether. However if this functionality is disabled, then flatpak can't function without the suid bit set on the bubblewrap binary (which is set in no-userns images). In this scenario bubblewrap creates namespaces on behalf of the user instead of the kernel, by running as root. This means trusting bubblewrap (a significantly less battle-tested piece of software than the kernel) to run as root. However, some see this as still a preferable tradeoff (trusting one small program with root in exchange for reducing the kernel's attack surface).

Ultimately we leave both options available because it's a tradeoff and neither is demonstrably preferable from a security standpoint. It should also be noted that podman, toolbox, and distrobox require unprivileged user namespaces to function and are therefore not included in the non-userns images.

Canonical considers user namespaces to be a substantial risk, too, and has restricted them via a global AppArmor policy [since 23.10 by opt-in](https://discourse.ubuntu.com/t/spec-unprivileged-user-namespace-restrictions-via-apparmor-in-ubuntu-23-10/37626) and [since 24.04 by default](https://ubuntu.com/blog/whats-new-in-security-for-ubuntu-24-04-lts).

Future endeavours should aim at bringing comparable, granular restrictions via a global SELinux policy, ideally upstream in Fedora.

There is currently an open inquiry as to whether or not the (legacy) SUID-based sandbox in Chromium provides the same sandbox quality as their namespace-based sandbox: https://issues.chromium.org/issues/364119466
