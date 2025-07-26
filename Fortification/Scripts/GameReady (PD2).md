# "GameReady Assembly Implementation (PD2)"

## Metadata
- **ID**: 135003
- **Description**: "GameReady Assembly Implementation for Project Diablo 2"
- **Options**: Game state validation
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
  This script implements comprehensive game state validation in assembly.
  It checks multiple game components to ensure the game is in a valid,
  playable state before executing other functions.
  
  Based on the GameReady() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and pointers from D2Ptrs.h
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(p_D2WIN_FirstControl,0x214A0)    // VARPTR(D2WIN, FirstControl, Control*, 0x214A0)
define(D2CLIENT_PlayerArea,0x11C34C)    // VARPTR(D2CLIENT, PlayerArea, int, 0x11C34C)
define(VALIDPTR,CUSTOM_FUNCTION)        // Custom pointer validation function

// Hook for game state checking
aobscanmodule(gameStateHook,D2CLIENT.dll,85 C0 74 ?? 8B ?? ?? ?? ?? ??85)

gameStateHook:
  call GameReadyRoutine                 // Call our game ready check
  test eax,eax                          // Test return value
  // Original code
  test eax,eax                          // Original instruction
  je gameNotReady                       // Original jump
  jmp returnFromGameStateHook

GameReadyRoutine:
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  push edx
  
  // Check if pMe exists
  mov eax,[pMe]
  test eax,eax
  jz GameNotReady                       // Return FALSE if pMe is NULL
  
  // Check D2CLIENT_GetPlayerUnit()
  call D2CLIENT_GetPlayerUnit
  test eax,eax
  jz GameNotReady                       // Return FALSE if player unit invalid
  mov ebx,eax                           // Save player unit in ebx
  
  // Check if UI controls are not active (!*p_D2WIN_FirstControl)
  mov eax,[p_D2WIN_FirstControl]
  test eax,eax
  jz CheckInventory                     // Good if no active controls
  mov eax,[eax]                         // Dereference the pointer
  test eax,eax
  jnz GameNotReady                      // Return FALSE if controls active
  
CheckInventory:
  // Check player inventory (pInventory)
  mov eax,[ebx+60h]                     // Offset to pInventory (adjust for PD2)
  test eax,eax
  jz GameNotReady                       // Return FALSE if no inventory
  push eax                              // Save inventory pointer
  call VALIDPTR                         // Validate inventory pointer
  add esp,4
  test eax,eax
  jz GameNotReady                       // Return FALSE if invalid
  
CheckPath:
  // Check player path (pPath)
  mov eax,[ebx+38h]                     // Offset to pPath (adjust for PD2)
  test eax,eax
  jz GameNotReady                       // Return FALSE if no path
  push eax                              // Save path pointer
  call VALIDPTR                         // Validate path pointer
  add esp,4
  test eax,eax
  jz GameNotReady                       // Return FALSE if invalid
  mov ecx,eax                           // Save path in ecx
  
  // Check xPos
  mov eax,[ecx+10h]                     // Offset to xPos
  test eax,eax
  jz GameNotReady                       // Return FALSE if xPos is 0
  
CheckRoom1:
  // Check pRoom1
  mov eax,[ecx+20h]                     // Offset to pRoom1
  test eax,eax
  jz GameNotReady                       // Return FALSE if no room
  push eax                              // Save room1 pointer
  call VALIDPTR                         // Validate room1 pointer
  add esp,4
  test eax,eax
  jz GameNotReady                       // Return FALSE if invalid
  mov edx,eax                           // Save room1 in edx
  
CheckRoom2:
  // Check pRoom2
  mov eax,[edx+18h]                     // Offset to pRoom2
  test eax,eax
  jz GameNotReady                       // Return FALSE if no room2
  push eax                              // Save room2 pointer
  call VALIDPTR                         // Validate room2 pointer
  add esp,4
  test eax,eax
  jz GameNotReady                       // Return FALSE if invalid
  mov edx,eax                           // Save room2 in edx
  
CheckLevel:
  // Check pLevel
  mov eax,[edx+94h]                     // Offset to pLevel
  test eax,eax
  jz GameNotReady                       // Return FALSE if no level
  push eax                              // Save level pointer
  call VALIDPTR                         // Validate level pointer
  add esp,4
  test eax,eax
  jz GameNotReady                       // Return FALSE if invalid
  mov edx,eax                           // Save level in edx
  
  // Check dwLevelNo
  mov eax,[edx+1Ch]                     // Offset to dwLevelNo
  test eax,eax
  jz GameNotReady                       // Return FALSE if levelNo is 0
  
CheckAct:
  // Check pAct
  mov eax,[ebx+20h]                     // Offset to pAct
  test eax,eax
  jz GameNotReady                       // Return FALSE if no act
  push eax                              // Save act pointer
  call VALIDPTR                         // Validate act pointer
  add esp,4
  test eax,eax
  jz GameNotReady                       // Return FALSE if invalid
  
CheckPlayerArea:
  // Check D2CLIENT_PlayerArea != 0
  mov eax,[D2CLIENT_PlayerArea]         // Get current area
  test eax,eax
  jz GameNotReady                       // Return FALSE if no area
  
GameReady:
  mov eax,1                             // Return TRUE
  jmp GameReadyExit
  
GameNotReady:
  xor eax,eax                           // Return FALSE
  
GameReadyExit:
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

gameNotReady:
returnFromGameStateHook:

[DISABLE]

// Restore original code
gameStateHook:
  db 85 C0 74 ?? 8B ?? ?? ?? ?? ?? 85   // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(D2CLIENT_GetPlayerUnit)
unregistersymbol(p_D2WIN_FirstControl)
unregistersymbol(GetPlayerArea)
unregistersymbol(VALIDPTR)
```

## Configuration

### Validation Checks
The function performs multiple validation steps:
- **Player Unit**: Ensures pMe and D2CLIENT_GetPlayerUnit() are valid
- **UI State**: Checks that no UI controls are blocking gameplay
- **Inventory**: Validates player inventory exists and is accessible
- **Path Data**: Ensures player path and position data are valid
- **Room Data**: Validates current room and level information
- **Act Data**: Checks that act information is properly loaded

### Memory Offsets
Memory offsets may need adjustment for different PD2 versions:
- **pInventory**: +0x60 from player unit base
- **pPath**: +0x38 from player unit base  
- **xPos**: +0x10 from path base
- **pRoom1**: +0x20 from path base
- **pRoom2**: +0x18 from room1 base
- **pLevel**: +0x94 from room2 base
- **dwLevelNo**: +0x1C from level base
- **pAct**: +0x20 from player unit base

## Features

1. **Comprehensive Validation**: Checks all critical game state components
2. **Null Pointer Protection**: Validates all pointers before dereferencing
3. **UI State Awareness**: Ensures no blocking UI elements are active
4. **Position Validation**: Verifies player has valid position data
5. **Level Validation**: Confirms player is in a valid game level
6. **Fast Execution**: Optimized assembly for minimal performance impact

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `D2CLIENT_GetPlayerUnit()`: Game function to get player unit
- `p_D2WIN_FirstControl`: UI control state pointer
- `GetPlayerArea()`: Function to get current area
- `VALIDPTR()`: Pointer validation function

## Return Values

- **EAX = 1**: Game is ready and all checks passed
- **EAX = 0**: Game is not ready or validation failed

## Notes

- This function should be called before any game manipulation
- Memory offsets are version-specific and may need updates
- All register states are preserved except EAX (return value)
- Function is designed for high-frequency calling with minimal overhead
