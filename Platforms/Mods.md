# Mods

- [Cheat Engine](#cheatengine)
- [Xbox 360](#xbox360)
- [Hex Info](#hexinfo)

## <a name="cheatengine"></a>Cheat Engine

The save data is structured the exact same way on Xbox 360, so you can use all of the information below to edit the save data.

Here are the steps to start modifying the save data:
- Step 1. Find the character data through cheat engine, first select castle.exe in the process menu
- Step 2. Click on "Memory View" under the scan list
- Step 3. Use ctrl + f or click "Search" then "Find memory" to bring up the memory search box
- Step 4. Select (Array of) byte then input your character data in hexidecimal form
- Step 4 example: If my character has maxed out stats and has completed normal mode, I would search: 19 19 19 19 FF FF 01 (first 4 arrays are character stats, then the next 3 arrays are level progress)
- Step 5. If done correctly, the bottom half of the memory viewer will display your save data, which you can edit from the main menu using the information below.

## <a name="xbox360"></a>Xbox 360

The requirements are:
- Xbox 360 Transfer Cable/XSATA/XPORT/USB (Hardware)
- Xplorer360/Xport360/Horizon/Modio/Velocity (Pick a Software)
- Profile with Castle Crashers save data
- Hex Editor (HxD is great)
- Rehasher and resigner
Alternativly, if you have an RGH/JTAG/Dev you can FTP the save data to/from the console

How to mod your Xbox profile data using a hex editor 

Part 1: Getting your profile onto your computer from your Xbox360 
- Tutorial for using a USB and Horizon which I find easiest for non RGH/JTAG users: https://youtu.be/0N90fDUcSCA?t=15

Part 2: Hex editing the profile data (I'm using HxD for the hex editing instructions)
- Step 1. Use ctrl + f or click "Search" then "Find" to bring up the memory search box
- Step 2. Select "Hex-values" then input your character data in hexidecimal form
- Step 2. example: If my character has maxed out stats and has completed normal mode, I would search: 19 19 19 19 FF FF 01 (first 4 arrays are character stats, then the next 3 arrays are level progress)
- Step 3. If done correctly, the bottom half of the memory viewer will display your save data, which you can edit from the main menu using the information below.

Part 3: Rehash and Resign (Using Horizon)
- Step 1. At the top of Horizon, click on "Quick Fix"
- Step 2. Select your profile, then click "Ok"

Part 4: Moving the profile back to Xbox (Using USB and Horizon)
- Step 1. Drag and drop the profile into Horizon
- Step 2. Click "Save to Device" and select your USB
- Step 3. It should ask you to overwrite, select Yes
- Step 4. Eject the USB from your computer (Windows 10 or 11 tutorial: https://youtu.be/IX7DLzFMLpk?t=67)
- Step 5. Plug the USB into your console, then move the profile back to the device you want it on

That is it for modding it on Xbox 360, if you want to transfer that data to your Xbox One do this:
- Step 1. Go to "Player Statistics" while in Castle Crashers
- Step 2. Upload data to Xbox One
- Step 3. Go on the same profile on your Xbox One
- Step 4. Go in Castle Crashers Remastered, then open the Player Statistics menu
- Step 5. Download data from Xbox 360

## <a name="hexinfo"></a>Hex Info

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

Specific values for weapons or pets, go to this link and use the Hex values: https://docs.google.com/spreadsheets/d/18odK4GIBrYB85iHiTFYLg1QZPojBO3gHNoS2s9FbLeQ/edit?usp=sharing

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
