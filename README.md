# Harvest Moon DS (EU Rev 1.1) - Memory Research

**Game Version:** Harvest Moon DS (Europe)  
**Platform:** Nintendo DS (NDS)  
**Revision:** 1.1  
**SHA-256:** 846ca75ca86781edd46b0d7891d5d73bb87830712ba1c992c5916ea2ac77d9f5

---

## 1. Overview  
I just wanted to marry the Witch Princess...  
This repository documents the memory addresses I have researched as well as some Action Replay codes I've made, specifically for the **European (EU) version** of Harvest Moon DS. Most existing documentation and codes covers both US versions, but close to nothing for EU. Provided memory addresses are EU (Rev 1.1) unless specified.

### Acknowledgements  
The original HMDS Technical Reference Guide folks have already researched and explained most of the stuff. The only thing I'm really doing is just providing additional info on the EU memory locations.  
[https://www.speedrun.com/hmds/guides/retgt](https://www.speedrun.com/hmds/guides/retgt)

### Global Memory Offsets
| Region | Base Shift | Notes |
| :--- | :--- | :--- |
| **US (NA 1.0/1.1)** | `0x00` | Reference Baseline |
| **EU (Rev 1.1)** | `- 0x60-0x64` | Most addresses are shifted back by roughly 96-100 bytes. |

---

## 2. Mining Memory Map

| Description | Address (EU) | Type | Notes |
| :--- | :--- | :--- | :--- |
| **Current Mine Floor** | `0x023DBCCA` | `uint16` | Safe to write to. When going down a staircase, the game takes this floor and +1 to it. |

---

## 3. NPC Data & Structure

The game stores NPC data in a contiguous array of structs. Each NPC block is exactly **40 bytes** (`0x28`) long.  

* **Base Address:** `0x023DBCCC` (NPC ID 0 starts here)
* **Stride:** `40` bytes (`0x28`)

### NPC Data Structure
```cpp
// C++ Representation of the NPC Memory Block
struct NPC_Data {
    uint8_t currentLoc;      // +0x00: Current Map ID (Player only?)
    uint8_t entranceID;      // +0x01: Entrance ID used
    uint8_t previousLoc;     // +0x02: Previous Map ID
    uint8_t exitID;          // +0x03: Exit ID used
    uint8_t friendship;      // +0x04: FP (0-255)
    uint16_t interaction;    // +0x05: Interaction Flags (Talked to, Gifted...)
    uint8_t _pad1;           // +0x07: Padding/Unknown
    uint16_t lovePoints;     // +0x08: LP / AP (0 - 65,535)
    uint8_t _unknown[10];    // +0x0A: Unknown Data
    uint64_t pathData;       // +0x14: NPC Pathing Information
    uint16_t pathWait;       // +0x1C: Timer waiting for next path action
    uint16_t _unknown2;      // +0x1E: Unknown
    uint16_t collisionWait;  // +0x20: Timer when blocked by player
    uint8_t _padding[6];     // +0x22: Padding to align to 40 bytes
};
```

### NPC ID List
Calculate specific addresses using: `Base (0x023DBCCC) + (ID * 0x28)`.  
| ID | Name | ID | Name | ID | Name | ID | Name |
|:---|:---|:---|:---|:---|:---|:---|:---|
| 0 | Player Character* | 1 | Celia | 2 | Muffy | 3 | Nami |
| 4 | Romana | 5 | Sebastian | 6 | Lumina | 7 | Wally |
| 8 | Chris | 9 | Grant | 10 | Kate | 11 | Hugh |
| 12 | Carter | 13 | Flora | 14 | Vesta | 15 | Marlin |
| 16 | Ruby | 17 | Rock | 18 | Dr. Hardy | 19 | Galen |
| 20 | Nina | 21 | Daryl | 22 | Cody | 23 | Gustafa |
| 24 | Griffin | 25 | Vans | 26 | Kassey | 27 | Patrick |
| 28 | Murrey | 29 | Takakura | 30 | | 31 | |
| 32 | | 33 | | 34 | | 35 | |
| 36 | | 37 | | 38 | | 39 | Kai |
| 40 | | 41 | | 42 | | 43 | |
| 44 | | 45 | Harvest Goddess | 46 | Thomas | 47 | Gotz |
| 48 | | 49 | Leia | 50 | Keira | 51 | Witch Princess |
| 52 | † (sprites) | | | | | | |

**Example Calculations**  
Flora (ID 13)  
Base: 0x023DBCCC  
Offset: 13 * 40 = 520 (0x208)  
Result: 0x023DBCCC + 0x208 = 0x023DBED4 (Start of Flora)  
Love Points (Offset +0x08): 0x023DBEDC

---

## 4. Tools and Experience
The tool system separates the "EXP Bar" from "Level".  
| Tool | EXP Address (uint16) | Level Address (uint16) |
|:---|:---|:---|
| Milker | 0x023DDED2 | 0x023DDEB4 |

> Note: Address 0x027E326C contains the displayed Milker level. Modifying this value has no effect on gameplay logic.

> Note: Above hypothesis regarding tool addresses is partially wrong for "Level Address".  
> Newly observed behaviour:
> When the milker levels up, the byte at 0x23DDEB4 increments by 4. At level 64, it increments the byte at 0x23DDEB5 by 1.
> As for the shear, when the shear levels up, it increments by 2. At shear level 99, 0x23DDEB5 value would be C6. But if milker level is >= 64, value would be C7.
> The precise code to modify both to level 99 would be 123DDEB4 0000C78E


### Experience Logic
The **EXP Address** (`0x023DDED2`) is a standard 16-bit integer (Little Endian).  
**Byte 1 (LSB):** Sub-level experience (0-255). Increments by `100` (`0x64`) per use.  
**Byte 2 (MSB):** "Internal Level". When LSB overflows, this increments. When this overflows `0xFF`, the *Level Address* updates (this is the value that actually affects your tool).  
The **Level Address** (`0x023DDEB4`) does **not** store the level number (e.g., 1, 2, 3). Instead, the number stored at this address is `(level * 4) + 2`. If anybody knows more about NDS game programming, please teach me. I would love to understand this code/optimization.   

---

## 5. Some Action Replay Codes
Flora AP max  
`123DBEDC 0000FFFF`

Select + Start to set Mine Level to 254. Going down one staircase brings you to 255.  
Change 0000XXXX in the second line of code to your desired destination.
```
94000130 000003F3  
123DBCCA 000000FE
D2000000 00000000
```

Milker and Shears Max Level (99)  
`123DDEB4 0000C78C`

Milker +1 Level Every Use  
`123DDED2 0000FFFF`

