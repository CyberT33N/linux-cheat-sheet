# Update OpenSSH on Ubuntu

Two lawful update paths exist. Choose deliberately; never mix them silently on one host.

## Path A - Distribution packages (pinned, zero maintenance)

```bash
sudo apt update
apt list -upgradable 'openssh*'
sudo apt install -y openssh-client openssh-server openssh-sftp-server
sudo systemctl restart ssh
```

- Ubuntu ships a patched upstream version; on 25.10 (questing) the archive pins the
  `1:10.0p1-5ubuntu5.x` series. Security fixes arrive as patch increments of that series.
- This path never reaches a newer upstream major/minor than the release carries.
- Check afterwards: `dpkg -l 'openssh*' | grep '^ii'` and `ssh -V`.

## Path B - Upstream source update (current features)

Use when newer upstream features are required (for example the post-quantum hybrid
signature `ssh-mldsa44-ed25519`, available from OpenSSH 10.6). Repeat the full verified
procedure from [install.md](./install.md) with the new version number:

1. Resolve the current release and its published SHA-256 digest from
   `https://www.openssh.com/releasenotes.html`.
2. Download tarball + `.asc` + `RELEASE_KEY.asc` from `https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/`.
3. Verify GPG signature and base64 SHA-256 digest before building.
4. Re-run the collision check for `/usr/local/bin` firejail symlinks before `make install`.
5. Build with the same configure flags; `make tests` must pass before installing.
6. `sudo make install`, then verify `ssh -V`, `ssh-keygen -V`, `ssh -Q key-sig | grep mldsa`.

## Detecting a version split (client vs tools)

A `/usr/local` install shadows the packaged client only if `PATH` prefers it. Check every
entry point explicitly:

```bash
command -v ssh ssh-keygen ssh-add ssh-agent sshd
/usr/local/bin/ssh -V
/usr/bin/ssh-keygen -V
/usr/sbin/sshd -V
```

- `command -v` lists the binary each name resolves to; mixed `/usr/local` and `/usr/bin`
  results mean a split install.
- The running sshd service is the one systemd starts, normally `/usr/sbin/sshd`
  (package path). A source build under `/usr/local` does not change the service.
- A stale `/usr/local/bin/ssh` from an older source build keeps winning for interactive
  shells; remove it deliberately when reverting to Path A: `sudo rm /usr/local/bin/ssh`.

## Security notes

- Never update by piping remote scripts into a shell; only pinned, signed release artifacts.
- OpenSSH 10.1+ warns when a connection uses a non post-quantum key exchange; keep the
  default `WarnWeakCrypto` enabled and upgrade servers that trigger the warning.
- After any update, rotate nothing automatically: existing keys stay valid; only keys in
  the earlier experimental post-quantum format (with the `@openssh.com` suffix) must be
  regenerated as `ssh-mldsa44-ed25519`.
- Rollback: Path A restores packaged binaries via `sudo apt install --reinstall
  openssh-client openssh-server`; Path B removes the `/usr/local` files as described in
  [install.md](./install.md).
