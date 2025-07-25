# "AutoLife Assembly Implementation (PD2)"

## Metadata
- **ID**: 135001
- **Description**: "AutoLife Assembly Implementation for Project Diablo 2"
- **Options**: Configurable life percentage threshold
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
  This script implements an automatic health potion usage system in assembly.
  It monitors the player's life percentage and automatically uses health potions
  when the life drops below a configurable threshold. The script includes
  cooldown protection and health potion effect checking to prevent spam usage.
  
  Based on the AutoLife() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses (these would need to be updated for actual PD2 offsets)
define(pMe,40000000)                    // Player unit pointer
define(GetTickCount,kernel32.GetTickCount)
define(v_LastLife,50000000)             // Last health potion use timestamp
define(v_AutoLifePercent,50000004)      // Life percentage threshold (default 30)

// Function pointers (would need actual PD2 addresses)
define(GameReady,60000000)              // GameReady function
define(GetUnitStat,60000004)            // GetUnitStat function  
define(GetUnitState,60000008)           // GetUnitState function
define(Health,6000000C)                 // Health potion use function
define(CalcPercent,60000010)            // Percentage calculation function

// Constants
define(STAT_HP,6)                       // Current HP stat ID
define(STAT_MAXHP,7)                    // Maximum HP stat ID  
define(AFFECT_HEALTHPOT,100)            // Health potion effect ID
define(COOLDOWN_TIME,500)               // 500ms cooldown between uses

// Main AutoLife hook - inject into game loop
aobscanmodule(gameLoopHook,D2CLIENT.dll,8B 0D ?? ?? ?? ?? 85 C9 74 ??)

gameLoopHook:
  pushad                                // Save all registers
  call AutoLifeRoutine                  // Call our AutoLife function
  popad                                 // Restore all registers
  // Original code
  mov ecx,[????????]                    // Original instruction
  test ecx,ecx                          // Original instruction
  jmp returnFromHook                    // Jump back to original code

AutoLifeRoutine:
  push ebp
  mov ebp,esp
  
  // Check if game is ready
  call GameReady
  test eax,eax
  jz AutoLifeExit                       // Exit if game not ready
  
  // Calculate current life percentage
  // LifePercent = CalcPercent((GetUnitStat(pMe, STAT_HP) >> 8), (GetUnitStat(pMe, STAT_MAXHP) >> 8))
  
  // Get current HP
  push STAT_HP                          // stat ID
  push [pMe]                            // player unit
  call GetUnitStat
  add esp,8                             // Clean stack
  shr eax,8                             // Shift right by 8 bits
  push eax                              // Save current HP
  
  // Get max HP  
  push STAT_MAXHP                       // stat ID
  push [pMe]                            // player unit
  call GetUnitStat
  add esp,8                             // Clean stack
  shr eax,8                             // Shift right by 8 bits
  mov edx,eax                           // max HP in edx
  pop eax                               // current HP in eax
  
  // Calculate percentage: (current * 100) / max
  push edx                              // Save max HP
  push eax                              // current HP
  push edx                              // max HP
  call CalcPercent
  add esp,8                             // Clean stack
  mov ebx,eax                           // Life percentage in ebx
  
  // Check cooldown timer
  call GetTickCount
  mov ecx,[v_LastLife]                  // Last use time
  sub eax,ecx                           // Current time - last use time
  cmp eax,COOLDOWN_TIME                 // Compare with 500ms cooldown
  jl AutoLifeExit                       // Exit if still in cooldown
  
  // Check if health potion effect is active
  push AFFECT_HEALTHPOT                 // Health potion effect ID
  push [pMe]                            // Player unit
  call GetUnitState
  add esp,8                             // Clean stack
  test eax,eax
  jnz AutoLifeExit                      // Exit if health potion effect active
  
  // Check if life percentage is below threshold
  mov eax,[v_AutoLifePercent]           // Load life percentage threshold
  cmp ebx,eax                           // Compare current life % with threshold
  jg AutoLifeExit                       // Exit if life is above threshold
  
  // Use health potion
  call Health                           // Call health potion function
  test eax,eax
  jz AutoLifeExit                       // Exit if no potion was used
  
  // Update last use timestamp
  call GetTickCount
  mov [v_LastLife],eax                  // Store current tick count
  
AutoLifeExit:
  mov esp,ebp
  pop ebp
  ret

returnFromHook:

[DISABLE]

// Restore original code
gameLoopHook:
  db 8B 0D ?? ?? ?? ??                  // Original bytes would go here
  db 85 C9 74 ??                        // Original bytes continued

// Clean up allocated memory
unregistersymbol(pMe)
unregistersymbol(v_LastLife)
unregistersymbol(v_AutoLifePercent)
unregistersymbol(GameReady)
unregistersymbol(GetUnitStat)
unregistersymbol(GetUnitState)
unregistersymbol(Health)
unregistersymbol(CalcPercent)
```

## Configuration

### Life Percentage Threshold
The `v_AutoLifePercent` variable controls when health potions are automatically used:
- **Default**: 30 (use potion when life drops to 30% or below)
- **Range**: 0-100
- **Location**: Memory address 0x50000004

### Cooldown Timer
The script includes a 500ms cooldown between potion uses to prevent spam:
- **Constant**: COOLDOWN_TIME = 500ms
- **Purpose**: Prevents multiple potions from being used rapidly

## Features

1. **Game State Checking**: Only operates when the game is ready and loaded
2. **Percentage Calculation**: Accurately calculates current life as percentage of maximum
3. **Cooldown Protection**: 500ms minimum delay between potion uses
4. **Effect Checking**: Won't use another potion if health potion effect is still active
5. **Threshold Based**: Configurable life percentage threshold for activation
6. **Memory Safe**: Proper register preservation and stack management

## Dependencies

This script requires the following functions to be properly hooked:
- `GameReady()`: Checks if game state is valid
- `GetUnitStat()`: Retrieves unit statistics (HP, Max HP)
- `GetUnitState()`: Checks for active effects/states
- `Health()`: Triggers health potion usage
- `CalcPercent()`: Calculates percentage values

## Notes

- Memory addresses marked with `??` need to be determined for actual PD2 implementation
- The script hooks into the main game loop for continuous monitoring
- All register states are preserved to prevent game crashes
- Compatible with Project Diablo 2 health potion item codes (589-593)
