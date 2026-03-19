# Practical Field Manual

This manual is written for real use, not for theory.

Use it when you need to:

- prepare a Linux machine for a sharing session
- verify a logic-analyzer software stack quickly
- run a demo without hardware
- perform a first capture with real hardware
- decode UART, I2C, or SPI with the least amount of guesswork

The examples assume Arch Linux, but the workflow itself is generic.

## 1. Prepare the machine

### Install the default stack

```bash
sudo pacman -S pulseview sigrok-cli sigrok-firmware-fx2lafw
```

What each package is for:

- `pulseview`: GUI capture and decoder workflow
- `sigrok-cli`: command-line verification and scripted demos
- `sigrok-firmware-fx2lafw`: support for common FX2-based analyzers

### Verify that the binaries exist

```bash
command -v pulseview
command -v sigrok-cli
```

If either command prints nothing, the install is not complete.

## 2. Verify the stack without hardware

This is the safest first check.

### Scan devices

```bash
sigrok-cli --scan
```

A healthy software-only result should still show the built-in demo device.

### Inspect the demo driver

```bash
sigrok-cli --driver demo --show
```

This confirms that:

- the backend loads
- the driver is visible
- the channel groups are available

### Run a small capture

```bash
sigrok-cli -d demo -c samplerate=1000 -C D0,D1,D2,D3 --samples 32 -O ascii
```

If you want a more compact output:

```bash
sigrok-cli -d demo -c samplerate=1000 -C D0,D1,D2,D3 --samples 32 -O bits
```

If these commands work, the software stack is already usable for a live walkthrough.

## 3. Open the GUI before hardware arrives

Start the GUI even if no analyzer is connected yet.

```bash
pulseview
```

Check the following:

- the application starts normally
- the decode UI is reachable
- adding protocol decoders does not crash
- session save and export menus are present

This matters because workshop failures are often GUI failures, not device failures.

## 4. First capture with real hardware

Once hardware is available, use this sequence.

### Step 1: Detect the device

```bash
sigrok-cli --scan
```

If no device appears:

- reconnect USB
- try a different cable
- try a different port
- re-run the scan

### Step 2: Open PulseView

Choose the detected device and set:

- active channels
- sample rate
- capture depth or capture time
- threshold or trigger settings if the device supports them

### Step 3: Keep the first capture simple

For the first run:

- enable only the channels you need
- use a moderate sample rate
- avoid complicated trigger rules
- verify that the waveform looks electrically plausible before adding decoders

### Step 4: Decode only after the waveform looks correct

Do not start by tuning decoder settings blindly.

First confirm:

- idle level looks correct
- edges are stable
- the clock actually toggles
- chip select or start condition is visible

Then add protocol decoders.

## 5. UART recipe

Use this when debugging boot logs, MCU shells, or sensor modules with TX/RX.

### Wiring checklist

- connect analyzer ground to target ground
- probe at least one data line: `TX` or `RX`
- probe both if you want duplex visibility

### Capture checklist

- start with a sample rate comfortably above the baud rate
- verify the idle level
- identify start bits and stop bits visually

### Decoder settings

Typical values to check:

- baud rate
- data bits
- parity
- stop bits
- bit order
- inverted or non-inverted line

### Common failure patterns

If the decoded text is nonsense:

- wrong baud rate
- inverted logic level
- wrong probe point
- insufficient sample rate

## 6. I2C recipe

Use this for sensors, PMICs, EEPROMs, and board bring-up.

### Wiring checklist

- connect ground
- probe `SCL`
- probe `SDA`

### What to verify in the raw waveform

- both lines idle high
- `SDA` changes while `SCL` is low, except for start and stop
- start and stop conditions are clearly visible

### Decoder settings

The basic decoder setup is usually enough if the waveform is clean.

If decoding fails:

- check whether the line is actually I2C and not a GPIO imitation
- confirm pull-ups exist
- reduce the number of enabled channels
- increase sample rate

### Common failure patterns

- no pull-up, so the lines never return high
- noise or ringing causes false transitions
- probing on the wrong side of level shifters

## 7. SPI recipe

Use this for flash, displays, radios, and many digital peripherals.

### Wiring checklist

- connect ground
- probe `CLK`
- probe `MOSI`
- probe `MISO` if needed
- probe `CS#` whenever possible

### What to verify in the raw waveform

- clock is present and stable
- data transitions match clock edges
- chip select brackets each transaction

### Decoder settings

The important settings are:

- clock polarity
- clock phase
- bit order
- word size
- chip-select polarity

### Fast diagnosis

If the bytes look shifted or wrong:

- try the other SPI mode
- verify whether the device is `MSB-first` or `LSB-first`
- check whether `CS#` is active-low or active-high

## 8. Minimal workshop flow

If you need a short live session, use this order:

1. Show the package install command
2. Run `sigrok-cli --scan`
3. Run the built-in demo capture
4. Open `PulseView`
5. Explain how a real analyzer would replace the demo source
6. Show one decoder workflow, not three

For a 10 to 15 minute slot, this is usually enough.

## 9. Chinese-friendly fallback

If the audience benefits from Chinese UI or Chinese-facing materials, keep `DSView` as a backup plan.

Use it only if:

- it was built and verified beforehand
- decoder resources are confirmed to load
- startup logs are clean

Do not rely on a first-time `DSView` build during a live session on a modern rolling-release Linux system.

## 10. Troubleshooting checklist

When the setup does not work, check in this order.

### Software layer

- does `sigrok-cli --scan` run at all
- does the demo driver work
- does `pulseview` start cleanly

### USB layer

- cable quality
- port quality
- hub vs direct connection
- insufficient power

### Signal layer

- shared ground
- correct voltage domain
- correct probe point
- enough sample rate

### Decoder layer

- correct protocol choice
- correct channel mapping
- correct polarity or inversion
- correct timing parameters

## 11. Cleanup after the share

If the machine was only used for preparation or a one-off workshop, remove everything cleanly.

Example package cleanup on Arch:

```bash
sudo pacman -Rns pulseview sigrok-cli libsigrok libsigrokdecode sigrok-firmware-fx2lafw boost
```

If local tools or builds were added, also remove:

- user-local binaries
- desktop entries
- temporary source archives
- temporary session files
- user-local decoder directories

## 12. Final advice

For most Linux-based embedded work:

- use `PulseView + sigrok-cli` as the default recommendation
- keep the first live demo hardware-free if necessary
- treat raw waveforms as the truth and decoders as helpers
- prepare a fallback path before every public session
