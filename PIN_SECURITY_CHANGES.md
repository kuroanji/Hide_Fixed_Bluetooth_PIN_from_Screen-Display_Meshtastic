# PIN Security — Code Changes

## Modified Files

### `src/nimble/NimbleBluetooth.cpp`

In `onPassKeyRequest()` / `onPassKeyDisplay()`, modified screen display logic:

```diff
 screen->startAlert([passkey](OLEDDisplay *display, OLEDDisplayUiState *state, int16_t x, int16_t y) -> void {
     int x_offset = display->width() / 2;
     int y_offset = display->height() <= 80 ? 0 : 12;
     display->setTextAlignment(TEXT_ALIGN_CENTER);
     display->setFont(FONT_MEDIUM);
     display->drawString(x_offset + x, y_offset + y, "Bluetooth");

+    // Hide PIN for FIXED_PIN mode unless user explicitly enabled "Show PIN"
+    bool hidePin = (config.bluetooth.mode == meshtastic_Config_BluetoothConfig_PairingMode_FIXED_PIN) &&
+                   !config.bluetooth.device_show_fixed_pin;
+
+    if (hidePin) {
+        // Predefined PIN: don't reveal the code on screen
+        display->setFont(FONT_SMALL);
+        y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_MEDIUM - 4 : y_offset + FONT_HEIGHT_MEDIUM + 5;
+        display->drawString(x_offset + x, y_offset + y, "Pairing with");
+        y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_SMALL - 5 : y_offset + FONT_HEIGHT_SMALL + 5;
+        display->drawString(x_offset + x, y_offset + y, "predefined PIN");
+    } else {
         char btPIN[16] = "888888";
         snprintf(btPIN, sizeof(btPIN), "%06u", passkey);
+#if !defined(M5STACK_UNITC6L)
         display->setFont(FONT_SMALL);
         y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_MEDIUM - 4 : y_offset + FONT_HEIGHT_MEDIUM + 5;
         display->drawString(x_offset + x, y_offset + y, "Enter this code");
+#endif
         display->setFont(FONT_LARGE);
         char pin[8];
         snprintf(pin, sizeof(pin), "%.3s %.3s", btPIN, btPIN + 3);
         y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_SMALL - 5 : y_offset + FONT_HEIGHT_SMALL + 5;
         display->drawString(x_offset + x, y_offset + y, pin);
+    }

     display->setFont(FONT_SMALL);
     // ... device name display
 });
```

**Log output unchanged:**
```cpp
LOG_INFO("*** Enter passkey %d on the peer side ***", passkey);
```

---

### `src/platform/nrf52/NRF52Bluetooth.cpp`

In `onPairingPasskey()`, modified screen display logic:

```diff
 screen->startAlert([](OLEDDisplay *display, OLEDDisplayUiState *state, int16_t x, int16_t y) -> void {
     int x_offset = display->width() / 2;
     int y_offset = display->height() <= 80 ? 0 : 12;
     display->setTextAlignment(TEXT_ALIGN_CENTER);
     display->setFont(FONT_MEDIUM);
     display->drawString(x_offset + x, y_offset + y, "Bluetooth");

+    // Hide PIN for FIXED_PIN mode unless user explicitly enabled "Show PIN"
+    bool hidePin = (config.bluetooth.mode == meshtastic_Config_BluetoothConfig_PairingMode_FIXED_PIN) &&
+                   !config.bluetooth.device_show_fixed_pin;
+
+    if (hidePin) {
+        // Predefined PIN: don't reveal the code on screen
+        display->setFont(FONT_SMALL);
+        y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_MEDIUM - 4 : y_offset + FONT_HEIGHT_MEDIUM + 5;
+        display->drawString(x_offset + x, y_offset + y, "Pairing with");
+        y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_SMALL - 5 : y_offset + FONT_HEIGHT_SMALL + 5;
+        display->drawString(x_offset + x, y_offset + y, "predefined PIN");
+    } else {
         char btPIN[16] = "888888";
         snprintf(btPIN, sizeof(btPIN), "%06u", configuredPasskey);

         display->setFont(FONT_SMALL);
         y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_MEDIUM - 4 : y_offset + FONT_HEIGHT_MEDIUM + 5;
         display->drawString(x_offset + x, y_offset + y, "Enter this code");

         display->setFont(FONT_LARGE);
         String displayPin(btPIN);
         String pin = displayPin.substring(0, 3) + " " + displayPin.substring(3, 6);
         y_offset = display->height() == 64 ? y_offset + FONT_HEIGHT_SMALL - 5 : y_offset + FONT_HEIGHT_SMALL + 5;
         display->drawString(x_offset + x, y_offset + y, pin);
+    }

     display->setFont(FONT_SMALL);
     // ... device name display
 });
```

**Log output unchanged:**
```cpp
LOG_INFO("Bluetooth pin set to '%i'", configuredPasskey);
// ...
LOG_INFO("BLE pair process started with passkey %s %s", passkey1, passkey2);
```

---

## Summary

| File | Screen Display | Log Output |
|------|----------------|------------|
| NimbleBluetooth.cpp | Modified (conditional) | Unchanged |
| NRF52Bluetooth.cpp | Modified (conditional) | Unchanged |

---

## Required: Protobuf Submodule Changes

**IMPORTANT:** The firmware changes above reference `config.bluetooth.device_show_fixed_pin` which does NOT exist in the current protobuf definition. The following changes must be made to the `protobufs` submodule:

### `protobufs/meshtastic/config.proto`

Add `device_show_fixed_pin` field to `BluetoothConfig` message:

```diff
 message BluetoothConfig {
   // ...
   PairingMode mode = 2;
   uint32 fixed_pin = 3;
+  /*
+   * Show the fixed PIN on the device screen during pairing.
+   * When false (and mode is FIXED_PIN), screen shows "Pairing with predefined PIN"
+   * instead of revealing the actual PIN code.
+   * Default: true (PIN is shown)
+   */
+  bool device_show_fixed_pin = 4;
 }
```

### After modifying config.proto:

1. Run the protobuf generator to create updated `.pb.h` files:
   ```bash
   cd protobufs
   ./generate.sh
   ```

2. Copy generated files to firmware:
   ```bash
   cp -r generated/* ../firmware/src/mesh/generated/
   ```

---

## Required: Mobile App Changes

The Meshtastic mobile apps (iOS/Android) need to implement a toggle switch in the Bluetooth settings UI:

### App UI Requirements

- **Location:** Bluetooth Config screen
- **Control:** Toggle switch
- **Label:** "Show PIN on Screen" or "Display PIN during pairing"
- **Description:** "When disabled, the fixed PIN will not be shown on the device screen during Bluetooth pairing"
- **Visibility:** Only shown when pairing mode is `FIXED_PIN`
- **Default:** ON (true) — matches current behavior

### App Proto Integration

The apps need to:
1. Update to use the modified `config.proto`
2. Read/write `config.bluetooth.device_show_fixed_pin` field
3. Add UI toggle in Bluetooth settings
