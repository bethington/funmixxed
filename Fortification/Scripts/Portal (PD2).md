# "Portal Assembly Implementation (PD2)"

## Metadata
- **ID**: 135008
- **Description**: "Portal Assembly Implementation for Project Diablo 2"
- **Options**: Town Portal usage with book and scroll support
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
  This script implements Town Portal functionality in assembly.
  It automatically searches for TP tomes or scrolls across all storage
  locations and uses them, with proper book quantity checking.
  
  Based on the Portal() function from Fortification\D2Helpers.cpp
}

[ENABLE]

// Memory addresses and functions from D2Ptrs.h
define(pMe,0x11BBFC)                    // VARPTR(D2CLIENT, PlayerUnit, UnitAny *, 0x11BBFC)
define(FindItem,CUSTOM_FUNCTION)        // Custom function using inventory traversal
define(UseItem,CUSTOM_FUNCTION)         // Custom function using D2NET_SendPacket
define(D2COMMON_GetUnitStat,-10973)     // FUNCPTR(D2COMMON, GetUnitStat, DWORD __stdcall, (UnitAny* pUnit, DWORD dwStat, DWORD dwStat2), -10973)
define(STORAGE_STASH,1)                 // Storage location constants
define(STORAGE_BELT,2)
define(STORAGE_INVENTORY,3)
define(STORAGE_CUBE,4)
define(STAT_AMMOQUANTITY,70)            // Ammunition quantity stat
define(TP_TOME_CODE,520)                // PD2 TP Tome item code
define(TP_SCROLL_CODE,531)              // PD2 TP Scroll item code

// Hook for portal usage
aobscanmodule(portalHook,D2CLIENT.dll,E8 ?? ?? ?? ?? 84 C0 74 ??)

portalHook:
  call PortalRoutine                    // Call our portal function
  test al,al                            // Test return value (BOOL)
  jmp returnFromPortalHook

PortalRoutine:
  push ebp
  mov ebp,esp
  sub esp,16                            // Local variables space
  push ebx
  push ecx
  push edx
  push esi
  push edi
  
  // Local variables:
  // [ebp-4]  = Book flag (BOOL)
  // [ebp-8]  = Quantity (INT)
  // [ebp-12] = Item pointer (UnitAny*)
  
  // Initialize local variables
  mov dword ptr [ebp-4],0               // Book = FALSE
  mov dword ptr [ebp-8],0               // Quantity = 0
  mov dword ptr [ebp-12],0              // Item = NULL
  
  // Check if cursor item exists
  mov eax,[pMe]
  test eax,eax
  jz PortalFailed                       // Exit if no player
  
  mov eax,[eax+60h]                     // pInventory offset
  test eax,eax
  jz PortalFailed                       // Exit if no inventory
  
  mov eax,[eax+B0h]                     // pCursorItem offset
  test eax,eax
  jnz PortalFailed                      // Exit if cursor item exists
  
  // Search for TP Tome in Stash
  push STORAGE_STASH                    // Storage location
  push TP_TOME_CODE                     // Item code (520)
  call FindItem                         // Search for TP tome
  add esp,8                             // Clean stack
  mov [ebp-12],eax                      // Store item pointer
  test eax,eax
  jz CheckInventoryTome                 // Not found in stash
  
  // Check tome quantity in stash
  push STAT_AMMOQUANTITY                // Stat ID
  push eax                              // Item pointer
  call GetUnitStat                      // Get ammunition quantity
  add esp,8                             // Clean stack
  test eax,eax
  jnz StashTomeValid                    // Tome has charges
  
CheckInventoryTome:
  // Search for TP Tome in Inventory
  push STORAGE_INVENTORY                // Storage location
  push TP_TOME_CODE                     // Item code (520)
  call FindItem                         // Search for TP tome
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckCubeTome                      // Not found in inventory
  mov [ebp-12],eax                      // Store item pointer
  
  // Check tome quantity in inventory
  push STAT_AMMOQUANTITY                // Stat ID
  push eax                              // Item pointer
  call GetUnitStat                      // Get ammunition quantity
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckCubeTome                      // Empty tome, continue searching
  
InventoryTomeValid:
  mov dword ptr [ebp-4],1               // Book = TRUE
  jmp UsePortalItem                     // Use the tome
  
CheckCubeTome:
  // Search for TP Tome in Cube
  push STORAGE_CUBE                     // Storage location
  push TP_TOME_CODE                     // Item code (520)
  call FindItem                         // Search for TP tome
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckBeltScroll                    // Not found in cube
  mov [ebp-12],eax                      // Store item pointer
  
  // Check tome quantity in cube
  push STAT_AMMOQUANTITY                // Stat ID
  push eax                              // Item pointer
  call GetUnitStat                      // Get ammunition quantity
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckBeltScroll                    // Empty tome, continue searching
  
CubeTomeValid:
  mov dword ptr [ebp-4],1               // Book = TRUE
  jmp UsePortalItem                     // Use the tome
  
CheckBeltScroll:
  // Search for TP Scroll in Belt
  push STORAGE_BELT                     // Storage location
  push TP_SCROLL_CODE                   // Item code (531)
  call FindItem                         // Search for TP scroll
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckInventoryScroll               // Not found in belt
  mov [ebp-12],eax                      // Store item pointer
  jmp UsePortalItem                     // Use the scroll
  
