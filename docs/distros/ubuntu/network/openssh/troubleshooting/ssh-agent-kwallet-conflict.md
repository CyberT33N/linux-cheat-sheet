# SSH authentication hangs or fails on Ubuntu with KDE (gcr-ssh-agent vs KWallet)

Symptom family: `git push` / `git fetch` over SSH hangs without a prompt or fails with
`git@github.com: Permission denied (publickey)` even though the key is registered on the
account — usually after a reboot into an **Ubuntu + KDE Plasma** session. Terminal output
may additionally carry `ksshaskpass` noise (`Couldn't start kwalletd: QDBusError(...)`,
`libEGL warning: ...`, `kf.config.core: ...`).

## Root cause: two credential stacks, the wrong one wins

Ubuntu+KDE sessions run **three** credential surfaces at once:

| Surface | Socket / mechanism | Health on KDE |
|---|---|---|
| `gcr-ssh-agent` (GNOME Component Reference — pulled in by GNOME/portal packages) | `/run/user/1000/gcr/ssh` | **Broken on KDE**: accepts `ssh-add` and lists keys, but its secrets live in the GNOME Secret Service and it needs a GNOME prompter that does not exist in a Plasma session → **it silently refuses to sign** |
| `ssh-agent.service` (OpenSSH agent, Ubuntu ships it enabled) | `/run/user/1000/openssh_agent` | Healthy, but loses the environment race |
| KWallet + `pam_kwallet5` + `ksshaskpass` (KDE-native path) | `/run/user/1000/kwallet5.socket`, `SSH_ASKPASS=/usr/bin/ksshaskpass` | Healthy: PAM unlocks the wallet at login, ksshaskpass serves passphrases from it |

`/usr/lib/systemd/user/gcr-ssh-agent.socket` contains the comment *"If gcr is installed,
take priority in setting SSH_AUTH_SOCK"* and its `ExecStartPost` sets
`SSH_AUTH_SOCK=%t/gcr/ssh` **session-wide** — so ssh talks to the broken agent by default.

The decisive diagnostic signature (agent holds the key, server accepts it, signature never
happens):

```text
debug1: Offering public key: /home/testuser/.ssh/github/id_ecdsa ECDSA SHA256:... explicit
debug1: Server accepts key: ... explicit
debug2: we did not send a packet, disable method        ← agent refused to sign
debug1: No more authentication methods to try.
```

## Diagnosis (read-only, one by one)

```bash
# which agent does the desktop session advertise?
tr '\0' '\n' < /proc/$(pgrep -u "$USER" -x plasmashell | head -1)/environ | grep SSH_AUTH_SOCK

# both agent sockets exist?
ls -la /run/user/1000/gcr/ssh /run/user/1000/openssh_agent

# is the key in the healthy agent, and does it authenticate there?
SSH_AUTH_SOCK=/run/user/1000/openssh_agent ssh-add -l
SSH_AUTH_SOCK=/run/user/1000/openssh_agent ssh -o BatchMode=yes \
  -o IdentityAgent=/run/user/1000/openssh_agent -T git@github.com

# is the KDE wallet stack actually healthy (it usually is)?
systemctl --user status plasma-kwallet-pam.service --no-pager | head -4
timeout 10 qdbus6 org.kde.kwalletd6 /modules/kwalletd6 org.kde.KWallet.isOpen kdewallet
```

If the gcr agent lists the fingerprint but the `-T git@github.com` probe ends in
`Permission denied (publickey)` with the `we did not send a packet` line above, the gcr
signing refusal is confirmed.

## Permanent fix (architecturally correct end state)

Remove the broken authority, keep the KDE-native stack, declare one agent path:

```bash
# 1) mask the GNOME agent (disable alone would be revived by preset on package updates)
systemctl --user mask gcr-ssh-agent.socket gcr-ssh-agent.service

# 2) declare the healthy agent for every shell (login shell reads zshenv/zprofile)
echo 'export SSH_AUTH_SOCK=/run/user/1000/openssh_agent' >> ~/.zshenv

# 3) load the auth key into the healthy agent (one prompt now)
SSH_AUTH_SOCK=/run/user/1000/openssh_agent ssh-add ~/.ssh/github/id_ecdsa

# 4) silent re-load at every login: autostart entry fetches the passphrase from KWallet
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/ssh-add-github.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=ssh-add github key
Exec=bash -c 'SSH_ASKPASS_REQUIRE=force DISPLAY=:1 SSH_AUTH_SOCK=/run/user/1000/openssh_agent ssh-add ~/.ssh/github/id_ecdsa >/dev/null 2>&1'
X-KDE-autostart-after=panel
EOF
```

Then log out and in (the session's stale `SSH_AUTH_SOCK` and the gcr socket file only
disappear with a fresh session). Verify:

```bash
echo "$SSH_AUTH_SOCK"                     # → /run/user/1000/openssh_agent
ssh -T git@github.com                     # → "Hi <user>! You've successfully authenticated"
ssh -T git@github.com 2>&1 | grep -c warning   # → 0 (no ksshaskpass call at all anymore)
```

## Emergency workaround (works inside the current session, no re-login)

Bypass agents entirely for the affected host; ssh reads the key file and `ksshaskpass`
serves the passphrase from the already-unlocked KWallet:

```sshconfig
Host github.com
  IdentityAgent none
  IdentitiesOnly yes
```

This authenticates immediately — but every uncached git command spawns `ksshaskpass`, and
because **ssh strips `DBUS_SESSION_BUS_ADDRESS` from askpass helpers**, ksshaskpass prints
`Couldn't start kwalletd: QDBusError(...)` and `libEGL/MESA` warnings on stderr while still
reading the wallet over the direct socket `/run/user/1000/kwallet5.socket`. Exit codes stay
correct; the noise is cosmetic. Use this as the bridge until the permanent fix + re-login
removes the askpass-per-command pattern.

## Notes

- A passphrase-protected auth key is a feature; `ssh -o BatchMode=yes` failing against the
  `IdentityAgent none` path is expected (BatchMode forbids any prompt) — not a regression.
- `BatchMode=yes` tests against a healthy agent DO pass once the key is loaded — use them
  as the noise-free verification lane.
- The signing key used for `git commit -S` (e.g. `id_ed25519_signing`) is a different
  credential concept from the SSH auth key; gcr may list it correctly yet still refuse
  signatures for non-signing keys — keep both keys' paths covered by the healthy agent.
- Keep `~/.ssh/config` host blocks (`IdentityFile`, `User`) untouched; the fix only adds
  agent authority, never rewrites identity selection.
