# "ManageTele Assembly Implementation (PD2)"

## Metadata

- **ID**: 135017
- **Description**: "ManageTele Assembly Implementation for Project Diablo 2"
- **Options**: Main teleportation management and execution
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
  This script implements the main teleportation management system in assembly.
  It coordinates skill checking, path execution, mana management, and timing
  for safe automated teleportation sequences.
  
  Based on the ManageTele() function from Fortification\AutoTele.cpp
}

[ENABLE]

// Game state and management variables
define(TELEPORT_SKILL_ID,54)            // Teleport skill identifier
define(TELEPORT_COOLDOWN,300)           // Cooldown between teleports (ms)
define(MIN_MANA_PERCENT,30)             // Minimum mana required (30%)

// Teleportation state tracking
teleState:
  dd 0                                  // Last teleport timestamp
  dd 0                                  // Teleportation active flag
  dd 0                                  // Current target X
  dd 0                                  // Current target Y
  dd 0                                  // Mana check required flag

// Hook for teleportation management
aobscanmodule(manageTeleHook,D2CLIENT.dll,55 8B EC 53 56 57 83 EC)

manageTeleHook:
  call ManageTeleRoutine                // Call our teleportation manager
  jmp returnFromManageTeleHook

ManageTeleRoutine:
  // Parameters: targetX (DWORD), targetY (DWORD)
  // Returns: EAX = 1 if teleportation initiated, 0 if blocked
  push ebp
  mov ebp,esp
  sub esp,16                            // Local variables
  push ebx
  push ecx
  push edx
  push esi
  push edi
  
  mov eax,[ebp+8]                       // targetX parameter
  mov ebx,[ebp+12]                      // targetY parameter
  
  // Store target coordinates
  mov [teleState+8],eax                 // Store target X
  mov [teleState+12],ebx                // Store target Y
  
  // Check if game is ready for teleportation
  call GameReady                        // Validate game state
  test eax,eax                          // Check result
  jz TeleportBlocked                    // Exit if game not ready
  
  // Check cooldown timer
  call GetTickCount                     // Get current time
  mov ecx,[teleState]                   // Last teleport time
  sub eax,ecx                           // Time difference
  cmp eax,TELEPORT_COOLDOWN             // Compare with cooldown
  jl TeleportCooldown                   // Exit if still cooling down
  
  // Verify teleport skill is active
  call GetSkillRoutine                  // Get current right hand skill
  cmp eax,TELEPORT_SKILL_ID             // Check if teleport
  je SkillReady                         // Jump if teleport active
  
  // Set teleport skill
  push 0                                // Right mouse button
  push TELEPORT_SKILL_ID                // Teleport skill ID
  call SetSkillRoutine                  // Set teleport skill
  add esp,8                             // Clean stack
  
  // Brief delay for skill change
  push 100                              // 100ms delay
  call Sleep
  add esp,4
  
SkillReady:
  // Check mana requirements
  call CheckManaForTeleport             // Verify sufficient mana
  test eax,eax                          // Check result
  jz InsufficientMana                   // Handle low mana
  
  // Create teleportation path
  mov eax,[teleState+8]                 // Target X
  mov ebx,[teleState+12]                // Target Y
  push ebx                              // Target Y parameter
  push eax                              // Target X parameter
  call MakePathRoutine                  // Create path to target
  add esp,8                             // Clean stack
  
  test eax,eax                          // Check if path created
  jz PathCreationFailed                 // Handle pathfinding failure
  
  // Execute first teleport step
  call ExecuteTeleportStep              // Execute next path step
  test eax,eax                          // Check execution result
  jz TeleportExecutionFailed            // Handle execution failure
  
  // Update teleportation state
  call GetTickCount                     // Get current time
  mov [teleState],eax                   // Update last teleport time
  mov dword ptr [teleState+4],1         // Set active flag
  
  mov eax,1                             // Return success
  jmp TeleportComplete
  
TeleportBlocked:
  // Game not ready for teleportation
  xor eax,eax                           // Return failure
  jmp TeleportComplete
  
TeleportCooldown:
  // Still in cooldown period
  xor eax,eax                           // Return failure
  jmp TeleportComplete
  
InsufficientMana:
  // Not enough mana for teleportation
  call AutoManaRoutine                  // Try to use mana potion
  xor eax,eax                           // Return failure (try again next cycle)
  jmp TeleportComplete
  
PathCreationFailed:
  // Could not create path to target
  mov dword ptr [teleState+4],0         // Clear active flag
  xor eax,eax                           // Return failure
  jmp TeleportComplete
  
