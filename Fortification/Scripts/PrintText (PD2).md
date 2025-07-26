# "PrintText Assembly Implementation (PD2)"

## Metadata
- **ID**: 135002
- **Description**: "PrintText Assembly Implementation for Project Diablo 2"
- **Options**: Color-coded text output
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
  This script implements formatted text printing with color support in assembly.
  It provides a way to display formatted text messages in the game with various
  color codes, similar to printf functionality but for Diablo II.
  
  Based on the PrintText() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and function pointers from D2Ptrs.h
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2CLIENT_PrintGameString,0x7D850) // FUNCPTR(D2CLIENT, PrintGameString, void __stdcall, (wchar_t *wMessage, int nColor), 0x7D850)
define(vsprintf_s,msvcrt.vsprintf_s)    // Format function
define(MultiByteToWideChar,kernel32.MultiByteToWideChar) // Convert function

// Hook for text printing calls
aobscanmodule(printTextHook,D2CLIENT.dll,E8 ?? ?? ?? ?? 83 C4 ?? 85 C0)

printTextHook:
  call PrintTextRoutine                 // Call our print function
  // Original code
  call [originalFunction]               // Original call
  add esp,??                           // Original instruction
  test eax,eax                         // Original instruction
  jmp returnFromPrintHook

PrintTextRoutine:
  push ebp
  mov ebp,esp
  sub esp,200                          // Local buffer space (152 + 48 for wide buffer)
  
  // Check if game is ready
  call GameReady
  test eax,eax
  jz PrintTextExit                     // Exit if game not ready
  
  // Parameters: Color (DWORD), Text (char*), ... (va_args)
  mov eax,[ebp+8]                      // Color parameter
  push eax                             // Save color
  
  // Format the string using va_args
  lea eax,[ebp-152]                    // Buffer address
  push eax                             // Buffer
  push 152                             // Buffer size
  push [ebp+12]                        // Format string
  lea eax,[ebp+16]                     // va_args start
  push eax                             // va_args
  call vsprintf_s                      // Format string
  add esp,16                           // Clean stack
  
  // Convert to wide character
  lea eax,[ebp-152]                    // Source buffer (char)
  lea edx,[ebp-456]                    // Destination buffer (wchar_t)
  
  push 304                             // Destination size (152 * 2)
  push edx                             // Destination buffer
  push 152                             // Source size
  push eax                             // Source buffer  
  push 1                               // Flags
  push 0                               // Code page (CP_ACP)
  call MultiByteToWideChar             // Convert to wide chars
  add esp,24                           // Clean stack
  
  // Print the game string
  pop eax                              // Restore color
  push eax                             // Color parameter
  lea edx,[ebp-456]                    // Wide string buffer
  push edx                             // Wide string
  call D2CLIENT_PrintGameString        // Print to game
  add esp,8                            // Clean stack
  
PrintTextExit:
  mov esp,ebp
  pop ebp
  ret

returnFromPrintHook:

[DISABLE]

// Restore original code
printTextHook:
  db E8 ?? ?? ?? ??                    // Original bytes
  db 83 C4 ?? 85 C0                    // Original bytes

// Clean up
unregistersymbol(GameReady)
unregistersymbol(D2CLIENT_PrintGameString)
```

## Configuration

### Color Codes
The color parameter supports various game color codes:
- **0**: White text
- **1**: Red text  
- **2**: Green text
- **3**: Blue text
- **4**: Gold text
- **5**: Gray text
- **9**: Orange text

### Format String Support
Supports standard printf format specifiers:
- **%d**: Integer values
- **%s**: String values
- **%c**: Character values
- **%.2f**: Floating point with precision

## Features

1. **Variable Arguments**: Supports printf-style formatting with multiple parameters
2. **Game State Checking**: Only prints when game is ready and loaded
3. **Color Support**: Full color code support for game text display
4. **Wide Character Conversion**: Properly converts to wide characters for game display
5. **Buffer Management**: Safe buffer handling to prevent overflows
6. **Stack Management**: Proper parameter passing and cleanup

## Dependencies

This script requires the following functions:
- `GameReady()`: Validates game state before printing
- `D2CLIENT_PrintGameString()`: Game's text display function
- `vsprintf_s()`: Format string with variable arguments
- `MultiByteToWideChar()`: Convert to wide characters

## Usage Example

```asm
// Print colored text
push 4                               // Gold color
push szFormat                        // "Player: %s, Level: %d"
push szPlayerName                    // "Paladin"
push 85                             // Level value
call PrintTextRoutine
add esp,16                          // Clean stack
```

## Notes

- Buffer sizes are fixed to prevent overflow (152 bytes for formatted text)
- Wide character buffer is double the size (304 bytes) for Unicode support
- All register states are preserved during execution
- Compatible with Diablo II's internal text rendering system
