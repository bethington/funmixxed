# "SetSkill Assembly Implementation (PD2)"

## Metadata
- **ID**: 135014
- **Description**: "SetSkill Assembly Implementation for Project Diablo 2"
- **Options**: Skill selection with packet sending
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
  This script implements skill selection functionality in assembly.
  It sends network packets to set the active skill for left or right
  mouse button, enabling automated skill management.
  
  Based on the SetSkill() function from Fortification\AutoTele.cpp
}

[ENABLE]

// Memory addresses and functions
define(D2NET_SendPacket,60000000)       // Network packet function

// Hook for skill setting
aobscanmodule(setSkillHook,D2CLIENT.dll,6A ?? 68 ?? ?? ?? ?? E8)

setSkillHook:
  call SetSkillRoutine                  // Call our skill setting function
  test al,al                            // Test return value (bool)
  jmp returnFromSetSkillHook

SetSkillRoutine:
  // Parameters: wSkillId (WORD), Left (bool)
  push ebp
  mov ebp,esp
  sub esp,8                             // Space for packet buffer (5 bytes + alignment)
  push ebx
  push ecx
  push edx
  
  mov bx,[ebp+8]                        // wSkillId parameter
  mov cl,[ebp+12]                       // Left parameter
  
  // Create skill selection packet (5 bytes)
  // Packet structure: [PacketType][SkillId_low][SkillId_high][HandType][0x00]
  
  // Determine packet type based on Left parameter
  test cl,cl                            // Check if Left is true
  jz SetRightSkill                      // Jump if setting right skill
  
  // Set left skill (packet type 0x3C)
  mov byte ptr [ebp-8],3Ch              // Packet[0] = 0x3C (left skill)
  jmp SetSkillData
  
SetRightSkill:
  // Set right skill (packet type 0x3D)
  mov byte ptr [ebp-8],3Dh              // Packet[0] = 0x3D (right skill)
  
SetSkillData:
  // Set skill ID (bytes 1-2)
  mov [ebp-7],bx                        // Packet[1-2] = wSkillId
  
  // Set hand type based on Left parameter
  test cl,cl                            // Check Left parameter again
  jz SetRightHand                       // Jump if right hand
  
  mov byte ptr [ebp-5],80h              // Packet[3] = 0x80 (left hand)
  jmp SetFinalByte
  
SetRightHand:
  mov byte ptr [ebp-5],00h              // Packet[3] = 0x00 (right hand)
  
SetFinalByte:
  mov byte ptr [ebp-4],00h              // Packet[4] = 0x00 (padding)
  
  // Send skill selection packet: D2NET_SendPacket(5, 1, Packet)
  lea eax,[ebp-8]                       // Packet buffer address
  push eax                              // Packet buffer
  push 1                                // Priority 1 (high priority)
  push 5                                // Packet size
  call D2NET_SendPacket                 // Send packet
  add esp,12                            // Clean stack
  
  mov eax,1                             // Return true (success)
  
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromSetSkillHook:

[DISABLE]

// Restore original code
setSkillHook:
  db 6A ?? 68 ?? ?? ?? ?? E8            // Original bytes

// Clean up
unregistersymbol(D2NET_SendPacket)
```

## Configuration

### Packet Types
Different packet types for skill assignment:
- **0x3C**: Left mouse button skill assignment
- **0x3D**: Right mouse button skill assignment

### Hand Types
Hand identifier values in packet:
- **0x80**: Left hand/mouse button
- **0x00**: Right hand/mouse button

### Packet Structure
5-byte packet format for skill selection:
- **Byte 0**: Packet type (0x3C or 0x3D)
- **Bytes 1-2**: Skill ID (WORD)
- **Byte 3**: Hand type (0x80 or 0x00)
- **Byte 4**: Padding (0x00)

## Features

1. **Dual Mouse Support**: Handles both left and right mouse button skills
2. **Skill ID Validation**: Accepts any valid skill identifier
3. **High Priority**: Uses priority 1 for immediate application
4. **Simple Interface**: Boolean parameter for left/right selection
5. **Network Integration**: Uses game's packet sending system
6. **Return Status**: Boolean return indicating success

## Dependencies

This script requires the following:
- `D2NET_SendPacket()`: Network packet transmission function
- Valid skill ID parameter

## Function Signature

```asm
SetSkillRoutine:
  ; Parameters:
  ; [ebp+8]  - wSkillId (WORD) - Skill identifier to assign
  ; [ebp+12] - Left (bool) - True for left mouse, false for right mouse
  ; Returns: EAX = 1 (always successful)
```

## Usage Example

```asm
push 0                               // Right mouse button
push 54                              // Teleport skill ID
call SetSkillRoutine                 // Set teleport on right click
add esp,8                            // Clean stack
```

## Common Skill IDs

### Popular Skills for Automation
```asm
; Teleport skill
push 0                               // Right mouse
push 54                              // D2S_TELEPORT
call SetSkillRoutine

; Town Portal skill  
push 1                               // Left mouse
push 50                              // D2S_TOWNPORTAL
call SetSkillRoutine

; Static Field skill
push 0                               // Right mouse
push 42                              // D2S_STATICFIELD
call SetSkillRoutine
```

### Combat Skills
```asm
; Fireball skill
push 0                               // Right mouse
push 47                              // D2S_FIREBALL
call SetSkillRoutine

; Lightning skill
push 1                               // Left mouse
push 49                              // D2S_LIGHTNING
call SetSkillRoutine
```

## Network Protocol

The function sends packets using Diablo II's skill protocol:
- **Left Packet**: 0x3C with hand type 0x80
- **Right Packet**: 0x3D with hand type 0x00
- **Priority**: 1 (high priority for immediate application)
- **Size**: 5 bytes total

## Integration with Automation

The function is commonly used in automation sequences:

```asm
; Teleport setup sequence
push 0                               // Right mouse
push 54                              // Teleport skill
call SetSkillRoutine                 // Set teleport
add esp,8

; Wait brief moment for server acknowledgment
push 100                             // 100ms delay
call Sleep
add esp,4

; Now teleport is ready for use
push 0                               // Right mouse
push targetY                         // Y coordinate
push targetX                         // X coordinate
call TeleOnMapRoutine               // Execute teleport
add esp,12
```

## Skill Categories

### Movement Skills
- **54**: Teleport (Sorceress)
- **132**: Charge (Paladin)
- **223**: Leap (Barbarian)

### Utility Skills
- **50**: Town Portal
- **217**: Find Item (Barbarian)
- **226**: Find Potion (Barbarian)

### Combat Skills
- **42**: Static Field (Sorceress)
- **47**: Fireball (Sorceress)
- **117**: Blessed Hammer (Paladin)

## Notes

- Function always returns true (1) as packets are sent optimistically
- Server handles skill availability and mana cost validation
- Skill must be learned and available for server to accept assignment
- Function preserves all registers except EAX (return value)
- Designed for high-frequency calling in automation scripts
- Compatible with all character classes and skill trees
- Packet format is standard across all Diablo II versions
