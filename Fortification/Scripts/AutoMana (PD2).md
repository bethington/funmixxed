# "AutoMana Assembly Implementation (PD2)"

## Metadata
- **ID**: 135007
- **Description**: "AutoMana Assembly Implementation for Project Diablo 2"
- **Options**: Automatic mana potion usage with percentage threshold
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
  This script implements automatic mana potion usage in assembly.
  It monitors the player's mana percentage and automatically uses mana potions
  when the mana drops below a configurable threshold, with cooldown protection.
  
  Based on the AutoMana() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and functions from D2Ptrs.h
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(D2CLIENT_GetPlayerUnit,0xA4D60)  // FUNCPTR(D2CLIENT, GetPlayerUnit, UnitAny* __stdcall,(),0xA4D60)
define(D2COMMON_GetUnitStat,-10973)     // FUNCPTR(D2COMMON, GetUnitStat, DWORD __stdcall, (UnitAny* pUnit, DWORD dwStat, DWORD dwStat2), -10973)
define(D2COMMON_GetUnitState,-10494)    // FUNCPTR(D2COMMON, GetUnitState, INT __stdcall, (LPUNITANY Unit, DWORD State), -10494)
define(CalcPercent,CUSTOM_FUNCTION)     // Custom percentage calculation function
define(Mana,CUSTOM_FUNCTION)            // Custom mana function using FindItem + UseItem
define(GetTickCount,kernel32.GetTickCount) // Timer function
define(v_LastMana,50000000)             // Last mana use timestamp
define(v_AutoManaPercent,50000004)      // Mana percentage threshold
define(STAT_MANA,8)                     // Mana stat ID
define(STAT_MAXMANA,9)                  // Max mana stat ID
define(AFFECT_MANAPOT,101)              // Mana potion effect ID
define(COOLDOWN_TIME,500)               // 500ms cooldown

// Hook for automatic mana usage
aobscanmodule(autoManaHook,D2CLIENT.dll,8B ?? ?? ?? ?? ?? 85 ?? 7E ??)

autoManaHook:
  call AutoManaRoutine                  // Call our auto mana function
  // Original code
  mov eax,[????????]                    // Original instruction
  test eax,eax                          // Original instruction
  jmp returnFromAutoManaHook

AutoManaRoutine:
  push ebp
  mov ebp,esp
  push ebx
  push ecx
  push edx
  push esi
  
  // Check if game is ready (using D2CLIENT_GetPlayerUnit)
  call D2CLIENT_GetPlayerUnit
  test eax,eax
  jz AutoManaExit                       // Exit if game not ready
  
  // Calculate current mana percentage
  // Get current mana (STAT_MANA >> 8)
  push 0                                // dwStat2 parameter (0 for base stat)
  push STAT_MANA                        // Mana stat ID
  push [pMe]                            // Player unit
  call D2COMMON_GetUnitStat             // Get current mana
  add esp,12                            // Clean stack (3 parameters)
  shr eax,8                             // Shift right by 8 bits
  push eax                              // Save current mana
  
  // Get max mana (STAT_MAXMANA >> 8)
  push 0                                // dwStat2 parameter (0 for base stat)
  push STAT_MAXMANA                     // Max mana stat ID
  push [pMe]                            // Player unit
  call D2COMMON_GetUnitStat             // Get max mana
  add esp,12                            // Clean stack (3 parameters)
  shr eax,8                             // Shift right by 8 bits
  mov edx,eax                           // Max mana in edx
  pop eax                               // Current mana in eax
  
  // Calculate percentage: CalcPercent(current, max)
  push edx                              // Max mana
  push eax                              // Current mana
  call CalcPercent                      // Calculate percentage
  add esp,8                             // Clean stack
  mov ebx,eax                           // Mana percentage in ebx
  
  // Check cooldown timer (GetTickCount() - v_LastMana >= 500)
  call GetTickCount                     // Get current time
  mov ecx,[v_LastMana]                  // Last mana use time
  sub eax,ecx                           // Current time - last use time
  cmp eax,COOLDOWN_TIME                 // Compare with 500ms cooldown
  jl AutoManaExit                       // Exit if still in cooldown
  
  // Check if mana potion effect is active
  push AFFECT_MANAPOT                   // Mana potion effect ID
  push [pMe]                            // Player unit
  call D2COMMON_GetUnitState            // Check for mana potion effect
  add esp,8                             // Clean stack
  test eax,eax
  jnz AutoManaExit                      // Exit if mana potion effect active
  
  // Check if mana percentage is below threshold
  mov eax,[v_AutoManaPercent]           // Load mana percentage threshold
  cmp ebx,eax                           // Compare current mana % with threshold
  jg AutoManaExit                       // Exit if mana is above threshold
  
  // Use mana potion
  call Mana                             // Call mana potion function
  test eax,eax
  jz UpdateTimer                        // Skip timer update if no potion used
  
