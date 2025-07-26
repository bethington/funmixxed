# Script to D2Ptrs.h Pointer Mapping Analysis

## Overview
This document maps each script in the `Fortification\Scripts` folder to the specific pointer locations defined in `Fortification\D2Ptrs.h`. Each script uses placeholder addresses that need to be replaced with the actual addresses from D2Ptrs.h.

---

## 🎯 **Script-by-Script Pointer Mapping**

### **1. AutoLife.md**
**Purpose**: Automatic health potion usage with percentage threshold

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(pMe,40000000)                    // Player unit pointer
define(GameReady,60000000)              // GameReady function  
define(GetUnitStat,60000004)            // GetUnitStat function
define(GetUnitState,60000008)           // GetUnitState function
define(Health,6000000C)                 // Health potion use function

// AFTER (D2Ptrs.h Actual):
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2COMMON_GetUnitStat,-10973)     // FUNCPTR(D2COMMON, GetUnitStat, DWORD __stdcall, (UnitAny* pUnit, DWORD dwStat, DWORD dwStat2), -10973)
define(D2COMMON_GetUnitState,-10494)    // FUNCPTR(D2COMMON, GetUnitState, INT __stdcall, (LPUNITANY Unit, DWORD State), -10494)
define(Health,CUSTOM_FUNCTION)          // Custom function using D2NET_SendPacket
```

#### **Required Stats/Constants:**
- `STAT_HP`: 6 (Current health points)
- `STAT_MAXHP`: 7 (Maximum health points)
- `AFFECT_HEALTHPOT`: 100 (Health potion effect state)

---

### **2. AutoMana (PD2).md**
**Purpose**: Automatic mana potion usage with percentage threshold

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(pMe,40000000)                    // Player unit pointer
define(GameReady,60000000)              // GameReady function
define(GetUnitStat,60000004)            // GetUnitStat function
define(GetUnitState,60000008)           // GetUnitState function
define(Mana,60000010)                   // Mana function

// AFTER (D2Ptrs.h Actual):
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2COMMON_GetUnitStat,-10973)     // FUNCPTR(D2COMMON, GetUnitStat, DWORD __stdcall, (UnitAny* pUnit, DWORD dwStat, DWORD dwStat2), -10973)
define(D2COMMON_GetUnitState,-10494)    // FUNCPTR(D2COMMON, GetUnitState, INT __stdcall, (LPUNITANY Unit, DWORD State), -10494)
define(Mana,CUSTOM_FUNCTION)            // Custom function using FindItem + UseItem
```

#### **Required Stats/Constants:**
- `STAT_MANA`: 8 (Current mana points)
- `STAT_MAXMANA`: 9 (Maximum mana points)
- `AFFECT_MANAPOT`: 101 (Mana potion effect state)

---

### **3. Health (PD2).md**
**Purpose**: Health potion selection and usage from inventory

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(pMe,40000000)                    // Player unit pointer
define(GameReady,60000000)              // GameReady function
define(InTown,60000004)                 // InTown function
define(FindItem,60000008)               // FindItem function
define(UseItem,6000000C)                // UseItem function

// AFTER (D2Ptrs.h Actual):
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2CLIENT_PlayerArea,0x11C34C)    // VARPTR(D2CLIENT, PlayerArea, int, 0x11C34C) - for town checking
define(FindItem,CUSTOM_FUNCTION)        // Custom function using inventory traversal
define(UseItem,CUSTOM_FUNCTION)         // Custom function using D2NET_SendPacket
```

#### **Health Potion Item Codes (PD2):**
- `SUPER_HEALTH_POTION`: 593 (Best quality)
- `GREATER_HEALTH_POTION`: 592
- `HEALTH_POTION`: 591
- `LIGHT_HEALTH_POTION`: 590
- `MINOR_HEALTH_POTION`: 589 (Lowest quality)

---

### **4. GameReady (PD2).md**
**Purpose**: Comprehensive game state validation

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(pMe,40000000)                    // Player unit pointer
define(D2CLIENT_GetPlayerUnit,60000000) // Get player function
define(p_D2WIN_FirstControl,60000004)   // UI control pointer
define(GetPlayerArea,60000008)          // Player area function

// AFTER (D2Ptrs.h Actual):
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(p_D2WIN_FirstControl,0x214A0)    // VARPTR(D2WIN, FirstControl, Control*, 0x214A0)
define(D2CLIENT_PlayerArea,0x11C34C)    // VARPTR(D2CLIENT, PlayerArea, int, 0x11C34C)
```

