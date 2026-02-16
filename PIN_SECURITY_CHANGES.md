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
