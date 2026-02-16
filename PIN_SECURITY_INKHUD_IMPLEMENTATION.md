# PIN Security for InkHUD — Hide Fixed PIN from Screen

This document describes the security feature that hides the fixed Bluetooth PIN from the InkHUD e-ink display during pairing.

## Overview

When using Bluetooth with a fixed PIN (`FIXED_PIN` mode), the PIN is shown on the device screen during pairing. This creates a security risk — anyone who can see the screen can learn the PIN.

This implementation adds an option to hide the PIN from the InkHUD display while still showing "Pairing with predefined PIN" message.

## Scope

This implementation affects **only InkHUD devices** (T-Echo, T-Echo Plus, etc.). It modifies a single file:
- `src/graphics/niche/InkHUD/Applets/System/Pairing/PairingApplet.cpp`

Other Meshtastic devices with OLED screens are not affected.

## Security Model

| Mode | PIN on Screen | PIN in Logs |
|------|---------------|-------------|
| RANDOM_PIN | ✓ Shown | ✓ Shown |
| FIXED_PIN (default) | ✓ Shown | ✓ Shown |
| FIXED_PIN + hide option | ✗ Hidden | ✓ Shown |

## Configuration

The feature is controlled by `config.bluetooth.device_show_fixed_pin`:

- `true` — Show PIN on screen (default behavior)
- `false` — Hide PIN, show "Pairing with predefined PIN"

This setting only affects `FIXED_PIN` mode. Random PINs are always shown.

## Implementation

### Condition for Hiding

```cpp
bool hidePin = (config.bluetooth.mode == meshtastic_Config_BluetoothConfig_PairingMode_FIXED_PIN) &&
               !config.bluetooth.device_show_fixed_pin;
```

### Screen Display

When `hidePin` is true:
```
+-------------------+
|     Bluetooth     |
|                   |
|   Pairing with    |
|   predefined PIN  |
|                   |
|  Name: device-xx  |
+-------------------+
```

When `hidePin` is false (or random PIN):
```
+-------------------+
|     Bluetooth     |
|  Enter this code  |
|                   |
|     123 456       |
|                   |
|  Name: device-xx  |
+-------------------+
```

## File Modified

**`src/graphics/niche/InkHUD/Applets/System/Pairing/PairingApplet.cpp`**

Only the `onRender()` method is modified. No changes to headers or other files.

## Testing

1. Flash firmware to InkHUD device (T-Echo Plus)
2. Set Bluetooth mode to `FIXED_PIN`
3. Set a PIN (e.g., 123456)
4. Set `device_show_fixed_pin = false`
5. Initiate pairing from phone
6. Verify e-ink screen shows "Pairing with predefined PIN" (not the actual PIN)
7. Enter the PIN on phone — pairing should succeed
8. Test with `device_show_fixed_pin = true` — PIN should be visible

## Notes

- Minimal change — only one file modified
- No impact on other device types
- No changes to Bluetooth stack or logging
- Uses existing InkHUD font system and layout methods
