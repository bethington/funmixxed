# "CastOnMap Assembly Implementation (PD2)"

## Metadata
- **ID**: 135010
- **Description**: "CastOnMap Assembly Implementation for Project Diablo 2"
- **Options**: Generic spell casting on map coordinates with cooldown
- **VariableType**: Auto Assembler Script
- **Source File**: SendPacket Assembly

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.13c (Project Diablo 2)
  Date   : 2025-07-25
  Author : Fortification Team

  Description:
  This script implements generic spell casting on map coordinates in assembly.
  It creates and sends network packets for casting spells at specific map
  locations with cooldown protection.
  
  Based on the CastOnMap() function from Fortification\SendPacket.cpp
}

[ENABLE]

// Memory addresses and functions from D2Ptrs.h
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2NET_SendPacket,-10024)         // FUNCPTR(D2NET, SendPacket, void __stdcall, (DWORD aLen, DWORD arg1, BYTE* aPacket), -10024)
define(GetTickCount,kernel32.GetTickCount) // Timer function
define(v_LastPrecast,50000000)          // Last precast timestamp
define(COOLDOWN_TIME,500)               // 500ms cooldown

// Hook for cast on map
aobscanmodule(castOnMapHook,D2CLIENT.dll,6A ?? 6A ?? 68 ?? ?? ?? ??)

castOnMapHook:
  call CastOnMapRoutine                 // Call our cast function
  test al,al                            // Test return value (bool)
  jmp returnFromCastOnMapHook

CastOnMapRoutine:
  // Parameters: x (WORD), y (WORD), Left (bool)
  push ebp
  mov ebp,esp
  sub esp,8                             // Space for packet buffer
  push ebx
  push ecx
  push edx
  push esi
  
  mov bx,[ebp+8]                        // x coordinate
  mov cx,[ebp+12]                       // y coordinate
  mov dl,[ebp+16]                       // Left flag
  
  // Check if game is ready (using D2CLIENT_GetPlayerUnit)
  call D2CLIENT_GetPlayerUnit
  test eax,eax
  jz CastOnMapFailed                    // Return 0 if game not ready
  
  // Check cooldown timer (GetTickCount() - v_LastPrecast >= 500)
  call GetTickCount                     // Get current time
  mov esi,[v_LastPrecast]               // Last precast time
  sub eax,esi                           // Current time - last precast time
  cmp eax,COOLDOWN_TIME                 // Compare with 500ms cooldown
  jl CastOnMapFailed                    // Return 0 if still in cooldown
  
  // Create packet buffer (5 bytes)
  // aPacket[0] = Left ? 0x08 : 0x0F
  test dl,dl                            // Check Left flag
  jz SetRightClick                      // Jump if not left click
  mov byte ptr [ebp-8],08h              // Set left click packet type
  jmp SetCoordinates
  
SetRightClick:
  mov byte ptr [ebp-8],0Fh              // Set right click packet type
  
SetCoordinates:
  // aPacket[1-2] = x coordinate (WORD)
  mov [ebp-7],bx                        // Set x coordinate
  
  // aPacket[3-4] = y coordinate (WORD)  
  mov [ebp-5],cx                        // Set y coordinate
  
  // Send packet: D2NET_SendPacket(5, 0, aPacket)
  lea eax,[ebp-8]                       // Packet buffer address
  push eax                              // Packet buffer
  push 0                                // Priority 0
  push 5                                // Packet size
  call D2NET_SendPacket                 // Send the packet
  add esp,12                            // Clean stack
  
  // Update last precast timestamp
  call GetTickCount                     // Get current time
  mov [v_LastPrecast],eax               // Store current tick count
  
  mov eax,1                             // Return 1 (success)
  jmp CastOnMapExit
  
CastOnMapFailed:
  xor eax,eax                           // Return 0 (failure)
  
CastOnMapExit:
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromCastOnMapHook:

[DISABLE]

// Restore original code
castOnMapHook:
  db 6A ?? 6A ?? 68 ?? ?? ?? ??         // Original bytes

// Clean up
unregistersymbol(GameReady)
unregistersymbol(D2NET_SendPacket)
unregistersymbol(v_LastPrecast)
```

## Configuration

### Packet Types
Cast packet types based on mouse button:
- **0x08**: Left mouse button casting
- **0x0F**: Right mouse button casting

### Cooldown Timer
- **Duration**: 500ms minimum between casts
- **Purpose**: Prevents packet spam and server overload
- **Variable**: v_LastPrecast timestamp storage

### Packet Structure
5-byte packet format for map casting:
- **Byte 0**: Packet type (0x08 or 0x0F)
- **Bytes 1-2**: X coordinate (WORD)
- **Bytes 3-4**: Y coordinate (WORD)

## Features

1. **Game State Validation**: Only casts when game is ready
2. **Cooldown Protection**: 500ms minimum delay between casts
3. **Mouse Button Support**: Handles both left and right click casting
4. **Coordinate Handling**: Proper WORD coordinate packing
5. **Network Integration**: Uses game's packet sending system
6. **Timer Management**: Automatic cooldown tracking

## Dependencies

This script requires the following:
- `GameReady()`: Game state validation function
- `D2NET_SendPacket()`: Network packet transmission function
- `GetTickCount()`: System timer function
- `v_LastPrecast`: Global timestamp variable

## Function Signature

```asm
CastOnMapRoutine:
  ; Parameters:
  ; [ebp+8]  - x (WORD) - Target X coordinate
  ; [ebp+12] - y (WORD) - Target Y coordinate
  ; [ebp+16] - Left (bool) - Left mouse button flag
  ; Returns: EAX = 1 if successful, 0 if failed
```

## Usage Example

```asm
push 0                               // Right mouse button
push 6789                            // Y coordinate
push 2345                            // X coordinate
call CastOnMapRoutine                // Cast spell
add esp,12                           // Clean stack
test eax,eax                         // Check if successful
jz CastFailed                        // Jump if failed
```

## Network Protocol

The function sends packets using Diablo II's network protocol:
- **Packet Size**: 5 bytes total
- **Priority**: 0 (normal priority)
- **Format**: [Type][X-low][X-high][Y-low][Y-high]

## Algorithm Flow

1. **Validation**: Check game state and cooldown timer
2. **Packet Building**: Create 5-byte packet with coordinates
3. **Type Selection**: Choose packet type based on mouse button
4. **Network Send**: Transmit packet via D2NET_SendPacket
5. **Timer Update**: Update cooldown timestamp
6. **Return Status**: Report success or failure

## Notes

- Function enforces 500ms cooldown to prevent server flooding
- Coordinates are stored in little-endian format (Intel x86)
- Packet types 0x08 and 0x0F are standard Diablo II casting opcodes
- Function preserves all registers except EAX (return value)
- Designed for integration with teleport and other spell casting systems
- Compatible with both single-player and multiplayer modes