TeleportExecutionFailed:
  // Teleport execution failed
  mov dword ptr [teleState+4],0         // Clear active flag
  xor eax,eax                           // Return failure
  jmp TeleportComplete
  
TeleportComplete:
  pop edi
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

CheckManaForTeleport:
  // Check if player has sufficient mana for teleportation
  // Returns: EAX = 1 if sufficient, 0 if insufficient
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  
  call D2CLIENT_GetPlayerUnit           // Get player unit
  test eax,eax                          // Check if valid
  jz InsufficientManaCheck
  
  mov ebx,eax                           // Player unit in EBX
  
  // Get current mana
  push 1                                // Mana stat ID
  push ebx                              // Player unit
  call GetUnitStat                      // Get current mana
  add esp,8                             // Clean stack
  mov ecx,eax                           // Current mana in ECX
  
  // Get maximum mana
  push 2                                // Max mana stat ID
  push ebx                              // Player unit
  call GetUnitStat                      // Get maximum mana
  add esp,8                             // Clean stack
  
  // Calculate mana percentage
  test eax,eax                          // Check max mana not zero
  jz InsufficientManaCheck              // Avoid division by zero
  
  imul ecx,100                          // current * 100
  div ecx                               // (current * 100) / max = percentage
  
  cmp eax,MIN_MANA_PERCENT              // Compare with minimum required
  jge SufficientManaCheck               // Jump if sufficient
  
InsufficientManaCheck:
  xor eax,eax                           // Return insufficient
  jmp ManaCheckComplete
  
SufficientManaCheck:
  mov eax,1                             // Return sufficient
  
ManaCheckComplete:
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

ExecuteTeleportStep:
  // Execute the next step in the teleportation path
  // Returns: EAX = 1 if successful, 0 if failed
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  
  // Check if path is available
  mov eax,[pathState+16]                // Total steps
  test eax,eax                          // Check if path exists
  jz NoPathAvailable                    // Exit if no path
  
  mov ebx,[pathState+20]                // Current step index
  cmp ebx,eax                           // Compare with total steps
  jge PathComplete                      // Exit if path complete
  
  // Get current waypoint coordinates
  mov ecx,ebx                           // Step index
  shl ecx,2                             // Multiply by 4 (DWORD)
  add ecx,pathState+24                  // Point to waypoint
  mov eax,[ecx]                         // Waypoint X
  mov ebx,[ecx+2]                       // Waypoint Y
  
  // Execute teleport to waypoint
  push 0                                // Right mouse button
  push ebx                              // Target Y
  push eax                              // Target X
  call TeleOnMapRoutine                 // Execute teleport
  add esp,12                            // Clean stack
  
  // Advance to next step
  inc dword ptr [pathState+20]          // Increment step index
  
  mov eax,1                             // Return success
  jmp TeleportStepComplete
  
NoPathAvailable:
  xor eax,eax                           // Return failure
  jmp TeleportStepComplete
  
PathComplete:
  // Path execution complete
  mov dword ptr [teleState+4],0         // Clear active teleportation flag
  mov eax,1                             // Return success
  
TeleportStepComplete:
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromManageTeleHook:

[DISABLE]

// Restore original code
manageTeleHook:
  db 55 8B EC 53 56 57 83 EC           // Original bytes

// Clean up
unregistersymbol(teleState)
```

## Configuration

### Teleportation Parameters

- **TELEPORT_SKILL_ID**: Skill ID for teleport (54)
- **TELEPORT_COOLDOWN**: Minimum time between teleports (300ms)
- **MIN_MANA_PERCENT**: Minimum mana percentage required (30%)

### State Management

The function maintains teleportation state:

- **Last Timestamp**: Tracks cooldown timing
- **Active Flag**: Indicates ongoing teleportation sequence
- **Target Coordinates**: Current destination
- **Mana Check**: Tracks mana requirements

## Features

1. **Comprehensive Management**: Handles all aspects of teleportation
2. **Cooldown Protection**: Prevents excessive teleportation
3. **Skill Validation**: Ensures teleport skill is active
4. **Mana Management**: Checks and manages mana requirements
5. **Path Integration**: Works with pathfinding system
6. **Error Handling**: Graceful failure recovery
7. **State Tracking**: Maintains teleportation progress

## Dependencies

This script requires the following:

- `GameReady()`: Game state validation
- `GetSkillRoutine()`: Current skill checking
- `SetSkillRoutine()`: Skill assignment
- `MakePathRoutine()`: Pathfinding system
- `TeleOnMapRoutine()`: Teleport execution
- `AutoManaRoutine()`: Mana potion usage
- `GetUnitStat()`: Player statistics
- `D2CLIENT_GetPlayerUnit()`: Player unit access

## Function Signature

```asm
ManageTeleRoutine:
  ; Parameters:
  ; [ebp+8]  - targetX (DWORD) - Destination X coordinate
  ; [ebp+12] - targetY (DWORD) - Destination Y coordinate
  ; Returns: EAX = 1 if teleportation initiated, 0 if blocked/failed
