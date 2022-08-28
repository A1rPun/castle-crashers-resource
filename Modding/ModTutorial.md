# Mod Tutorial

- [Requirements](#requirements)
- [Creating a simple mod from scratch](#tutorial)
- [Advanced mods](#advanced)

## <a name="requirements"></a>Requirements

- Extracted game files, these contain assets and code.
- JPEXS Decompiler, to view assets and code.
- Cheatengine, to run created mods/scripts.
- Any text editor, to view/edit code and scripts.
- MAIN.SWF's Frame 2 code, somewhat readable form of the main code.
- MAIN.SWF's Frame 2 P-code, to extract the ConstantPool and look up bytes.

## <a name="tutorial"></a>Creating a simple mod from scratch

Let's say we want to create a mod that gets us infinite health in game. Where do I start?

#### Finding the code

We need to understand how the game handles health, for this we need to look at MAIN.SWF's Frame 2 code.
Now that we have the code open we have to look for the line that decreases health when damaged. This process takes a lot of code reading and understanding how Tom Fulps brain was structured when he made this.

After a bit of searching you come to the conclusion that in all places where the health should be reduced the function `f_Damage()` is called with a few parameters. So our goal is now to modify some code in `f_Damage` so that in all places where this function is called we have control over it.

Search for `function f_Damage(` to go the the definition of the function. In this large function we can see a line that we are looking for:

```
zone.health = zone.health - damage_pow;
```

`zone` is a parameter of the f_Damage function, in this case it means the player
`.health` is a property of that player
`damage_pow` is a local variable that holds the damage calculation

#### Translating the code

For the sake of this mod we want to remove this line of code because we don't want any health reduction. How are we going to remove this line with CheatEngine?

For this we need a unique byte representation of code, we can look up the bytes for `f_Damage` in MAIN.SWF's Frame 2 P-code. For example this is a unique array of bytes that will point to the f_Damage function in memory:

```
8E 51 00 66 5F 44 61 6D 61 67 65 00 06 00 15
```

To validate an array of bytes is unique you can search for the array of bytes in CheatEngine's memory view.
You can also search the string in MAIN.SWF's Frame 2 P-code but this isn't too reliable.

With these bytes we can now "aobscan" and find the corresponding address in memory. All we need to know now is what bytes represent the entire line of code `zone.health = zone.health - damage_pow;` and what offset these bytes have compared to the f_Damage function.

As we can see in MAIN.SWF's Frame 2 P-code the bytes we want to replace look like this:

```
96 0A 00 04 03 09 76 01 04 03 09 76 01 4E 96 02 00 04 0C 0B 4F
```

Replacing it with "nothing" is simple, we just have to make `00`'s of it.

There are `2836` bytes between the start of f_Damage and the start of the line `zone.health = zone.health - damage_pow;`. This means we can calculate the byte offset for use in the script. For now we just use the hexadecimal representation of the number 2836 which is `B14`.

#### Creating a CheatEngine script

Now we have all the information to craft a CheatEngine script. This tutorial won't go into detail on how to create CheatEngine scripts but here is the script for this tutorial, I annotated it with comments (//):

```
[ENABLE]
// Scans the memory for a unique array of bytes and gives the address the name "fDamage"
aobscanregion(fDamage,30000000,40000000,8E 51 00 66 5F 44 61 6D 61 67 65 00 06 00 15)
// Create variable
registersymbol(health)

// Variable definition
label(health)
// Using the address+offset definition to tell CE what bytes need to be replaced
fDamage+B14:
// The replacement code, the length should be the same as the original bytes
health:
 db 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

[DISABLE]
// Original bytes
health:
 db 96 0A 00 04 03 09 76 01 04 03 09 76 01 4E 96 02 00 04 0C 0B 4F

// Destroy variable
unregistersymbol(health)
```

## <a name="advanced"></a>Advanced mods

- Make a copy of any .SWF file to use as edit fodder
- Go to frame 1 or 2, a frame that has a ConstantPool
- Delete the code that was already there
- Copy a function you want to edit into JPEXS ActionScript window
- Save after you are done
- Edit the P-code
- Highlight the (auto-generated) ConstantPool
- Replace the highlighted ConstantPool with the MAIN.SWF ConstantPool
- Save it
- Now you're left with bytes that can be used to make a CE script