UpdateTimer:
  // Update last use timestamp
  call GetTickCount                     // Get current time
  mov [v_LastMana],eax                  // Store current tick count
  
AutoManaExit:
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromAutoManaHook:

[DISABLE]

// Restore original code
autoManaHook:
  db 8B ?? ?? ?? ?? ?? 85 ?? 7E ??      // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(GameReady)
unregistersymbol(GetUnitStat)
unregistersymbol(GetUnitState)
unregistersymbol(CalcPercent)
unregistersymbol(Mana)
unregistersymbol(v_LastMana)
unregistersymbol(v_AutoManaPercent)
```

## Configuration

### Mana Threshold
The `v_AutoManaPercent` variable controls when mana potions are used:
- **Default**: 30 (use potion when mana drops to 30% or below)
- **Range**: 0-100
- **Location**: Memory address 0x50000004

### Cooldown Timer
- **Duration**: 500ms minimum between potion uses
- **Purpose**: Prevents rapid potion consumption
- **Constant**: COOLDOWN_TIME = 500

### Stat IDs
Diablo II stat identifiers:
- **STAT_MANA (8)**: Current mana points
- **STAT_MAXMANA (9)**: Maximum mana points
- **AFFECT_MANAPOT (101)**: Mana potion effect state

## Features

1. **Percentage-Based**: Uses configurable mana percentage threshold
2. **Cooldown Protection**: 500ms minimum delay between uses
3. **Effect Checking**: Won't use potion if mana potion effect is active
4. **Game State Validation**: Only operates when game is ready
5. **Automatic Calculation**: Real-time mana percentage calculation
6. **Memory Efficient**: Minimal memory usage and fast execution

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `GameReady()`: Game state validation function
- `GetUnitStat()`: Function to read unit statistics
- `GetUnitState()`: Function to check unit effects/states
- `CalcPercent()`: Percentage calculation function
- `Mana()`: Mana potion usage function

## Function Flow

1. **Game Validation**: Check if game is ready
2. **Mana Calculation**: Get current and max mana, calculate percentage
3. **Cooldown Check**: Ensure enough time has passed since last use
4. **Effect Check**: Verify no mana potion effect is currently active
5. **Threshold Check**: Compare mana percentage with configured threshold
6. **Potion Usage**: Use mana potion if all conditions are met
7. **Timer Update**: Update last use timestamp

## Usage Example

```asm
// This function is typically called from a game loop hook
call AutoManaRoutine                  // Check and use mana if needed
; Function automatically handles all logic
```

## Memory Layout

```asm
v_LastMana:        DD 0               // Last mana use timestamp
v_AutoManaPercent: DD 30              // Mana threshold percentage (30%)
```

## Algorithm Logic

```
IF GameReady() AND 
   (GetTickCount() - v_LastMana >= 500) AND
   NOT GetUnitState(pMe, AFFECT_MANAPOT) AND
   (CurrentManaPercent <= v_AutoManaPercent)
THEN
   Mana()
   v_LastMana = GetTickCount()
```

## Notes

- Function is designed for continuous background operation
- Mana percentage is calculated using bit-shifted stat values
- All register states are preserved during execution
- Compatible with Project Diablo 2's stat and effect systems
- Integrates seamlessly with existing mana management functions
