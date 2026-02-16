# PIN Security for InkHUD — Code Changes

## Modified Files

### 1. `src/graphics/niche/InkHUD/Applets/System/Menu/MenuAction.h`

Added new action enum for toggling PIN visibility:

```diff
 enum MenuAction {
     // ... existing actions ...
     TOGGLE_BLUETOOTH_PAIR_MODE,
+    TOGGLE_SHOW_FIXED_PIN,
     // ... rest of actions ...
 };
```

---

### 2. `src/graphics/niche/InkHUD/Applets/System/Menu/MenuApplet.cpp`

#### Change 2.1: Add action handler in `executeAction()`:

```diff
+case TOGGLE_SHOW_FIXED_PIN:
+    // Toggle whether to show fixed PIN on pairing screen
+    nodeDB->saveToDisk(SEGMENT_CONFIG);
+    config.bluetooth.device_show_fixed_pin = !config.bluetooth.device_show_fixed_pin;
+    nodeDB->saveToDisk(SEGMENT_CONFIG);
+    break;
```

#### Change 2.2: Add menu item in Bluetooth config page:

```diff
 // In NODE_CONFIG_BLUETOOTH page builder:
 items.push_back(MenuItem(pairLabel, MenuAction::TOGGLE_BLUETOOTH_PAIR_MODE, MenuPage::NODE_CONFIG_BLUETOOTH));

+// Show PIN toggle - only visible when FIXED_PIN mode is active
+if (config.bluetooth.mode == meshtastic_Config_BluetoothConfig_PairingMode_FIXED_PIN) {
+    const char *showPinLabel = config.bluetooth.device_show_fixed_pin ? "Show PIN: Yes" : "Show PIN: No";
+    items.push_back(MenuItem(showPinLabel, MenuAction::TOGGLE_SHOW_FIXED_PIN, MenuPage::NODE_CONFIG_BLUETOOTH));
+}
```

---

### 3. `src/graphics/niche/InkHUD/Applets/System/Pairing/PairingApplet.cpp`

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

| File | Changes |
|------|---------|
| `MenuAction.h` | Add `TOGGLE_SHOW_FIXED_PIN` enum value |
| `MenuApplet.cpp` | Add action handler + menu item (~15 lines) |
| `PairingApplet.cpp` | Add conditional PIN hiding (~12 lines) |

| Aspect | Details |
|--------|---------|
| Files changed | 3 |
| Total lines added | ~30 |
| Lines removed | ~4 |
| Dependencies | Requires `device_show_fixed_pin` field in protobuf |
| Header changes | MenuAction.h |

---

## Required: Protobuf Changes

**IMPORTANT:** This implementation requires `config.bluetooth.device_show_fixed_pin` field which must be added to the protobuf definition. See `PIN_SECURITY_CHANGES.md` for details.
