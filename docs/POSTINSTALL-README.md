# secureblue

After rebasing to secureblue, follow the following steps in order.

## Subscribe to secureblue release notifications

[FAQ](FAQ.md#releases)

## Nvidia
If you are using an nvidia image, run this after installation:

```
ujust set-kargs-nvidia
```

You may also need this (solves flickering and luks issues on some nvidia hardware):
```
rpm-ostree kargs \
    --append-if-missing=initcall_blacklist=simpledrm_platform_driver_init
```

### Nvidia optimus laptop
If you are using an nvidia image on an optimus laptop, run this after installation:

```
ujust configure-nvidia-optimus
```

## Enroll secureboot key

```
ujust enroll-secure-boot-key
```

## Set hardened kargs

> [!NOTE]
> Learn about the hardening applied by the kargs set by the command below [here](KARGS.md).

```
ujust set-kargs-hardening
```
This command applies a fixed set of hardened boot parameters, and asks you whether or not the following kargs should *also* be set along with those (all of which are documented in the link above):

### 32-bit support
If you answer `N`, or press enter without any input, support for 32-bit programs will be disabled on the next boot. If you run exclusively modern software, chances are likely you don't need this, so it's safe to disable for additional attack surface reduction.

However, there are certain exceptions. A couple common usecases are if you need Steam, or run an ocassional application in Wine you'll likely want to keep support for 32-bit programs. If this is the case, answer `Y`.

### Force disable simultaneous multithreading
If you answer `Y` when prompted, simultaneous multithreading (SMT, often called Hyperthreading) will be disabled on all hardware, regardless of known vulnerabilities. This can cause a reduction in the performance of certain tasks in favor of security.

### Unstable hardening kargs
If you answer `Y` when prompted, unstable hardening kargs will be additionally applied, which can cause issues on some hardware, but are stable on other hardware.

## Setup USBGuard

*This will generate a policy based on your currently attached USB devices and block all others, then enable usbguard.*

```
ujust setup-usbguard
```

## GRUB
### Set a password

Setting a GRUB password helps protect the device from physical tampering and mitigates various attack vectors, such as booting from malicious media devices and changing boot or kernel parameters.

To set a GRUB password, use the following command. By default, the password will be required when modifying boot entries, but not when booting existing entries.

```
sudo grub2-setpassword
```

GRUB will prompt for a username and password. The default username is root.

If you wish to password-protect booting existing entries, you can add the `grub_users root` entry in the specific configuration file located in the `/boot/loader/entries` directory.

## Create a separate wheel account for admin purposes

Creating a dedicated wheel user and removing wheel from your primary user helps prevent certain attack vectors, like:

- https://www.kicksecure.com/wiki/Dev/Strong_Linux_User_Account_Isolation#LD_PRELOAD
- https://www.kicksecure.com/wiki/Root#Prevent_Malware_from_Sniffing_the_Root_Password

> [!CAUTION]
> If you do these steps out of order, it is possible to end up without the ability to administrate your system. You will not be able to use the [traditional GRUB-based method](https://linuxconfig.org/recover-reset-forgotten-linux-root-password) of fixing mistakes like this, either, as this will leave your system in a broken state. However, simply rolling back to an older snapshot of your system, should resolve the problem.
> 
1. `run0`
2. `adduser admin`
3. `usermod -aG wheel admin`
4. `passwd admin`
5. `exit`
6. `reboot`

> [!NOTE]
> We log in as admin to do the final step of removing the user account's wheel privileges in order to make the operation of removing those privileges depend on having access to your admin account, and the admin account functioning correctly first.

5. Log in as `admin`
6. `run0 gpasswd -d {your username here} wheel`
7. `reboot`

When using a non-wheel user, you can add the user to other groups if you want. For example:

- use libvirt: `libvirt`
- use `adb` and `fastboot`: `plugdev`
- use systemwide flatpaks: `flatpak`

## Setup system DNS

Interactively setup system DNS resolution for systemd-resolved (optionally also set the resolver for hardened-chromium via management policy):

```
ujust dns-selector
```

NOTE: If you intend to use a VPN, use the system default state (network provided resolver). This will ensure your system uses the VPN provided DNS resolver to prevent DNS leaks. ESPECIALLY avoid setting the browser DNS policy in this case.

## Bash environment lockdown

To mitigate [LD_PRELOAD attacks](https://github.com/Aishou/wayland-keylogger), run:

```
ujust toggle-bash-environment-lockdown
```

## LUKS TPM2 Unlock

> [!WARNING]
> Do not use this if you have an AMD CPU.

To enable TPM2 LUKS unlocking, run:

```
ujust setup-luks-tpm-unlock
```
Type `Y` when asked if you want to set a PIN.

## Validation

To validate your secureblue setup, run:

```
ujust audit-secureblue
```

## Optional: `hardened-chromium` Flags
The included [hardened-chromium](https://github.com/secureblue/hardened-chromium) browser has some additional settings in `chrome://flags` you *may* want to set for additional hardening and convenience (can cause functionality issues in some cases).
You can read about these settings [here](https://github.com/secureblue/hardened-chromium?tab=readme-ov-file#post-install).

## Read the FAQ

Lots of important stuff is covered in the [FAQ](FAQ.md). AppImage toggles, GNOME extension toggles, Xwayland toggles, etc.
