# Vendor detections

Installer `.exe` — SHA-256
`505134aa3d1ef51be225b3cab7dc3b1549bbd4151c79e9bc1b253b51decc092e`

VirusTotal first analysis: **5 / 70 detections.** A low first-day count is normal for a
freshly packed loader; re-scan later for a higher count.

| Vendor | Verdict |
|---|---|
| ESET-NOD32 | `Win64/Kryptik.GXY` trojan |
| Elastic | Malicious (high confidence) |
| Rising | `Trojan.ShellCodeLoader!1.12EA8 (CLASSIC)` |
| Tencent | `Trojan.Win64.Kryptik.16003858` |
| SecureAge | Malicious |

- **Popular threat label:** `trojan.shellcodeloader`
- **Threat category:** trojan
- **Family label:** `shellcodeloader`

The `Win64` label on a 32-bit Inno Setup installer refers to the **packed 64-bit
payload** carried inside the installer, not the installer stub itself.
