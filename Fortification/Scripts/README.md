# "D2Helpers Assembly Scripts Index (PD2)"

## Overview

This directory contains assembly implementations of all major functions from `Fortification\D2Helpers.cpp`, converted to Cheat Engine Auto Assembler scripts for Project Diablo 2.

## Created Scripts

### Core System Functions
1. **[PrintText (PD2).md](PrintText%20(PD2).md)** - Formatted text output with color support
2. **[GameReady (PD2).md](GameReady%20(PD2).md)** - Comprehensive game state validation
3. **[AutoLife Assembly (PD2).md](../AutoLife%20Assembly%20(PD2).md)** - Automatic health potion usage

### Item Management Functions
4. **[FindItem (PD2).md](FindItem%20(PD2).md)** - Item searching by code and location
5. **[UseItem (PD2).md](UseItem%20(PD2).md)** - Item usage with packet sending

### Potion and Automation Functions
6. **[Health (PD2).md](Health%20(PD2).md)** - Health potion selection and usage
7. **[AutoMana (PD2).md](AutoMana%20(PD2).md)** - Automatic mana potion usage with thresholds

### Utility Functions
8. **[Portal (PD2).md](Portal%20(PD2).md)** - Town Portal usage with tome/scroll support

## Remaining Functions to Implement

The following functions from `D2Helpers.cpp` still need assembly implementations:

### Core Helper Functions
- `GetPlayerArea()` - Get current player area/level
- `GetLevel()` - Get level structure by ID
- `IsTownLevel()` - Check if area is a town
- `GetDistance()` / `GetDistanceSquared()` - Distance calculations
- `GetTileLevelNo()` - Get level from tile information

### Event and Input Handling
- `GameEventHandler()` - Main input/event processing
- `MiddleMouseButton()` - Teleport on middle mouse click
- `ScreenToAutoMap()` - Screen coordinate conversion
- `GetCurrentSkill()` - Get active skill information
- `GetRoom()` - Get room from coordinates

### Item and Inventory Functions
- `GetItemLocation()` - Determine item storage location
- `UseItemIds()` - Get item IDs for usage
- `D2IsPotion()` - Identify potion types
- `GetItemCode()` - Get item code string
- `GetTotalBeltTPs()` / `GetTotalCubeTPs()` / etc. - Count items by type

### Potion Management Functions
- `Mana()` - Mana potion usage
- `Rejuvenation()` - Rejuvenation potion usage
- `AutoJuv()` - Automatic rejuvenation usage
- `GetTotalBeltRejuvenation()` / etc. - Count rejuvenation potions
- `GetTotalBeltMana()` / etc. - Count mana potions
- `GetTotalBeltHealth()` / etc. - Count health potions

### Combat and Skill Functions
- `UseSkillOnUnit()` - Cast skills on targets
- `GetUnitDist()` - Calculate unit distances
- `FindNearestUnit()` - Find closest enemy
- `UsingSkill()` - Check if casting/attacking
- `AttackRightMonster()` / `AttackLeftMonster()` - Auto-attack functions

### Safety and Automation Functions
- `SinglePlayerChicken()` - Emergency town portal
- `Exit()` - Emergency game exit
- `AutoPrecast()` - Automatic skill precasting
- `ResetPrecast()` - Reset precast state

## Implementation Notes

### Common Features
All assembly implementations include:
- **Memory Safety**: Comprehensive null pointer checking
- **Register Preservation**: All registers preserved except return values
- **Game State Validation**: Integration with GameReady() checks
- **Error Handling**: Proper failure modes and exit conditions
- **Performance Optimization**: Minimal overhead assembly code

### Memory Addresses
All scripts use placeholder memory addresses that need to be updated for actual PD2 implementation:
- Function pointers: `60000000` series
- Global variables: `40000000` and `50000000` series
- These addresses must be determined through reverse engineering

### Constants and Offsets
Memory offsets and constants are based on Project Diablo 2 structure analysis:
- Item codes updated for PD2 (517-598 range)
- Storage location constants (1-4 range)
- Stat IDs matching PD2 values
- Effect IDs for potion states

### Integration Points
Scripts are designed to hook into various game functions:
- Game loop hooks for automatic functions
- Input handling hooks for manual functions
- Network packet hooks for item usage
- UI event hooks for configuration

## Usage Instructions

1. **Memory Address Resolution**: Update all placeholder addresses with actual PD2 values
2. **Offset Verification**: Verify structure offsets match current PD2 version
3. **Hook Point Identification**: Locate appropriate injection points in game code
4. **Testing and Validation**: Test each function individually before integration
5. **Performance Monitoring**: Ensure minimal impact on game performance

## Development Priority

Recommended implementation order:
1. **Core Functions**: GameReady, PrintText, GetPlayerArea
2. **Item Management**: FindItem, UseItem, GetItemLocation
3. **Basic Automation**: Health, Mana, AutoLife, AutoMana
4. **Advanced Features**: Portal, combat functions, safety systems
5. **UI and Input**: GameEventHandler, mouse/keyboard handling

## Compatibility

All scripts are designed for:
- **Game**: Diablo II Version 1.14d
- **Mod**: Project Diablo 2
- **Tool**: Cheat Engine Auto Assembler
- **Architecture**: x86 (32-bit)

## Notes

- Scripts maintain compatibility with original C++ function signatures
- Assembly implementations preserve original algorithmic logic
- Memory management follows game's allocation patterns
- Network protocol compatibility maintained for multiplayer support
