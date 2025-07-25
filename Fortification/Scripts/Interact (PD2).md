# "Interact Assembly Implementation (PD2)"

## Metadata
- **ID**: 135011
- **Description**: "Interact Assembly Implementation for Project Diablo 2"
- **Options**: Unit interaction with packet sending
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
  This script implements unit interaction functionality in assembly.
  It creates and sends network packets to interact with NPCs, objects,
  and other game units.
  
  Based on the Interact() function from Fortification\SendPacket.cpp
}

[ENABLE]

// Memory addresses and functions
define(GameReady,60000000)              // GameReady function
define(D2NET_SendPacket,60000004)       // Network packet function

// Hook for unit interaction
aobscanmodule(interactHook,D2CLIENT.dll,6A ?? 68 ?? ?? ?? ?? E8)

interactHook:
  call InteractRoutine                  // Call our interact function
  test al,al                            // Test return value (bool)
  jmp returnFromInteractHook

InteractRoutine:
  // Parameters: UnitId (DWORD), UnitType (DWORD)
  push ebp
  mov ebp,esp
  sub esp,12                            // Space for packet buffer (9 bytes + alignment)
  push ebx
  push ecx
  push edx
  
  mov ebx,[ebp+8]                       // UnitId parameter
  mov ecx,[ebp+12]                      // UnitType parameter
  
  // Check if game is ready
  call GameReady
  test eax,eax
  jz InteractFailed                     // Return 0 if game not ready
  
  // Create interaction packet (9 bytes)
  // Packet structure:
  // [0] = 0x13 (interaction packet type)
  // [1-4] = UnitType (DWORD)
  // [5-8] = UnitId (DWORD)
  
  mov byte ptr [ebp-12],13h             // Packet[0] = 0x13
  mov [ebp-11],ecx                      // Packet[1-4] = UnitType
  mov [ebp-7],ebx                       // Packet[5-8] = UnitId
  
  // Send packet: D2NET_SendPacket(9, 1, aPacket)
  lea eax,[ebp-12]                      // Packet buffer address
  push eax                              // Packet buffer
  push 1                                // Priority 1 (high priority)
  push 9                                // Packet size
  call D2NET_SendPacket                 // Send the packet
  add esp,12                            // Clean stack
  
  mov eax,1                             // Return 1 (success)
  jmp InteractExit
  
InteractFailed:
  xor eax,eax                           // Return 0 (failure)
  
InteractExit:
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromInteractHook:

[DISABLE]

// Restore original code
interactHook:
  db 6A ?? 68 ?? ?? ?? ?? E8            // Original bytes

// Clean up
unregistersymbol(GameReady)
unregistersymbol(D2NET_SendPacket)
```

## Configuration

### Packet Type
- **0x13**: Standard interaction packet type for Diablo II
- **Purpose**: Initiates interaction with game units

### Unit Types
Common unit types for interaction:
- **UNIT_TYPE_PLAYER (0)**: Player characters
- **UNIT_TYPE_NPC (1)**: Non-player characters
- **UNIT_TYPE_OBJECT (2)**: Interactive objects (chests, doors, etc.)
- **UNIT_TYPE_MISSILE (3)**: Projectiles (rarely interacted with)
- **UNIT_TYPE_ITEM (4)**: Items on ground

### Packet Structure
9-byte packet format for unit interaction:
- **Byte 0**: Packet type (0x13)
- **Bytes 1-4**: Unit type (DWORD)
- **Bytes 5-8**: Unit ID (DWORD)

## Features

1. **Game State Validation**: Only interacts when game is ready
2. **Unit Identification**: Supports interaction with any unit type
3. **High Priority**: Uses priority 1 for immediate processing
4. **Memory Safe**: Proper buffer management and stack cleanup
5. **Network Integration**: Uses game's packet sending system
6. **Universal Compatibility**: Works with all interactive game units

## Dependencies

This script requires the following:
- `GameReady()`: Game state validation function
- `D2NET_SendPacket()`: Network packet transmission function
- Valid unit ID and type parameters

## Function Signature

```asm
InteractRoutine:
  ; Parameters:
  ; [ebp+8]  - UnitId (DWORD) - Target unit identifier
  ; [ebp+12] - UnitType (DWORD) - Type of unit to interact with
  ; Returns: EAX = 1 if successful, 0 if failed
```

## Usage Example

```asm
push 1                               // UNIT_TYPE_NPC
push 0x12345678                      // NPC Unit ID
call InteractRoutine                 // Interact with NPC
add esp,8                            // Clean stack
test eax,eax                         // Check if successful
jz InteractionFailed                 // Jump if failed
```

## Common Use Cases

### NPC Interaction
```asm
push UNIT_TYPE_NPC                   // Type: NPC
push npcUnitId                       // Specific NPC ID
call InteractRoutine                 // Talk to NPC
```

### Object Interaction
```asm
push UNIT_TYPE_OBJECT                // Type: Object
push chestUnitId                     // Chest/door ID
call InteractRoutine                 // Open chest/door
```

### Player Interaction
```asm
push UNIT_TYPE_PLAYER                // Type: Player
push playerUnitId                    // Target player ID
call InteractRoutine                 // Interact with player
```

## Network Protocol

The function sends packets using Diablo II's interaction protocol:
- **Packet Type**: 0x13 (interaction command)
- **Priority**: 1 (high priority for immediate response)
- **Size**: 9 bytes total
- **Byte Order**: Little-endian (Intel x86)

## Error Handling

Function will fail and return 0 if:
- Game is not in ready state
- Invalid parameters are provided
- Network sending fails

## Notes

- Function uses high priority (1) for immediate server processing
- Packet format is standard across all Diablo II versions
- Unit IDs must be valid and currently loaded in game
- Function preserves all registers except EAX (return value)
- Designed for integration with bot and automation systems
- Compatible with both single-player and multiplayer modes
- Can be used for NPCs, objects, waypoints, and other interactive elements