CheckInventoryScroll:
  // Search for TP Scroll in Inventory
  push STORAGE_INVENTORY                // Storage location
  push TP_SCROLL_CODE                   // Item code (531)
  call FindItem                         // Search for TP scroll
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckStashScroll                   // Not found in inventory
  mov [ebp-12],eax                      // Store item pointer
  jmp UsePortalItem                     // Use the scroll
  
CheckStashScroll:
  // Search for TP Scroll in Stash
  push STORAGE_STASH                    // Storage location
  push TP_SCROLL_CODE                   // Item code (531)
  call FindItem                         // Search for TP scroll
  add esp,8                             // Clean stack
  test eax,eax
  jz CheckCubeScroll                    // Not found in stash
  mov [ebp-12],eax                      // Store item pointer
  jmp UsePortalItem                     // Use the scroll
  
CheckCubeScroll:
  // Search for TP Scroll in Cube
  push STORAGE_CUBE                     // Storage location
  push TP_SCROLL_CODE                   // Item code (531)
  call FindItem                         // Search for TP scroll
  add esp,8                             // Clean stack
  test eax,eax
  jz PortalFailed                       // Not found anywhere
  mov [ebp-12],eax                      // Store item pointer
  
UsePortalItem:
  // Check if we found an item
  mov eax,[ebp-12]                      // Get item pointer
  test eax,eax
  jz PortalFailed                       // No item found
  
  // Use the item
  push eax                              // Item pointer
  call UseItem                          // Use the portal item
  add esp,4                             // Clean stack
  
  // If it was a book, get the quantity for reference
  mov eax,[ebp-4]                       // Check book flag
  test eax,eax
  jz PortalSuccess                      // Not a book, skip quantity check
  
  // Get book quantity for reference
  mov eax,[ebp-12]                      // Item pointer
  push STAT_AMMOQUANTITY                // Stat ID
  push eax                              // Item pointer
  call GetUnitStat                      // Get quantity
  add esp,8                             // Clean stack
  mov [ebp-8],eax                       // Store quantity
  
PortalSuccess:
  mov eax,1                             // Return TRUE
  jmp PortalExit
  
StashTomeValid:
  mov dword ptr [ebp-4],1               // Book = TRUE
  jmp UsePortalItem                     // Use the tome
  
PortalFailed:
  xor eax,eax                           // Return FALSE
  
PortalExit:
  pop edi
  pop esi
  pop edx
  pop ecx
  pop ebx
  mov esp,ebp
  pop ebp
  ret

returnFromPortalHook:

[DISABLE]

// Restore original code
portalHook:
  db E8 ?? ?? ?? ?? 84 C0 74 ??        // Original bytes

// Clean up
unregistersymbol(pMe)
unregistersymbol(FindItem)
unregistersymbol(UseItem)
unregistersymbol(GetUnitStat)
```

## Configuration

### Item Codes (PD2)
Project Diablo 2 portal item codes:
- **520**: Town Portal Tome (book with multiple charges)
- **531**: Town Portal Scroll (single use)

### Search Priority
The function searches in this specific order:
1. **TP Tome in Stash** (with quantity check)
2. **TP Tome in Inventory** (with quantity check)
3. **TP Tome in Cube** (with quantity check)
4. **TP Scroll in Belt** (quick access)
5. **TP Scroll in Inventory** (main storage)
6. **TP Scroll in Stash** (secondary storage)
7. **TP Scroll in Cube** (alternative storage)

### Validation Checks
- **Cursor Item**: Ensures no item is held on cursor
- **Tome Quantity**: Verifies tomes have remaining charges
- **Item Availability**: Confirms portal item exists before use

## Features

1. **Intelligent Priority**: Prefers tomes over scrolls for efficiency
2. **Quantity Validation**: Checks tome charges before usage
3. **Multi-Location Search**: Searches all storage areas systematically
4. **Cursor Safety**: Prevents usage when cursor is occupied
5. **Fallback Support**: Multiple fallback options if preferred items unavailable
6. **Book Tracking**: Tracks tome usage and remaining charges

## Dependencies

This script requires the following:
- `pMe`: Global player unit pointer
- `FindItem()`: Item searching function
- `UseItem()`: Item usage function
- `GetUnitStat()`: Function to read item statistics

## Function Signature

```asm
PortalRoutine:
  ; Parameters: None
  ; Returns: EAX = 1 if portal opened successfully, 0 if failed
```

## Usage Example

```asm
call PortalRoutine                    // Attempt to open portal
test eax,eax                          // Check if successful
jz NoPortalAvailable                  // Jump if failed
; Portal was opened successfully
```

## Memory Offsets

Key offsets used:
- **pInventory**: +0x60 from player unit
- **pCursorItem**: +0xB0 from inventory

## Algorithm Flow

1. **Validation**: Check cursor state and player unit
2. **Tome Search**: Search for TP tomes with quantity validation
3. **Scroll Fallback**: Search for TP scrolls if no valid tomes
4. **Usage**: Use first available portal item found
5. **Tracking**: Record book usage and quantity information

## Notes

- Function prioritizes tomes over scrolls for resource efficiency
- Empty tomes are skipped during search
- All storage locations are searched systematically
- Function preserves all registers except EAX (return value)
- Designed for integration with escape and transportation systems
- Compatible with Project Diablo 2's item and inventory systems
