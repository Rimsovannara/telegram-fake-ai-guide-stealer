# Telegram Desktop session theft (`tdata`) — defensive detail

Stealers in this family target the Telegram Desktop session folder. This note explains
**why** that folder is enough to take over an account, so defenders can recognise and
prevent it. It contains no offensive code or steps.

## Location

```
%AppData%\Telegram Desktop\tdata\
├─ key_datas              # local encryption key for the profile
├─ D877F783D5D3EF8C\      # per-account data directory
│   ├─ maps               # index of the encrypted stores
│   └─ ...                # cached messages, media, settings
├─ D877F783D5D3EF8Cs\     # settings for that account
└─ ...
```

(Directory names vary between versions.)

## Why copying it = copying the login

- **Not bound to hardware.** By default the session is not tied to your machine, disk,
  or Windows user. Dropped onto another PC's `tdata`, it authenticates as you.
- **No local passcode by default.** Without a Local Passcode, `key_datas` is not gated
  by a secret only you know.
- **Bypasses login & 2FA.** Cloud password and SMS/app codes guard **new** logins.
  Restoring an existing session never triggers that flow — the attacker is already
  "logged in."
- **Often silent.** The stolen session appears only as another active device, easy to
  miss unless you check.

The same principle applies to browser cookie/token stores, which is why stealers grab
Telegram **and** browser data in one pass.

## Defenses

| Action | Where | Effect |
|---|---|---|
| Set a **Local Passcode** | Settings → Privacy | Encrypts the session with a secret not stored in `tdata`; a stolen copy can't be unlocked. **Most effective.** |
| **Two-Step Verification** | Privacy → 2-Step | Guards new logins and account reset. |
| **Review Active Sessions** | Settings → Devices | Terminate unrecognised clients; instantly kills a stolen session. |
| **Auto-terminate idle sessions** | Devices → If inactive | A stolen session expires on its own. |

## If compromise is suspected

From a **different** device: terminate all sessions, change the cloud password, rotate
all browser-saved passwords, and move crypto to new wallets. Then reinstall the affected
PC — deleting the malware file does not undo prior exfiltration.