#### **Validation Chain Pointers:**
- Player unit validation
- UI control state checking
- Inventory pointer validation
- Path and room data validation
- Act and level validation

---

### **5. TeleOnMap (PD2).md**
**Purpose**: Teleport casting on map coordinates

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(GameReady,60000000)              // GameReady function
define(D2CLIENT_GetPlayerUnit,60000004) // Get player unit function
define(CastOnMap,60000008)              // Cast on map function

// AFTER (D2Ptrs.h Actual):
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(CastOnMap,CUSTOM_FUNCTION)       // Custom function using D2NET_SendPacket
define(D2NET_SendPacket,-10024)         // FUNCPTR(D2NET, SendPacket, void __stdcall, (DWORD aLen, DWORD arg1, BYTE* aPacket), -10024)
```

#### **Constants:**
- `D2S_TELEPORT`: 54 (Teleport skill ID)

---

### **6. FindItem (PD2).md**
**Purpose**: Item searching by code and storage location

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(pMe,40000000)                    // Player unit pointer
define(GetItemLocation,60000000)        // GetItemLocation function
define(GetUnitStat,60000004)            // GetUnitStat function

// AFTER (D2Ptrs.h Actual):
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(GetItemLocation,CUSTOM_FUNCTION) // Custom function using item data analysis
define(D2COMMON_GetUnitStat,-10973)     // FUNCPTR(D2COMMON, GetUnitStat, DWORD __stdcall, (UnitAny* pUnit, DWORD dwStat, DWORD dwStat2), -10973)
```

#### **Constants:**
- `UNIT_TYPE_ITEM`: 4 (Item unit type)
- `STAT_AMMOQUANTITY`: 70 (For tome quantity checking)
- Storage location constants (1-4)

---

### **7. UseItem (PD2).md**
**Purpose**: Item usage with network packet sending

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(pMe,40000000)                    // Player unit pointer
define(GetItemLocation,60000000)        // GetItemLocation function
define(D2NET_SendPacket,60000004)       // Packet sending function

// AFTER (D2Ptrs.h Actual):
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(GetItemLocation,CUSTOM_FUNCTION) // Custom function using item data analysis
define(D2NET_SendPacket,-10024)         // FUNCPTR(D2NET, SendPacket, void __stdcall, (DWORD aLen, DWORD arg1, BYTE* aPacket), -10024)
```

#### **Packet Types:**
- `0x20`: Inventory/Stash/Cube usage
- `0x26`: Belt usage

---

### **8. CastOnMap (PD2).md**
**Purpose**: Generic spell casting on map coordinates

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(GameReady,60000000)              // GameReady function
define(D2NET_SendPacket,60000004)       // Network packet function

// AFTER (D2Ptrs.h Actual):
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2NET_SendPacket,-10024)         // FUNCPTR(D2NET, SendPacket, void __stdcall, (DWORD aLen, DWORD arg1, BYTE* aPacket), -10024)
```

#### **Packet Types:**
- `0x08`: Left mouse button casting
- `0x0F`: Right mouse button casting

---

### **9. PickItem (PD2).md**
**Purpose**: Item pickup with network packet sending

#### **Current Script Placeholders → D2Ptrs.h Replacements:**
```asm
// BEFORE (Script Placeholders):
define(D2NET_SendPacket,60000000)       // Network packet function

// AFTER (D2Ptrs.h Actual):
define(D2NET_SendPacket,-10024)         // FUNCPTR(D2NET, SendPacket, void __stdcall, (DWORD aLen, DWORD arg1, BYTE* aPacket), -10024)
```

