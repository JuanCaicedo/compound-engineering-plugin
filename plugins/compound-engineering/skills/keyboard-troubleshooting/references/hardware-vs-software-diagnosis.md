# Hardware vs Software Diagnosis Framework

## Key Diagnostic Principle

**When behavior differs across devices, suspect hardware first. When behavior is consistent across all devices, suspect software.**

This framework helps distinguish between hardware issues, firmware configuration, and operating system settings when keyboards behave unexpectedly.

## Diagnostic Decision Tree

### Step 1: Device Variance Check

Test the behavior across multiple input devices.

**Question:** Does the problem affect ALL keyboards or only SOME keyboards?

**If ALL keyboards (built-in + all external keyboards):**
- Problem is SOFTWARE (macOS settings, system configuration, software key remapping)
- Investigate: System Preferences, third-party apps (Karabiner, BetterTouchTool, Alfred, etc.)

**If ONLY ONE specific keyboard:**
- Problem is HARDWARE or FIRMWARE (keyboard-specific)
- Next: Go to Step 2

**If SOME but not ALL keyboards:**
- Problem is HARDWARE or FIRMWARE (specific to those keyboards or their connection mode)
- Next: Go to Step 2

### Step 2: Firmware Mode Detection

When behavior is device-specific, check for firmware toggles and mode switches.

**Common firmware modes in consumer Bluetooth keyboards:**

1. **Mac/Windows Mode Toggle**
   - Many budget Bluetooth keyboards support dual-mode operation
   - Mac mode: Keys labeled as Cmd, Option, Control map correctly
   - Windows mode: Keys remapped as Ctrl, Alt, Windows (appears "swapped" on Mac)
   - Activation: Usually Fn + specific key combination (varies by brand)

2. **Single Device vs Multi-Device Pairing Mode**
   - Some keyboards can pair with multiple devices simultaneously
   - Different pairing modes may have different key behaviors
   - Activation: Usually Fn + number keys (Fn+1, Fn+2, Fn+3)

3. **Bluetooth vs Wired Connection Mode**
   - Some keyboards work in both modes with different key layouts
   - Physical connection may trigger mode change
   - Activation: Usually automatic on connection type change, or Fn combination

4. **Gaming vs Standard Mode**
   - Gaming-oriented keyboards may have separate FN lock or gaming mode
   - Gaming mode disables Windows key to prevent accidental focus loss
   - Activation: Usually Fn+G or dedicated gaming mode key

### Step 3: Physical Hardware Checks

If firmware mode is not the issue, check for physical problems.

**Inspect the keyboard:**
- Look for DIP switches on the underside
- Check for mode indicator LEDs
- Look for printed Fn key combinations (usually in blue or lighter color)
- Examine connection (loose USB, weak Bluetooth signal)
- Check for physical key damage or debris

**Test the keyboard:**
- Connect to a different computer to isolate the keyboard
- Try USB wired connection if Bluetooth is problematic
- Restart the keyboard (power off, remove batteries, reconnect)
- Force-forget Bluetooth pairing and re-pair from scratch

### Step 4: Software Configuration Check

Only investigate software settings if hardware and firmware have been ruled out.

**Check in this order:**
1. macOS System Settings → Keyboard → Modifier Keys (for this specific keyboard)
2. macOS System Settings → Keyboard → Input Sources (language/layout issues)
3. Third-party key remapping tools (Karabiner-Elements, BetterTouchTool, Alfred, Keyboard Maestro)
4. System event viewers for key event conflicts
5. macOS accessibility settings that might affect key behavior

## Red Flags for Hardware Issues

These patterns strongly indicate hardware/firmware problems rather than software:

1. **Device-specific behavior variation** - The problem affects only one keyboard while others work fine
2. **User rarely customizes device settings** - If the user has no keyboard-specific configurations, hardware is more likely
3. **Behavior is symmetric and consistent** - All instances of the affected key(s) behave the same way (all Commands, all Options)
4. **Multiple keys affected identically** - Left and right versions of the same key all exhibit the same issue (both Command keys swapped, both Option keys swapped)
5. **No recent system updates or software changes** - Problem appeared without software changes
6. **Printed key labels don't match behavior** - Keyboard shows one thing but acts differently (Mac legend with Windows behavior)
7. **Problem persists after macOS restart** - Ruling out temporary system state issues
8. **Keyboard works correctly with different macOS version** - Testing on another Mac or bootcamp partition

## Red Flags for Software Issues

These patterns indicate software rather than hardware:

1. **Affects ALL input devices equally** - Problem exists on built-in keyboard, all external keyboards, trackpad
2. **Problem appeared after system update** - macOS update changed behavior system-wide
3. **Problem appeared after software installation** - New application installation preceded the issue
4. **Settings reveal the culprit** - System Preferences, Karabiner, or other tool shows suspicious configuration
5. **Behavior varies by application** - Problem only exists in certain apps (not system-wide)
6. **Fix involves software changes** - Uninstalling an app or changing settings resolves it
7. **Problem affects keyboard across all physical keyboards but not trackpad** - System keyboard handling is affected, not peripheral hardware

## Common Hardware Issues and Solutions

### Issue: Modifier Keys Swapped (Cmd/Option/Control)

**Typical cause:** Mac/Windows firmware mode toggle

**Diagnosis:**
- Only affects specific keyboard
- Other keyboards work correctly
- All instances of each key exhibit same behavior

**Solution:**
- Identify the keyboard brand/model
- Check user manual for Mac/Windows mode toggle
- Look for Fn key combinations (typically printed on keyboard)
- Common combinations: Fn+R, Fn+W, Fn+M, Fn+A

**Example:** Meetion KB-BT2 uses Fn+R to toggle Mac/Windows mode

### Issue: Fn Key Not Working as Expected

**Typical cause:** FN Lock mode enabled or firmware mode issue

**Diagnosis:**
- Fn key combinations don't trigger special functions
- Function keys (F1-F12) work but don't trigger intended actions
- Usually only affects specific keyboard

**Solution:**
- Check for dedicated Fn Lock key or mode toggle
- Try Fn+Esc (common toggle for FN Lock)
- Verify keyboard firmware mode matches operating system
- Check keyboard's manual for FN Lock behavior

### Issue: Keyboard Works in One Mode (USB/Bluetooth) but Not Another

**Typical cause:** Firmware expects different connection type

**Diagnosis:**
- Keyboard works via USB but not Bluetooth (or vice versa)
- Problem is consistent and reproducible
- Other keyboards work in both modes

**Solution:**
- Check if keyboard supports both connection types
- Verify firmware is up-to-date (some keyboards support firmware updates)
- Test with different connection type
- Factory reset the keyboard (usually Fn+R or holding power button)

### Issue: Specific Modifier Key Stuck or Not Registering

**Typical cause:** Hardware failure or debris

**Diagnosis:**
- Key physically appears stuck or has reduced tactile feedback
- Modifier key has no effect when pressed
- Other keys on same keyboard work fine
- Only affects physical key, not software layer

**Solution:**
- Clean underneath the key with compressed air
- Check for physical damage
- Test on different computer to isolate
- May require keyboard replacement

## Prevention Strategy

Before investing debugging effort:

1. **Always test multiple devices first** (2-5 minutes)
2. **Check printed legends on keyboard** (1 minute)
3. **Search for Fn key combinations in manual** (5 minutes)
4. **Only then investigate software settings** (if #1-3 don't resolve)

This saves significant time and frustration by catching hardware issues early.
