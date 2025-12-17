# BACKPORT
A very helpful community member named Alrysc helped back-port awaited `2.5` features into the older `2.0` engine.
He did this so that everyone can enjoy new content sooner and iron out innaccuracies in PVP.

This effort, dubbed "the backport" was unplanned and volunteered by him because he is very cool. 
Because he was also helpful, he was brought into the official team for `2.5` to work on the C++ codebase.

# 2.1 CHANGELOG
1. `Hit.Flinch` is now allowed to queue with `Hit.Drag`, so you will flinch when hit by `Hit.Drag` if you don't have armor/`DefenseRule`-s.
2. `Hit.Stun` and `Hit.Freeze` replace each other instead of allowing you to be stunned and frozen at the same time.
3. Grass heal works properly.
4. Fixed an issue where you could skip the battle start text when form changing if any Player took weakness damage since the last transform state.
5. Fixed TFC softlock if you did TFC on the last possible frame.
6. Breaking change: `Player.charged_time_table_func` was renamed to `Player.charge_time_func`, and now takes Player (self) and the Charge level as input.
   
    !!! NOTE
        You will not get errors if you don't fix this, but it will not work as expected!

7. Breaking change: `PlayerForm.calculate_charge_time_func` was renamed to `PlayerForm.charge_time_func`, and now takes Player (self) and the Charge level as input, so it works the same as the Player's version. 

    !!! CAUTION
        This will make the mod unnusable if you don't fix this.

8. Fixed mistake on `Player.charge_time_func`.
9. Added `Entity.is_dragged` so you can check if you're currently under the effects of Drag, which prevents all action.
10. Fixed `Entity.can_attack`.
11. Drag now returns false for `is_moving` and `is_sliding` after movement ends for the rest of the duration of Drag.
12. Added Confuse status, which lasts for 110 frames when applied. You can access with `Hit.Confuse`.
    1. Added `Entity.is_confused` so you can check if you are under the effects on Confusion. 
    2. Enemies may want to use this to choose different logic. Kirby summarizes how enemies tend to react to Confuse in later games.
