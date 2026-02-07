---
name: keyboard-troubleshooting
description: This skill should be used when debugging keyboard behavior issues on macOS. It provides a hardware-first diagnostic framework for identifying whether keyboard problems stem from hardware/firmware configuration or software settings. Use this skill when facing modifier key swaps, function key failures, or other keyboard behavior anomalies, especially when behavior differs between keyboards or connection types.
---

# Keyboard Troubleshooting

## Overview

Keyboard problems can be frustratingly difficult to debug because the root cause might be hardware firmware configuration, operating system settings, or third-party software. This skill provides a systematic hardware-first diagnostic framework that quickly identifies whether a keyboard problem is hardware/firmware-based or software-based by testing device variance patterns.

The key insight: when behavior differs across devices (some keyboards work fine, others don't), suspect hardware first. When all keyboards exhibit the same problem, suspect software.

## Workflow Decision Tree

### Step 1: Device Variance Test (2-5 minutes)

The fastest way to narrow down the root cause is to test keyboard behavior across multiple input devices.

**Test with:**
1. Mac's built-in keyboard (trackpad keyboard on MacBook or Magic Keyboard on desktop)
2. At least one external keyboard (if available)
3. Another external keyboard (if available)

**Test behavior:** Try simple command sequences like Cmd+C, Cmd+V, Fn key combinations, or any behavior the user reported as problematic.

**Analyze results:**

- **Same problem on ALL keyboards (built-in + external)?**
  Go to **Step 3: Software Investigation**

- **Problem only on ONE specific keyboard?**
  Go to **Step 2: Hardware/Firmware Diagnosis**

- **Problem on SOME but not ALL external keyboards?**
  Go to **Step 2: Hardware/Firmware Diagnosis**

### Step 2: Hardware/Firmware Diagnosis

When problem is device-specific, check for firmware toggles and physical hardware configuration.

**Sub-step 2a: Check for Mac/Windows Mode Toggle**

Many Bluetooth keyboards, especially budget models, support dual-mode operation for Mac and Windows. When in Windows mode on a Mac, keys appear "swapped" because firmware remaps modifier keys.

- Look at the keyboard for printed Fn combinations (usually in blue or lighter color)
- Check keyboard underside for DIP switches or mode indicators
- Consult keyboard manual or manufacturer support page
- Try common toggles: Fn+R, Fn+W, Fn+M, Fn+A, Fn+X, Fn+Delete
- Test immediately after toggle attempt (no restart needed)

**Sub-step 2b: Check for Multi-Device Pairing Mode**

Some keyboards support simultaneous pairing with multiple devices. Different pairing modes or device slots may have different key behaviors.

- Check for mode indicator LEDs on keyboard
- Try toggling device profiles (often Fn+1, Fn+2, Fn+3)
- Verify keyboard is paired to current Mac (may need to re-pair)
- Test with different connection type (USB if using Bluetooth, or vice versa)

**Sub-step 2c: Physical Hardware Inspection**

- Examine keyboard for physical damage, debris, or stuck keys
- Check for DIP switches on underside
- Look for firmware version or mode indicators
- Test USB connection (if available) vs Bluetooth
- Force-forget Bluetooth pairing and re-pair from scratch
- Check if firmware update is available for the keyboard

**Sub-step 2d: Consult Keyboard Documentation**

Search for the keyboard model online and find:
- User manual (usually PDF from manufacturer)
- Firmware toggle combinations
- Known issues for macOS
- Firmware update procedures

**Sub-step 2e: When Nothing Works**

If hardware/firmware diagnosis doesn't resolve the issue:
- Test keyboard on another Mac to isolate the device
- Contact keyboard manufacturer support with model/firmware information
- Consider keyboard may have hardware failure
- Escalate to Step 3 if problem is actually system-wide

### Step 3: Software Investigation

Only investigate software settings if device variance test showed the problem affects ALL keyboards equally.

**Check in this order:**

1. **macOS System Settings → Keyboard → Modifier Keys**
   - Verify modifier key remapping is correct for this keyboard
   - Check if another keyboard's settings are incorrectly applied
   - Reset to defaults if configuration appears wrong

2. **macOS System Settings → Keyboard → Input Sources**
   - Verify keyboard layout matches expected (U.S. English, etc.)
   - Check for unexpected multiple layouts that might conflict
   - Disable non-essential input sources

3. **Third-party key remapping tools** (highest priority)
   - Karabiner-Elements (most common source of key remapping issues)
   - BetterTouchTool (may have key binding conflicts)
   - Alfred (custom hotkey conflicts)
   - Keyboard Maestro (complex macro conflicts)
   - Any other app with key mapping functionality

4. **macOS Accessibility Settings**
   - Slow Keys mode enabled
   - Sticky Keys enabled
   - Mouse Keys enabled
   - Full Keyboard Access enabled (may cause unexpected behavior)

5. **System logs and diagnostics**
   - Check Console for keyboard-related errors
   - Verify no OS update recently changed keyboard behavior
   - Consider macOS restart to clear temporary state

**Debug approach for software issues:**
- Quit each third-party app one at a time to isolate culprit
- Check System Settings for any unexpected configuration
- Read recent macOS update notes for keyboard behavior changes
- Create new macOS user account to test with clean software state

## Quick Reference: Hardware Toggle Patterns

See `references/common-keyboards-firmware.md` for specific models and their firmware toggles.

**Common patterns to try when model is unknown:**
- Fn + R (most common for budget keyboards)
- Fn + W (Windows mode)
- Fn + M (Mac mode)
- Fn + A (Alt/Windows)
- Fn + X (device profile switching)
- Fn + Delete (less common but documented)
- Long-press power button (factory reset, last resort)

Test each toggle attempt on the problematic keyboard. If behavior changes after toggle, firmware mode was successfully changed.

## Case Studies and Real Examples

See `references/case-studies.md` for six real-world examples demonstrating:
- Swapped modifier keys on Meetion KB-BT2
- Function keys not working as expected
- Mixed keyboard setups with different firmware modes
- False positive diagnosis (actually software issue)
- Hybrid problems (hardware + connection quality)
- DIP switch configuration issues

These examples illustrate pattern recognition and decision-making across different problem types.

## Diagnostic Framework Details

See `references/hardware-vs-software-diagnosis.md` for comprehensive coverage of:
- Device variance decision tree
- Firmware mode detection
- Physical hardware checks
- Red flags for hardware vs software issues
- Common hardware issues and solutions
- Prevention strategies

## Prevention Strategy

The most effective approach saves significant debugging time:

1. **Always test multiple keyboards first** (2-5 minutes)
   - Device variance emerges immediately
   - Points directly to hardware or software category

2. **Check keyboard printed legends** (1 minute)
   - Blue or highlighted text usually indicates Fn combinations
   - Tells you what firmware modes are available

3. **Search for Fn key combinations in manual** (5 minutes)
   - Most keyboards have easily accessible documentation online
   - Manufacturer support pages often list common toggles

4. **Only then investigate software settings** (if #1-3 don't resolve)
   - After hardware/firmware has been ruled out
   - Focus on third-party apps first (Karabiner, BetterTouchTool)
   - Then check macOS settings

This approach typically resolves keyboard problems in 10-20 minutes instead of hours of unproductive software debugging.

## Key Insight: Device Variance Pattern

The single most important diagnostic principle is understanding device variance:

**When one keyboard works but another doesn't** = Hardware/firmware problem in the affected keyboard

**When all keyboards behave the same way (wrong)** = Software problem affecting the entire system

This principle applies universally. Testing multiple devices first is the fastest way to categorize the problem and point to the correct solution direction.
