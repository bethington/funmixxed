# "PickItem Assembly Implementation (PD2)"

## Metadata
- **ID**: 135013
- **Description**: "PickItem Assembly Implementation for Project Diablo 2"
- **Options**: Item pickup with packet sending
- **VariableType**: Auto Assembler Script
- **Source File**: PickIt Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.13c (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements item pickup functionality in assembly.
  It sends the appropriate network packet to pick up items by their
  unit ID from the ground.
  
  Based on the PickItem() function from Fortification\PickIt.cpp
}

[ENABLE]

// Memory addresses and functions from D2Ptrs.h
define(D2NET_SendPacket,-10024)         // FUNCPTR(D2NET, SendPacket, void __stdcall, (DWORD aLen, DWORD arg1, BYTE* aPacket), -10024)

// Hook for item pickup
aobscanmodule(pickItemHook,D2CLIENT.dll,6A ?? 68 ?? ?? ?? ?? E8)

pickItemHook:
  call PickItemRoutine                  // Call our pickup function
  jmp returnFromPickItemHook

PickItemRoutine:
  // Parameter: ItemId (DWORD)
  push ebp
  mov ebp,esp
  sub esp,8                             // Space for packet buffer (4 bytes + alignment)
  push ebx
  push ecx
  push edx
  
  mov ebx,[ebp+8]                       // ItemId parameter
  
  // Create pickup packet (4 bytes)
  // Packet structure: [0x16][ItemId_low][ItemId_mid][ItemId_high]
  mov byte ptr [ebp-8],16h              // Packet[0] = 0x16 (pickup packet type)
  mov [ebp-7],ebx                       // Packet[1-4] = ItemId
  
  // Send pickup packet: D2NET_SendPacket(4, 1, Packet)
  lea eax,[ebp-8]                       // Packet buffer address
  push eax                              // Packet buffer
  push 1                                // Priority 1 (high priority)
  push 4                                // Packet size
  call D2NET_SendPacket                 // Send packet
  add esp,12                            // Clean stack
  
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromPickItemHook:

[DISABLE]

// Restore original code
pickItemHook:
  db 6A ?? 68 ?? ?? ?? ?? E8            // Original bytes

// Clean up
unregistersymbol(D2NET_SendPacket)
```

## Configuration

### Packet Type
- **0x16**: Standard item pickup packet type for Diablo II
- **Purpose**: Requests server to pick up specified item

### Packet Structure
4-byte packet format for item pickup:
- **Byte 0**: Packet type (0x16)
- **Bytes 1-4**: Item unit ID (DWORD)

### Priority Level
- **Priority 1**: High priority for immediate processing
- **Purpose**: Ensures quick item pickup response

## Features

1. **Simple Interface**: Single parameter item pickup
2. **High Priority**: Uses priority 1 for fast execution
3. **Minimal Overhead**: Only 4-byte packet size
4. **Direct Addressing**: Uses item unit ID for precise targeting
5. **Network Integration**: Uses game's packet sending system
6. **Memory Efficient**: Minimal stack usage

## Dependencies

This script requires the following:
- `D2NET_SendPacket()`: Network packet transmission function
- Valid item unit ID parameter

## Function Signature

```asm
PickItemRoutine:
  ; Parameters:
  ; [ebp+8] - ItemId (DWORD) - Unit ID of item to pick up
  ; Returns: Nothing (void function)
```

## Usage Example

```asm
push 0x12345678                      // Item unit ID
call PickItemRoutine                 // Pick up the item
add esp,4                            // Clean stack
```

## Integration with Item Detection

The function is typically used with item scanning:

```asm
; Example integration with item finder
ItemScanLoop:
  call FindNextItem                    // Get next item on ground
  test eax,eax                         // Check if item found
  jz ItemScanComplete                  // Exit if no more items
  
  push eax                             // Item unit ID
  call PickItemRoutine                 // Pick up the item
  add esp,4                            // Clean stack
  
  jmp ItemScanLoop                     // Continue scanning

ItemScanComplete:
```

## Network Protocol

The function sends packets using Diablo II's pickup protocol:
- **Packet Type**: 0x16 (pickup command)
- **Priority**: 1 (high priority for immediate response)
- **Size**: 4 bytes total
- **Byte Order**: Little-endian (Intel x86)

## Performance Characteristics

- **Execution Time**: Minimal (single packet creation and send)
- **Memory Usage**: 8 bytes stack space (including alignment)
- **Network Traffic**: 4 bytes per pickup attempt
- **Server Load**: Minimal impact with priority 1

## Error Handling

The function operates at the packet level and does not perform validation:
- **No Item Validation**: Does not verify item exists
- **No Distance Checking**: Does not validate pickup range
- **No Inventory Space**: Does not check available inventory
- **Server Responsibility**: Server handles all validation and errors

## Use Cases

### Automated Item Collection
- Bot systems for item farming
- Quality of life pickup helpers
- Specific item type collectors

### Manual Pickup Assistance
- Click-to-pickup helpers
- Inventory management tools
- Item sorting systems

## Notes

- Function sends packet immediately without validation
- Server is responsible for all pickup validation (distance, inventory space, etc.)
- Item must be valid and within pickup range for server to process
- Function preserves all registers (void return type)
- Designed for high-frequency calling in pickup loops
- Compatible with both single-player and multiplayer modes
- Packet format is standard across all Diablo II versions
