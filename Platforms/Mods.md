# Mods

- [Cheatengine](#cheatengine)
- [xbox 360](#xbox360)

## <a name="cheatengine"></a>Cheatengine

*TODO: details*

## <a name="xbox360"></a>xbox 360

The requirements are:
- Xbox 360 Transfer Cable/XSATA/XPORT/USB (Hardware)
- Xplorer360/Xport360/Horizon/Modio/Velocity (Pick a Software)
- Profile with Castle Crashers save data
- Hex Editor (HxD is great)
- Rehasher and resigner
Alternativly, if you have an RGH/JTAG/Dev you can FTP the save data to/from the console

Save data info:

All Characters Unlockable Items:
- Data starts before Green Knight's data, the first character in the save. This includes weapons, pets & iventory.
00 00 00 00 00 00 00 00 - First 4 arrays are pets, next 2 are unlockable items (includes hidden ones such as mittens) then 2 arrays of padding.
00 00 00 00 00 00 00 00 - Weapons set 1
00 00 00 00 00 00 00 00 - Weapons set 2
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
- (then Green Knight's data starts here)
80 62 00 00 FF FF 40 11 E.T.C.

Individual Character Data
- bool Unlocked – One byte, unlock flag for the character (Locked is 0x00, unlocked is 0x80)
- byte Level – Stores level value, can be edited to 0xFF for maximum (level 256)
- int EXP – int32, stores EXP value
- byte Weapon – Weapon ID
- byte Pet – Pet ID
- byte Strength – Strength value
- byte Defense – Defense value
- byte Magic – Magic Value
- byte Agility – Agility Value
- byte LevelProgress1 – Progress for story on left side of the dock (0xFF Unlocks all)
- byte LevelProgress2 - Progress for story on right side of the dock (0xFF Unlocks all)
- byte LevelProgress3 – Progress for Wizard Castle and Final Battle
- byte Potions – Stores number of potions
- byte Bombs – Stores number of bombs
- byte Sandwiches – Stores number of sandwiches
- byte Collectables – Collectable items such as the Compass and Telescope. Change to 0xFE to unlock all
- int Gold - in32, stores Gold value
- byte EnableInsaneMode – Unlock insane mode. Change to 0x01 to enable
- byte InsaneLevelProgress1 – Progress for story on left side of the dock (0xFF Unlocks all)
- byte InsaneLevelProgress2 - Progress for story on right side of the dock (0xFF Unlocks all)
- byte InsaneLevelProgress3 – Progress for Wizard Castle and Final Battle
- byte Skull - What skull the character has, 0x00 for none, 0x01 for normal, 0x02 for gold
- int Padding – Empty space that is not used, int32
