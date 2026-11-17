# Install OpenSSH 10.6p1 from verified source (Ubuntu)

Verified procedure for installing a current upstream OpenSSH release on Ubuntu when the
distribution archive pins an older version. Every artifact is pinned and cryptographically
verified before execution. Run all steps on the machine that will use the binaries.

## Prerequisites

- Ubuntu 25.10 (questing) or newer; `build-essential`, `libssl-dev`, `zlib1g-dev` installed
  (verify with `dpkg -l build-essential libssl-dev zlib1g-dev`).
- FIDO2/security-key support requires the development headers:

```bash
sudo apt install -y libfido2-dev libcbor-dev libpam0g-dev
```

- Verify the headers are visible to the build (each command is a separate step):

```bash
ls -l /usr/include/fido/credman.h
/usr/bin/pkg-config --modversion libfido2
```

## Collision check before any install into /usr/local

A previous `firecfg` run may have replaced `/usr/local/bin` entries with symlinks to
`/usr/bin/firejail`. `make install` follows existing symlinks and would write through them,
which can corrupt the sandbox launcher. Check first:

```bash
find /usr/local/bin /usr/local/sbin -maxdepth 1 -type l -lname '*firejail*'
```

If `ssh`, `scp`, `sftp`, `ssh-keygen`, `ssh-add` or `ssh-agent` appear, remove only those
specific symlinks before installing (firejail itself stays untouched):

```bash
sudo rm /usr/local/bin/ssh
```

## Download, verify, build, install

```bash
build_dir=$(mktemp -d "${TMPDIR:-/tmp}/openssh-10.6p1-build.XXXXXXXX")
cd "$build_dir"
curl -fSLO https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/RELEASE_KEY.asc
curl -fSLO https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-10.6p1.tar.gz
curl -fSLO https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-10.6p1.tar.gz.asc
mkdir -p gnupg && chmod 700 gnupg
gpg --homedir "$build_dir/gnupg" --batch --import RELEASE_KEY.asc
gpg --homedir "$build_dir/gnupg" --batch --verify openssh-10.6p1.tar.gz.asc openssh-10.6p1.tar.gz
openssl dgst -sha256 -binary openssh-10.6p1.tar.gz | openssl base64
```

- The signature must be `Good signature` from `Damien Miller <djm@mindrot.org>`.
- The base64 SHA-256 digest must match the value published in the release notes at
  `https://www.openssh.com/releasenotes.html` for `openssh-10.6p1.tar.gz`.
- Stop on any mismatch; never continue with an unverified tarball.

```bash
tar zxf openssh-10.6p1.tar.gz
cd openssh-10.6p1
```

If the interactive shell loads conda or other toolchains into `PATH`, pin the build to the
system toolchain so headers and libraries stay consistent:

```bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
./configure --prefix=/usr/local \
  --sysconfdir=/etc/ssh \
  --with-pam \
  --with-sandbox=seccomp_filter \
  --with-security-key-builtin \
  --with-pie
make -j"$(nproc)"
make tests
sudo make install
```

Configure flag rationale:

| Flag | Effect |
|---|---|
| `--prefix=/usr/local` | Installs client binaries under `/usr/local/{bin,sbin}`; the packaged `sshd` service keeps running from `/usr/sbin/sshd` |
| `--sysconfdir=/etc/ssh` | Shares one configuration truth with the packaged tools |
| `--with-pam` | Pluggable authentication support for sshd |
| `--with-sandbox=seccomp_filter` | Privilege-separation sandbox for sshd on Linux |
| `--with-security-key-builtin` | Built-in FIDO2/U2F support (needs libfido2 headers) |
| `--with-pie` | Position-independent executables for hardening |

## Post-install verification

```bash
/usr/local/bin/ssh -V
/usr/local/bin/ssh-keygen -V
/usr/local/bin/ssh -Q key-sig | grep mldsa
/usr/local/bin/ssh -Q KexAlgorithms | grep -E 'mlkem|sntrup'
```

Expected: `OpenSSH_10.6p1`, `ssh-mldsa44-ed25519` in the key signature list, and
`mlkem768x25519-sha256` plus `sntrup761x25519-sha512` in the KEX list.

Resolution check (which binary wins for interactive shells):

```bash
command -v ssh ssh-keygen
```

## Rollback

- Remove the installed binaries: `sudo make -C "$build_dir/openssh-10.6p1" uninstall`
  (or delete the files listed in the install log under `/usr/local`).
- Restore a pre-change system snapshot if the package manager created one automatically
  (e.g. a `timeshift-autosnap-apt` BTRFS snapshot taken during `sudo apt install`).
- The packaged `openssh-server`/`sshd` service is unaffected by a `/usr/local` install.
