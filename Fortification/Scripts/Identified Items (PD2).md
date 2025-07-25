# "Identified Items (PD2)"

## Metadata
- **ID**: 134787
- **Description**: "Identified Items (PD2)"
- **Options**: None
- **VariableType**: Auto Assembler Script
- **Source File**: Diablo II.ct

## AssemblerScript

```asm
{ 
  Game   : Diablo II
  Version: 1.14d
  Date   : 2025-07-24
  Author : Xerzes

  Description:
  This script forces all dropped items in Diablo II (Project Diablo 2 mod) to be automatically identified. It patches multiple locations in D2GAME.dll to set the identified flag for set, unique, rare, and magic items when they are created or dropped.
}

[ENABLE]

// Scan for set, unique, rare, and magic item drop routines
// Each patch sets the push value to 1 (identified) instead of 0 (unidentified)
aobscanmodule(dropIdentifiedSet,D2GAME.dll,6A 00 6A 10 56 E8 AB) // should be unique
aobscanmodule(dropIdentifiedUnique,D2GAME.dll,6A 00 6A 10 53) // should be unique
aobscanmodule(dropIdentifiedRare,D2GAME.dll,6A 00 6A 10 56 E8 12) // should be unique
aobscanmodule(dropIdentifiedMagic,D2GAME.dll,6A 00 6A 10 50 E8 94) // should be unique

dropIdentifiedSet:
  push 1 // Set dropped set item as identified
dropIdentifiedUnique:
  push 1 // Set dropped unique item as identified
dropIdentifiedRare:
  push 1 // Set dropped rare item as identified
dropIdentifiedMagic:
  push 1 // Set dropped magic item as identified

[DISABLE]

// Restore original push value (0 = unidentified)
dropIdentifiedSet:
  db 6A 00
dropIdentifiedUnique:
  db 6A 00
dropIdentifiedRare:
  db 6A 00
dropIdentifiedMagic:
  db 6A 00

{
// ORIGINAL CODE - INJECTION POINT: D2GAME.Ordinal10013+BE9

D2GAME.Ordinal10013+BCA: 5D                    - pop ebp
D2GAME.Ordinal10013+BCB: 33 C0                 - xor eax,eax
D2GAME.Ordinal10013+BCD: 5B                    - pop ebx
D2GAME.Ordinal10013+BCE: 81 C4 0C 08 00 00     - add esp,0000080C
D2GAME.Ordinal10013+BD4: C2 08 00              - ret 0008
D2GAME.Ordinal10013+BD7: 8B 44 C4 1C           - mov eax,[esp+eax*8+1C]
D2GAME.Ordinal10013+BDB: 8B B4 24 20 08 00 00  - mov esi,[esp+00000820]
D2GAME.Ordinal10013+BE2: 50                    - push eax
D2GAME.Ordinal10013+BE3: 56                    - push esi
D2GAME.Ordinal10013+BE4: E8 01 74 FF FF        - call D2GAME.dll+B1AA
// ---------- INJECTING HERE ----------
D2GAME.Ordinal10013+BE9: 6A 00                 - push 00
// ---------- DONE INJECTING  ----------
D2GAME.Ordinal10013+BEB: 6A 10                 - push 10
D2GAME.Ordinal10013+BED: 56                    - push esi
D2GAME.Ordinal10013+BEE: E8 AB 68 FF FF        - call D2GAME.dll+A65E
D2GAME.Ordinal10013+BF3: 6A 00                 - push 00
D2GAME.Ordinal10013+BF5: 6A 00                 - push 00
D2GAME.Ordinal10013+BF7: 6A 00                 - push 00
D2GAME.Ordinal10013+BF9: 56                    - push esi
D2GAME.Ordinal10013+BFA: 6A 00                 - push 00
D2GAME.Ordinal10013+BFC: 6A 04                 - push 04
D2GAME.Ordinal10013+BFE: E8 0D 6C FF FF        - call D2GAME.dll+A9D0
}

{
// ORIGINAL CODE - INJECTION POINT: D2GAME.dll+F6BD

D2GAME.dll+F6A0: 3B 48 28        - cmp ecx,[eax+28]
D2GAME.dll+F6A3: 75 4D           - jne D2GAME.dll+F6F2
D2GAME.dll+F6A5: 8B 5C 24 24     - mov ebx,[esp+24]
D2GAME.dll+F6A9: 56              - push esi
D2GAME.dll+F6AA: 53              - push ebx
D2GAME.dll+F6AB: E8 FA BA FF FF  - call D2GAME.dll+B1AA
D2GAME.dll+F6B0: 8B 44 24 20     - mov eax,[esp+20]
D2GAME.dll+F6B4: E8 07 F7 FF FF  - call D2GAME.dll+EDC0
D2GAME.dll+F6B9: 85 C0           - test eax,eax
D2GAME.dll+F6BB: 74 2D           - je D2GAME.dll+F6EA
// ---------- INJECTING HERE ----------
D2GAME.dll+F6BD: 6A 00           - push 00
// ---------- DONE INJECTING  ----------
D2GAME.dll+F6BF: 6A 10           - push 10
D2GAME.dll+F6C1: 53              - push ebx
D2GAME.dll+F6C2: E8 97 AF FF FF  - call D2GAME.dll+A65E
D2GAME.dll+F6C7: 6A 00           - push 00
D2GAME.dll+F6C9: 6A 00           - push 00
D2GAME.dll+F6CB: 6A 00           - push 00
D2GAME.dll+F6CD: 53              - push ebx
D2GAME.dll+F6CE: 6A 00           - push 00
D2GAME.dll+F6D0: 6A 03           - push 03
D2GAME.dll+F6D2: E8 F9 B2 FF FF  - call D2GAME.dll+A9D0
}

{
// ORIGINAL CODE - INJECTION POINT: D2GAME.Ordinal10013+2282

D2GAME.Ordinal10013+2261: 8B 44 24 0C        - mov eax,[esp+0C]
D2GAME.Ordinal10013+2265: 8B 4C 24 24        - mov ecx,[esp+24]
D2GAME.Ordinal10013+2269: 40                 - inc eax
D2GAME.Ordinal10013+226A: 3B C1              - cmp eax,ecx
D2GAME.Ordinal10013+226C: 89 44 24 0C        - mov [esp+0C],eax
D2GAME.Ordinal10013+2270: 0F 8C FA FE FF FF  - jl D2GAME.Ordinal10013+2170
D2GAME.Ordinal10013+2276: 85 DB              - test ebx,ebx
D2GAME.Ordinal10013+2278: 75 08              - jne D2GAME.Ordinal10013+2282
D2GAME.Ordinal10013+227A: 85 ED              - test ebp,ebp
D2GAME.Ordinal10013+227C: 0F 84 9A 00 00 00  - je D2GAME.Ordinal10013+231C
// ---------- INJECTING HERE ----------
D2GAME.Ordinal10013+2282: 6A 00              - push 00
// ---------- DONE INJECTING  ----------
D2GAME.Ordinal10013+2284: 6A 10              - push 10
D2GAME.Ordinal10013+2286: 56                 - push esi
D2GAME.Ordinal10013+2287: E8 12 52 FF FF     - call D2GAME.dll+A65E
D2GAME.Ordinal10013+228C: 33 FF              - xor edi,edi
D2GAME.Ordinal10013+228E: 8B FF              - mov edi,edi
D2GAME.Ordinal10013+2290: 57                 - push edi
D2GAME.Ordinal10013+2291: 56                 - push esi
D2GAME.Ordinal10013+2292: E8 6D 55 FF FF     - call D2GAME.dll+A9C4
D2GAME.Ordinal10013+2297: 0F B7 C0           - movzx eax,ax
D2GAME.Ordinal10013+229A: 50                 - push eax
}

{
// ORIGINAL CODE - INJECTION POINT: D2GAME.dll+104C0

D2GAME.dll+104AF: 66 85 ED        - test bp,bp
D2GAME.dll+104B2: 75 0A           - jne D2GAME.dll+104BE
D2GAME.dll+104B4: 5F              - pop edi
D2GAME.dll+104B5: 5E              - pop esi
D2GAME.dll+104B6: 5D              - pop ebp
D2GAME.dll+104B7: 33 C0           - xor eax,eax
D2GAME.dll+104B9: 5B              - pop ebx
D2GAME.dll+104BA: 59              - pop ecx
D2GAME.dll+104BB: C2 08 00        - ret 0008
D2GAME.dll+104BE: 8B 06           - mov eax,[esi]
// ---------- INJECTING HERE ----------
D2GAME.dll+104C0: 6A 00           - push 00
// ---------- DONE INJECTING  ----------
D2GAME.dll+104C2: 6A 10           - push 10
D2GAME.dll+104C4: 50              - push eax
D2GAME.dll+104C5: E8 94 A1 FF FF  - call D2GAME.dll+A65E
D2GAME.dll+104CA: 8B 4C 24 1C     - mov ecx,[esp+1C]
D2GAME.dll+104CE: 8B 06           - mov eax,[esi]
D2GAME.dll+104D0: 51              - push ecx
D2GAME.dll+104D1: E8 0A 39 00 00  - call D2GAME.Ordinal10013+C20
D2GAME.dll+104D6: 5F              - pop edi
D2GAME.dll+104D7: 5E              - pop esi
D2GAME.dll+104D8: 5D              - pop ebp
}

```
