# A1rMods explanation

All changes are done on main.swf (frame 2) unless stated otherwise.

## Infinite health

Remove damage substraction from health in f_Damage

```diff
function f_Damage(zone, damage_pow, damage_type, damage_flags, recoil_x, recoil_y)
{
  // code
-    zone.health = zone.health - damage_pow;
  // code
}
```

## Player is lightweight

Set weightplus to 0 in f_SpawnPlayer

```diff
function f_SpawnPlayer(u_num, n_type, x, y)
{
  // code
-    _loc2_.weightplus = 1;
+    _loc2_.weightplus = 0;
  // code
}
```

## Player is heavyweight

Set resists[DMG_MELEE] to 65 in f_UpdatePlayerAttributes. Uses 9 `int()` calls to align bytes

```diff
function f_UpdatePlayerAttributes(zone)
{
  // code
-    zone.resists[DMG_MELEE] = 40 + _loc4_ / 2;
+    zone.resists[DMG_MELEE] = int(65);
  // code
}
```

## Projectiles can hit players while tossed

Let if statement pass in f_ProjectileHit

```diff
function f_ProjectileHit(zone)
{
  // code
    if(_loc7_ < _loc6_ - _loc3_.h)
    {
-      if(!_loc3_.nohit or zone.hitnohit)
+      if(1 == 1)
       {
          if(!(zone.splashattack and _loc3_.toss_clock > 0))
          {
  // code
}
```

## Safety decapitation - blocks first light attack

Replace multiple properties in f_SpawnPlayer

```diff
function f_SpawnPlayer(u_num, n_type, x, y)
{
  // code
-   _loc2_.rage = _loc3_.rage;
-   _loc2_.rage_goal = _loc3_.rage_goal;
+   _loc2_.blocks = true;
+   _loc2_.block_odds = int(1);
    _loc2_.recovery = _loc3_.recovery;
    _loc2_.health_max = _loc3_.health_max;
    _loc2_.health = _loc3_.health;
-   _loc2_.poison_timer = 0;
-   _loc2_.fire_timer = 0;
+   _loc2_.air_block_odds = 1;
+   _loc2_.full_block_odds = 1;
  // code
}
```

## Disable ice physics

Set slide speed to 0.0 in f_Walk

```diff
function f_Walk(zone)
{
  // code
    if(level == 23 or level == 102)
    {
       var _loc5_ = 1;
    }
    else if(level == 51 or level == 49 or level == 57)
    {
-      _loc5_ = 0.3;
+      _loc5_ = 0.0;
    }
    else
    {
       _loc5_ = 0;
    }
  // code
}
```

## Ice physics for every floor

Set slide speed to 0.3 in f_Walk.

```diff
function f_Walk(zone)
{
  // code
    if(level == 23 or level == 102)
    {
        var _loc5_ = 1;
    }
    else if(level == 51 or level == 49 or level == 57)
    {
        _loc5_ = 0.3;
    }
    else
    {
-       _loc5_ = 0;
+       _loc5_ = 0.3;
    }
  // code
}
```

## Remove mounts

Make noop's of f_MakeHorse f_MakeMount2 f_MakeMount3 f_MakeMount4

```diff
function f_MakeHorse(x, y)
{
-  // all code
}
function f_MakeMount2(x, y)
{
-  // all code
}
function f_MakeMount3(x, y)
{
-  // all code
}
function f_MakeMount4(x, y)
{
-  // all code
}
```

## Infinite bombs

Remove bombs substraction in f_Punch and f_HorseAttack

```diff
function f_Punch(zone)
{
  // code
    case 8:
        if(zone.bombs > 0)
        {
-           zone.bombs = zone.bombs - 1;
  // code
}
function f_HorseAttack(zone)
{
  // code
    case 8:
        if(zone.bombs > 0)
        {
-           zone.bombs = zone.bombs - 1;
  // code
}
```

