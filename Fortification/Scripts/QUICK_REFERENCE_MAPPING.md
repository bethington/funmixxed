# Quick Reference: Script Placeholder → D2Ptrs.h Address Mapping

## 🎯 **Immediate Address Replacements Needed**

### **Core Player & Game State Pointers**
```asm
// Replace in ALL scripts:
define(pMe,40000000)                    → define(pMe,0x11BBFC)                    // D2CLIENT_PlayerUnit
define(GameReady,60000000)              → define(D2CLIENT_GetPlayerUnit,0xA4D60)  // + custom validation logic
define(D2CLIENT_GetPlayerUnit,60000000) → define(D2CLIENT_GetPlayerUnit,0xA4D60)  // Direct replacement
define(p_D2WIN_FirstControl,60000004)   → define(p_D2WIN_FirstControl,0x214A0)   // UI control checking
define(GetPlayerArea,60000008)          → define(D2CLIENT_PlayerArea,0x11C34C)   // Player area
```

### **Statistics & State Functions**
```asm
// Replace in AutoLife.md, AutoMana.md, FindItem.md:
define(GetUnitStat,60000004)            → define(D2COMMON_GetUnitStat,-10973)    // Statistics reading
define(GetUnitState,60000008)           → define(D2COMMON_GetUnitState,-10494)   // Effect checking
```

### **Network Packet Functions**
```asm
// Replace in TeleOnMap.md, CastOnMap.md, UseItem.md, PickItem.md:
define(D2NET_SendPacket,60000000)       → define(D2NET_SendPacket,-10024)        // Packet sending
define(D2NET_SendPacket,60000004)       → define(D2NET_SendPacket,-10024)        // Packet sending (alt)
```

---

## 📋 **Script-Specific Replacements**

### **AutoLife.md**
```asm
define(pMe,40000000)                    → 0x11BBFC    // D2CLIENT_PlayerUnit
define(GetUnitStat,60000004)            → -10973      // D2COMMON_GetUnitStat  
define(GetUnitState,60000008)           → -10494      // D2COMMON_GetUnitState
define(Health,6000000C)                 → CUSTOM      // Implement using FindItem + UseItem
```

### **AutoMana.md**
```asm
define(pMe,40000000)                    → 0x11BBFC    // D2CLIENT_PlayerUnit
define(GetUnitStat,60000004)            → -10973      // D2COMMON_GetUnitStat
define(GetUnitState,60000008)           → -10494      // D2COMMON_GetUnitState  
define(Mana,60000010)                   → CUSTOM      // Implement using FindItem + UseItem
```

### **Health.md**
```asm
define(pMe,40000000)                    → 0x11BBFC    // D2CLIENT_PlayerUnit
define(GameReady,60000000)              → 0xA4D60     // D2CLIENT_GetPlayerUnit + validation
define(InTown,60000004)                 → CUSTOM      // Implement using D2CLIENT_PlayerArea
define(FindItem,60000008)               → CUSTOM      // Implement inventory traversal
define(UseItem,6000000C)                → CUSTOM      // Implement using D2NET_SendPacket (-10024)
```

### **GameReady.md**
```asm
define(pMe,40000000)                    → 0x11BBFC    // D2CLIENT_PlayerUnit
define(D2CLIENT_GetPlayerUnit,60000000) → 0xA4D60     // D2CLIENT_GetPlayerUnit
define(p_D2WIN_FirstControl,60000004)   → 0x214A0     // D2WIN_FirstControl
define(GetPlayerArea,60000008)          → 0x11C34C    // D2CLIENT_PlayerArea
```

### **TeleOnMap.md**
```asm
define(GameReady,60000000)              → 0xA4D60     // D2CLIENT_GetPlayerUnit + validation
define(D2CLIENT_GetPlayerUnit,60000004) → 0xA4D60     // D2CLIENT_GetPlayerUnit
define(CastOnMap,60000008)              → CUSTOM      // Implement using D2NET_SendPacket (-10024)
```

### **FindItem.md**
```asm
define(pMe,40000000)                    → 0x11BBFC    // D2CLIENT_PlayerUnit
define(GetItemLocation,60000000)        → CUSTOM      // Implement using item data analysis
define(GetUnitStat,60000004)            → -10973      // D2COMMON_GetUnitStat
```

