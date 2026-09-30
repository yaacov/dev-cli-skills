---
name: docker-distro
description: Debug distributions and specific versions by running the distro's Docker image with a small test script — check what is installed, which files and configs exist, and verify behavior. Use when the user wants to inspect a distro or image, verify what a package installs, check config files, compare versions, or test behavior inside a distro image.
---

# docker-distro

1. Pick image from [ref-images.md](ref-images.md) (confirm version or print `/etc/os-release` first).
2. Write a small `sh` script: one `PASS:` / `FAIL:` line per assertion; use `set -u`, **not** `set -e`, so the full report still runs after a failure.
3. `docker run --rm` with the script mounted read-only; iterate or use `--entrypoint sh -it` / `sleep infinity` + `exec` for longer sessions.

## Rules (easy to get wrong)

- **Arch:** on Apple Silicon use `--platform linux/amd64` or `linux/arm64` when testing a specific arch — wrong arch fails quietly or under emulation.
- **Minimal / Alpine:** often no `bash` or `curl`; use `sh` and install tools in the disposable run.
- **No systemd** in the container — verify unit files under `/etc/systemd`, not `systemctl`.
- **Offline behavior:** `docker run --network none` when the case must not reach the network.
- **Cleanup:** confirm before `docker rm`, `docker rmi`, or `prune`.

## Run

```bash
docker run --rm --platform linux/amd64 \
  -v /tmp/dist-check.sh:/tmp/dist-check.sh:ro \
  <image-from-ref-images> sh /tmp/dist-check.sh
```

**Local RPM on UBI/RHEL family:** mount the file and `dnf install -y /tmp/pkg.rpm` in the same one-shot. See [ref-images.md](ref-images.md) for UBI `curl-minimal` vs `curl` (`--allowerasing`).

**Compare versions:** same script, multiple tags; diff captured stdout.

**Config drift on RPM systems:** dnf may leave `<file>.rpmorig` when an `/etc` file was updated — useful when diffing package vs live config.