## Infinite potions

Remove healthpots substraction in f_Punch, f_CheckHealthPots, f_HorseAttack and f_PlayerClock

```diff
function f_Punch(zone)
{
  // code
    case 9:
        if(zone.healthpots > 0 and zone.health < zone.health_max)
        {
            if(zone.health > 0)
            {
-               zone.healthpots = zone.healthpots - 1;
  // code
}
function f_HorseAttack(zone)
{
  // code
    case 9:
        if(zone.healthpots > 0 and zone.health < zone.health_max)
        {
            if(zone.health > 0)
            {
-               zone.healthpots = zone.healthpots - 1;
  // code
}
function f_CheckHealthPots(zone)
{
  // code
    if(_loc3_ != 0 and !_root.vs_fight and zone.healthpots > 0 and active_players > 1)
    {
-       zone.healthpots = zone.healthpots - 1;
  // code
}
function f_PlayerClock(zone)
{
  // code
    if(zone.healthpots > 0 and zone.health < zone.health_max)
    {
        if(zone.health > 0)
        {
-           zone.healthpots = zone.healthpots - 1;
  // code
}
```

## Infinite sandwiches

Remove beefies substraction in f_Punch

```diff
function f_Punch(zone)
{
  // code
    case 11:
        if(zone.beefies > 0 and !zone.beefy)
        {
-           zone.beefies = zone.beefies - 1;
  // code
}
```

## Infinite tek

Hard-code nohit = true in f_StandSettings

```diff
function f_StandSettings(zone)
{
  // code
-   zone.nohit = false;
+   zone.nohit = true;
  // code
}
```

## Walk in cutscenes

Remove all code from f_PlayersStop()

```diff
function f_PlayersStop()
{
-  // all code
}
```

## Alien can pick up weapons - not in Weapons Frog

Set p_type == 11 to 0 in f_PickupItem and f_GetPickup

```diff
function f_PickupItem(zone)
{
  // code
-   if(_loc2_.hud_pt.p_type == 11)
+   if(_loc2_.hud_pt.p_type == 0)
  // code
}
function f_GetPickup(zone, u_temp)
{
  // code
-   if(zone.hud_pt.p_type == 11)
+   if(zone.hud_pt.p_type == 0)
  // code
}
```

## 100% critical chance

Let if statement pass in f_Damage

```diff
function f_Damage(zone, damage_pow, damage_type, damage_flags, recoil_x, recoil_y)
{
  // code
    if(zone.hitby.weapon_critical > 0)
    {
-       if(random(zone.hitby.weapon_critical) == 0)
+       if(1 == 1)
  // code
}
```

## 100% elemental chance

Let if statement pass in f_Damage

```diff
function f_Damage(zone, damage_pow, damage_type, damage_flags, recoil_x, recoil_y)
{
  // code
    if(zone.hitby.weapon_magic_type > 0)
    {
-       if(random(zone.hitby.weapon_magic_chance) == 0)
+       if(1 == 1)
  // code
}
```

## Players unable to get knocked off of mounts

Remove all code from f_KnockOffHorse

```diff
function f_KnockOffHorse(zone)
{
- // all code
}
```

## Enemies unable to get knocked off of mounts

```diff
function f_Juggle1(zone)
{
    if(zone.horse)
    {
-       zone.horse.gotoAndStop("wait");
-       zone.horse = undefined;
+       return undefined;
    }
  // code
}
```

## Beefies don't grab you mid-air

Add a check for `body_y` so beefies don't grab the player if they fly over them.
They still grab the player in the air if they try to jump over the beefies to balance things out.

Also because of the removal of `nohit` which was needed to make place for the new check has a side-effect.
A player is able to reset the beefies hitcounter by flying over them without the need to be in tek.

