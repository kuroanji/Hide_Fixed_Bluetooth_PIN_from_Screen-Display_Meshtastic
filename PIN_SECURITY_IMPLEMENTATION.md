# PIN Security — Hide Fixed PIN from Screen

This document describes the security feature that hides the fixed Bluetooth PIN from the device screen during pairing.

## Overview

When using Bluetooth with a fixed PIN (`FIXED_PIN` mode), the PIN is shown on the device screen during pairing. This creates a security risk — anyone who can see the screen can learn the PIN.

This implementation adds an option to hide the PIN from the screen while still showing "Pairing with predefined PIN" message. The PIN remains in debug logs for troubleshooting.

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
+-----------------+
|    Bluetooth    |
|  Pairing with   |
|  predefined PIN |
| Name: device-xx |
+-----------------+
```

When `hidePin` is false (or random PIN):
```
+-----------------+
|    Bluetooth    |
| Enter this code |
|    123 456      |
| Name: device-xx |
+-----------------+
```

## Files Modified

### Firmware
- `src/nimble/NimbleBluetooth.cpp` — ESP32/NimBLE pairing screen
- `src/platform/nrf52/NRF52Bluetooth.cpp` — nRF52 pairing screen

### Protobuf Submodule (required)
- `protobufs/meshtastic/config.proto` — add `device_show_fixed_pin` field to `BluetoothConfig`

See `PIN_SECURITY_CHANGES.md` for the exact protobuf changes required.

## Testing

1. Set Bluetooth mode to `FIXED_PIN`
2. Set a PIN (e.g., 123456)
3. Set `device_show_fixed_pin = false`
4. Initiate pairing from phone
5. Verify screen shows "Pairing with predefined PIN" (not the actual PIN)
6. Verify debug logs still show the PIN
7. Enter the PIN on phone — pairing should succeed

## Notes

- PIN is NOT hidden from logs — logs are typically only accessible to device owner
- This feature is for physical security (shoulder surfing), not log security
- Random PINs are always displayed — they're useless without seeing the screen anyway
