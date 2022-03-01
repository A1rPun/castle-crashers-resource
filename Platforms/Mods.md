## Xbox

The requirements are:
- Xbox 360 Transfer Cable/XSATA/XPORT (Hardware)
- Xplorer360/ Xport360 (Software)
- Profile with Castle Crashers save data
- Hex Editor
- Rehasher and resigner


bool Unlocked – One byte, unlock flag for the character (Locked is 0x00, unlocked is 0x80)
byte Level – Stores level value, can be edited to 0xFF for maximum (level 256)
int EXP – int32, stores EXP value
byte Weapon – Weapon ID
byte Pet – Pet ID
byte Strength – Strength value
byte Defense – Defense value
byte Magic – Magic Value
byte Agility – Agility Value
byte LevelProgress1 – Progress for story on left side of the dock (0xFF Unlocks all)
byte LevelProgress2 - Progress for story on right side of the dock (0xFF Unlocks all)
byte unk1 – Not known to do anything (yet?)
byte Potions – Stores number of potions
byte Bombs – Stores number of bombs
byte Sandwiches – Stores number of sandwiches
byte Collectables – Collectable items such as the Compass and Telescope. Change to 0xFE to unlock all
byte EnableInsaneMode – Unlock insane mode. Change to 0x01 to enable
int Padding – Empty space that is not used, int32

