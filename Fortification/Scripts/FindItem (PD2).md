# "FindItem Assembly Implementation (PD2)"

## Metadata
- **ID**: 135004
- **Description**: "FindItem Assembly Implementation for Project Diablo 2"
- **Options**: Item searching by code and location
- **VariableType**: Auto Assembler Script
- **Source File**: D2Helpers Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.13c (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements item searching functionality in assembly.
  It searches through the player's inventory for items matching a specific
  item code and storage location.
  
  Based on the FindItem() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and constants from D2Ptrs.h
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(GetItemLocation,CUSTOM_FUNCTION) // Custom function using item data analysis
define(D2COMMON_GetUnitStat,-10973)     // FUNCPTR(D2COMMON, GetUnitStat, DWORD __stdcall, (UnitAny* pUnit, DWORD dwStat, DWORD dwStat2), -10973)
define(UNIT_TYPE_ITEM,4)                // Item unit type constant
define(STAT_AMMOQUANTITY,70)            // Ammunition quantity stat
define(STORAGE_STASH,1)                 // Storage location constants
define(STORAGE_BELT,2)
define(STORAGE_INVENTORY,3)
define(STORAGE_CUBE,4)

// Hook for item searching
aobscanmodule(findItemHook,D2CLIENT.dll,8B ?? ?? ?? ?? ?? 85 ?? 74 ??)

findItemHook:
  // This would be hooked where FindItem is called
  jmp returnFromFindItemHook

FindItemRoutine:
  // Parameters: dwCode (item code), nLoc (location)
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  push edx
  push esi
  
  mov ebx,[ebp+8]                       // dwCode parameter
  mov ecx,[ebp+12]                      // nLoc parameter
  
  // Get player unit
  mov eax,[pMe]
  test eax,eax
  jz FindItemNotFound                   // Return FALSE if no player
  
  // Get inventory
  mov eax,[eax+60h]                     // Offset to pInventory
  test eax,eax
  jz FindItemNotFound                   // Return FALSE if no inventory
  
  // Get first item
  mov esi,[eax+A0h]                     // Offset to pFirstItem
  test esi,esi
  jz FindItemNotFound                   // Return FALSE if no items
  
FindItemLoop:
  // Check if current item is valid
  test esi,esi
  jz FindItemNotFound                   // End of list, item not found
  
  // Check if item type is UNIT_TYPE_ITEM
  mov eax,[esi+0Ch]                     // Offset to dwType
  cmp eax,UNIT_TYPE_ITEM
  jne FindItemNext                      // Skip if not item type
  
  // Check if item code matches (dwTxtFileNo)
  mov eax,[esi+4h]                      // Offset to dwTxtFileNo
  cmp eax,ebx                           // Compare with target code
  jne FindItemNext                      // Skip if code doesn't match
  
  // Special check for item code 520 (PD2 TP Tome)
  cmp eax,520
  jne CheckItemLocation                 // Skip special check if not TP tome
  
  // Check ammunition quantity for TP tome
  push 0                                // dwStat2 parameter (0 for base stat)
  push STAT_AMMOQUANTITY                // Stat ID
  push esi                              // Item unit
  call D2COMMON_GetUnitStat             // Get ammo quantity
  add esp,12                            // Clean stack (3 parameters)
  test eax,eax                          // Check if quantity is 0
  jz FindItemNext                       // Skip if empty tome
  
CheckItemLocation:
  // Check if item is in correct location
  push esi                              // Item parameter
  call GetItemLocation                  // Get item location
  add esp,4                             // Clean stack
  cmp eax,ecx                           // Compare with target location
  je FindItemFound                      // Found matching item
  
FindItemNext:
  // Move to next item in inventory
  mov eax,[esi+30h]                     // Offset to pItemData
  test eax,eax
  jz FindItemNotFound                   // No item data
  mov esi,[eax+38h]                     // Offset to pNextInvItem
  jmp FindItemLoop                      // Continue searching
  
FindItemFound:
  mov eax,esi                           // Return item pointer
  jmp FindItemExit
  
FindItemNotFound:
  xor eax,eax                           // Return NULL (FALSE)
  
FindItemExit:
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromFindItemHook:

[DISABLE]

// Restore original code
findItemHook:
  db 8B ?? ?? ?? ?? ?? 85 ?? 74 ??      // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(GetItemLocation)
unregistersymbol(GetUnitStat)
```

## Configuration

### Item Codes (PD2)
Common item codes for Project Diablo 2:
- **517**: Small Rejuvenation Potion
- **518**: Full Rejuvenation Potion
- **520**: Town Portal Tome
- **531**: Town Portal Scroll
- **589-593**: Health Potions (Minor to Super)
- **594-598**: Mana Potions (Minor to Super)

### Storage Locations
Storage location constants:
- **STORAGE_STASH (1)**: Player stash
- **STORAGE_BELT (2)**: Belt slots
- **STORAGE_INVENTORY (3)**: Main inventory
- **STORAGE_CUBE (4)**: Horadric Cube

### Memory Offsets
Key memory offsets for item traversal:
- **pInventory**: +0x60 from player unit
- **pFirstItem**: +0xA0 from inventory
- **dwType**: +0x0C from item unit
- **dwTxtFileNo**: +0x04 from item unit
- **pItemData**: +0x30 from item unit
- **pNextInvItem**: +0x38 from item data

## Features

1. **Item Code Matching**: Searches for specific item types by code
2. **Location Filtering**: Only returns items in specified storage location
3. **Special Handling**: Custom logic for tome items with quantity checks
4. **Linked List Traversal**: Efficiently walks through inventory item chain
5. **Null Safety**: Comprehensive null pointer checking
6. **Fast Execution**: Optimized assembly for minimal performance impact

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `GetItemLocation()`: Function to determine item storage location
- `GetUnitStat()`: Function to read item statistics
- Item type and stat constants defined in game headers

## Function Signature

```asm
FindItemRoutine:
  ; Parameters (pushed in reverse order):
  ; [ebp+12] - nLoc (storage location to search)
  ; [ebp+8]  - dwCode (item code to find)
  ; Returns: EAX = Item pointer or NULL if not found
```

## Usage Example

```asm
push STORAGE_BELT                     // Search in belt
push 518                             // Full rejuvenation potion
call FindItemRoutine                 // Search for item
add esp,8                            // Clean stack
test eax,eax                         // Check if item found
jz NoItemFound                       // Jump if not found
; eax now contains pointer to found item
```

## Notes

- Function returns NULL (0) if item is not found
- Special handling for TP tomes checks ammunition quantity
- Memory offsets are specific to PD2 and may need adjustment
- Function preserves all registers except EAX (return value)
- Designed for high-frequency calling with minimal overhead
