# PIN Security for InkHUD — Code Changes

## Modified File

### `src/graphics/niche/InkHUD/Applets/System/Pairing/PairingApplet.cpp`

Only the `onRender()` method is modified:

```diff
 void InkHUD::PairingApplet::onRender(bool full)
 {
     // Header
     setFont(fontMedium);
     printAt(X(0.5), Y(0.25), "Bluetooth", CENTER, BOTTOM);
-    setFont(fontSmall);
-    printAt(X(0.5), Y(0.25), "Enter this code", CENTER, TOP);

-    // Passkey
-    setFont(fontMedium);
-    printThick(X(0.5), Y(0.5), passkey.substr(0, 3) + " " + passkey.substr(3), 3, 2);
+    // Hide PIN for FIXED_PIN mode unless user explicitly enabled "Show PIN"
+    bool hidePin = (config.bluetooth.mode == meshtastic_Config_BluetoothConfig_PairingMode_FIXED_PIN) &&
+                   !config.bluetooth.device_show_fixed_pin;
+
+    if (hidePin) {
+        // Predefined PIN: don't reveal the code on screen
+        setFont(fontSmall);
+        printAt(X(0.5), Y(0.45), "Pairing with", CENTER, BOTTOM);
+        printAt(X(0.5), Y(0.55), "predefined PIN", CENTER, TOP);
+    } else {
+        setFont(fontSmall);
+        printAt(X(0.5), Y(0.25), "Enter this code", CENTER, TOP);
+
+        // Passkey
+        setFont(fontMedium);
+        printThick(X(0.5), Y(0.5), passkey.substr(0, 3) + " " + passkey.substr(3), 3, 2);
+    }

     // Device's bluetooth name, if it will fit
     setFont(fontSmall);
     std::string name = "Name: " + parse(getDeviceName());
     if (getTextWidth(name) > width()) // Too wide, try without the leading "Name: "
         name = parse(getDeviceName());
     if (getTextWidth(name) < width()) // Does it fit?
         printAt(X(0.5), Y(0.75), name, CENTER, MIDDLE);
 }
```

---

## Summary

| Aspect | Details |
|--------|---------|
| Files changed | 1 |
| Lines added | ~12 |
| Lines removed | ~4 |
| Dependencies | None (uses existing config) |
| Header changes | None |
