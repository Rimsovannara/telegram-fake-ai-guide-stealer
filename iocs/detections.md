# Vendor detections

Both artifacts carry the same popular threat label — **`trojan.shellcodeloader`**
(category: trojan; family: `shellcodeloader`) — and are flagged by the same two
high-signal engines. A low count is normal for a freshly packed loader that evades
sandboxes (see behavior tags below); re-scan later for a higher count.

## Installer `.exe`

SHA-256 `505134aa3d1ef51be225b3cab7dc3b1549bbd4151c79e9bc1b253b51decc092e`
VirusTotal: **5 / 70**

| Vendor | Verdict |
|---|---|
| ESET-NOD32 | `Win64/Kryptik.GXY` trojan |
| Elastic | Malicious (high confidence) |
| Rising | `Trojan.ShellCodeLoader!1.12EA8 (CLASSIC)` |
| Tencent | `Trojan.Win64.Kryptik.16003858` |
| SecureAge | Malicious |

## Outer archive (`.7z`-named RAR5)

SHA-256 `3b305ebee74814e35941fcc6f9517a542e4c22ac14d9379376b1721054abdcf5`
VirusTotal: **2 / 64**

| Vendor | Verdict |
|---|---|
| ESET-NOD32 | `Win64/Kryptik.GXY` trojan |
| Rising | `Trojan.ShellCodeLoader!1.12EA8 (CLASSIC)` |

## VirusTotal tags & sandbox behavior (archive)

- File tags: `rar`, `malware`
- Behavior: **`detect-debug-environment`**, **`long-sleeps`**

Both behavior tags are **anti-analysis** techniques: the payload checks for a
debugger/analysis environment and uses long sleeps to outlast automated sandbox
timeouts. This evasion is a direct reason the static detection count stays low.

---

The `Win64` label on a 32-bit Inno Setup installer / RAR archive refers to the **packed
64-bit payload** carried inside, not the installer stub or the container itself.
