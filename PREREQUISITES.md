# ATLAS — Prerequisites

> **Audience:** Shinynecrid community administrators only.
> This module is privately maintained and not intended for public deployment.

The ATLAS dedicated server binary (`ShooterGameServer`) was compiled against
**OpenSSL 1.0.0** and has never been updated by the developer. This requires
a manual one-time setup step on Linux hosts before the server can launch.

---

## Windows

No additional prerequisites. The Windows server binary has no OpenSSL dependency.

| Requirement | Notes |
|---|---|
| Windows Server 2019/2022 or Windows 10/11 (64-bit) | |
| [VC++ Redistributable 2015-2022 x64](https://aka.ms/vs/17/release/vc_redist.x64.exe) | Required by the Unreal Engine binary |

---

## Linux — AlmaLinux 8 / 9

```bash
sudo dnf install -y compat-openssl10
```

---

## Linux — AlmaLinux 10

`compat-openssl10` was removed from RHEL 10 repos. Install it manually from
the AlmaLinux 9 package:

```bash
wget https://repo.almalinux.org/almalinux/9/AppStream/x86_64/os/Packages/compat-openssl10-1.0.2o-5.el9.x86_64.rpm
sudo rpm -ivh --nodeps compat-openssl10-1.0.2o-5.el9.x86_64.rpm
sudo ldconfig
```

### Verify

```bash
ldconfig -p | grep libssl.so.1.0
```

Expected output:
```
libssl.so.1.0.0 (libc6,x86-64) => /usr/lib64/libssl.so.1.0.0
```

---

## Podman / Container Note

Libraries installed on the host may not be visible inside the AMP Podman
container. If ATLAS still fails to find `libssl.so.1.0.0` after installing
on the host, contact your hosting provider to have it included in the
container image or bind-mounted into the instance.
