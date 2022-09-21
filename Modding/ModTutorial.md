# Mod Tutorial

- [Requirements](#requirements)
- [Creating a simple mod from scratch](#tutorial)
- [Creating an advanced mod from scratch](#advanced)

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

## <a name="advanced"></a>Creating an advanced mod from scratch

1. Make a copy of any .SWF file to use as edit fodder
2. In JPEXS go to scripts and then to frame 1 or 2 (a frame that has a ConstantPool)
3. Edit and delete the ActionScript code that was already there
4. Copy a function you want to edit into the ActionScript window
5. Save after you are done
6. Edit the P-code
7. Highlight the (auto-generated) ConstantPool line
8. Replace the ConstantPool with the MAIN.SWF ConstantPool
9. Save it
10. Now you're left with bytes that can be used to make a CheatEngine script

### Example

_Some of the steps require [this tool](https://a1rpun.github.io/castle-crashers-resource/) to help extract and align bytes._

We start with a mod idea and an empty CheatEngine Table.

**example.CT**

```xml
<?xml version="1.0" encoding="utf-8"?>
<CheatTable CheatEngineTableVersion="34">
  <CheatEntries>
    <CheatEntry>
      <ID>1</ID>
      <Description>"Mod idea"</Description>
      <LastState/>
      <VariableType>Auto Assembler Script</VariableType>
      <AssemblerScript>[ENABLE]
// Description of mod idea
aobscanregion(FixMe,30000000,40000000,00 00)
registersymbol(fixme)

label(fixme)
FixMe+00:
fixme:
 db

[DISABLE]
fixme:
 db

unregistersymbol(fixme)
</AssemblerScript>
    </CheatEntry>
  </CheatEntries>
  <UserdefinedSymbols/>
</CheatTable>
```

#### Step 1, looking up original code

This example tries to modify the `f_PaintClock` function. Let's find `f_PaintClock` in the P-code so we can use the bytes for disabling your script and for length calculations. This what the function looks like in ActionScript:

```
function f_PaintClock(zone)
{
   if(zone.paint_timer > 0)
   {
      if(zone.paint_timer % 3 == 0)
      {
         var _loc2_ = 70 + random(30);
         var _loc3_ = f_FX(zone.x - 15 + random(30),zone.y + zone.body_y - (20 + random(10)),zone.y - 1,"paintswirl2",_loc2_,_loc2_);
         _loc3_.body.body.body.body.gotoAndStop(zone.paint_type);
      }
      zone.paint_timer = zone.paint_timer - 1;
   }
}
```

The function in P-Code looks like this:

<details><summary>Show P-Code</summary><pre><code>
; 8e 1a 00 66 5f 50 61 69 6e 74 43 6c 6f 63 6b 00 01 00 04 2a 00 01 7a 6f 6e 65 00 fd 00
}
loc46506:DefineFunction2 "f_PaintClock" 1 4 false false true false true false true false false 1 "zone"  {
; 96 05 00 04 01 09 d6 05
Push register1 "paint_timer"
; 4e
GetMember
; 96 09 00 06 00 00 00 00 00 00 00 00
Push 0.0
; 67
Greater
; 12
Not
; 9d 02 00 e1 00
If loc46620
; 96 05 00 04 01 09 d6 05
Push register1 "paint_timer"
; 4e
GetMember
; 96 05 00 07 03 00 00 00
Push 3
; 3f
Modulo
; 96 09 00 06 00 00 00 00 00 00 00 00
Push 0.0
; 49
Equals2
; 12
Not
; 9d 02 00 ac 00
If loc46610
; 96 0a 00 07 46 00 00 00 07 1e 00 00 00
Push 70 30
; 30
RandomNumber
; 47
Add2
; 87 01 00 02
StoreRegister 2
; 17
Pop
; 96 0b 00 04 02 04 02 09 d7 05 04 01 08 c9
Push register2 register2 "paintswirl2" register1 "y"
; 4e
GetMember
; 96 05 00 07 01 00 00 00
Push 1
; 0b
Subtract
; 96 04 00 04 01 08 c9
Push register1 "y"
; 4e
GetMember
; 96 04 00 04 01 08 ca
Push register1 "body_y"
; 4e
GetMember
; 47
Add2
; 96 0a 00 07 14 00 00 00 07 0a 00 00 00
Push 20 10
; 30
RandomNumber
; 47
Add2
; 0b
Subtract
; 96 04 00 04 01 08 c8
Push register1 "x"
; 4e
GetMember
; 96 05 00 07 0f 00 00 00
Push 15
; 0b
Subtract
; 96 05 00 07 1e 00 00 00
Push 30
; 30
RandomNumber
; 47
Add2
; 96 08 00 07 06 00 00 00 09 97 01
Push 6 "f_FX"
; 3d
CallFunction
; 87 01 00 03
StoreRegister 3
; 17
Pop
; 96 05 00 04 01 09 d8 05
Push register1 "paint_type"
; 4e
GetMember
; 96 0a 00 07 01 00 00 00 04 03 09 91 01
Push 1 register3 "body"
; 4e
GetMember
; 96 03 00 09 91 01
Push "body"
; 4e
GetMember
; 96 03 00 09 91 01
Push "body"
; 4e
GetMember
; 96 03 00 09 91 01
Push "body"
; 4e
GetMember
; 96 02 00 08 13
Push "gotoAndStop"
; 52
CallMethod
; 17
Pop
</code></pre></details>

The function definition in P-code is usable for scanning because it's **always unique**, it the first 29 bytes in this example:

```
8e 1a 00 66 5f 50 61 69 6e 74 43 6c 6f 63 6b 00 01 00 04 2a 00 01 7a 6f 6e 65 00 fd 00
```

`29` in hex is `1d` so we can use that as offset because we only want to overwrite the body of the function.

Extract the rest the bytes (not the first 29 bytes) and create a so called byte array that we can use in a script.

```
96 05 00 04 01 09 d6 05 4e 96 09 00 06 00 00 00 00 00 00 00 00 67 12 9d 02 00 e1 00 96 05 00 04 01 09 d6 05 4e 96 05 00 07 03 00 00 00 3f 96 09 00 06 00 00 00 00 00 00 00 00 49 12 9d 02 00 ac 00 96 0a 00 07 46 00 00 00 07 1e 00 00 00 30 47 87 01 00 02 17 96 0b 00 04 02 04 02 09 d7 05 04 01 08 c9 4e 96 05 00 07 01 00 00 00 0b 96 04 00 04 01 08 c9 4e 96 04 00 04 01 08 ca 4e 47 96 0a 00 07 14 00 00 00 07 0a 00 00 00 30 47 0b 96 04 00 04 01 08 c8 4e 96 05 00 07 0f 00 00 00 0b 96 05 00 07 1e 00 00 00 30 47 96 08 00 07 06 00 00 00 09 97 01 3d 87 01 00 03 17 96 05 00 04 01 09 d8 05 4e 96 0a 00 07 01 00 00 00 04 03 09 91 01 4e 96 03 00 09 91 01 4e 96 03 00 09 91 01 4e 96 03 00 09 91 01 4e 96 02 00 08 13 52 17
```

#### Step 2, prepare script

Now we can fill in some details in our script:

```xml
<AssemblerScript>[ENABLE]
// Set the priority of all enemies to 4 in f_PaintClock
aobscanregion(fPaintClock,30000000,40000000,8e 1a 00 66 5f 50 61 69 6e 74 43 6c 6f 63 6b 00 01 00 04 2a 00 01 7a 6f 6e 65)
registersymbol(paintClock)

label(paintClock)
fPaintClock+1d:
paintClock:
 db

[DISABLE]
paintClock:
 db 96 05 00 04 01 09 d6 05 4e 96 09 00 06 00 00 00 00 00 00 00 00 67 12 9d 02 00 e1 00 96 05 00 04 01 09 d6 05 4e 96 05 00 07 03 00 00 00 3f 96 09 00 06 00 00 00 00 00 00 00 00 49 12 9d 02 00 ac 00 96 0a 00 07 46 00 00 00 07 1e 00 00 00 30 47 87 01 00 02 17 96 0b 00 04 02 04 02 09 d7 05 04 01 08 c9 4e 96 05 00 07 01 00 00 00 0b 96 04 00 04 01 08 c9 4e 96 04 00 04 01 08 ca 4e 47 96 0a 00 07 14 00 00 00 07 0a 00 00 00 30 47 0b 96 04 00 04 01 08 c8 4e 96 05 00 07 0f 00 00 00 0b 96 05 00 07 1e 00 00 00 30 47 96 08 00 07 06 00 00 00 09 97 01 3d 87 01 00 03 17 96 05 00 04 01 09 d8 05 4e 96 0a 00 07 01 00 00 00 04 03 09 91 01 4e 96 03 00 09 91 01 4e 96 03 00 09 91 01 4e 96 03 00 09 91 01 4e 96 02 00 08 13 52 17

unregistersymbol(paintClock)
</AssemblerScript>
```
Do note that I shortened the aobscanregion byte array with 3 bytes for no reason... it's shorter :).

#### Step 3, finalize script

The only thing missing is the bytes to overwrite the original function and the mod is ready for testing, [use these steps to create the bytes](#advanced).

Use this ActionScript to put in JPEXS as example, it sets the priority of all enemies to 4:

```js
function f_PaintClock(zone)
{
  var i = 1;
  while(i < total_enemies)
  {
    var enemy = loader.game.game["e" + int(i)];
    if(enemy.alive and enemy.priority < 4)
    {
      enemy.priority = 4;
      enemy.chases = false;
    }
    i += 1;
  }
}
```

Use this MAIN.SWF frame 2 ConstantPool for replacing the auto-generated ConstantPool in JPEXS:

<details><summary>Show ConstantPool</summary><pre><code>
ConstantPool "main" "n_state" "b_do_init" "fp_Main" "level" "LOGPush" "s_levelpath" "f_LoadNewLevel" "f_SetMainfp" "g_bExitGame" "GetFlashGlobal" "g_bNoPause" "SetFlashGlobal" "ShowLoadIcon" "GetGameMode" "p_game" "p" "f_SavePlayerInfoToHudForLevelChange" "blank" "gotoAndStop" "fader" "f_JumpOut" "f_ResetLevelVars" "f_UnloadLevelClips" "f_RemoveLevelClips" "f_SetLetterbox" "console_version" "exit" "loader" "b_loaded" "" "kills" "portals_active" "kills_goal" "leash_left_x" "leash_right_x" "leash_left_locked" "leash_right_locked" "HideLoadIcon" "f_Init" "player" "wins" "losses" "winner" "SetScores" "g_nPort1CharId" "g_nPort2CharId" "g_nPort3CharId" "g_nPort4CharId" "g_bStartedArena" "g_bStartedQuaff" "g_nArenaPoints" "u_point" "Object" "gravity" "g_dash_timer" "PI" "players" "total_enemies" "total_towers" "total_animals" "total_horses" "total_chickens" "total_corpses" "total_fx" "extra_fx" "total_boss" "gore" "GetGore" "spawn_portal_num" "num_portals" "portal1" "portal2" "portal3" "portal4" "portal5" "portal6" "portal7" "portal8" "portal9" "DMG_MELEE" "DMG_POISON" "DMG_ELEC" "DMG_ICE" "DMG_FIRE" "DMG_OBJECT" "DMG_MAGIC" "DMGFLAG_NONE" "DMGFLAG_JUGGLE" "DMGFLAG_STUN" "DMGFLAG_NORESIST" "DMGFLAG_BLOODY" "DMGFLAG_NO_ELEM_EFFECT" "DMGFLAG_SPARKLE_EFFECT" "STUCK_THRESHOLD" "CHEATS" "nowaypoints" "ez_enemies" "GetCheat" "all_combos_unlocked" "Array" "f_InitSaveSystem" "f_InitCharColors" "friendly_fire" "f_FirstTimeInitPlayers" "GetWidescreen" "SCREEN_WIDTH" "IsPalMode" "SCREEN_HEIGHT" "HALF_SCREEN_WIDTH" "HALF_SCREEN_HEIGHT" "ASPECT_RATIO" "GAME_WIDTH" "GAME_HEIGHT" "HALF_GAME_WIDTH" "HALF_GAME_HEIGHT" "GAME_ASPECT_RATIO" "GAME_SCALE" "GAME_X_OFFSET" "GAME_Y_OFFSET" "MAX_ZOOM" "EDGE_MARGIN" "DEFAULT_LEASH_WIDTH" "f_PositionHud" "../sounds/sounds.swf" "sounds" "low" "../lobby/lobby.swf" "f_ChangeLevel" "letterbox_top" "_x" "_y" "letterbox_bot" "_xscale" "_yscale" "cinema_letterbox" "g_bMap" "g_bMenu" "p_cinema_clip" "active" "go_arrow" "current_depth_mod" "total_huds" "combo_count" "combo_timer" "current_shadow" "object_index" "static_index" "stuck_timer" "healthmeter" "boss_fight" "f_ClearUnlockDisplay" "f_CreatePickupArray" "fp_FunctionLine" "fp_FunctionLineProjectile" "grass" "grass_total" "shovelspots" "shovelspots_total" "bombspots" "bombspots_total" "secrets" "secrets_total" "trees" "trees_total" "exits" "exits_total" "ladders" "ladders_total" "ripoffs" "ripoffs_total" "ride_rotation" "AutoRun" "level_dust" "dust_brown" "f_ResetHudGoldCounters" "sky_type" "../sky1/sky1.swf" "sky" "../sky2/sky2.swf" "../sky3/sky3.swf" "../sky4/sky4.swf" "../sky5/sky5.swf" "../sky6/sky6.swf" "fog_type" "../fog/fog.swf" "fog" "current_fx" "extra_fx_current" "f_GetDepthModAssignment" "fx" "invisObject" "attachMovie" "../fx/fx.swf" "depth_mod" "total_shadows" "s" "shadow" "game" "off" "x" "y" "body_y" "warp_timer" "bsp_timer" "getDepth" "../player/player.swf" "p_temp" "e_tower" "../etower/etower.swf" "animal" "../animals/animals.swf" "total_slimes" "e_slime" "../eslime/eslime.swf" "total_imps" "e_imp" "../eimp/eimp.swf" "horse" "../horse/horse.swf" "total_mount2" "mount2" "../mount2/mount2.swf" "total_mount3" "mount3" "../mount3/mount3.swf" "total_mount4" "mount4" "../mount4/mount4.swf" "corpse" "chicken" "../chicken/chicken.swf" "e_human" "e_bodyparts" "static" "u_object" "Math" "abs" "zone" "h" "_height" "w" "_width" "top" "left" "right" "n_height" "bottom" "hitzone" "object" "f_UpdateObject" "depth_y" "f_Depth" "diagonal" "left_x" "right_x" "left_y" "right_y" "h2" "zone2" "w2" "x1" "x2" "f_AddNeutral" "g" "cut" "dug" "bomb_pt_x" "bomb_pt" "bomb_pt_y" "bomb_pt_w" "ripoff" "animals" "f_UnloadSpecificClips" "f_UnloadClips" "total_bigtrolls" "e_trollx" "total_bigtroll" "e_troll" "total_bees" "e_bee" "total_fish" "e_fish" "total_bats" "e_bat" "total_scorpions" "e_scorpion" "total_ufo" "ufo" "ufodust" "total_beetles" "e_beetle" "f_RemoveSpecificClips" "f_RemoveFX" "f_RemoveCorpses" "f_RemoveChickens" "f_RemoveHorses" "f_RemoveAnimals" "f_RemoveTowers" "f_RemoveEnemies" "f_RemovePlayers" "f_RemoveMount2" "scale" "scale_goal" "scale_mod" "speed" "camera_x" "camera_y" "camera_oldx" "camera_oldy" "camera_x_goal" "camera_y_goal" "last_good_player_x" "last_good_player_y" "last_good_player_groundtype" "focus_things" "player_move" "player_x_old" "leash_left_y" "cam_catchup" "cam_catchup_speed" "leash_right_y" "rot" "safe_screen_size" "forced_zoom" "waypoint" "newwaypoint" "lastwaypoint" "n_bottomplayer" "n_topplayer" "bg0_c" "bg1_c" "bg2_c" "bg3_c" "bg4_c" "shake" "shake_intensity" "a_shake" "rotrad" "bumpup" "f_GetCameraCoords" "compensated_camera_y" "edge1" "edge2" "edge3" "max_top" "max_bottom" "max_right" "max_left" "_rotation" "bg0" "bg1" "bg2" "bg3" "game_x" "game_y" "scaled_screen_width" "f_HiFps_CameraReset" "n_width" "playerArrayOb" "p_pt" "onscreen" "onscreenbody" "port" "VibrateController" "active_players" "alive" "hud_pt" "f_UpdateScrollZoom" "hifps_camera_reset" "cam_cathcup" "active_enemies" "enemyArrayOb" "e" "health" "hitground1" "sqrt" "f_GetClosestWaypoint" "max" "min" "HiFps_Reset" "bg" "_c" "f_LocalToGame" "door" "npc" "body_y_mod" "last_good_groundtype" "n_groundtype" "length" "splice" "total_players" "m_zoom" "f_BottomPlayerY" "portal" "open" "f_func" "fp_activate" "target_level" "f_FadeOut" "shadow_pt" "body" "f_BSPHitTest" "f_Damage" "speed_toss_y" "speed_toss_x" "impact1" "f_FX" "f_CallJuggle1" "f_BSPCheckLastHitType" "f_BSPCheckLastHitSlope" "f_MoveCharV" "floor" "jumped" "jumping" "blocking" "hanging" "ladder" "busy" "jump_attack" "speed_jump" "speed_launch" "f_ShadowSize" "jump" "done" "body_table_y" "f_Ripple" "one" "on" "waterbox" "f_Water2MCH" "No Deep Water" "Error" "f_BSPCheckLastHitIndex" "HiFps_ResetRecursive" "climb" "dashing" "dashjump" "f_PlainMCV" "pre_ladder_x" "pre_ladder_y" "human" "upright" "f_Water2MCV" "ladder_slope" "ladder_endpt_top" "pow" "ladder_endpt_bot" "f_DashReset" "f_MoveCharH" "walking" "cframe" "max_cframe" "f_FlipChar" "button_left" "Key" "isDown" "left_timer" "button_right" "right_timer" "button_up" "up_timer" "button_down" "down_timer" "button_jump" "speed_x" "cos" "ripple" "wave" "splash" "water_default" "f_ColorSwap" "f_ProjectileHitWallH" "f_PlainProjectileMove" "f_StairsProjectileMove" "f_InsideProjectileMove" "f_TableProjectileMove" "speed_y" "bounces" "bounces_max" "f_ProjectileMove" "f_ProjectileMoveY" "n_slope" "f_Quantize" "bounds" "Undefined Ground type!" "f_PlayerCheckPickups" "f_SZ_PlayerXOnScreen" "f_SZ_PlayerWithinSafeScreen" "f_HitWallH" "f_PlainMCH" "f_StairsMCH" "f_DeathBoxMCH" "f_Water1MCH" "f_XBPortalThing" "f_InsideMCH" "f_LadderMCH" "f_TableMCH" "f_LargeObjectRanges" "f_SetXY" "u_temp" "hitwall_h" "r" "beefy" "f_SZ_OnScreenBody" "f_SZ_PlayerYOnScreen" "f_SZ_PlayerYDiffMax" "f_HitWallV" "f_InsideMCV" "f_DeathBoxMCV" "f_Water1MCV" "f_LadderMCV" "f_SideLadderMCV" "f_TableMCV" "punching" "sl_top" "sl_bot" "q" "n_type" "sideclimb_up" "f_LedgeProjectile" "cinema" "f_WarpIn" "_parent" "charcolor" "stats" "profile" "available" "last_up" "p_type" "f_HudSetProfile" "vy" "button_start" "pressed_start" "select" "player_num" "next_hud" "hud" "f_GetButtons" "player_type" "color_green" "color_red" "color_blue" "color_orange" "char_colors" "f_SZ_PlayerXOnScreenMax" "f_ResetPlayerInfoAndStats" "ground_slam" "player_pt" "f_SpawnPlayer" "stop" "f_PlayerArray" "fp_StandAnim" "f_SwimStand" "fp_WalkAnim" "f_SwimWalk" "jump_speed_y" "f_JumpAction" "deer" "fp_HorseRide" "f_HorseRide" "spawn" "_currentframe" "pressed_right" "pressed_left" "button_accept" "f_HudPressedSelect" "_visible" "g_bHudHidden" "g_nPort" "State" "was_paused" "f_ClearButtons" "dropped" "f_CheckHealth" "../map/map.swf" "SetPortState" "f_CopyButtons" "f_UpdateOverlay" "slideWait" "hud_start_y" "gold" "goldcounter_targ" "goldchangetimer" "goldcounter" "goldshowtimer" "goldstats" "goldicon" "gotoAndPlay" "store" "goldtext" "SetTextNumeric" "goldbg" "text" "show_gold" "round" "fp_HudTick" "f_HudSlide" "vs_cinema" "wait" "name" "overlay" "nameoverlay" "press_start" "animal_unlocks" "AnimalHandler" "f_UpdateAchievement" "UnlockAvatarItem" "unlock_display" "f_UnlockAnimal" "f_PushUnlockDisplay" "frame" "item" "gamertag" "GetLocalGamerTag" "nurse" "weapon_offset" "f_SetWeaponUnlocked" "save_data_info" "char_offset" "char_size" "WriteStorage" "characterunlock" "f_UnlockItem" "blacksmith" "item_unlocks" " (" ")" "king" "arrow_type" "f_UnlockCombo" "f_DisplayUnlockMagic" "unlockfunc" "unlockzone" "unlockitem" "CharacterColor" "Color" "setTransform" "ra" "ga" "ba" "aa" "rb" "gb" "bb" "ab" "num_characters" "f_NewColor" "color_dark" "s_Swing" "start" "s_MeatyPunch" "s_Cling" "s_FistPunch" "s_Clang" "s_Block" "f_SoundPan" "percent" "u_pan" "setPan" "setVolume" "g_rad" "sin" "g_sin45" "y1" "y2" "x3" "y3" "x4" "y4" "lerp_x" "lerp_y" "g_deg" "f_GetEnemySpawnWP" "f_GetWPX" "f_GetWPY" "f_NewFriendNum" "stand" "insane_mode" "health_max" "attack_pow" "punch_pow_low" "punch_pow_medium" "punch_pow_high" "punch_pow_max" "arrow_pow" "magic_pow" "e_pt" "f_NewEnemy" "combo_unlocks" "f_NewShadow" "tossable" "zapping" "helmet" "face_hit1" "face_hit2" "arrow_speed_y" "arrow_speed_x" "arrow_gravity" "shot_timer_default" "weight" "grab" "lifebar" "punch_clock" "punch_count" "poison_timer" "fire_timer" "rolling" "magician" "magic_delay" "magic_wait" "magic_chain" "magic_air" "magic_splash" "magic_bullet" "magic_timer" "hitby" "weapon_type" "item_type" "fp_DeathAction" "arrowplink" "aggressiveness" "grapple_aggression" "recovery" "resists" "attack_type" "frozen" "fp_PunchHit" "f_EnemyAttack" "fp_CharClock" "f_EnemyClock" "fp_Blocking" "f_EnemyBlocking" "fp_MidAttack" "f_EnemyMelee" "f_WalkType1" "f_StandType1" "fp_Jab" "f_PunchSet100" "fp_DieAction" "f_Null" "fp_hitreaction" "f_HumanoidDefaults" "f_ClearTimers" "retreater" "roller" "anti_air" "blocks" "block_odds" "full_block_odds" "air_block_odds" "chases" "color_default" "e_type" "fp_BlockFunction" "fp_Ranged" "f_KidSettings" "f_InsertEnemy" "f_SetEnemyHealth" "f_SpawnEnemy" "weapon" "emblem" "shield" "fp_StandSettings" "f_NPCMidJump" "fp_Character" "f_DasherWalk" "f_ShootArrow" "fp_GetUpAction" "f_EnemyGetUp" "f_EnemyCheckBlock" "hop_timer" "f_WeaponStats" "f_SetPlayerMagic" "f_BadDude" "f_SetEnemyPow" "random_dash_jump" "random_dash_roll" "dash_timer" "f_ThiefWalk" "archer" "total_trolls" "f_GeneralWalk" "f_Taunt1" "f_PunchSet300" "fp_Fierce" "fp_ExtremeDeath1" "fp_Hit1" "f_HitSlime" "fp_Hit2" "fp_Hit3" "fp_Juggle" "intro_timer" "humanoid" "dropstuff" "nohit" "f_BeeWalk" "fly_timer" "flying" "fly" "f_EnemyQuickBomb" "f_ShootFire" "weightplus" "f_JumpKickInit" "f_NinjaWarp" "f_Beetle" "f_BeetleAttack" "f_HitBeetle" "arrowhit_function" "mode" "mode_timer" "hit_number" "f_AlienWalk" "f_ShootAlienGun" "f_Fish" "f_FishDive" "f_FishAttack" "f_HitFish" "damage_all" "haswall" "f_SetInWater" "current_speed" "f_UnresponsiveDefaults" "f_HitSiegeTower" "riders" "walk" "f_SpawnBarbarian" "enemy_archer" "vehicle" "rider1" "f_BigTrollWalk" "prey" "f_PickRandomPlayer" "f_Hit1Reaction" "skiptarget" "health_cp" "birth_timer" "attack" "speed_z" "s_SlimeHit" "captor" "hostage" "slime_glob" "hit_function" "f_ShrapnelBounce" "die" "hop" "invincible_timer" "s_Punch3" "prev_StandAnim" "prev_WalkAnim" "prev_Character" "magicmode" "beefy_blobbed" "blobbed" "f_SlimeAttack" "s_SlimeLand" "hasloot" "bagrun" "f_EnemyDie" "loot" "f_ConfirmPickup" "hit1" "total_pickups" "pickupArrayOb" "pickup" "f_OnScreen" "f_PickupPop" "dodge" "f_Imp" "f_Hit2Reaction" "f_Hit3Reaction" "fp_FlipHit" "f_FlipHit" "f_Juggle1" "u_boss" "pet" "state" "lost" "owner" "f_ItemSpawn" "f_EnemyDropWeapon" "animal_type" "f_RandomItemSpawn" "gem_type" "f_RemoveShadow" "f_SetTargets" "fp_SpecialEvent" "fp_SpecialEvent2" "f_SetLeash" "f_CheckEQ" "standing" "f_MagicJump" "magic_jump" "f_AutoJumpInit" "magic_type" "tornado" "fp_MagicMove" "f_ShootMagic" "magic1" "f_MagicBullet" "lastpunch" "punch1_4" "priority" "f_BlockFunction" "EnemyQ_Total" "EnemyQ" "EnemyMax" "EnemyQ_Current" "f_LaunchEnemies" "hit_chain" "beefy_block" "beefy_hit1" "fp_CharacterDefault" "f_BarbarianBossWalk" "f_WalkType2" "f_StandType2" "f_PunchSet201" "f_GetUpPuch201_2" "f_Level7BossDie" "f_BarbBossHit" "enemy_spawn_timer" "player_used_magic" "target_body_y" "../efish/efish.swf" "hit2" "f_RangedWalkInit" "f_EnemyRangeAttack" "f_EnemyWalkInit" "f_EnemyClose" "f_EnemyWalk" "punch1_1" "punch1_3" "../ebeetle/ebeetle.swf" "speed_x_max" "speed_y_max" "punch2_1" "punch2_3" "toss_clock" "f_BlockSound" "block1" "f_PunchSound" "boomerangexploit" "head_x" "head_y" "spawn_timer" "shot_timer" "shot_timer2" "f_HitAntlion" "f_PushCharsOut" "s_AntlionDie" "f_FaceClosestPlayer" "head" "head2" "u_temp2" "general_projectile" "f_Shoot" "projectile_type" "punch_pt" "antlion" "s_Chomp" "../ufo/ufo.swf" "../ufodust/ufodust.swf" "onground" "fp_CheckYSpace" "f_ObjectCheckYSpace" "f_HitUFO" "alien_timer" "stones" "s_Engine" "f_SoundX" "f_SetActiveEnemies" "ufostone" "ufostonedrop" "s_DarkForce" "hit" "f_UFOCheckDead" "destin_x" "thruster" "f_SpawnAlien" "s_DarkMagic1" "impact4" "f_Juggle1Setup" "juggle1" "stone_timer" "beam" "smokefade2" "body1" "body2" "body3" "f_MakeShrapnel" "s_SlamGround" "f_ScreenShake" "burried" "remove" "punch_group" "blocked" "f_DropHostage" "thrown" "f_BeefyGrabEnemy" "f_EnemyWalkToss" "f_BeefyGrapple" "f_BeefyEnemyWalk" "f_BeefyEnemyDie" "f_BeefyHit" "f_BeefyJuggle" "grapple_min" "grapple_mod" "f_SpawnImp" "num_items" "num_animals" "num_levels" "num_relics" "num_items_expansion" "relic_offset" "item_unlocks_expansion" "level_unlocks" "relic_unlocks" "normal_level_unlocks" "insane_level_unlocks" "ReadStorage" "exp" "exp_mod" "exp_next" "exp_start" "default_weapon" "f_GetDefaultWeapon" "strength" "defense" "magic" "agility" "xpgained" "healthpots" "bombs" "beefies" "gold_start" "f_LoadLevelUnlocks" "achievement" "exp_last" "loaded" "IsFullGame" "IsLocalPlayerInSession" "f_SaveLevelUnlocks" "WriteSaveGame" "f_IsArenaUnlocked" "arenaunlock" "select_players" "showselections" "i" "c" "g_nPlayer" "Port" "restore_anim" "f_SetNPCDefaultInfo" "combo_pt" "combo" "pusher" "scroller" "magic_max" "magic_current" "magic_regen" "magic_sustain_pow" "shield_pow" "combo_max" "combo_current" "equippeditem" "arenabattle" "arena_mode" "f_AssignAnimal" "arrow_hit_function" "rage" "rage_goal" "flag" "spawned" "float_timer" "f_StandSettings" "f_Character" "fp_DashAnim" "f_DashType1" "fp_ThrowAction" "f_ThrowEnemy" "f_PunchHit" "f_PlayerClock" "f_JumpAttack" "f_PunchSet1" "f_PunchSet2" "fp_JabFierce" "f_Uppercut" "fp_Wait" "u_punch2_2" "color_grey" "f_NPCWalk" "f_UpdatePlayerAttributes" "weapon_agility" "weapon_magic" "magic_unlocks" "weapon_strength" "weapon_defense" "f_UpdatePlayerAgility" "button_back" "button_select" "button_walk_left" "button_walk_up" "button_walk_right" "button_walk_down" "button_punch1" "button_punch2" "button_projectile" "button_shield" "button_magic" "button_l2" "button_r2" "intro" "punched" "pressed_punch1" "xp" "pic" "picbg" "itembg" "wasdead" "f_ProjectileHitSpark" "wings" "fp_timer" "f_AnimalTimerNull" "f_CardinalTimer" "f_OwletTimer" "f_RammyTimer" "f_FroggletTimer" "f_PolarBearTimer" "timer" "f_BatTimer" "f_TrollPetTimer" "f_PazzoTimer" "f_HawksterTimer" "pause" "f_InstallBallTimer" "f_PelterTimer" "f_DragonTimer" "f_WhaleTimer" "weapon_critical" "weapon_magic_type" "weapon_magic_chance" "weapon_level" "p_target" "goal_x" "goal_y" "carrying" "love_timer" "heart" "f_SetAnimalTimer" "follow" "f_UnAttachAnimal" "unlock" "f_UnlockAnimalAchievement" "ignore_timer" "dist" "launch" "f_GetSomeLove" "f_lerp" "barn_x" "barn_y" "sleep" "bubble" "gotosleep" "sleep_z" "f_AttachAnimal" "angry" "happy" "f_SZ_OnScreenMax" "f_IsAPlayerClose" "wake" "counter" "fp_GotoAction" "sit" "f_AnimalGrabSecretItem" "goto" "dig" "dust_snow" "f_AnimalRamHit" "_scale" "f_FlipSame" "f_AnimalGrabFruit" "f_AnimalDig" "f_Heal" "animalproof" "ram" "ram_intro" "f_InstallBounce" "s_Photon" "photon" "damage_type" "peck" "f_AnimalPeck" "tonguelength" "tonguespeed" "tongue" "f_GetPickup" "maul" "f_AnimalMaul" "bathead" "bat" "f_BatAttack" "KISS" "Traditional" "TheTraitor" "DeerTrainer" "ConscientiousObjector" "MaximumFirepower" "ArenaMaster" "Glork" "MeleeIsBest" "TreasureHunter" "Medic!" "f_ReceivedAllKisses" "UnlockAchievement" "f_BeatAllLevels" "f_WhaleActive" "treefall" "fall" "goldshoot" "falling" "onfire" "damage_chain" "current_weight" "root" "f_PunchReset" "horse_move" "spinning" "sheathing" "grappler" "s_Ground3" "f_KnockOff" "throwmove" "f_ClearHostage" "f_LoseHorse" "rider2" "rider3" "rider4" "skipjuggle" "beefy_hitground1" "collide" "punch" "punch_function" "hitwall" "f_Collide" "f_KidHitKids" "f_BloodShrapnel2" "explode" "s_Explosion6" "f_Explosion" "f_StaticRange" "f_SpawnMask" "big_splash" "s_Ground6" "s3" "level_shockwave" "s_Ground4" "s_Ground5" "bounce2" "bounce1" "missilemode" "f_Jump" "impact_block" "s_Swing4" "land" "f_MagicBulletDown" "magic_air_down" "mount_type" "punch_num" "punch11_2" "punch11_1" "autojump" "splashattack" "unblockable" "trail_function" "victim" "nospin" "stunned" "lightning_strike" "smoking_timer" "u_zone" "lightning_timer" "inert" "f_ProjectileHitGeneral" "impact2" "hit_range" "gate" "arrow_pt" "hitleft" "hitbydamage" "fx_x" "fx_y" "fx_body_y" "uniquehit" "fp_UniqueHit" "no_wall_damage" "bod_y" "f_TeamCheck" "throw_source_y" "throw_source_x" "throw_x" "s_PoisonCloud" "elemental_poison" "s_MagicLightning" "elemental_lightning" "s_ShootIce" "elemental_ice" "s_ShootFire" "elemental_fire" "hit_impact" "f_SwordClang" "f_HardPunchSound" "s_Smack1" "f_SetFloat" "hit_y" "f_MakeBodyPart" "stomping" "hitnohit" "hit_x" "block_timer" "block2" "block_clock" "nofight" "f_Hit1" "f_Hit2" "f_Hit3" "poison_owner" "fire_owner" "force_y" "force_x" "impact_type" "f_BloodBathShrapnel" "spark_timer" "spark_owner" "frog_tongue" "temp_attack_type" "rider" "f_MeleeCheckHit" "angle" "g_sin30" "s_GrassCut" "f_MeleeImpactSound" "speed_slam" "end_attack" "s_Splash1" "s_Splash2" "temple_splash" "specialmode" "topspin" "generaljump" "chasejump" "f_Magic" "wings_subframe" "s_SplashOut" "s2" "f_TopSpinInit" "hold" "end" "beefy_blocking" "magicshield_end" "button_sheath" "sheathed" "weapon_equipped" "unsheath" "sheath" "gobeefy" "pressed_magic" "magicclock" "freeze" "f_MagicMode" "f_Walk" "f_Jumping" "f_Punch" "f_CharacterSlide" "buzzsaw" "f_BuzzSaw" "punched2" "slamdown" "uniqueride" "anim_ride" "f_AlignBody" "f_Block" "reverse" "retreat" "dashing_timer" "truespeed" "f_BeefyBlock" "f_BeefyThrow" "wall_x" "wall_y" "f_HitWallGroupH" "active_walls" "wallArrayOb" "wall" "f_HitWallGroupV" "arrow" "f_ProjectileHitStick" "damage_pow" "* TO MANY DEPTH MODS!!!" "total_gold_collected" "damage_val" "color_gold" "f_ShowVal" "toss" "toss_speed_x" "f_IsWeaponUnlocked" "IsSameProfile" "getrelic" "critical" "outline" "ten" "hundred" "kid_pointer" "ready" "f_UpdateHUD" "f_Exp" "f_ElementalPunch" "f_IceShatter" "stone" "f_StoneHeart" "color_dark_green" "zapped" "idle" "pink_stun" "stun" "blood_timer" "sparklefx_timer" "u_digit" "f_CheckHealthPots" "invincible_recover" "got_hit" "limit_bottomright" "limit_topleft" "right_last" "right_last2" "left_last" "left_last2" "cpr" "thrusting" "prev_fp_UniqueHit" "f_FlipInverse" "throw" "f_BeefyCarryEnemy" "f_StandTypeBeefyHostage" "f_WalkTypeBeefyHostage" "f_CharacterBeefyCarry" "f_BeefyThrowAction" "beefy_grab" "shieldbash" "s_Swing3" "uppercut" "punch200_1" "dashpunch" "s_Swing2" "s_Swing5" "punch6_3" "punch6_4" "punch1_2" "f_PunchInit" "magic0" "magic2" "pink_cast" "punch2_2b" "punch2_2" "dashslam" "f_StompRange" "stomp" "punch5_1" "punch6_1" "punch4_4" "punch5_2" "punch100_2" "punch100_3" "punch100_1" "punch100_" "pull" "propeller_timer" "propeller" "punch200_2" "punch200_3" "punch200_4" "beefy_dashpunch" "punch201_2" "punch201_3" "punch201_4" "punch201_1" "beefy_slam" "p1" "fire_fx2" "f_RandomOverlay" "fire" "temp_flame" "smokefade" "f_SmokeFade" "paint_timer" "paintswirl2" "paint_type" "sparkleblast_fx" "beefy_timer" "beefycount" "beefy_smoke_timer" "f_GoDefault" "beefysmoke" "poison_fx2" "poison_fx4" "poison_fx1" "poison_fx3" "glowset" "f_EnemyMagicClock" "f_PoisonClock" "f_FireClock" "f_BloodClock" "f_SmokingClock" "f_SparkClock" "f_SparkleEffectClock" "spark" "f_ShrapnelVanish" "stonewall_shrapnel" "f_RubbleBounce" "door_shrapnel" "s_Explosion3" "s_Explosion4" "area_fire" "body4" "trail_timer" "cannonball_bomb" "f_CannonBallBombExplode" "crosshair" "explosion3" "pillar18" "table" "table2" "skull" "ribs" "armbone" "legbone" "smallgib1" "smallgib2" "smallgib3" "smallgib4" "blood_drop" "noblood" "general_shrapnel" "shrapnel_type" "embersmoke" "ember_shrapnel" "f_EmberBounce" "hatty_gem_shards" "f_IceBounce" "ice_shrapnel" "f_EnemyWalkingAwayLeft" "f_EnemyWalkingAwayRight" "runscared" "s_IceShatter" "f_IceShrapnel" "plank_shrapnel" "mushroom1" "mushroom2" "stoneheart" "deadbee" "f_ProjectileHitBoomerang" "directionaless_blocking" "hitdown" "s_ArrowHit" "body_r" "arrow_stuck" "s_arrowHit" "penguin_explode" "impact_general" "impact_gem" "f_ExplosionCheckHit" "f_MagicExplosionCheckHit" "f_StompCheckHit" "smokeblast" "poison_trail" "spit_trail" "lightning_trail" "boomerangtrail" "penguintrail" "hattyteartrail" "daggertrail" "cyclopsfiretrail" "trail_interval" "generaltrail" "blackmagictrail" "explosion1" "photon_hit" "s_PhotonHit" "spit_hit" "acid_splash" "f_ProjectileHit" "speed_return" "prev_y" "f_ObjectCheckPickups" "boomeranged" "boomerang" "spit" "area_frogburp" "f_utilNormalizeScale" "f_utilLerp" "current_chain" "area_rainbow" "s_AreaPoison" "area_poison1" "f_AreaGeneral" "f_ShootLightning" "s_AreaIce" "area_ice" "s_AreaFire" "area_fire1" "s_AreaBarb" "area_barb" "s_AreaSawblade" "area_buzzsaw" "area_bees" "s_AreaKing" "area_heal" "s_AreaEarth" "area_earth" "s_AreaNecro" "area_skeleton" "area_hellfire" "s_AreaDark" "area_blackmagic" "s_AreaNinja" "area_ninjasmoke" "s_PinkSplashMagic" "f_AreaRainbow" "s_PurpleSplashMagic" "f_FrogMagicBurp" "s_HattySplashMagic" "area_whale" "area_golddrop" "s_AreaThief" "area_arrows" "s_MagicWind" "saracenwind" "hatty_tears" "shoot" "f_LightningRelease" "beefy_die" "vs_fight" "itemselect" "icon2" "icon" "removechar_mod" "swapDepths" "removeMovieClip" "hit_front" "has_shadow" "closed" "f_HitDummy" "f_ArrowHitDummy" "treasurechest" "f_UniqueTrue" "f_HitGeneral" "f_HumanoidCheckYSpace" "f_PlankShrapnel" "barrel3" "f_BarrelShrapnel" "barrel2" "barrel1" "f_HitBarrel" "chair_shrapnel2" "chair_shrapnel3" "f_HitRoundTable" "f_HitTombstone" "current" "boss" "s1" "s_Glass1" "f_HitGenerator" "hay1" "hay2" "hay3" "hay4" "hay5" "f_HitHayCart" "purplerock2" "purplerock1" "f_HitPurpleTower" "f_RandomTreeItem" "f_HitTree" "chair_shrapnel1" "f_HitRoundTableChair" "f_GeneralBounce" "f_CheckDead" "f_HitGuardTower" "n_health" "f_UnSetDoor" "fp_SmashAction" "n_health_max" "doorsmack" "f_DoorShrapnel" "f_ProjectileHitStickFall" "door_pt" "arrow_fall" "f_HitStoneWall" "f_ArrowHitDoor" "f_HitDoor" "locked" "f_HitPadlockDoor" "f_HitLavarockDoor" "explosion_range_y" "explosion_range_x" "f_MagicClock" "s_LevelUp" "levelup_timer" "go" "levelup" "SetRichPresence" "f_ItemToggle" "pressed_projectile" "f_CheckInsideBSP" "f_SZ_OnScreen" "truespeed_timer" "f_PaintClock" "f_BeefyClock" "mid" "n_top_player" "f_ExtremeDeath1" "nextFrame" "statics_min" "total_statics" "total_objects" "object_pt" "hills_total" "hills" "hillx" "hills_min" "hills_max" "f_ArrayInsert" "f_ArraySplice" "u_size" "s_Pig2" "s_Eat" "s_Pig1" "targ_hud" "pos" "localToGlobal" "globalToLocal" "flytohud" "t" "f_GetGold" "s_Coin1" "s_Coin2" "marker" "f_DisplayWeaponBonus" "s_GetWeapon" "f_GetWeapon" "buttonpress" "horse_punch1_2" "horse_punch1_1" "bite" "horse_bow" "oncatapult" "horse_idle" "f_Wait" "fp_PrevHorseRide" "f_HorseWaitCheck" "escape" "f_HorseAttack" "peaceful" "f_FollowWalkInit" "f_NPCWalking" "f_EnemyHorseMelee" "f_HorseRam" "anim_walk" "jumper" "s_DeerJump" "horsejump" "scorpion" "legs" "u_priority" "orderArray" "u_mod" "hunter" "fp_taskcomplete" "prey_num" "follow_order" "ceil" "guard" "f_GoalWalk" "fp_Goal" "f_Guard" "target_x" "target_y" "wander" "temp_speed_x" "temp_speed_y" "wander_timer" "wait_timer" "f_WaitTimer" "collide_h" "collide_v" "last_x" "last_y" "f_RemovalLeft" "f_RemovePlayer" "f_RemoveNPC" "dash" "f_RemovalRight" "beefy_walk" "hiding_walk" "beefy_stand" "hiding_stand" "beefy_stand_hostage" "hostage_walk" "beefy_walk_hostage" "prev_x" "success_x" "success_y" "return_val" "f_EnemyDashInit" "dash_speed_y" "dash_speed_x" "dash_length" "f_EnemyDashSpeed" "roll" "hop_speed_y" "hop_speed_x" "hops" "f_EnemyHopSpeed" "f_EnemyHopInit" "bomb" "bow" "gunshoot" "north" "f_EnemyDashCheck" "f_EnemyDashing" "f_WalkToInit" "f_EnemyMagician" "f_EnemyHopCheck" "barbarians" "f_SpawnMeleeBarb" "f_EnemyTargetCloseEnemy" "f_BossGrabEnemy" "level7die" "f_KillEnemies" "f_EnemyBeefyGrab" "f_EnemyBeefyThrow" "f_NPCPickPlayer" "f_SlideDust" "shoulderslide" "dash_speed" "f_EnemyDashEnd" "shoulder" "f_EnemyDashRange" "shockwave2" "casted" "pause_timer" "spell_bolt" "pattern" "goleft" "spawnleft" "go_up" "loops" "loop_speed" "f_WraithRise" "tookshot" "f_SZ_PlayerYOnScreenMax" "f_SpawnSkeleton" "cinema_butt2" "s_FireHit" "geyser" "f_FireBurnHit" "f_FireWallBurnHit" "ember_timer" "ember" "bk" "evilsmoke" "introdash" "_start" "walk_timer" "hunted" "idle_timer" "go_timer" "f_LevelUpAttack" "recover_timer" "f_EndHit" "walkto_x" "walkto_y" "WalkToAction" "u_thru" "walkto_speed" "horse_walkto" "beefy_walkthru" "walkthru" "beefy_walkto" "walkto" "f_WalkThru" "f_WalkTo" "f_PlayersClose" "swordlock" "f_KnockOffHorse" "hit3" "hostage_thrown" "beefy_throw" "s_DarkFly" "f_CharacterBeefy" "f_PunchSet200" "f_BeefyJabFierce" "transform" "magic_gap" "f_GoBeefy" "bar" "face" "horse_wait" "vs_time" "death_time" "beefylaugh" "dance" "IsNetGame" "f_SetVsTrueskill" "f_PlayerStop" "f_GoDance" "fp_PrevStandAnim" "fp_PrevWalkAnim" "fp_PrevDashAnim" "fp_PrevCharacter" "f_PlayerGo" "guide" "f_CannonBallBomb" "splat" "f_ExitOnHorse" "f_ClearQ" "door2" "f_MakeFriend" "f_PlayersStop" "f_NPCHorseExit" "horseblood" "f_HorseHeadIdle" "f_SetHorseHead" "pressed_button_l2" "pressed_button_r2" "f_PlayerGetsPickups" "initOverlayY" "u_wall_top" "u_wall_bottom" "u_wall_right" "u_wall_left" "u_wall_center_h" "u_wall_center_v" "smasher" "smash_pow" "f_CheckPushCharH" "f_CheckPushCharV" "f_CheckPushCharOut" "hrange" "yrate" "scalerate" "_alpha" "fp_WaitTimer" "f_NPCWalkRight" "f_NPCWalkLeft" "taunt1" "p_pt1" "kiss" "f_ScoreTally" "vs" "vs_center" "vs_intro" "newT" "oldT" "maxF" "f_BeefyGrappleTarget" "grapple" "f_AnyButtonPressed" "grapple_score" "grapple_timer" "beefythrown" "beefythrowbeefy" "winning" "torso" "losing" "draw" "s_Smack2" "StopMusic" "PlayMusic" "score_exp" "score_kills" "score_gold" "tally" "f_SetLevelCompleted" "f_SetLevelVisited" "f_PlayersGo" "door6" "catapult1" "catapult2" "f_HudWait" "../level36/level36.swf" "swim_stand" "swim_walk" "big_splish" "up_last" "up_last2" "down_last" "down_last2" "speed_x_slide" "side" "ladderclimb_sidedown" "ladderclimb_sideup" "speed_y_slide" "ladderclimb" "NO DEPTH: " " tried to move to: " "abs_bottom" "abs_top" "current_depth" "NO DEPTH and zone is undefined." "reach" "pressed_projectile_timer" "hattymagair1" "hatty_cast" "horn" "container" "f_GetPlayerClip" "monster_h" "monster_body_h" "monster_speed_x_max" "monster_speed_y_max" "monster_speed_x" "monster_speed_y" "v_fly_lock" "shooter_fireball" "u_left" "u_right" "f_ShooterProjectileMove" "jumpkick" "s_MagicNinja" "kick" "ninjerlog" "f_ShrapnelPermaBounce" "jumpkickwarp" "../escorp/escorp.swf" "scorpions" "f_ScorpionReset" "scorp_top" "scorp_bottom" "f_HitScorpion" "bowup" "playerbombexplode" "f_RandomGoldSpawn" "tornado_timer" "old_attack_pow" "tornadodone" "ladder_goal" "sideclimb_finish" "play" "finish" "f_IsCharacterUnlocked" "f_UnlockCharacter" "total" "door1" "door3" "beefy_jump" "stat1" "stat2" "stat3" "stat4" "text1" "plusminus" "stat" "poison" "lightning" "ice" "color_red2" "color_red3" "color_darkred" "color_yellow" "color_purple" "color_verydark" "color_lightgreen" "color_lightred" "color_lightorange" "color_lightblue" "color_bright" "color_white" "color_dirt1" "color_dirt2" "color_water1" "color_water2" "color_water3" "color_lightblue2" "music/Waterflame_RaceVictory.wav" "RegisterMusic" "fp_fpsLimiter" "fpsLimiter" "_root"

</code></pre></details>

After creating the bytes and copy-pasting them in the ENABLE section you'll need to make sure the byte array is the **same length** as the original byte array that we extracted from P-code.

You can prepend or append `00` bytes to align the length with the original bytes. This is only possible because we are overwriting the whole function body. In other cases you might need one or multiple `int()` calls to align bytes.

The script is now ready for test. If you are happy with the results you can share the `.CT`!

## Common problems

#### Scan regions for users

The function `aobscanregion(name,30000000,40000000,....` uses ranges to look for bytes, these can be on different places depending on the system it's running on. Widen the range or find out which range `castle.exe` uses on the users system to fix the issue.

#### Custom bytes are longer than original bytes

This happens. Rewrite the whole function you're working on and/or replace and call other functions to save bytes.

#### I did everthing right but it didn't work

This can be multiple things unfortunately. The most common thing I encounter while modding is that JPEXS uses the wrong registers so you might want to use placeholder function parameters.

If you need zone to be on register2 for example in f_HitSiegeTower(zone):

```
function f_HitSiegeTower(register1placeholder, zone)
{

}
```

Write a function like this and JPEXS decompiles to the right registers. I used an offset so CE doen't actually inject a new argument.