13. TimeFreeze attacks don't-counter hit anymore.
14. Turns now last 512 frames by default (~8.5 seconds vs. old 10 seconds).
15. `Battle.get_turn_count`, which returns the current turn number. `0.0` initially, then increases by `1.0` every Battle Start banner.
16. `Battle.get_cust_gauge_value`, which returns a double ranging from `0.0` to `1.0`, as a percentage of the Cust gauge (or maybe it can go over `1.0`, but `1.0` is full. I'll update this text later once I check.).
17. `Battle.get_cust_gauge_time`, which returns a number in frames for the current gauge time (e.g. `256.0` when gauge is half full with default duration).
18. `Battle.get_cust_gauge_max_time`, which returns a number in frames for the current max gauge time (e.g. `512.0` for default time).
19. `Battle.get_default_cust_gauge_max_time`, which returns the default gauge time, `512.0` frames.
20. `Battle.set_cust_gauge_time`, takes in a frametime (like `frames(256)`) to set current gauge time to.
21. `Battle.set_cust_gauge_max_time`, takes in frametime to set max time to (current time is automatically readjusted to match the percentage of total time it had before, e.g. max time 512 and current time 256 will have current time 100 if you set max time to 200).
22. `Battle.reset_cust_gauge_to_default`, sets max time to default time.
23. Added secondary element to HitProps, called `element2`. You can access with `props.element2`.
24. Form change resets emotion.
25. The old HitProps constructor is deprecated! But it exists for now. In the future and now, you can create a HitProps with `HitProps.new()` or `HitProps.new(context)`. Then you use a series of functions (builder design pattern) to customize.
26. Fixes duplicate name that made it impossible to read a HitProps Drag. The function drag is now `drg`.
27. Obstacles cannot become blind or confused.
28. The player session tracks BugFrags, and then can be set (and rewarded with battle rewards) from the server.
29. Fixed a crash related to leaving the server in recent builds
30. Obstacles are now deleted immediately when hitting 0 health, rather than waiting for their next update like all other Entities.
31. Unfinished `Battle.UIComponent` is included, but does not yet support layering.
32. Low health beep is back, but plays at slightly wrong times. To be fixed later.
33. Because the timing of battle results signals has changed, liberations may react sooner than expected.
34. Opacity is set correctly by the server for drawn sprites.
35. Hiding the HUD will no longer allow you to open an invisible menu.
    1. Pressing Start will still interrupt other inputs. For example, moving and pressing Start will stop moving for a frame.
36. Instead of saying "No Data", results will say "Pending" in the text box if the server indicated a card _would_ be received but had not finished installing.
37. Fixed a crash when editing folder.
38. UIComponents now respond to layering and can be drawn over the Cust gauge at certain layers.
39. UI component draw fixes when perspective flipping.
40. `mob:no_results()` skips showing battle results, but results still go to servers (for tourney style battles).
41. Battle results `run` is now replaced with `reason` on the server.
    1. `win` - when the battle is over b/c the player won
    2. `lose` - when the battle is over b/c the player lost
    3. `draw` - when the battle is over but the match was a draw
    4. `runaway` - "   " b/c the player requested to run away
    5. `debugesc` - "   " b/c the user player aborted (panic)
42. The engine now responds to a unique heuristic: if the user presses `ESC` three times in a row within a second, the scenes can respond to a user panic operation. This trigger aborts the battle scene and the overworld online scene. This is useful if you get stuck and replaces the debug `ESC` behavior entirely.
    1. 2 scenes now implement this: battle + overworld online. Performing this operation will return you to the previous scene.
43. Custom battle hit SFX. This makes the engine more accurate in behavior.
    1. `Entity:set_hurt_sfx(audio)` changes the entity hurt sfx.
    2. `Entity:use_default_hurt_sfx(bool)` toggles whether to use the default hurt sfx from before.
    3. `DefenseFrameStateJudge:set_hit_sfx(audio)` changes the _final_ sfx to play. Adopts the victim entity's hurt sfx if one is provided.
    4. `DefenseFrameStateJudge:hit_spawn_gfx(bool)` adds a hint to the defense frame state judge whether or not to spawn a hit graphic effect.
    5. `DefenseFrameStateJudge:hint_spawn_gfx()` - used in hitbox attack + collision func to read if the entity should obey the hint to spawn image graphics or not.
    6. Additionally, attack and collision callbacks now receive an optional last parameter `judge` object so you can read this hint and respond to it.
    
    !!! TIP
        For those who care about mod accuracy, custom sfx during combat resolution is a must.
        For most people, this means you don't have to provide hit sounds in most of your mods anymore, or handle playing them.

44. `Hit.cancel` is automatically added to `Hit.stun`, `Hit.freeze`, `Hit.flinch`, and `Hit.drag`.
45. You can add a callback event to react to this.
46. `Hit.cancel` can be erased by defense rules.
47. The purpose of this flag is to cancel a chip or interrupt an enemy.
    1. This behavior used to be applied to everything that used a `CardAction` for attacking or behavior, but was inconsistent with the games. So now it's still "automatic" but you can opt-out of this behavior.
    2. For example: when viruses are stunned, they **do not** have their action interrupted. They continue after un-stun. However, bosses **do** handle interrupts.
    3. Side effects: if you are dragged and remove `Hit.cancel`, it's now possible to move mid-action. This may or may not be what you want, if it happens. Just handle this case.
    4. TL;DR if you're programming a virus, use virus body defense rule  or remove `Hit.cancel` in the defense rule step. This assumes you're using `CardAction`-s for behaviors (correct).
48. FullSynchro is no longer removed accidentally in places where it shouldn't be (when `DefenseOrder.Always` blocked the attack, or when anybody using a `CardAction`).
49. `ActionOrder.Immediate` now causes the last action added to go first, so queuing multiple actions with `ActionOrder.Immediate` will have reversed order now (FILO aka stack).
50. Losing in PvE battles can no longer end in a `draw`. Either `win` or `lose` (with the exception of `userAbort`).
51. The fade time after battle was changed, for people who noticed it was different last build. The results widget now stays around while fading out. This is like the games.
52. New title screen. May be the final title screen depending on if we get custom art by the time public release happens.
    1. `bg_blue.png` -> `bg.png`
    2. added `vnum.png` which you can customize but the location is pre-coded.
 53. Servers can play the email ring.
 54. Servers can provide card mod assets and cause them to load on the client.
 55. Servers can send battle rewards, which display in battle for real!
     1. There can be multiple rewards sent at the same time.

    <iframe src="https://youtube.com/embed/mZLwz1dRMo8" width="320" height="240" controls></iframe>

 56. Servers can tell the client to draw sprites on the screen in the overworld.
    
    <iframe src="https://youtube.com/embed/76d3I9BvhQM" width="320" height="240" controls></iframe>

 57. Servers can toggle the HUD between health and PET.
    
    !!! Note
        the PET and face shown are currently just the ones in the resource folder, but they may read from the mod in the future.

     1. New API endpoints for:
         1. Battle rewards
         2. Toggling HUD
         3. Sending emails
         4. Ringing PET
         5. Drawing sprites on screen
 58. Server sprites are drawn in an optimized manner for potentially hundreds of sprites over the network.
 59. `mob:no_results()` skips results presentation for tourney-style battles
 60. Asset type and package type hints (optional) when providing assets to your users (makes chip rewards work)
 61. For the sprite API, the `a` channel is now an alternative name for `opacity` channel.
 62. Sprite API supports `color_mode` and `rgb` channels.
 63. James released "Stardust" - a simple particle system plugin for ONB servers.
    1. Stardust API updated to support rgb color channels
    2. Added `enums.lua` to the server libs to start documenting enum values.
 64. Added `Net.virtual_input(event={player_id, events={{name, state}})` for the server to handle input.
    
    !!! NOTE
        Be sure to lock the player's client-side input first!
 
 65. Server-side sprite anims loop now like they were supposed to.
 66. When providing assets, you can now give an optional asset type and mod type on the server-side. 
    
    !!! TIP
        This helps resolve chips for giving them as rewards in battle.