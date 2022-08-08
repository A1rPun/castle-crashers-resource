# Mods

- [Cheatengine](#cheatengine)
- [xbox 360](#xbox360)

## <a name="cheatengine"></a>Cheatengine

The save data is structured the exact same way on Xbox 360, so you can use all of the information below to edit the save data.

Here are the steps to start modifying the save data:
- Step 1. Find the character data through cheat engine, first select castle.exe in the process menu
- Step 2. Click on "Memory View" under the scan list
- Step 3. Use ctrl + f or click "Search" then "Find memory" to bring up the memory search box
- Step 4. Select (Array of) byte then input your character data in hexidecimal form
- Step 4 example: If my character has maxed out stats and has completed normal mode, I would search: 19 19 19 19 FF FF 01 (first 4 arrays are character stats, then the next 3 arrays are level progress)
- Step 5. If done correctly, the bottom half of the memory viewer will display your save data, which you can edit from the main menu using the information below.

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

Locating: Data starts before Green Knight's data, the first character in the save. This includes weapons, pets & iventory.
- 00 00 00 00 00 00 00 00 - First 4 arrays are pets, next 2 are unlockable items (includes hidden ones such as mittens) then 2 arrays of padding. (set all to FF to unlock all)
- 00 00 00 00 00 00 00 00 - Weapons set 1 (set all to FF to unlock all)
- 00 00 00 00 00 00 00 00 - Weapons set 2 (set all to FF to unlock all)
- 00 00 00 00 00 00 00 00 
- 00 00 00 00 00 00 00 00 
- 00 00 00 00 00 00 00 00 
- 00 00 00 00 00 00 00 00
- (then Green Knight's data starts here)
- 80 62 00 00 FF FF 40 11 E.T.C.

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
