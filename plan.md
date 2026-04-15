# impliment plan

## Overview

This is the code base of touchscreen for 3d printer. It support many firmware types, but we will focus on RRF(reprap firmware) impliment.

## Current behavior

- The touchscreen will detect what type of host firmware it connected to.
- If the host is RRF, it will disable print from builtin SD function

## What we need

- The touchscreen not disable print from sd function on RRF
- BUT we we need other behavior for print from built-in print from SD in for RRF. When user trying to print a file in built-in sd, do the following.
    - send M28 <filename> command to start Begin write to SD card on the host.
    - Read the file on buit-in sd and send it (the host firmware will write that down to a file)
    - send M29 to complete file saving.
    - send M32 <filename> to start print from the host

## Implementation Steps

### Step 1: Enable TFT SD selection for RRF
COMPLETED
- Modify `menuPrint()` in `Print.c` to show source selection menu for RRF instead of bypassing to onboard media
- This allows users to select files from TFT SD card when connected to RRF
- The old RRF-specific bypass in `menuPrint()` has been removed, so RRF now uses the normal source selection flow

### Step 2: Add transfer logic for RRF TFT SD printing
IN PROGRESS
- Modified `startPrint()` in `Printing.c` to detect when printing from TFT SD on RRF
- Created `transferFileToRRFHost()` function that implements the M28/M29/M32 sequence
- **Current implemented behavior:**
  - Sends `M28 <filename>` to host to start file writing
  - Reads file from TFT SD line-by-line using FatFS
  - Sends file content via `Serial_Put()` to the host
  - Sends `M29` to complete file transfer
  - Sends `M32 <filename>` to start printing
  - Pauses periodic status queries (`autoReportSDStatus`) during transfer to prevent `M27` commands from interfering
  - Includes synchronization delays
  - Restores the status query setting after transfer completes
- Fixed the RRF host filename formatting so the transfer path now sends `/filename.gcode` instead of `//filename.gcode`
- Remaining work:
  - Review the transfer flow for robustness around command/data timing and host responses
  - Build and verify there are no compilation issues
  - Confirm the behavior on real RRF hardware

### Step 3: Consolidated into Step 2
- The file transfer function already exists as `transferFileToRRFHost()`
- Remaining implementation work is part of finishing and validating Step 2, rather than creating a new transfer function

### Step 4: Test and validate
NOT STARTED
- Test the new behavior with RRF host
- Ensure backward compatibility with other firmware types
- Verify file transfer integrity and print initiation
