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

## Deployment Recommendation — Host OS (Non-Containerized)

> [!IMPORTANT]
> **ATLAS must be run on the host OS directly. Podman / containerized AMP deployments are not supported.**

Because `ShooterGameServer` requires the legacy `libssl.so.1.0.0` library,
and AMP's Podman container image does not include it, the server will fail
to start in a containerized environment.

When installing AMP, select the **non-containerized** (host OS) deployment
option. This ensures that system libraries installed via `dnf` or `rpm` are
immediately visible to the ATLAS process without any additional configuration.

### Why not Podman?

- The `compat-openssl10` library installed on the host is **not automatically
  visible** inside the Podman container AMP uses for isolation.
- Bind-mounting the library into the container requires custom hosting
  configuration that is outside the scope of this module.
- ATLAS's multi-grid architecture (multiple server processes + Redis) is
  simpler to manage without an additional container networking layer.

If your hosting provider only offers containerized AMP, ATLAS is not
compatible with that environment.

---

## Security Hardening

> [!WARNING]
> `compat-openssl10` installs a library that reached **end-of-life on December 31, 2019**
> and will never receive security patches. Take the precautions below before exposing
> this server to the internet.

### OpenSSL 1.0.x Risk Scope

The legacy library only affects processes that explicitly link against it — specifically
`ShooterGameServer`. Modern system tools and other game servers compiled against
OpenSSL 1.1 or 3.x are **not impacted**. The exposure only exists while the ATLAS
process is actively running.

### RCON — Firewall Required

RCON uses the legacy SSL stack and must never be exposed to the public internet.

Restrict the RCON port to trusted admin IPs only:

```bash
# Replace <RCON_PORT> and <TRUSTED_IP> with your values
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="<TRUSTED_IP>" port port="<RCON_PORT>" protocol="tcp" accept'
sudo firewall-cmd --reload
```

### Run Under a Dedicated Unprivileged User

Do not run `ShooterGameServer` as `root` or as the AMP service account. Create a
dedicated system user with no login shell and no sudo access:

```bash
sudo useradd -r -s /sbin/nologin atlasserver
```

Configure AMP to run the ATLAS instance under this user. If a vulnerability in the
legacy SSL library is exploited, blast radius is limited to the permissions of this
account only.
