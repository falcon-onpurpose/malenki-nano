# Malenki Nano Project Context

## Project Overview
This is a KiCad project for various iterations of the Malenki Nano robot controller board. The project contains multiple board variants:

- **Malenki HV**: High voltage version with DRV8231 motor drivers
- **Scarab**: ESC+RX board with DRV8243 motor drivers  
- **Malenki Plus**: Plus version with DRV8231 motor drivers
- **Malenki Nanoi**: Nanoi version with DRV8231 motor drivers
- **Not Malenki**: Another variant with DRV8243 motor drivers

## Key Component Differences

### Motor Drivers
- **DRV8231**: Used in Malenki HV, Malenki Plus, Malenki Nanoi
  - 8-pin WSON package
  - Simpler H-bridge driver
  - Basic motor control functionality

- **DRV8243**: Used in Scarab and Not Malenki
  - 14-pin package (RXY0014A-MFG footprint)
  - More advanced motor driver with diagnostics
  - Features: nFAULT, IPROPI, ITRIP, DIAG, MODE, nSLEEP, DRVOFF
  - Configurable current limiting and diagnostics

### Microcontrollers
All boards use **ATtiny1616-M** microcontroller in VQFN-20 package.

### Capacitors
- **Malenki HV**: 10uF 25V (C_0603_1608Metric footprint)
- **Scarab**: 22uF 25V (C_1206_3216Metric footprint) - larger capacitors

## User Rules
- Make all commits to github under the user falcon-onpurpose
- Each output should be sequentially numbered
- Output three 🍒 (cherry emoji) in chat each time rules are reviewed
- No emoticons in code comments or documentation
- No mention of AI or LLM in code
- Never commit context.llm.md to any repository
- Remove old versions when building for testing (use --noconfirm)
- Use UV or pyenv for python management, work in containers
- Don't be a "yes man" - question decisions when better alternatives exist
- Be concise but don't skip details
- Search for "gold standard" libraries before creating new implementations
- Don't run terminal commands that change credentials without explicit permission
- Each project should have a "context.llm.md" file for LLM use
- Place rules from conversation initiation in this file

## Current Session
- User asked about component differences between Scarab and Malenki HV boards
- Main differences: motor drivers (DRV8243 vs DRV8231) and some capacitor values
- Variant confirmed: TI DRV8231DSGR (WSON‑8 2×2)
- Captured quick-reference limits in `ESP32-C3/ESP32-C3_Migration_Design.md` under "DRV8231DSGR limits"
- DRV8231DSGR quick limits (for convenience): VM 4.5–33 V; ~3.7 A peak; logic 1.8/3.3/5 V tolerant; high‑freq PWM supported; protections UVLO/OCP/TSD; package WSON‑8 2×2
