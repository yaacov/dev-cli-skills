# Distro → image catalog

On Apple Silicon, add `--platform linux/amd64` unless testing arm64.

| Distro | Image | Notes |
|--------|-------|-------|
| RHEL (via UBI) | `registry.access.redhat.com/ubi<ver>` (e.g. `ubi9`) | closest public RHEL userland |
| CentOS Stream | `quay.io/centos/centos:stream<ver>` | |
| Fedora | `fedora:<ver>` (`latest` = current) | |
| Rocky Linux | `rockylinux:<ver>` | |
| AlmaLinux | `almalinux:<ver>` | |
| Amazon Linux 2 | `amazonlinux:2` | |
| Amazon Linux 2023 | `amazonlinux:2023` | |
| openSUSE Leap | `opensuse/leap:<ver>` (e.g. `15.6`) | |
| SLES | `suse/sles<ver>` (e.g. `sles15`) | repos may need registration |
| Debian | `debian:<ver>` — `stable`, `12`/`bookworm`, `13`/`trixie` | |
| Ubuntu | `ubuntu:<ver>` (`24.04`, `22.04`) | |
| Alpine | `alpine:<ver>` (e.g. `3.20`) | `sh` only by default |
| busybox | `busybox:stable` | |
| Windows Server | `mcr.microsoft.com/windows/servercore:<ver>` | PowerShell — not for Linux-style `sh` checks |

## Gotchas

- **UBI `curl`:** on UBI 9, `curl-minimal` conflicts with full `curl` — `dnf install -y --allowerasing curl`.
- **UBI `dnf` warnings** about entitlement/consumer identity are noise; `ubi-*-rpms` repos work unregistered.
- **SLES:** `zypper refresh` may fail until the image is registered.
- **ENTRYPOINT:** some images (e.g. SLES) start a daemon — use `--entrypoint sh` for a shell-only container.
