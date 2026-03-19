# Arch Linux Logic-Analyzer Session Notes

This document captures a single practical session on an Arch Linux machine used to prepare a logic-analyzer software setup for embedded debugging and for a live sharing scenario.

## Short conclusion

If the goal is to get useful logic-analyzer tooling on Linux quickly, the best default is:

```bash
sudo pacman -S pulseview sigrok-cli sigrok-firmware-fx2lafw
```

If the goal also includes Chinese-friendly UI and Chinese-facing materials, keep `DSView` as a fallback rather than the primary recommendation.

## Why this stack

### PulseView + sigrok

This combination is the most practical default because it gives:

- a standard open-source workflow
- broad device support
- a simple command-line interface for reproducible demos
- a low-friction path for common embedded buses such as SPI, I2C, and UART

### DSView

`DSView` is useful when Chinese support matters more than having the cleanest install story.

In this session, the reasons to keep it as a fallback were:

- the project is clearly more Chinese-friendly
- it has Chinese language resources for UI and decoder options
- it worked in the end, but required local build fixes on a modern Arch setup

## What worked without hardware

The built-in `sigrok` demo driver was enough to validate the toolchain.

Useful checks:

```bash
sigrok-cli --scan
sigrok-cli --driver demo --show
sigrok-cli -d demo -c samplerate=1000 -C D0,D1,D2,D3 --samples 32 -O ascii
sigrok-cli -d demo -c samplerate=1000 -C D0,D1,D2,D3 --samples 32 -O bits
```

This is the best path when:

- no analyzer is physically available
- the session is for teaching
- the goal is to prove that the software stack is healthy before hardware arrives

## Notes on DSView on modern Arch Linux

`DSView 1.3.2` did run successfully in the end, but it was not a clean out-of-the-box install.

### What had to be handled

#### 1. CMake compatibility

The project uses an older CMake baseline, so configuration needed:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/.local -DCMAKE_POLICY_VERSION_MINIMUM=3.5
```

#### 2. New toolchain compatibility fixes

Two source-level fixes were needed during the build:

- include `strings.h` so `strcasecmp()` is declared
- replace `usleep()` with `g_usleep()` in the local build

#### 3. Decoder path and language resources

The binary was installed under `~/.local`, but decoder scripts and language resources also had to exist in the expected paths:

- `~/.local/share/DSView/lang`
- `~/.local/share/libsigrokdecode4DSL/decoders`

Without those paths, common decoders such as SPI, I2C, and UART were not loaded correctly.

#### 4. One bundled decoder shipped broken

The `sdcard_spi` decoder contained a Python syntax error and needed a local one-line fix before startup logs were clean.

### Recommendation for live sharing

For a live talk or workshop, do not rely on building `DSView` live on stage.

Better options:

- prebuild and verify it on the target machine beforehand
- ship screenshots or a recorded walkthrough instead of a live build
- use `PulseView + sigrok-cli` as the live demo path

## Suggested structure for an embedded sharing session

### Option A: safest live demo

1. Install `PulseView` and `sigrok-cli`
2. Show `sigrok-cli --scan`
3. Use the built-in demo driver
4. Explain where a real FX2-based analyzer would fit in

### Option B: broader comparison talk

1. Start with the open-source default stack
2. Explain when `DSView` is attractive
3. Highlight Chinese-support tradeoffs
4. End with a full rollback procedure

## Full rollback after the session

If the machine was only used to prepare the session, a full cleanup is reasonable.

Packages removed in this session:

- `pulseview`
- `sigrok-cli`
- `libsigrok`
- `libsigrokdecode`
- `sigrok-firmware-fx2lafw`
- `boost`
- auto-installed dependencies such as `libftdi`, `libieee1284`, and `libserialport`

Typical package removal command:

```bash
sudo pacman -Rns pulseview sigrok-cli libsigrok libsigrokdecode sigrok-firmware-fx2lafw boost
```

User-level files that were also removed:

- `~/.local/bin/DSView`
- `~/.local/bin/dsview`
- `~/.local/share/DSView`
- `~/.local/share/libsigrokdecode4DSL`
- `~/.local/share/applications/dsview.desktop`
- `~/.config/DreamSourceLab`
- temporary build and archive files under `/tmp`

## Final recommendation

For most embedded engineers on Linux:

- recommend `PulseView + sigrok-cli` first
- mention `DSView` as a Chinese-friendly fallback
- use the built-in `sigrok` demo driver when no hardware is available
- always prepare a rollback path if the machine is only being used for a share or workshop
