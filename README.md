# Fake "AI Programming Guide" → Telegram Session Stealer

Static malware analysis and Indicators of Compromise (IOCs) for a
**`trojan.shellcodeloader`** sample distributed over Telegram while disguised as a
programming tutorial.

> **Defensive research only.** This repository contains **hashes, IOCs, and analysis
> notes** — **no malware sample or payload is included**. Nothing here can be run to
> reproduce the attack. See [DISCLAIMER](#disclaimer).

---

## Summary

A file was delivered via Telegram with the social-engineering caption:

> *"If your mobile phone is not compatible for viewing, please use a computer to view
> the tutorial. Thank you."*

The "tutorial" is **not a document**. It is a Windows installer that unpacks and runs a
packed 64-bit payload. The caption exists to move the target from a safe mobile device
to a vulnerable Windows PC, since the payload cannot execute on Android or iOS.

The payload is a **shellcode loader** — it decrypts code and runs it in memory (leaving a
minimal on-disk footprint), which is consistent with its low first-day antivirus
detection. Loaders of this class typically deliver an **information stealer** that
harvests browser credentials, crypto wallets, and — the focus of this write-up — the
**Telegram Desktop session** (`tdata`), enabling full account takeover **without a
password or 2FA prompt**.

---

## Sample metadata

| Property | Value |
|---|---|
| Presented as | `2026 High-Efficiency Guide to Using AI for Computer Programming` |
| Delivery | Telegram file + lure caption |
| Outer container | **RAR5** archive named with a `.7z` extension |
| Inner file | Windows PE32 executable (32-bit GUI) |
| Installer framework | **Inno Setup 6.3.0** |
| Embedded payload | **packed x86-64 (PE32+) executable**, high entropy |
| Fake version resource | ProductName `L4XH3.exe`, ProductVersion `18.606.696.443`, blank publisher/copyright |
| Digital signature | none |
| Analysis method | **static only — never executed** |

### Hashes (SHA-256)

| Artifact | Size (bytes) | SHA-256 |
|---|---|---|
| Outer archive (`.7z`-named RAR5) | 21,316,997 | `3b305ebee74814e35941fcc6f9517a542e4c22ac14d9379376b1721054abdcf5` |
| Installer `.exe` | 21,783,784 | `505134aa3d1ef51be225b3cab7dc3b1549bbd4151c79e9bc1b253b51decc092e` |

VirusTotal (look up by hash — sample not distributed here):
- [Archive](https://www.virustotal.com/gui/file/3b305ebee74814e35941fcc6f9517a542e4c22ac14d9379376b1721054abdcf5)
- [Installer](https://www.virustotal.com/gui/file/505134aa3d1ef51be225b3cab7dc3b1549bbd4151c79e9bc1b253b51decc092e)

---

## Detections

VirusTotal, installer `.exe` — **5 / 70** at first analysis (typical for a freshly
packed loader; expect the count to rise on re-scan).

| Vendor | Verdict |
|---|---|
| ESET-NOD32 | `Win64/Kryptik.GXY` trojan |
| Elastic | Malicious (high confidence) |
| Rising | `Trojan.ShellCodeLoader!1.12EA8` |
| Tencent | `Trojan.Win64.Kryptik.16003858` |
| SecureAge | Malicious |

- **Popular threat label:** `trojan.shellcodeloader`
- **Family label:** `shellcodeloader`

Note the **`Win64`** label on a 32-bit installer: it refers to the packed **64-bit**
payload carried inside.

---

## Static analysis findings

1. **Extension mismatch.** The download is named `.7z` but its bytes are a **RAR5**
   archive (`Rar5`, method `v6:32M:m3`). Container/extension mismatch is a common way
   to evade naming-based filters.
2. **Document → executable.** The archive holds a single `.exe`. A legitimate guide
   would be a PDF/EPUB, not a program.
3. **Inno Setup wrapper.** The `.exe` is an Inno Setup 6.3.0 installer. The bulk of the
   file (~20 MB) is a compressed block (`zlb` magic, overlay entropy ≈ 8.0) — opaque to
   static string analysis.
4. **Embedded packed 64-bit PE.** A valid `PE32+` (x86-64, 6 sections) is embedded at
   file offset `0x0ba408`. Its strings are pure high-entropy noise → **packed /
   encrypted**, matching the ShellCodeLoader / Kryptik classification.
5. **Forged version resource.** Publisher, description, and copyright fields are blank
   padding; ProductName is a random token (`L4XH3.exe`) and the version is nonsense
   (`18.606.696.443`).
6. **No plaintext C2.** No URLs, IPs, or domains recoverable from the file — the C2
   configuration is inside the encrypted payload and would require dynamic analysis to
   extract (not performed).
7. **Stale build stamp.** The installer stub carries a 2024-06-10 compile timestamp
   against a 2026 archive date — a reused builder.

---

## Attack chain

| # | Stage | Description |
|---|---|---|
| 1 | Delivery & disguise | Telegram file, `.7z`-named RAR, document-style name, blank publisher. |
| 2 | Execution | Victim opens `.exe` on Windows; Inno Setup drops the packed 64-bit loader. |
| 3 | Shellcode loading | Loader decrypts and runs code **in memory** → minimal disk footprint, low AV detection. |
| 4 | Harvest | In-memory stealer sweeps browser stores, crypto wallets, and Telegram Desktop `tdata`. |
| 5 | Exfiltration | Collected data zipped and uploaded (attacker server or Telegram bot channel). |

---

## Why the Telegram session is the prize

Telegram Desktop stores logged-in state in `%AppData%\Telegram Desktop\tdata\`
(`key_datas`, an account directory, and a `maps` index). Once you are signed in, **this
folder is the account key**:

- **Not bound to your machine** by default — it works on another computer as-is.
- **No local passcode** unless you set one — the copy opens without a prompt.
- **Bypasses login & 2FA** — restoring an existing session never triggers the new-login
  flow that a cloud password or SMS/app code would guard.

Copying `tdata` is therefore equivalent to copying the login. See
[`iocs/telegram-tdata.md`](iocs/telegram-tdata.md).

---

## Defense & remediation

**Harden Telegram Desktop**
- Set a **Local Passcode** (Settings → Privacy) — the single most effective step; a
  stolen `tdata` cannot be unlocked without it.
- Enable **Two-Step Verification** (cloud password) with a recovery email you control.
- **Review Active Sessions** (Settings → Devices) and terminate anything unrecognised.
- Set sessions to **auto-terminate** after the shortest acceptable idle window.

**General**
- Never run "documents" that are executables/installers. Verify unknown files by hash on
  VirusTotal before opening.
- Treat "open it on a computer instead" as a red flag.

**If it was executed — assume compromise.** From a *different* device: terminate all
Telegram sessions, change the cloud password, rotate all passwords saved in the PC's
browser, and move crypto to new wallets. Then reinstall the affected PC — deleting the
file does not undo what a stealer already exfiltrated.

---

## Repository contents

```
README.md                 – this analysis
iocs/hashes.txt           – SHA-256 hashes (machine-readable)
iocs/telegram-tdata.md    – Telegram session-theft mechanism (defensive detail)
iocs/detections.md        – vendor detection table
```

---

## Disclaimer

This repository is published for **defensive security education and threat
intelligence**. It contains only hashes, indicators, and analysis notes describing
observed attacker behaviour at a conceptual level. **No malicious sample, payload, or
step-by-step offensive instructions are included.** The sample was analysed **statically
and never executed**; no file was uploaded to any third-party service. Vendor names and
verdicts are reproduced from public VirusTotal results. Use this information to detect,
defend against, and remediate the described threat.
