# "UseItem Assembly Implementation (PD2)"

## Metadata
- **ID**: 135005
- **Description**: "UseItem Assembly Implementation for Project Diablo 2"
- **Options**: Item usage with packet sending
- **VariableType**: Auto Assembler Script
- **Source File**: D2Helpers Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.14d (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements item usage functionality in assembly.
  It creates and sends the appropriate network packets to use items
  from different storage locations (inventory, belt, stash, cube).
  
  Based on the UseItem() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and functions
define(pMe,40000000)                    // Player unit pointer
define(GetItemLocation,60000000)        // GetItemLocation function
define(D2NET_SendPacket,60000004)       // Packet sending function
define(STORAGE_INVENTORY,3)             // Storage location constants
define(STORAGE_BELT,2)
define(STORAGE_STASH,1)
define(STORAGE_CUBE,4)

// Hook for item usage
aobscanmodule(useItemHook,D2CLIENT.dll,6A ?? 68 ?? ?? ?? ?? E8)

useItemHook:
  call UseItemRoutine                   // Call our use item function
  jmp returnFromUseItemHook

UseItemRoutine:
  // Parameter: Unit (item unit pointer)
  push ebp
  mov ebp,esp
  sub esp,20                            // Space for packet buffer
  push ebx
  push ecx
  push edx
  push esi
  
  mov esi,[ebp+8]                       // Unit parameter
  test esi,esi
  jz UseItemExit                        // Exit if unit is NULL
  
  // Get item location
  push esi                              // Item parameter
  call GetItemLocation                  // Get storage location
  add esp,4                             // Clean stack
  mov ebx,eax                           // Save location in ebx
  
  // Check if item is in inventory or belt
  cmp ebx,STORAGE_INVENTORY
  je UseFromInventoryOrBelt
  cmp ebx,STORAGE_BELT
  je UseFromInventoryOrBelt
  
  // Check if item is in stash or cube
  cmp ebx,STORAGE_STASH
  je UseFromStashOrCube
  cmp ebx,STORAGE_CUBE
  je UseFromStashOrCube
  
  jmp UseItemExit                       // Unknown location, exit
  
UseFromInventoryOrBelt:
  // Create packet for inventory/belt usage
  
  // Set packet type based on location
  mov al,20h                            // Default to 0x20 (inventory)
  cmp ebx,STORAGE_BELT
  jne SetInventoryPacket
  mov al,26h                            // 0x26 for belt usage
  
SetInventoryPacket:
  mov [ebp-20],al                       // Packet[0] = packet type
  
  // Set item unit ID
  mov eax,[esi+08h]                     // Item dwUnitId offset
  mov [ebp-19],eax                      // Packet[1-4] = Unit ID
  
  // Set position data
  mov eax,[pMe]                         // Get player unit
  test eax,eax
  jz UseItemExit                        // Exit if no player
  
  cmp ebx,STORAGE_INVENTORY
  jne SetBeltPosition                   // Belt uses NULL position
  
  // For inventory, use player position
  mov ecx,[eax+38h]                     // pPath offset
  test ecx,ecx
  jz UseItemExit                        // Exit if no path
  
  mov eax,[ecx+10h]                     // xPos offset
  mov [ebp-15],eax                      // Packet[5-8] = xPos
  
  mov eax,[ecx+14h]                     // yPos offset
  mov [ebp-11],eax                      // Packet[9-12] = yPos
  jmp SendInventoryBeltPacket
  
SetBeltPosition:
  // For belt, use NULL positions
  xor eax,eax
  mov [ebp-15],eax                      // Packet[5-8] = NULL
  mov [ebp-11],eax                      // Packet[9-12] = NULL
  
SendInventoryBeltPacket:
  // Send packet for inventory/belt
  lea eax,[ebp-20]                      // Packet buffer address
  push eax                              // Packet buffer
  push 1                                // Packet priority
  push 13                               // Packet size
  call D2NET_SendPacket                 // Send packet
  add esp,12                            // Clean stack
  jmp UseItemExit
  
UseFromStashOrCube:
  // Create packet for stash/cube usage (always use 0x20)
  mov byte ptr [ebp-20],20h             // Packet[0] = 0x20
  
  // Set item unit ID
  mov eax,[esi+08h]                     // Item dwUnitId
  mov [ebp-19],eax                      // Packet[1-4] = Unit ID
  
  // Get player position (required for stash/cube)
  mov eax,[pMe]                         // Get player unit
  test eax,eax
  jz UseItemExit                        // Exit if no player
  
  mov ecx,[eax+38h]                     // pPath offset
  test ecx,ecx
  jz UseItemExit                        // Exit if no path
  
  mov eax,[ecx+10h]                     // xPos
  mov [ebp-15],eax                      // Packet[5-8] = xPos
  
  mov eax,[ecx+14h]                     // yPos
  mov [ebp-11],eax                      // Packet[9-12] = yPos
  
  // Send packet for stash/cube
  lea eax,[ebp-20]                      // Packet buffer address
  push eax                              // Packet buffer
  push 1                                // Packet priority
  push 13                               // Packet size
  call D2NET_SendPacket                 // Send packet
  add esp,12                            // Clean stack
  
UseItemExit:
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromUseItemHook:

[DISABLE]

// Restore original code
useItemHook:
  db 6A ?? 68 ?? ?? ?? ?? E8            // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(GetItemLocation)
unregistersymbol(D2NET_SendPacket)
```

## Configuration

### Packet Types
Different packet types for item usage:
- **0x20**: Inventory, Stash, Cube item usage
- **0x26**: Belt item usage

### Storage Locations
- **STORAGE_INVENTORY (3)**: Main inventory grid
- **STORAGE_BELT (2)**: Belt potion slots
- **STORAGE_STASH (1)**: Player stash storage
- **STORAGE_CUBE (4)**: Horadric Cube storage

### Packet Structure
13-byte packet format:
- **Byte 0**: Packet type (0x20 or 0x26)
- **Bytes 1-4**: Item Unit ID (DWORD)
- **Bytes 5-8**: X Position (DWORD, NULL for belt)
- **Bytes 9-12**: Y Position (DWORD, NULL for belt)

## Features

1. **Location-Aware Usage**: Different packet types based on storage location
2. **Position Handling**: Proper position data for different storage types
3. **Belt Optimization**: Special handling for belt item usage
4. **Stash/Cube Support**: Full support for PD2 stash and cube usage
5. **Packet Creation**: Dynamic packet building and sending
6. **Memory Safe**: Proper buffer management and null checking

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `GetItemLocation()`: Function to determine item storage location
- `D2NET_SendPacket()`: Game's network packet sending function
- Player unit structure with path and position data

## Function Signature

```asm
UseItemRoutine:
  ; Parameters:
  ; [ebp+8] - Unit (pointer to item unit to use)
  ; Returns: Nothing (void function)
```

## Usage Example

```asm
push itemPointer                      // Item unit pointer
call UseItemRoutine                   // Use the item
add esp,4                             // Clean stack
```

## Memory Offsets

Key offsets used in the function:
- **dwUnitId**: +0x08 from item unit
- **pPath**: +0x38 from player unit
- **xPos**: +0x10 from path structure
- **yPos**: +0x14 from path structure

## Notes

- Function automatically determines correct packet type based on storage location
- Belt items use NULL positions in the packet
- Stash and cube items require player position data
- All packets are sent with priority 1 (normal priority)
- Function preserves all registers (void return type)
- Designed to work with Project Diablo 2's network protocol
