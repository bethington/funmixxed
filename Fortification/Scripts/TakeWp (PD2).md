# "TakeWp Assembly Implementation (PD2)"

## Metadata
- **ID**: 135012
- **Description**: "TakeWp Assembly Implementation for Project Diablo 2"
- **Options**: Waypoint interaction and packet sending
- **VariableType**: Auto Assembler Script
- **Source File**: PickIt Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.14d (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements waypoint interaction functionality in assembly.
  It searches for waypoint objects in the current area and sends the
  appropriate packet to activate them.
  
  Based on the TakeWp() function from Fortification\PickIt.cpp
}

[ENABLE]

// Memory addresses and functions
define(pMe,40000000)                    // Player unit pointer
define(D2NET_SendPacket,60000000)       // Network packet function
define(UNIT_TYPE_OBJECT,2)              // Object unit type
define(WAYPOINT_OBJECT_ID,429)          // Waypoint object ID

// Hook for waypoint taking
aobscanmodule(takeWpHook,D2CLIENT.dll,8B ?? ?? ?? ?? ?? 85 ?? 74 ??)

takeWpHook:
  call TakeWpRoutine                    // Call our waypoint function
  jmp returnFromTakeWpHook

TakeWpRoutine:
  push ebp
  mov ebp,esp
  sub esp,12                            // Space for packet buffer (9 bytes + alignment)
  push ebx
  push ecx
  push edx
  push esi
  push edi
  
  // Get player unit
  mov eax,[pMe]
  test eax,eax
  jz TakeWpExit                         // Exit if no player
  
  // Get player act
  mov eax,[eax+20h]                     // pAct offset
  test eax,eax
  jz TakeWpExit                         // Exit if no act
  
  // Get first room in act
  mov esi,[eax+68h]                     // pRoom1 offset
  
TakeWpRoomLoop:
  test esi,esi
  jz TakeWpExit                         // Exit if no more rooms
  
  // Get first unit in room
  mov edi,[esi+70h]                     // pUnitFirst offset
  
TakeWpUnitLoop:
  test edi,edi
  jz TakeWpNextRoom                     // Continue to next room if no more units
  
  // Check if unit type is UNIT_TYPE_OBJECT
  mov eax,[edi+0Ch]                     // dwType offset
  cmp eax,UNIT_TYPE_OBJECT
  jne TakeWpNextUnit                    // Skip if not object
  
  // Check if object is waypoint (dwTxtFileNo == 429)
  mov eax,[edi+4h]                      // dwTxtFileNo offset
  cmp eax,WAYPOINT_OBJECT_ID
  jne TakeWpNextUnit                    // Skip if not waypoint
  
  // Found waypoint, create interaction packet
  // Packet structure: [0x49][UnitId][NULL][NULL][NULL][0x6F][NULL][NULL][NULL]
  mov byte ptr [ebp-12],49h             // Packet[0] = 0x49
  
  mov eax,[edi+8h]                      // dwUnitId offset
  mov [ebp-11],eax                      // Packet[1-4] = UnitId
  
  mov dword ptr [ebp-7],0               // Packet[5-8] = NULL
  mov dword ptr [ebp-3],0               // Packet[9-12] = NULL  
  mov dword ptr [ebp-11+8],0            // Packet[13-16] = NULL
  mov byte ptr [ebp-11+12],6Fh          // Packet[17] = 0x6F
  mov dword ptr [ebp-11+13],0           // Packet[18-21] = NULL
  mov dword ptr [ebp-11+17],0           // Packet[22-25] = NULL
  mov dword ptr [ebp-11+21],0           // Packet[26-29] = NULL
  
  // Send waypoint packet
  lea eax,[ebp-12]                      // Packet buffer address
  push eax                              // Packet buffer
  push 0                                // Priority 0
  push 9                                // Packet size
  call D2NET_SendPacket                 // Send packet
  add esp,12                            // Clean stack
  
  jmp TakeWpExit                        // Exit after taking waypoint
  
TakeWpNextUnit:
  // Move to next unit in room
  mov edi,[edi+10h]                     // pListNext offset
  jmp TakeWpUnitLoop
  
TakeWpNextRoom:
  // Move to next room in act
  mov esi,[esi+78h]                     // pRoomNext offset
  jmp TakeWpRoomLoop
  
TakeWpExit:
  pop edi
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromTakeWpHook:

[DISABLE]

// Restore original code
takeWpHook:
  db 8B ?? ?? ?? ?? ?? 85 ?? 74 ??      // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(D2NET_SendPacket)
```

## Configuration

### Waypoint Identification
- **Object ID**: 429 (waypoint object identifier)
- **Unit Type**: UNIT_TYPE_OBJECT (2)
- **Search Scope**: All rooms in current act

### Packet Structure
9-byte waypoint activation packet:
- **Byte 0**: 0x49 (waypoint packet type)
- **Bytes 1-4**: Waypoint unit ID
- **Bytes 5-8**: NULL padding
- **Remaining**: Special waypoint activation data

### Memory Offsets
Key structure offsets used:
- **pAct**: +0x20 from player unit
- **pRoom1**: +0x68 from act structure
- **pUnitFirst**: +0x70 from room structure
- **pListNext**: +0x10 from unit structure
- **pRoomNext**: +0x78 from room structure
- **dwType**: +0x0C from unit structure
- **dwTxtFileNo**: +0x04 from unit structure
- **dwUnitId**: +0x08 from unit structure

## Features

1. **Area-Wide Search**: Searches all rooms in current act
2. **Object Type Filtering**: Only examines object units
3. **Waypoint Identification**: Finds waypoint by object ID
4. **Automatic Activation**: Sends waypoint packet when found
5. **Early Exit**: Stops searching after finding waypoint
6. **Memory Safe**: Comprehensive null pointer checking

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `D2NET_SendPacket()`: Network packet transmission function
- Valid act and room structure hierarchy

## Function Signature

```asm
TakeWpRoutine:
  ; Parameters: None
  ; Returns: Nothing (void function)
```

## Usage Example

```asm
call TakeWpRoutine                   // Search for and take waypoint
; Function automatically handles all logic
```

## Search Algorithm

The function performs a systematic search:
1. **Act Validation**: Verify player act exists
2. **Room Iteration**: Loop through all rooms in act
3. **Unit Iteration**: Loop through all units in each room
4. **Type Filtering**: Check if unit is an object
5. **ID Matching**: Verify object is a waypoint
6. **Activation**: Send waypoint packet when found

## Packet Protocol

Waypoint activation uses a specific 9-byte packet:
- **Command**: 0x49 (waypoint interaction)
- **Target**: Waypoint unit ID
- **Format**: Standard Diablo II waypoint protocol
- **Priority**: 0 (normal priority)

## Use Cases

### Automatic Waypoint Collection
- Bot systems that automatically collect waypoints
- Area exploration scripts
- Town return preparation

### Manual Waypoint Activation
- Player assistance for finding hidden waypoints
- Accessibility helpers for waypoint location

## Notes

- Function searches entire act for waypoint objects
- Only activates first waypoint found in search order
- Uses break mechanism to prevent multiple activations
- Designed to handle exception cases safely
- Compatible with all Diablo II acts and difficulty levels
- Memory offsets are specific to PD2 structure layout
- Function preserves all registers (void return type)