```

## Usage Example

```asm
; Basic teleportation to coordinates
push 150                             // Target Y
push 100                             // Target X
call ManageTeleRoutine               // Manage teleportation
add esp,8                            // Clean stack

test eax,eax                         // Check result
jz TeleportationFailed               // Handle failure

; Teleportation initiated successfully
TeleportationSuccessful:
  ; Continue with other actions
  ret

TeleportationFailed:
  ; Handle failure (cooldown, mana, etc.)
  ret
```

## Automation Integration

Typical usage in automated bot loops:

```asm
AutoBotLoop:
  ; Check if teleportation is needed
  call DetermineNextTarget             // Get next destination
  test eax,eax                         // Check if target available
  jz NoTeleportNeeded                  // Skip if no target
  
  ; Attempt teleportation
  push ebx                             // Target Y from DetermineNextTarget
  push eax                             // Target X from DetermineNextTarget
  call ManageTeleRoutine               // Execute teleportation
  add esp,8
  
  test eax,eax                         // Check success
  jz TeleportationBlocked              // Handle blockage
  
  ; Wait for teleportation completion
  push 500                             // 500ms delay
  call Sleep
  add esp,4
  
  jmp AutoBotLoop                      // Continue loop

NoTeleportNeeded:
  ; Perform other bot actions
  call PerformOtherActions
  jmp AutoBotLoop

TeleportationBlocked:
  ; Handle blockage (wait, retry, etc.)
  push 1000                            // 1 second delay
  call Sleep
  add esp,4
  jmp AutoBotLoop
```

## State Management

The teleportation state structure:

```
teleState structure (20 bytes):
Offset 0x00: Last Teleport Timestamp (DWORD)
Offset 0x04: Active Flag (DWORD) - 1 if teleporting
Offset 0x08: Target X Coordinate (DWORD)
Offset 0x0C: Target Y Coordinate (DWORD)
Offset 0x10: Mana Check Required Flag (DWORD)
```

## Error Recovery

The function handles various failure scenarios:

1. **Game Not Ready**: Waits for game state to stabilize
2. **Cooldown Active**: Respects timing restrictions
3. **Wrong Skill**: Automatically sets teleport skill
4. **Low Mana**: Triggers mana potion usage
5. **Path Failure**: Clears state for retry
6. **Execution Failure**: Resets for next attempt

## Mana Management Integration

```asm
; Mana checking logic
CheckManaForTeleport:
  ; Get current mana percentage
  call CalculateManaPercent
  cmp eax,MIN_MANA_PERCENT             // 30% minimum
  jge ManaOK
  
  ; Try to restore mana
  call AutoManaRoutine                 // Use mana potion
  
  ; Recheck after potion
  call CalculateManaPercent
  cmp eax,MIN_MANA_PERCENT
  jge ManaOK
  
  ; Still insufficient
  xor eax,eax                          // Return failure
  ret

ManaOK:
  mov eax,1                            // Return success
  ret
```

## Performance Considerations

- **Cooldown Management**: Prevents spam and server issues
- **State Caching**: Avoids redundant calculations
- **Early Exits**: Fast failure detection
- **Memory Efficient**: Compact state structure
- **Register Preservation**: Maintains calling convention

## Integration with Other Systems

The function coordinates with multiple subsystems:

- **Pathfinding**: Uses MakePath for route planning
- **Skill Management**: Ensures proper skill selection
- **Mana System**: Manages resource requirements
- **Timing System**: Respects cooldowns and delays
- **Game State**: Validates conditions before action

## Notes

- Function is designed for continuous calling in bot loops
- Handles both single teleports and multi-step paths
- Automatically manages skill switching and mana
- Provides detailed error reporting through return values
- Compatible with all character classes that have teleport
- Respects server timing and anti-cheat measures
- Designed for safe, undetectable automated movement
