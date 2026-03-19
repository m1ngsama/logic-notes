# Logic Notes

Practical notes on logic-analyzer tooling for embedded debugging on Linux.

## What this repo captures

This repository documents one real Arch Linux session focused on:

- setting up a logic-analyzer software stack quickly
- running a hardware-free demo
- evaluating a Chinese-friendly fallback GUI
- removing everything cleanly after the session

## Recommended default stack

For a Linux machine, the clean default is:

- `PulseView` for the GUI
- `sigrok-cli` for scripted capture and demos
- `sigrok-firmware-fx2lafw` for common FX2-based analyzers

## Chinese-friendly fallback

If Chinese UI and Chinese-facing materials matter, `DSView` is the stronger fallback. It is based on the sigrok ecosystem, but in this session it required extra care on modern Arch Linux.

## Read the full notes

- [Arch Linux logic-analyzer session notes](./docs/arch-linux-logic-analyzer-session.md)
- [Practical field manual](./docs/practical-field-manual.md)

## Audience

This repo is written for embedded engineers, workshop hosts, and anyone preparing a live sharing session on logic-analyzer tooling.
