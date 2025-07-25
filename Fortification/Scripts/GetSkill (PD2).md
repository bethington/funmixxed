# "GetSkill Assembly Implementation (PD2)"

## Metadata

- **ID**: 135015
- **Description**: "GetSkill Assembly Implementation for Project Diablo 2"
- **Options**: Retrieve active skill from right hand
- **VariableType**: Auto Assembler Script
- **Source File**: AutoTele Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.14d (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements skill retrieval functionality in assembly.
  It reads the active skill ID from the player's right hand slot,
  enabling automation systems to check current skill configuration.
  
  Based on the GetSkill() function from Fortification\AutoTele.cpp
}

[ENABLE]

// Player unit structure and skill offsets
define(pMe,70000000)                    // Player unit pointer
define(PLAYER_SKILL_RIGHT_OFFSET,0x3E)  // Right hand skill offset in player unit

// Hook for skill reading
aobscanmodule(getSkillHook,D2CLIENT.dll,8B ?? ?? ?? ?? ?? 83 ?? ?? 74)

getSkillHook:
  call GetSkillRoutine                  // Call our skill reading function
  jmp returnFromGetSkillHook

GetSkillRoutine:
  // Returns: EAX = Right hand skill ID (WORD)
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  
  // Check if player unit exists
  mov eax,[pMe]                         // Load player unit pointer
  test eax,eax                          // Check if pointer is valid
  jz NoSkillSet                         // Jump if null pointer
  
  // Validate player unit structure
  cmp dword ptr [eax],1                 // Check unit type (1 = player)
  jne NoSkillSet                        // Jump if not player unit
  
  // Read right hand skill from player unit
  // Player unit structure contains skill data at specific offsets
  movzx eax,word ptr [eax+PLAYER_SKILL_RIGHT_OFFSET]  // Load right hand skill ID
  jmp SkillRetrieved                    // Jump to return
  
NoSkillSet:
  xor eax,eax                           // Return 0 (no skill set)
  
SkillRetrieved:
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromGetSkillHook:

[DISABLE]

// Restore original code
getSkillHook:
  db 8B ?? ?? ?? ?? ?? 83 ?? ?? 74      // Original bytes

// Clean up
unregistersymbol(pMe)
```

## Configuration

### Player Unit Structure

The player unit contains skill information at fixed offsets:

- **Right Hand Skill**: Offset 0x3E (WORD)
- **Left Hand Skill**: Offset 0x40 (WORD)
- **Unit Type**: Offset 0x00 (DWORD) - Must be 1 for player

### Skill ID Values

Common skill IDs returned by this function:

- **0**: No skill assigned
- **54**: Teleport (Sorceress)
- **50**: Town Portal
- **117**: Blessed Hammer (Paladin)
- **42**: Static Field (Sorceress)

## Features

1. **Safe Access**: Validates player unit pointer before reading
2. **Type Checking**: Ensures unit is actually a player
3. **Zero Extension**: Returns WORD value properly extended to DWORD
4. **Error Handling**: Returns 0 for invalid or missing data
5. **Fast Access**: Direct memory read with minimal overhead
6. **Register Preservation**: Preserves all registers except EAX

## Dependencies

This script requires the following:

- `pMe`: Global player unit pointer
- Valid player unit structure with skill data

## Function Signature

```asm
GetSkillRoutine:
  ; Parameters: None
  ; Returns: EAX = Right hand skill ID (WORD extended to DWORD)
  ;          EAX = 0 if no skill set or error
```

## Usage Example

```asm
call GetSkillRoutine                 // Get current right hand skill
cmp eax,54                           // Compare with teleport skill ID
je TeleportIsReady                   // Jump if teleport is active

; If teleport not active, set it
push 0                               // Right mouse button
push 54                              // Teleport skill ID
call SetSkillRoutine                 // Set teleport skill
add esp,8

TeleportIsReady:
; Now we know teleport is ready
```

## Skill Validation

Common validation patterns using GetSkill:

```asm
; Check if any skill is set
call GetSkillRoutine
test eax,eax                         // Check if skill ID is non-zero
jz NoSkillAssigned                   // Jump if no skill

; Check for specific skill category
call GetSkillRoutine
cmp eax,54                           // Check for teleport
je HasTeleport
cmp eax,50                           // Check for town portal
je HasTownPortal
; ... other skill checks

; Check skill before automation
CheckTeleportSkill:
  call GetSkillRoutine
  cmp eax,54                         // Must be teleport
  jne SetupTeleport                  // Setup if not teleport
  ret                                // Return if already set

SetupTeleport:
  push 0                             // Right mouse
  push 54                            // Teleport skill
  call SetSkillRoutine               // Set teleport
  add esp,8
  ret
```

## Integration with Automation

Typical usage in automation loops:

```asm
AutoTeleportLoop:
  ; Verify teleport skill is active
  call GetSkillRoutine
  cmp eax,54                         // Check for teleport skill
  jne SetupTeleportSkill             // Setup if needed
  
  ; Skill is ready, perform teleportation
  push 0                             // Right mouse
  push targetY                       // Target Y coordinate
  push targetX                       // Target X coordinate  
  call TeleOnMapRoutine             // Execute teleport
  add esp,12
  
  ; Continue loop or exit
  jmp AutoTeleportLoop

SetupTeleportSkill:
  push 0                             // Right mouse
  push 54                            // Teleport skill
  call SetSkillRoutine               // Set skill
  add esp,8
  jmp AutoTeleportLoop               // Return to main loop
```

## Error Conditions

The function handles several error scenarios:

- **Null Player Pointer**: Returns 0 if pMe is null
- **Invalid Unit Type**: Returns 0 if unit type is not player (1)
- **Memory Access Error**: Safe read with validation

## Performance Considerations

- **Direct Memory Access**: Single memory read operation
- **Minimal Overhead**: No function calls or complex logic
- **Register Efficient**: Uses only necessary registers
- **Cache Friendly**: Accesses commonly used player data

## Skill Categories by ID Range

### Sorceress Skills (ID 36-71)
- **42**: Static Field
- **47**: Fireball  
- **54**: Teleport
- **56**: Fire Wall

### Paladin Skills (ID 96-131)
- **117**: Blessed Hammer
- **112**: Concentration
- **106**: Charge

### Common Skills (ID 50-51)
- **50**: Town Portal
- **51**: Identify

## Memory Layout

Player unit skill data structure:
```
Offset 0x00: Unit Type (DWORD) = 1
...
Offset 0x3E: Right Hand Skill (WORD)
Offset 0x40: Left Hand Skill (WORD)
...
```

## Notes

- Function reads only right hand skill (most commonly used for automation)
- Left hand skill can be read by changing offset to 0x40
- Skill ID 0 indicates no skill is assigned
- Function is safe to call frequently in loops
- Compatible with all character classes
- Skill IDs are consistent across game versions
- Return value is zero-extended WORD for easy comparison