### **UseItem.md**
```asm
define(pMe,40000000)                    → 0x11BBFC    // D2CLIENT_PlayerUnit
define(GetItemLocation,60000000)        → CUSTOM      // Implement using item data analysis
define(D2NET_SendPacket,60000004)       → -10024      // D2NET_SendPacket
```

### **CastOnMap.md**
```asm
define(GameReady,60000000)              → 0xA4D60     // D2CLIENT_GetPlayerUnit + validation
define(D2NET_SendPacket,60000004)       → -10024      // D2NET_SendPacket
```

### **PickItem.md**
```asm
define(D2NET_SendPacket,60000000)       → -10024      // D2NET_SendPacket
```

---

## 🔄 **Mass Find & Replace Operations**

### **Step 1: Global Replacements**
```bash
# Replace in ALL .md files:
find: define(pMe,40000000)
replace: define(pMe,0x11BBFC)

find: define(D2NET_SendPacket,60000000)
replace: define(D2NET_SendPacket,-10024)

find: define(D2NET_SendPacket,60000004)  
replace: define(D2NET_SendPacket,-10024)

find: define(GetUnitStat,60000004)
replace: define(D2COMMON_GetUnitStat,-10973)

find: define(GetUnitState,60000008)
replace: define(D2COMMON_GetUnitState,-10494)
```

### **Step 2: Script-Specific Replacements**
```bash
# GameReady.md only:
find: define(D2CLIENT_GetPlayerUnit,60000000)
replace: define(D2CLIENT_GetPlayerUnit,0xA4D60)

find: define(p_D2WIN_FirstControl,60000004)
replace: define(p_D2WIN_FirstControl,0x214A0)

# TeleOnMap.md only:
find: define(D2CLIENT_GetPlayerUnit,60000004)
replace: define(D2CLIENT_GetPlayerUnit,0xA4D60)
```

---

## ⚠️ **Functions Requiring Custom Implementation**

### **Missing Functions (Not in D2Ptrs.h)**
1. **GameReady()** - Composite validation function
   - Uses: D2CLIENT_GetPlayerUnit, D2WIN_FirstControl, D2CLIENT_PlayerArea
   - Logic: Validate player unit, UI state, inventory, path, room, level data

2. **InTown()** - Town area detection
   - Uses: D2CLIENT_PlayerArea
   - Logic: Check if current area ID matches town area IDs

3. **FindItem()** - Inventory item search
   - Uses: D2CLIENT_PlayerUnit, D2COMMON_GetUnitStat
   - Logic: Traverse inventory linked list, match item codes

4. **GetItemLocation()** - Storage location detection
   - Uses: Item data structure analysis
   - Logic: Determine if item is in inventory, belt, stash, or cube

5. **Health()** & **Mana()** - Potion usage functions
   - Uses: FindItem + UseItem
   - Logic: Find best available potion, use via packet sending

---

## 📊 **Address Type Reference**

### **FUNCPTR Addresses (Function Pointers)**
```cpp
D2CLIENT_GetPlayerUnit      → 0xA4D60     // Positive address
D2COMMON_GetUnitStat        → -10973      // Negative ordinal (DLL export)
D2COMMON_GetUnitState       → -10494      // Negative ordinal (DLL export)
D2NET_SendPacket            → -10024      // Negative ordinal (DLL export)
```

### **VARPTR Addresses (Variable Pointers)**
```cpp
D2CLIENT_PlayerUnit         → 0x11BBFC    // Memory address
D2WIN_FirstControl          → 0x214A0     // Memory address  
D2CLIENT_PlayerArea         → 0x11C34C    // Memory address
```

---

## 🎯 **Next Steps**

1. **Apply mass replacements** for all placeholder addresses
2. **Implement missing custom functions** (GameReady, InTown, FindItem, etc.)
3. **Verify memory structure offsets** for PD2 compatibility
4. **Test each script individually** before integration
5. **Update addresses if needed** for actual PD2 version differences

This reference provides the exact address mappings needed to convert all assembly scripts from placeholders to working PD2 bot functionality.
