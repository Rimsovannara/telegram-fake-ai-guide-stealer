# Fake "AI programming guide" → Telegram session stealer

Static analysis and indicators for a `trojan.shellcodeloader` sample that was sent over
Telegram disguised as a programming tutorial.

This repo has hashes, indicators, and notes only — **no sample or payload**. Nothing here
can be run to reproduce the attack.

## What it is

The file arrived over Telegram with the caption *"If your mobile phone is not compatible
for viewing, please use a computer to view the tutorial."* It's not a document — it's a
Windows installer (Inno Setup 6.3) that carries a packed 64-bit payload. The caption is
the social-engineering part: the payload can't run on a phone, so it pushes the target to
a Windows PC.

The payload is a shellcode loader — it decrypts code and runs it in memory, which is why
so few engines flag it at first. Loaders like this usually drop an info-stealer that goes
after browser credentials, crypto wallets, and the Telegram Desktop session (`tdata`),
which is enough to take over the account without a password or 2FA prompt.

## Sample

| | |
|---|---|
| Presented as | `2026 High-Efficiency Guide to Using AI for Computer Programming` |
| Container | RAR5 archive named with a `.7z` extension |
| Inner file | Windows PE32, Inno Setup 6.3.0 |
| Payload | packed x86-64 PE, high entropy |
| Signature | none |
| Analysis | static only — never executed |

### Hashes (SHA-256)

```
3b305ebee74814e35941fcc6f9517a542e4c22ac14d9379376b1721054abdcf5  archive (RAR5, .7z name)
505134aa3d1ef51be225b3cab7dc3b1549bbd4151c79e9bc1b253b51decc092e  installer .exe
```

VirusTotal: [archive](https://www.virustotal.com/gui/file/3b305ebee74814e35941fcc6f9517a542e4c22ac14d9379376b1721054abdcf5)
· [installer](https://www.virustotal.com/gui/file/505134aa3d1ef51be225b3cab7dc3b1549bbd4151c79e9bc1b253b51decc092e)

## Detections

- Installer: 5/70 — ESET `Win64/Kryptik.GXY`, Elastic (high), Rising `ShellCodeLoader`,
  Tencent `Kryptik`, SecureAge.
- Archive: 2/64 — ESET `Win64/Kryptik.GXY`, Rising `ShellCodeLoader`.
- VT sandbox tags: `detect-debug-environment`, `long-sleeps` — anti-analysis, which is
  why the static count stays low.

The `Win64` label on a 32-bit installer refers to the packed 64-bit payload inside.

## Static findings

- Named `.7z` but the bytes are a RAR5 archive.
- The archive holds a single `.exe` — a real guide would be a PDF/EPUB.
- ~20 MB compressed block (Inno `zlb`, entropy ≈ 8.0), opaque to string analysis.
- A valid packed x86-64 PE sits at offset `0x0ba408`; its strings are noise (encrypted).
- Version resource is faked: blank publisher/copyright, random ProductName (`L4XH3.exe`),
  nonsense version (`18.606.696.443`).
- No plaintext C2 — the config is inside the encrypted payload.

## Why the Telegram session is the target

Telegram Desktop keeps the logged-in session in `%AppData%\Telegram Desktop\tdata\`. By
default it isn't tied to the machine and isn't protected by a local passcode, so copying
that folder is the same as copying the login — and restoring an existing session never
triggers the 2FA that guards new logins. Details in
[`iocs/telegram-tdata.md`](iocs/telegram-tdata.md).

## Defending against it

- Set a **Local Passcode** in Telegram Desktop — encrypts the session with a secret
  that isn't in `tdata`, so a stolen copy can't be opened. This is the main defense.
- Turn on Two-Step Verification.
- Check Settings → Devices and terminate sessions you don't recognise; set them to
  auto-terminate after a short idle window.
- Don't run "documents" that are executables. Check unknown files by hash on VirusTotal.

If it was already run: from another device, terminate all Telegram sessions, change the
cloud password, rotate browser-saved passwords, move any crypto to new wallets, then
reinstall the PC.
