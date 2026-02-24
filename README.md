# Rogue Mega Evolution Implementation

This system allows for **Wild Boss Encounters** where a Pokémon can Mega Evolve without a Trainer or a Mega Ring. It uses a specific flag to bypass standard engine restrictions while keeping the player's Mega Evolution requirements intact.

---


https://github.com/user-attachments/assets/bae476c4-e821-4f31-9e3a-d8ae88f6e029



## 1. Requirements & Core Logic
* **Flag Trigger**: Controlled by **Flag 0x950**. When this flag is **ON**, the wild opponent is permitted to Mega Evolve.
* **Wild Only**: The logic explicitly checks for `B_SIDE_OPPONENT` and `!BATTLE_TYPE_TRAINER` to ensure the player cannot Mega Evolve without a Keystone.
* **Held Item**: The wild Pokémon must hold the correct Mega Stone (e.g., Mewtwo holding Mewtwonite X).

---

## 2. C Code Modifications (`mega.c`)

### Enabling the Evolution
In `MegaEvolutionEnabled`, add a check for the flag and the opponent side. This tells the engine that the "Keystone" requirement is satisfied by the environment:

```c
bool8 MegaEvolutionEnabled(u8 bank)
{
    if (gBattleTypeFlags & (BATTLE_TYPE_LINK))
        return TRUE;

    // Allow Rogue Megas for the opponent side only
    if (FlagGet(0x950) && SIDE(bank) == B_SIDE_OPPONENT)
        return TRUE;

    if (FindBankKeystone(bank) == ITEM_NONE)
        return FALSE;

    return TRUE;
}
```
### Redirecting to a Custom Battle Script

In `DoMegaEvolution`, check for the flag and redirect to a custom battle script while skipping standard Keystone and item buffer preparation:

```c
const u8* DoMegaEvolution(u8 bank)
{
    // ... standard evolution lookups ...

    if (evolutions != NULL)
    {
        // ... standard form change logic ...

        PREPARE_MON_NICK_BUFFER(gBattleTextBuff1, bank, gBattlerPartyIndexes[bank]);

        if (FlagGet(0x950) && SIDE(bank) == B_SIDE_OPPONENT)
        {
            PREPARE_SPECIES_BUFFER(gBattleTextBuff3, SPECIES(bank));
            return BattleScript_RougeMega; // Custom Rogue Script
        }

        // ... standard Keystone/Item buffers and return ...
    }

    return NULL;
}

C
extern u8 BattleScript_RougeMega[];
```
---

### 3. Assembly & String Implementation
To avoid text errors like character offset mismatches, the strings must be defined according to your specific character table.

**Rogue Reacting String:** `[BUFFER][00] is reacting to the surrounding Mega energy!`

```assembly
MegaReactingRogue: .byte 0xFD, 0x00, 0x00, 0xDD, 0xE7, 0x00, 0xE6, 0xD9, 0xD5, 0xD7, 0xE8, 0xDD, 0xE2, 0xDB, 0x00, 0xE8, 0xE3, 0x00, 0xE8, 0xDC, 0xD9, 0xFE, 0xE7, 0xE9, 0xE6, 0xE6, 0xE3, 0xE9, 0xE2, 0xD8, 0xDD, 0xE2, 0xDB, 0x00, 0xC7, 0xD9, 0xDB, 0xD5, 0x00, 0xD9, 0xE2, 0xD9, 0xE6, 0xDB, 0xED, 0xAB, 0xFF

```

---

### 4. Header Declaration
Add this to `include/new/mega_battle_scripts.h` to make the script visible to the C compiler and prevent "undefined reference" errors:

```c
extern u8 BattleScript_RougeMega[];
```

### Links:
https://github.com/Shiny-Miner/CFRU-expansion
