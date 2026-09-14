<p align="center">
  <img src="banner.svg" alt="EliteDesk 800 G2 DM Hackintosh EFI" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-Sequoia%2015-black?style=flat-square&logo=apple&logoColor=white">
  <img src="https://img.shields.io/badge/OpenCore-1.0.5-blue?style=flat-square">
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square&color=30d158">
  <img src="https://img.shields.io/badge/Tested-v2.14.0-blue?style=flat-square&color=0a84ff">
  <img src="https://img.shields.io/badge/Support-None-critical?style=flat-square&color=ff453a">
</p>

**A complete, working OpenCore EFI for the HP EliteDesk 800 G2 DM (65W) — i7-6700, HD 530 spoofed to Kaby Lake, fully accelerated.**
Built the hard way, over months, so you don't have to. Shared publicly in case it saves someone else the time. Nothing more.

---

## What's actually in this

- **Accelerated graphics** — HD 530 spoofed as Kaby Lake (`ig-platform-id 0x59120000`), correct DVMT stolen-memory split for HP's 32MB BIOS default, all three framebuffer connectors force-typed for real-world monitor compatibility
- **Full USB port map** — every physical port on this board mapped by address, not guessed, including the front USB-C combo port, tested and confirmed live rather than assumed
- **Working native sleep (S3)** — the three ACPI binary patches and SSDTs this board actually needs (EC rename, HP RTC power-loss patch, GPRW→XPRW), not a generic template
- **Bluetooth that survives non-genuine hardware** — the board-id bypass and NVRAM keys the internal BCM20702A0 needs to pass Apple's validation
- **A SMBIOS generator that isn't a copy-paste serial** — see below

## Hardware this targets

| Component | Status |
|---|---|
| CPU | Intel Core i7-6700 (Skylake) |
| iGPU | HD 530, spoofed Kaby Lake — accelerated |
| Ethernet | Intel I219-LM — native |
| Audio | Realtek ALC221 — native |
| Bluetooth | Broadcom BCM20702A0 (internal) — working |
| Wi-Fi | Broadcom BCM43228 — **no macOS driver exists, don't ask, see below** |
| Sleep | S3 — working |
| USB | All ports mapped and tested, including front USB-C |

If your board isn't this exact model, this EFI is a reference at best. It will very likely not just work.

## Quick start

Pick whichever matches the machine you're running the generator on. Both produce an identical, independently-validated `EFI-GENERATED-<timestamp>/` folder — the only difference is the platform doing the generating.

### macOS

```
chmod +x generate-smbios.sh
./generate-smbios.sh
```

Builds `macserial` fresh from Acidanthera's own source on the spot, generates a Serial/MLB/UUID, decodes and checksum-validates all of it before writing anything.

### Windows

For prepping a machine that's still running Windows — including the exact box you're about to wipe for macOS, which is genuinely the best case, since it lets the tool grab the real onboard Ethernet MAC for ROM before the OS goes away.

```powershell
.\generate-smbios.ps1
```

If PowerShell blocks the script (unsigned script execution policy), run once first:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```
This only affects the current PowerShell window, not your system-wide policy.

Uses a Windows build of the same Acidanthera `macserial` source (`windows\macserial.exe`, cross-compiled, not a reimplementation), same generate → decode → checksum-verify → write → sanity-check pipeline as the macOS version.

### Then, either way

Copy the generated `EFI` folder onto your USB installer's EFI partition. Boot it, confirm it's stable, then copy it to your internal drive's EFI partition.

Every run produces a unique identity. **Never reuse one generated EFI's identity on a second physical machine** — that's how you get iMessage/FaceTime activation locked, and Apple support can only undo that so many times before they stop being nice about it.

## Why a generator instead of a fixed SMBIOS

Every hardware-identifying field in this repo's shipped `config.plist` — Serial, MLB, UUID, ROM — is a placeholder. Deliberately obvious ones (`GENERATEME01`, all-zero ROM), so nobody mistakes them for real values and boots on them by accident. The generator fills these in itself, validates the result structurally against Apple's own encoding before it lets you use it, and never touches anything else in the config — same graphics, same ACPI, same USB map, same everything that took months to get right.

`Automatic: True` in `PlatformInfo` means OpenCore derives board-id and the firmware feature flags from those four values on its own. Get the four right and the rest follows — which is exactly what the generator checks before it writes anything.

## A note on the Windows build

`windows\macserial.exe` is cross-compiled from the identical source used for the macOS version, with a standard, mature toolchain, and produced no compiler errors or warnings. It has not been run-tested on an actual Windows machine before this release — if it does something wrong, open an issue (see disclaimer below on how much weight that carries) and, ideally, a PR.

## Known limitations

- **Wi-Fi (BCM43228) does not work and will not work.** No macOS driver exists for this chip. This is a hardware fact, not a config problem. The only fix is a compatible card swap (BCM94360NG in the M.2 slot). This is not a bug report.
- **DP→HDMI adapters may need a manual replug after the first wake** on some monitor/adapter combinations. This is a documented, known consequence of the connector-type patch that's required to get acceleration working at all — not specific to this EFI, and not something a config file can fully engineer around on every possible cable.

---

## Disclaimer

> This EFI and its generator script are provided **"as is,"** built for one specific machine, and shared publicly on the off chance it's useful to someone else. There is no support. There is no roadmap. There are no planned updates for future macOS versions. Issues may be read; they will not be triaged, and there is no commitment to respond to, fix, or even acknowledge them. If it doesn't boot on your hardware, that's a you problem — this was never built to be general-purpose. **Back up your data before doing anything with EFI files, on any machine, ever.** Use entirely at your own risk.

---

<p align="center">Personal hobby project · No support · No warranty · Not affiliated with Apple, HP, or Acidanthera</p>
