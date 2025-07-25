# "TeleOnMap Assembly Implementation (PD2)"

## Metadata
- **ID**: 135009
- **Description**: "TeleOnMap Assembly Implementation for Project Diablo 2"
- **Options**: Teleport casting on map coordinates
- **VariableType**: Auto Assembler Script
- **Source File**: SendPacket Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.14d (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements teleport casting on map coordinates in assembly.
  It validates the teleport skill is selected and calls the generic map
  casting function to execute the teleport.
  
  Based on the TeleOnMap() function from Fortification\SendPacket.cpp
}

[ENABLE]

// Memory addresses and functions
define(GameReady,60000000)              // GameReady function
define(D2CLIENT_GetPlayerUnit,60000004) // Get player unit function
define(CastOnMap,60000008)              // Cast on map function
define(D2S_TELEPORT,54)                 // Teleport skill ID

// Hook for teleport on map
aobscanmodule(teleOnMapHook,D2CLIENT.dll,E8 ?? ?? ?? ?? 84 C0 75 ??)

teleOnMapHook:
  call TeleOnMapRoutine                 // Call our teleport function
  test al,al                            // Test return value (bool)
  jmp returnFromTeleOnMapHook

TeleOnMapRoutine:
  // Parameters: x (WORD), y (WORD), Left (bool)
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  push edx
  
  mov bx,[ebp+8]                        // x coordinate
  mov cx,[ebp+12]                       // y coordinate  
  mov dl,[ebp+16]                       // Left flag
  
  // Check if game is ready
  call GameReady
  test eax,eax
  jz TeleOnMapFailed                    // Return 0 if game not ready
  
  // Get player unit
  call D2CLIENT_GetPlayerUnit
  test eax,eax
  jz TeleOnMapFailed                    // Return 0 if no player unit
  
  // Check if teleport is the selected right skill
  // pInfo->pRightSkill->pSkillInfo->wSkillId
  mov eax,[eax+14h]                     // pInfo offset
  test eax,eax
  jz TeleOnMapFailed                    // Return 0 if no info
  
  mov eax,[eax+0Ch]                     // pRightSkill offset
  test eax,eax
  jz TeleOnMapFailed                    // Return 0 if no right skill
  
  mov eax,[eax]                         // pSkillInfo offset
  test eax,eax
  jz TeleOnMapFailed                    // Return 0 if no skill info
  
  movzx eax,word ptr [eax+04h]          // wSkillId offset
  cmp eax,D2S_TELEPORT                  // Compare with teleport skill ID
  jne TeleOnMapFailed                   // Return 0 if not teleport
  
  // Call CastOnMap function
  movzx edx,dl                          // Extend Left flag to DWORD
  push edx                              // Left parameter
  movzx ecx,cx                          // Extend y to DWORD
  push ecx                              // y parameter
  movzx ebx,bx                          // Extend x to DWORD
  push ebx                              // x parameter
  call CastOnMap                        // Call generic cast function
  add esp,12                            // Clean stack
  
  // Return result from CastOnMap
  jmp TeleOnMapExit
  
TeleOnMapFailed:
  xor eax,eax                           // Return 0 (false)
  
TeleOnMapExit:
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromTeleOnMapHook:

[DISABLE]

// Restore original code
teleOnMapHook:
  db E8 ?? ?? ?? ?? 84 C0 75 ??        // Original bytes

// Clean up
unregistersymbol(GameReady)
unregistersymbol(D2CLIENT_GetPlayerUnit)
unregistersymbol(CastOnMap)
```

## Configuration

### Skill Validation
- **D2S_TELEPORT (54)**: Teleport skill identifier
- **Right Skill Check**: Validates teleport is selected on right mouse button
- **Skill Info Chain**: Follows pInfo->pRightSkill->pSkillInfo->wSkillId

### Coordinate System
- **x, y**: Map coordinates (WORD values)
- **Left Flag**: Boolean indicating left/right mouse button cast

### Memory Offsets
Player unit structure offsets:
- **pInfo**: +0x14 from player unit
- **pRightSkill**: +0x0C from info structure  
- **pSkillInfo**: +0x00 from skill structure
- **wSkillId**: +0x04 from skill info structure

## Features

1. **Game State Validation**: Ensures game is ready before casting
2. **Player Unit Verification**: Validates player unit exists
3. **Skill Verification**: Confirms teleport skill is selected
4. **Parameter Validation**: Checks all required parameters
5. **Generic Casting**: Delegates actual casting to CastOnMap function
6. **Return Status**: Boolean return indicating success/failure

## Dependencies

This script requires the following:
- `GameReady()`: Game state validation function
- `D2CLIENT_GetPlayerUnit()`: Function to get player unit
- `CastOnMap()`: Generic map casting function
- Player unit structure with skill information

## Function Signature

```asm
TeleOnMapRoutine:
  ; Parameters:
  ; [ebp+8]  - x (WORD) - Target X coordinate
  ; [ebp+12] - y (WORD) - Target Y coordinate  
  ; [ebp+16] - Left (bool) - Left mouse button flag
  ; Returns: EAX = 1 if successful, 0 if failed
```

## Usage Example

```asm
push 1                               // Left mouse button
push 5432                            // Y coordinate
push 1234                            // X coordinate
call TeleOnMapRoutine                // Attempt teleport
add esp,12                           // Clean stack
test eax,eax                         // Check if successful
jz TeleportFailed                    // Jump if failed
```

## Validation Chain

The function performs validation in this order:
1. **Game Ready**: Basic game state check
2. **Player Unit**: Verify player unit exists
3. **Player Info**: Check player info structure
4. **Right Skill**: Verify right skill exists
5. **Skill Info**: Check skill info structure
6. **Skill ID**: Confirm teleport is selected

## Notes

- Function returns immediately if teleport is not the selected skill
- All coordinate parameters are extended from WORD to DWORD for function calls
- Boolean parameters are properly extended for stack calling convention
- Function preserves all registers except EAX (return value)
- Designed to prevent accidental casting with wrong skills selected
- Integrates with existing CastOnMap infrastructure
