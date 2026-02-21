# Hide Fixed Bluetooth PIN from Screen Display [Meshtastic devices + InkHUD]

Adds option to hide the fixed Bluetooth PIN from the device screen during pairing, while still displaying "Pairing with predefined PIN" message.

New implementation integrated in InkHUD2: https://github.com/kuroanji/InkHUD2

## Why

When using a fixed PIN for Bluetooth pairing, displaying it on screen creates a security risk — anyone nearby can see the PIN. This is particularly relevant for devices in semi-public locations or shared spaces.

See `PIN_SECURITY_IMPLEMENTATION.md` for instructions (or `PIN_SECURITY_INKHUD_IMPLEMENTATION.md` for InkHUD only).

Merge branch for InkHUD devices ONLY: https://github.com/kuroanji/firmware/tree/Hide_fixed_PIN_InkHUD_devices

