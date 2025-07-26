# "Health Assembly Implementation (PD2)"

## Metadata
- **ID**: 135006
- **Description**: "Health Potion Usage Assembly Implementation for Project Diablo 2"
- **Options**: Automatic health potion selection and usage
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
  This script implements automatic health potion usage in assembly.
  It searches for the highest quality health potion available and uses it,
  checking multiple storage locations in priority order.
  
  Based on the Health() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and functions from D2Ptrs.h
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2CLIENT_PlayerArea,0x11C34C)    // VARPTR(D2CLIENT, PlayerArea, int, 0x11C34C) - for town checking
define(FindItem,CUSTOM_FUNCTION)        // Custom function using inventory traversal
define(UseItem,CUSTOM_FUNCTION)         // Custom function using D2NET_SendPacket
define(STORAGE_STASH,1)                 // Storage location constants
define(STORAGE_BELT,2)
define(STORAGE_INVENTORY,3)
define(STORAGE_CUBE,4)

// PD2 Health Potion Item Codes (Super to Minor)
define(SUPER_HEALTH_POTION,593)
define(GREATER_HEALTH_POTION,592)
define(HEALTH_POTION,591)
define(LIGHT_HEALTH_POTION,590)
define(MINOR_HEALTH_POTION,589)

// Hook for health potion usage
aobscanmodule(healthPotionHook,D2CLIENT.dll,E8 ?? ?? ?? ?? 85 C0 74 ??)

healthPotionHook:
  call HealthRoutine                    // Call our health function
  test eax,eax                          // Test return value
  jmp returnFromHealthHook

HealthRoutine:
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  push edx
  push esi
  push edi
  
  // Check if game is ready (using D2CLIENT_GetPlayerUnit)
  call D2CLIENT_GetPlayerUnit
  test eax,eax
  jz HealthFailed                       // Exit if game not ready
  
  // Check if player is in town (using D2CLIENT_PlayerArea)
  mov eax,[D2CLIENT_PlayerArea]         // Get current area ID
  test eax,eax
  jz HealthFailed                       // Exit if no area info
  
  // Check for town area IDs (1, 40, 75, 103, 109 are common town IDs)
  cmp eax,1                             // Rogue Encampment
  je HealthFailed
  cmp eax,40                            // Lut Gholein  
  je HealthFailed
  cmp eax,75                            // Kurast Docks
  je HealthFailed
  cmp eax,103                           // Pandemonium Fortress
  je HealthFailed
  cmp eax,109                           // Harrogath
  je HealthFailed
  
  // Check if cursor item exists
  mov eax,[pMe]
  mov eax,[eax+60h]                     // pInventory offset
  test eax,eax
  jz HealthFailed                       // Exit if no inventory
  
  mov eax,[eax+B0h]                     // pCursorItem offset
  test eax,eax
  jnz HealthFailed                      // Exit if cursor item exists
  
  // Item codes array: {593, 592, 591, 590, 589}
  mov edi,0                             // Initialize loop counter
  
HealthSearchLoop:
  cmp edi,5                             // Check if we've tried all 5 potions
  jge HealthFailed                      // Exit if no potions found
  
  // Get current item code
  mov ebx,SUPER_HEALTH_POTION           // Start with super health potion (593)
  sub ebx,edi                           // Subtract counter to get next potion code
  
  xor esi,esi                           // Clear item pointer
  
  // Try stash first
  push STORAGE_STASH                    // Storage location
  push ebx                              // Item code
  call FindItem                         // Search for item
  add esp,8                             // Clean stack
  test eax,eax
  jnz HealthItemFound                   // Found item in stash
  
  // Try belt next
  push STORAGE_BELT                     // Storage location
  push ebx                              // Item code
  call FindItem                         // Search for item
  add esp,8                             // Clean stack
  test eax,eax
  jnz HealthItemFound                   // Found item in belt
  
  // Try inventory next
  push STORAGE_INVENTORY                // Storage location
  push ebx                              // Item code
  call FindItem                         // Search for item
  add esp,8                             // Clean stack
  test eax,eax
  jnz HealthItemFound                   // Found item in inventory
  
  // Try cube last
  push STORAGE_CUBE                     // Storage location
  push ebx                              // Item code
  call FindItem                         // Search for item
  add esp,8                             // Clean stack
  test eax,eax
  jnz HealthItemFound                   // Found item in cube
  
  // No item found, try next potion type
  inc edi                               // Increment counter
  jmp HealthSearchLoop                  // Continue searching
  
HealthItemFound:
  mov esi,eax                           // Save item pointer
  
  // Use the found item
  push esi                              // Item pointer
  call UseItem                          // Use the health potion
  add esp,4                             // Clean stack
  
  mov eax,1                             // Return TRUE (success)
  jmp HealthExit
  
HealthFailed:
  xor eax,eax                           // Return FALSE (failure)
  
HealthExit:
  pop edi
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromHealthHook:

[DISABLE]

// Restore original code
healthPotionHook:
  db E8 ?? ?? ?? ?? 85 C0 74 ??        // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(GameReady)
unregistersymbol(InTown)
unregistersymbol(FindItem)
unregistersymbol(UseItem)
```

## Configuration

### Health Potion Priority
Potions are used in order of effectiveness (highest to lowest):
- **593**: Super Healing Potion (best)
- **592**: Greater Healing Potion
- **591**: Healing Potion
- **590**: Light Healing Potion
- **589**: Minor Healing Potion (weakest)

### Storage Priority
Storage locations are searched in this order:
1. **Stash**: Checked first for convenience
2. **Belt**: Quick access potions
3. **Inventory**: Main storage area
4. **Cube**: Alternative storage

### Usage Conditions
Function will only execute when:
- Game is ready and loaded
- Player is not in town
- No item is currently on cursor
- At least one health potion is available

## Features

1. **Quality Priority**: Automatically selects highest quality available potion
2. **Multi-Location Search**: Searches all storage areas systematically
3. **Town Safety**: Prevents usage in town areas
4. **Cursor Check**: Ensures no interference with held items
5. **Game State Validation**: Comprehensive safety checks
6. **Efficient Search**: Stops at first available potion of each type

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `GameReady()`: Game state validation function
- `InTown()`: Town detection function
- `FindItem()`: Item searching function
- `UseItem()`: Item usage function

## Function Signature

```asm
HealthRoutine:
  ; Parameters: None
  ; Returns: EAX = 1 if potion used successfully, 0 if failed
```

## Usage Example

```asm
call HealthRoutine                    // Use health potion
test eax,eax                          // Check if successful
jz NoHealthPotionUsed                 // Jump if failed
; Health potion was used successfully
```

## Memory Offsets

Key offsets used:
- **pInventory**: +0x60 from player unit
- **pCursorItem**: +0xB0 from inventory

## Algorithm Flow

1. **Validation**: Check game state, town status, cursor state
2. **Priority Loop**: Iterate through potion types (best to worst)
3. **Location Search**: For each potion type, search all storage locations
4. **Usage**: Use first available potion found
5. **Return**: Report success/failure status

## Notes

- Function returns immediately after using first available potion
- Higher quality potions are prioritized over quantity
- All safety checks must pass before attempting usage
- Function preserves all registers except EAX (return value)
- Designed for integration with automatic health management systems