```diff
function f_BeefyChain(zone)
{
+   if(!zone.human) // decompilation error fix
    {
        if(zone.hit_number > 0)
        {
            var _loc2_ = 1;
+           while(_loc2_ <= active_players) // decompilation error fix
            {
                var _loc3_ = playerArrayOb["p_pt" + int(_loc2_)];
-               if(_loc3_.alive and !_loc3_.nohit and !_loc3_.beefy)
+               if(_loc3_.alive and !_loc3_.beefy)
                {
-                   if(Math.abs(zone.x - _loc3_.x) < 100)
-                   {
-                       if(Math.abs(zone.y - _loc3_.y) < zone.speed)
-                       {
+                   if(Math.abs(zone.x - _loc3_.x) < 100 and Math.abs(zone.y - _loc3_.y) < zone.speed and _loc3_.body_y > -120)
+                   {
                        zone.prev_Character = zone.fp_Character;
                        f_BeefyGrabEnemy(zone,zone.prey);
                        f_SetTargets();
                        zone.fp_Character = f_EnemyWalkToss;
+                   }
-                       }
-                   }
                }
                _loc2_ += 1;
            }
        }
    }
    zone.hit_number = 0;
}
```

## Infinite wave training

Remove total and wave increment in f_ArenaGates by rewrting the whole function.
Remove weight/random_dash_jump/random_dash_roll from arena enemy.
Set health to 1000 and weightplus to 1 for Thief, Barbarian, Conehead, Snakey & Eskimo

_TODO: Add more code_

```diff
function f_ArenaGates(zone)
{
    if(_root.active_enemies > 0)
    {
        if(zone.door2.open)
        {
-           zone.wave = zone.wave + 1;
  // code
-           zone.total = zone.total + 1;
  // code
        }
    }
}
```

For all enemies:

```diff
function f_SpawnBarbarian(x, y, enemy_level)
{
  // code
-   _loc2_.health_max = f_SetEnemyHealth(50);
+   _loc2_.health_max = f_SetEnemyHealth(1000);
  // code
}
```

## Magic + light attack shoots an arched arrow

Custom code in f_PaintClock

```js
function f_PaintClock(zone)
{
   if(zone.hud_pt.item_unlocks[1] and zone.toss_clock == 0 and Key.isDown(zone.button_punch1) and Key.isDown(zone.button_magic))
   {
      zone.punching = true;
      if(zone.jumping and !zone.freeze)
      {
         zone.freeze = true;
         f_SetFloat(zone,11);
      }
      zone.gotoAndStop("bowup");
   }
}
```

## Arena enemies

Custom code in f_PaintClock

```js
function f_PaintClock(zone)
{
   var _loc2_ = 1;
   while(_loc2_ < total_enemies)
   {
      var _loc3_ = loader.game.game["e" + int(_loc2_)];
      if(_loc3_.alive and _loc3_.weight != 3)
      {
         _loc3_.block_odds = 3;
         _loc3_.aggressiveness = 1;
         _loc3_.shot_timer_default = 30;
         _loc3_.weight = 3;
         _loc3_.random_dash_jump = 1;
         _loc3_.random_dash_roll = 1;
         _loc3_.speed = 10;
      }
      _loc2_ += 1;
   }
}
```

## Cats do 999 damage

```diff
function f_MeleeCheckHit(zone, u_temp, x1, y1, x2, y2, top, bottom, left, right)
{
  // code
    case 301:
-       f_Damage(u_temp,zone.punch_pow_medium,zone.attack_type,DMGFLAG_JUGGLE | DMGFLAG_BLOODY,zone.force_x,zone.force_y);
+       f_Damage(u_temp,int(999),zone.attack_type,DMGFLAG_JUGGLE | DMGFLAG_BLOODY,zone.force_x,zone.force_y);
  // code
}
```

## Choose 1p arena

Partially change code in DefineSprite 140, Frame 152, select.swf

_TODO: Add code_

## Choose multiplayer arena

Partially change SelectExit function in aMenu.swf

_TODO: Add code_