#### **Constants:**
- Packet type `0x16`: Item pickup command

---

## 📊 **Pointer Usage Summary Table**

| D2Ptrs.h Pointer | Address | Type | Scripts Using | Function |
|------------------|---------|------|---------------|----------|
| `D2CLIENT_PlayerUnit` | 0x11BBFC | VARPTR | AutoLife, AutoMana, Health, GameReady, FindItem, UseItem | Core player reference |
| `D2NET_SendPacket` | -10024 | FUNCPTR | TeleOnMap, CastOnMap, UseItem, PickItem | Network packet sending |
| `D2COMMON_GetUnitStat` | -10973 | FUNCPTR | AutoLife, AutoMana, FindItem | Statistics reading |
| `D2CLIENT_GetPlayerUnit` | 0xA4D60 | FUNCPTR | Health, GameReady, TeleOnMap, CastOnMap | Player unit retrieval |
| `D2COMMON_GetUnitState` | -10494 | FUNCPTR | AutoLife, AutoMana | Effect state checking |
| `D2WIN_FirstControl` | 0x214A0 | VARPTR | GameReady | UI state validation |
| `D2CLIENT_PlayerArea` | 0x11C34C | VARPTR | Health, GameReady | Area/town checking |

---

## 🔧 **Implementation Notes**

### **Negative Address Convention**
D2COMMON functions use negative ordinals (DLL exports):
- `-10024`: D2NET_SendPacket
- `-10494`: D2COMMON_GetUnitState  
- `-10973`: D2COMMON_GetUnitStat

### **Positive Address Convention**
D2CLIENT functions use positive memory addresses:
- `0xA4D60`: D2CLIENT_GetPlayerUnit
- `0x11BBFC`: D2CLIENT_PlayerUnit (VARPTR)
- `0x214A0`: D2WIN_FirstControl (VARPTR)

### **DLL Mapping System**
The `DLLOFFSET(a1,b1)` macro maps to specific DLLs:
```cpp
enum {DLLNO_D2CLIENT, DLLNO_D2COMMON, DLLNO_D2GFX, DLLNO_D2LANG, 
      DLLNO_D2WIN, DLLNO_D2NET, DLLNO_D2GAME, DLLNO_D2LAUNCH, 
      DLLNO_FOG, DLLNO_BNCLIENT, DLLNO_STORM, DLLNO_D2CMP, DLLNO_D2MULTI};
```

---

## ⚠️ **Critical Compatibility Issues**

### **Version Mismatch**
- **D2Ptrs.h**: Comments indicate "Updated 1.13c"
- **Scripts**: Target "1.14d (Project Diablo 2)"
- **Resolution**: Addresses may need updating for PD2 compatibility

### **Custom Functions Required**
Several scripts reference functions that don't exist in D2Ptrs.h:
- `GameReady()`: Composite function using multiple validation pointers
- `InTown()`: Function to check if player is in town area
- `FindItem()`: Custom inventory traversal function
- `GetItemLocation()`: Function to determine item storage location

### **Memory Structure Offsets**
Scripts use hardcoded offsets that may be PD2-specific:
- Player inventory: `+0x60`
- Item data: `+0x30`
- Path data: `+0x38`
- These may need adjustment for PD2's memory layout

---

## 🎯 **Recommended Implementation Strategy**

1. **Phase 1**: Update all placeholder addresses with D2Ptrs.h values
2. **Phase 2**: Implement missing custom functions (GameReady, InTown, etc.)
3. **Phase 3**: Verify memory structure offsets for PD2 compatibility
4. **Phase 4**: Test and adjust addresses if needed for PD2 version

This mapping provides the foundation for converting the assembly scripts from placeholder addresses to working PD2 bot functionality using the exact pointer locations defined in D2Ptrs.h.
