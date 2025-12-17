# `2.5` CHANGELOG
There were many sleepless nights, long weekends, and multiple refactors of the ONB engine from version `2.0`. Huge thanks to team-member, Rune, for documenting the incremental changelogs during development
so that we can present them here.

!!! note
    This is a living document. Expect more changes to be added over time until `2.5` is released.

Efforts were made to preserve many of the scriptable API behavior while improving what was already available. Fixing some bugs revealed new issues and etc. Finally, the changelog reveals what has changed from the last public release to the grand `2.5` version of ONB.

!!! tip
    The items marked <span class="hidden">SECRET</span> are supposed to be surprises and as such will not be mentioned in any form of publicity even after a short time
    after the release of `2.5` in order to maximize fun for everyone.

1. Fixed missing local session code saving/restoring prog blocks.
	1. Fixed a logic issue in navi cust scene when inserting blocks into the grid.
2. Fixed crash while changing forms in the middle of a chip.
	1. Old code revealed animations were updating during draw and this queued graphics to be drawn which were later deleted when the transformation completed on a given frame.
3. Dark card effect properly dimming everything but mob health and card art.
4. Added default draw bits to card attachments and player form attachments.
	1. This required bumping all layer codes by 1 to support an _unset_ state.
5. Fixed many more issues with player sessions and folders which resulted in a partial rewrite. The sessions now switch between local and server without overwriting or affecting eachother.
6. Chip pool is also respected. Moving chips out of your pool will reduce the pool and these changes are tracked for that session.
7. Server mode settings: A) World B) Game
	1. Game mode can have a title screen and splash
		1. MiniGameFramework (MGFW) started.
	2. All servers should be able to provide an optional loading screen
		1. So people aren't just staring at a black loading screen
			1. A temp "solution" is in atm but MGFW would be more robust.
8. In Folder Edit: the `/` character is incorrect. e.g. `10930` instead of `10/30`.
9.  Animated navi previews
	1. Introduced `@node` attribute for ONB and boomsheets
		1. Users can use nodes for eyes and multiple independant parts.
		2. Limited supported scenes only (at the moment)
			1. mugshots
			2. navi preview
	2. Renamed `SyncNodeHandle` to `SpriteGraph`.
	3. We can support multiple nodes for the same animation.
	4. There needs to be two types of ways to initialize a "SyncNode":
		1. `create_sync_node(point)` 1. references the SAME animation as the entity. No copies.
		2. `create_sync_node_new_anim(point, anim)` 1. Provides a node its own animation file.
			1. This is ideal for sync nodes using completely different animations than the entity to which it is bound.
10. Bugfix: `cust_gauge_max_time` returns the default ma1. time if a custom ma1. time has not yet been provided.
11. Patch lua `print` to support variadic args.
12. Bugfix: `CardAction.on_action_end_func` did not run in time freeze. 
13. Added a new lua API for `HitProps`.
    
	The old code will still work. The new API will look like this:

	```lua
	local props = HitProps.new(user:get_context())
	```

	Or created manually:

	```lua
	HitProps.new()
	props:with_flags(Hit.Confuse | Hit.PierceInvis | Hit.	PierceGuard | Hit.PierceGround):cooldown(frames(2))
	confuse_atk:set_hit_props(props)
	```

	You can also chain off of `HitProps.new()`

	```lua
	local props = HitProps.new():add_flag(Hit.Confuse)	:cooldown(frames(2))
	```

  	Notice there's `:with_flags(...)` and `:add_flag(...)`. The former overwrites all at once. The latter can help format and add one at a time.

15. Patched status animations restarting when statuses are applied on that frame. Status animations (confused, blind, etc.) will continue to play if they were already playing even on subsequence status applications.
16. `DefenseFrameStateJudge` had unused `block_impact` and `is_impact_blocked` functions. It was repurposed and is now `block_collision` and `is_collision_blocked`. A new check for if `judge.is_collision_blocked` is `true` now occurs after `DefenseOrder::always` defense rule types.
17. Bugfix: Using Keristero's Guard mod, when using it and changing forms, confirmed that the `end_action` callback is correctly executed.
18. PlayerForms now support `set_description(str)`.


	<iframe src="https://youtube.com/embed/5DjTTTHhpWI" width="320" height="240" controls></iframe>


19. Battle background `.anim` files now support `@scroll left, top, frames` attribute.
	1. This attribute will overwite the `velx` and `vely` parameters in the scripted mob lua function `set_background`. Additionally, these two fields are now optional in lua and default to `0` if not provided. `mob:set_background(texture_path, anim_path, velx?, vely?)`.
	2. Similarly applied changes and exact behavior for scripted cards in lua which use animated background images.
20. Changing the charge table in lua now immediately applies the new values to the charge component.
21. Bugfix: A drag event's last **realized** step applies endlag.
22. Sea tiles add a +30 on aqua chips.
23. Feat: Mobs can replace the battle tiles with a custom thematic tileset.

	<iframe src="https://youtube.com/embed/YTWM8Y7WkbE" width="320" height="240" controls></iframe>

24. Card damager modifier changes:
	1. ... now takes an ID string in lua to identify the modification.
	2. ... can be cleared the same way.
	3. ... changes the value by id iff different from the last value.
25. Tile rule `TileRules.SeaBonusCheck` added.
26. If chip `can_boost` is `false` damage number and multiplier are hidden.
27. `Player.is_charging` returns false if not charging buster, true otherwise.
28. Sea panels also tick like poison for Fire characters.
29. Bugfix: Weird shader effects in battle start at times for characters
	1. Added Entity LOC into the RenderData table and made correct calls to refresh and re-apply the shader.
30. Removed all built-in background types except for Lan and Grid. This means you must provide your own backgrounds in servers or mods from here on out.
31. Backgrounds use `tickrate` for pixel-perfect scrolling behavior. 
	1. To add scrolling, add `@Scroll left=p1. right=p1. frames=count` to your animation files for the background.
	2. The lua API for setting a background is now just `set_background(texture_path, anim_path)`
	3. TiledMaps no longer read `Velocity` properties. Simply include the attribute to your animation file for servers.
32. Bugfix: Rainbow does not animate in NaviCust scene, or Status screen.
33. Bugfix: Can't leave folder until I set a REG chip.
34. Feature: Prevent backslashes to user name in config screen.
35. Bugfix: Having one item left in the navi cust makes the RUN button disappear when you select it.
36. Bugfix: Chip with + Damage is reset by navi who can charge on the first frame of battle
37. Feat: chip charging can replace a chip's action.
38. Feat: Tag chip support and data.
39. The engine should have a keyboard capture status so that pressing 0 key while in textbo1. does not reboot on you.
40. Invalid cards forever when you run out of chips fix.
41. Make special chars a lua property instead of a function.
42. Viewing an invalid card in the folder caused a crash fix.
43. Tagged chip pair not showing up fix.
44. If you have 30 chips and dont make an edit you can't leave FIX (AGAIN!).
45. Remove remaining pointers in recent chip cust refactor.
46. Provided way to do warpstep bug.
47. If I make a change to my config controls and save, the soft reset does not respect my changes.
48. Guaranteed frame-perfect key presses even during frame step on both single threaded and multithreaded modes.
49. Ability to determine if an entity is a local player or not (for exclusive sounds).
50. Added new optional custom property "Warp Out" which defaults to `true` if not specified, but if set to `false` it triggers the handler without warping the player out automatically.
    
	!!! TIP
    	This allows anyone to use custom warps for literally any kind of warping and to do custom animations.

51. Add additional elements: `Recov`, `PanelDestruction`, `Invisible`.
52. Allow `player.on_pre_move_func()` to return `nil` as a no-opt. When `nil` is returned, there is no animation + no movement.
53. Buster hit sound only plays on attack, not collision check 1. this allows guard to play _\*TING\*_ correctly.
54. Added ability to swap out with human character for overworld segments.
	1. Custom jack-in animation.

	<iframe src="https://youtube.com/embed/2TWqbFHjEew" width="320" height="240" controls></iframe>

55. Play buster hit sf1 on `attack_func`. Currently it is on hit which could be blocked by Guard/Defense Rules.
56. From Kirby: All panels animate during pause and other states. Magma panels dissapear after 980 frames.
57. Bugfix: Magma tiles deal null damage now (previously fire). Thanks Kirby!
58. From Kuri: They supplied custom chip cust graphics and the following information:
	1. Patched everything to do with forms and form customization:
		1. When opening the forms, the cursor is a frame late to reposition to the top
		2. After selecting a new form, the 1st frame of being open, the forms are not in the right place
		3. the 4th selected chip is underneath the chip lock sprite or missing entirely
		4. Form transformation timing is not correct
			1. 20 frames pass
			2. 21st frame, it starts to turn white
			3. 9 frames to become fully white
			4. 9 frames to fade out
		5. Pop in the health and emotion UI to the original top-left position and to the cust select widget during animations
59. Bugfix: The frame that an entire team is deleted, should results in all other entities receiving 0 damage and no status effects.
60. `Battle.is_local_player(e)` -> rename to the audio explicit version to prevent desyncs
	1. Renamed to `Battle.play_audio_for_player_or_default(sound1, sound2, priority)`.
61. Add Lua function for anims `.refresh()` and `.update()`
	1. these functions dont explicitly need a sprites anymore either. Provide this option.
62. Lua to add/remove nodes to tiles
	1. This way "custom" tiles can blink with the parent tile
63. `PlayerMeta.set_charged_attack_level()` is now -> `.set_charge_level()`
64. Add new API to `Game.cpp` to fetch the renderer to perform programmer-defined functions
	1. Currently the `RenderOptions` list is useful.
	2. `getController()->getRenderer<MyCustomCustomer>()->doCustomFunction();`
65. Added Swoosh support for non-renderer events.
66. Swoosh can now send data back to the next scene when called via `pop()` or `rewind()`!
67. Added dark hole.
68. `SubChip` renamed to `SubItem`.
69. SubItems have a category group for the `Head`, `Body`, and `Shoes` placement.
70. SubItems with the same category will ask to overwrite the active spot.
71. SubItems have a new `activateOnUse` field which, when true, places the item in the active spot client-side.
72. (C++) SubItems are cleared at the end of a session.
73. SubItems have an event that sends to the server.
74. Emails have an event that sends to the server.

	<iframe src="https://youtube.com/embed/mNXbxG1D3jA" width="320" height="240" controls></iframe>

75. Added `SubItemAdd` packet which was missing.
76. Added `SubItemActivate(subitem_id, active)` to toggle the active subitem if applicable
77. (C++) `OverworldSceneBase` class has a new `OnSubItemConsumeEvent()` to handle network callbacks in the online area scene.
78. Feat: API for sending chip data for after battles.
79. Feat: API sending other battle reward types (health, monies, bits, etc.).
    
	<iframe src="https://youtube.com/embed/mZLwz1dRMo8" width="320" height="240" controls></iframe>

80. Min and Max Emblem Size were 15x15. Size restrictions removed entirely.
81. Added a way to get viewport boundaries in screen/world space from Engine: `Engine.get_viewport_size(): {x=width, y=height}`
82. Similarly, added way to get tile from worldspace/screenspace point coordinates: `field:tile_from_pixels(x, y)`
83. Potentially fixed overworld collisions from allowing pixel-perfect diagonal crossing with a custom tri-leg algorithm approach.
84. `KeyItemScene` textbox scale issue fixed.
85. Bugfix: removed red tag when no cards and no reg.
86. Bugfix: Empty folder (and end of the folder) clears the codes from being visible.
87. Bugfix: Giga class cards are no longer default when folder is empty.
88. Moved multiplier out of `SelectedUI` widget to the chip properties as the new `x2` field
	1. When x2 is `true`, damage is doubled only once. When `x2` is `false`, damage is halved if ever doubled.
89. Noted that the `OnHit` callbacks were never executed if `props.damage <= 0`, so I took out those redundant checks IN the callback handler routine.
90. Kano moved the `card_delete` and `hand_delete` hit props resolution to the base entity class.
91. Card UUID property is now read-only.
92. Components now have a `is_timefrozen()` function for components attached to an entity.
93. Adjusted card damage modifier graphics on all states that print the modifiers based on the source material.
94. Camera packets that shake with intensity or duration of zero (0) will stop the shake effect.
95. Entities in battle can call `stop_camera_shake()` to also end the camera shake effect.
96. <span class="hidden">SECRET</span>.
97. <span class="hidden">SECRET</span>.
98. <span class="hidden">SECRET</span>.
99.  <span class="hidden">SECRET</span>.
100. Engine support `@loop` attribute everywhere for anims. `@loop start=x, end=y`.
101. Overworld supports `@on_move_change` for anims and only has two types of keys: `reverse` and `forward`.
102. <span class="hidden">SECRET</span>.
103. Added lua bindings to `tickrate` struct:
	1. `tickrate.new(units: int, delay: uint = 0)`.
	2. Fields: units, delay. 
	3. The only function: `digest(frames: frametime) -> int`.
104. Anim method `set_playback_speed()` now takes in `tickrate` to support frame delays (sub <1 frame per tick speeds).
105. Added an overload for `set_playback_speed()` to take in `frametime` for compatibility with olds mods.
106. Anim method `get_playback_speed()` likewise returns `tickrate`.
107. Updated function signature: `Explosion.new(count: int, speed: tickrate)`.
108. Bugfix: <span class="hidden">SECRET</span>.
109. Added optimization to animations so that cpu cycles for long-looping animations should stablize.
110. Ensured new "loop points" for overworld animations are working.
	1. See: Megalo_player_GyroMan.zip from D3str0y3d.
111. Bugfix: UI layer updates and draws itself on the correct frame as expected.
112. Feat: Server can now see what Cust Parts the client has installed for the current player mod:
	1. Server can now see this event with `Net.on('progblocks_upsert', event)`
		1. `event` = `{player_id, [pool], [active]}`
		2. `pool` = `{id, quantity}`
		3. `active` = `id, form?, color: string? center? rotates? compiled?`
		4. `form` = `[bool; 15]`
	2. Send whether or not a piece was on the compile line `compiled`.
	3. Fix the cursor alignment math when grabbing a bloc.
113. Feat: `GameSession` is now the source of truth for the player's stats in the engine.
114. Bugfix: Regression: Blocks are not inserting intro the grid after rotations correctly
115. Bugfix: Regression: Thread-friendly input consumers (frameskip, fullscreen, etc.) no longer work.
116. Bugfix: Regression: Fullscreen toggle in multithreaded mode crash.
117. Feat: `-f` or `--fullscreen` flags supported from console launch.
118. Bugfix: First-time fresh boot of client doesn't load any resources from mods and crashed in Navi Select.
119. Bugfix: Tweaked charge effect animation.
120. Bugfix: `Battle.play_audio_for_player_or_default(...)` crash fix.
121. Bugfix: Running out of cards now correctly erases the card icons from their slots.
122. Feat: <span class="hidden">SECRET</span>.
123. Bugfix: Fix stair layer ordering regression with new renderer in overworld.
124. Feature: During chip select, added a callback function on the hover card state to take in the player and mutate chips in the deck. e.g. Maramusa.
    1. `CardBuilderTrait.on_update` -> `CardBuilderTrait.on_tick`.
    2. `card_on_tick(actor, props)` for the lua mod, runs every frame tick during the Cust Selection Battle state.
    3. GUI caches damage value when it makes a new chip selection and draws this cached value like the games.
    4. `print_uncertain_damage` renamed -> `is_mystery_damage`.
    5. ALL next cards (top-most card) in a `Player`'s hand will now call `cards_on_tick` during combat as well.
125. Patch: Buster max charge time can never be less than zero + hard-coded 10 frames of startup delay.
126. Debug flag added to external device processor to draw sprite origins with color. This used used in the ONB Inspector tool.
127. Migrated all and partially rewrote player session data pertaining to folders into a new `FolderRecords` struct.
    1. This struct now has additional properties for the player's current session: `folderLimit`, `canCreate`, `canEdit`, `canEquip`, `forceEquip`
    2. The server can send these events and the client reads these values.
128. Renamed lua sprite methods `set_layer` to `set_draw_priority` which is more accurate now. This change was also made in the C++ code.
129. New `set_draw_bits` represents the group + order info used by the retro renderer to retarget sprites and object sprites.
    1. Method signature `set_draw_bits(unsigned int group, unsigned int destLayer)`.
    2. `destLayer` is optional!
    3. Groups: 
        1. `Background = 0`
        2. `Floor = 1`
        3. `Object = 2`
        4. `Attack = 3`
        5. `Effect = 4`
        6. `Misc = 5`
    4. Destination Layers are just `Groups`.
        1. This is to retarget object sprites and can be omitted.
130. Feat: Support sprite flipping at the node class and remove other implementations.
    1. Completed goal: Alpha mod shows that GBA layering is correct when flipping screen.
131. Feat: `status_cooldown: frames` field added to hitbox props.
132. Bufix: Restored freeze on ice panel when hit with water elements.
133. Bugfix: Restored cancel ice at EOF when hit with true breaking.
134. Feat: Optimized `Field` and `Tile` class iterations per tick.
135. Lua anim object has a new `set_keyframe(index, sprnode?)` function.
136. Added complementary `get_keyframe_index(): int` to lua anim objects.
137. Feat: Server can now overwrite player folder session traits and permissions.

	![](./images/folder_perms.png)

138. Feat: When using the link gate, the chip no longer auto-fires (manual activation).

	<iframe src="https://youtube.com/embed/fC3-eGVylLQ" width="320" height="240" controls></iframe>

139. Feat: You can now select a chip from the pool for overworld games.
    1. You can configure the event name, the types of chips that are allowed, the number of chips, etc.
    
	<iframe src="https://youtube.com/embed/p_Bf0Z1NSUo" width="320" height="240" controls></iframe>

140. Streams live logs to the registered external device (e.g. ONB Inspector tool) if ran with `-d` flag
141. Feat: No more console window. The debugger tool is now optional.

	![](./images/debugger.png)

142. Feat: camera zoom is enabled for a map for PC players via `PgUp` and `PgDn`.
143. Feat: Rewrote tile rendering and added camera-culling! Strictly tiles in view of the camera are drawn.
	1. Culling
		![](./images/map_culling.png)
	2. Layer special effects. [Link 1](https://x.com/OpenNetBattle/status/1880746436138246379?s=20). [Link 2](https://x.com/OpenNetBattle/status/1880746037142536685?s=20).
	3. Complete control of each tile. [Link 1](https://x.com/OpenNetBattle/status/1880745741314048152?s=20).

144. Feat: Server can overwrite/set/get Tag mem, Reg mem, and Card Limits.
145. Bugfix: Textbox responds to text escape codes properly (e.g. mute, slow, speed up).
146. Feat: Added `Layer 0` wiggle support.
147. Feat: "Fragments" info section appears in menu only when `fragments > 0`.

	<iframe src="https://youtube.com/embed/JYLfeZnmPi8" width="320" height="240" controls></iframe>

148. Bugfix: Characters which were not spawned by the mob spawner could not properly use timestop actions (the action does not execute, and does not freeze time).
149. Bugfix: Player `FinishConstructor()` function should overwrite shoot, swing, throw, and flinch animations too.
    1.   Flinch frame was overwriting non-existent entries, resulting in less frames to complete. This was unfair. Ensured overwrites generate valid anims that wait for duration specified.
150. Feat: Server API for creating, updating, equipping, etc. folders:
    1.   Client had to change from taking in a string to requiring a position index, starting at `1`.
    2.   This resolved most of the remaining known permission syncing issues.
151. Bugfix: Fixed the remainder of permission issues using wrong bitwise math.
152. Bugfix: Fixed `Net.write_player_folder()` API. All codes were being overwritten to `0`.
153. Bugfix: Fixed `Net.inc_player_card_pool()` and `Net.dec_player_card_pool()` API. Same problem as `write_player_folder()` as well plus the signature changed in rust but had not in lua.
    1.   Sig: `Net.inc_player_card_pool(player_id, list {id: str, code: u8, count: u8})`
    2.   Sig: `Net.dec_player_card_pool(player_id, list: id: str, code: u8, count: u8})`
154. Added MiniGameFramework (MGF) widget support.
	1. Support for Progress Bars Implemented
	2. Support for Textures as the Progress Bar Implemented
     

		!!! note
			This event was a precursor to Sprite API in `2.1`.
			This is a client-side widget for upcoming minigame framework.


155. Feat: <span class="hidden">SECRET</span>.
156. Feat: Client + Server can send/read mail events.
157. Flavored Text implemented engine-wide:
	1.   Wavy Text implemented
	2.   Jitter Text implemented
	3.   Rainbow Text implemented
	4.   These effects automatically work whereever custom strings are allowed.

	<iframe src="https://youtube.com/embed/kvBPSNw4TVA" width="320" height="240" controls></iframe>

158. New Start Menu + Start Up Sequence + Logo
159. Imported, configured, and programmed correct fonts for folder widgets.
160. Feat: Replaced LIMIT system with chips with highly requested MB system.

	<iframe src="https://youtube.com/embed/TV8o9Rixa_g" width="320" height="240" controls></iframe>

161. Feat: Stackable Cust Parts

	!!! note
		Finally, no one has to make multiple of the same mod and the engine
		tracks dupes client-side correctly.

	<iframe src="https://youtube.com/embed/N1S1Jj45jY0" width="320" height="240" controls></iframe>

162. Feat: Scriptable Program Advances!
    
	<iframe src="https://youtube.com/embed/xX1AJwItG8E" width="320" height="240" controls></iframe>

163. Bugfix: form crash fix.
164. Feat: Chips can be queued.
165. Feat: Scriptable Character spawn intros.
166. Feat: Popular Request: anything can be deleted/removed in intros.
167. Feat: Popular Request: added `-z` flag skips the client boot and title screen.
168. Feat: Eensured that title screen layer quantity can be made dynamic.
     1. The client reads the directory for titlescreen assets.
     2. This allows for even more client customization.
169. Bugfix: Removes a function ptr copy to the animation's interrupt routine when copy_from is invoked (this was running CardAction's end routine any time set_state() was ran subsequently and causing a crash)
170. Feat: added _fairly_ cust-accurate emblem.
     1. Optional for player mods.
171. Added `TileState.length` to TileState enum in lua to make field randomization easier.
172. Bugfix: Fixed volcano sprite refresh reported issue.
173. Feat: Tile pack provided by Rune.
174. Bugfix: Ice slide bug.
175. Feat: Custom cust screen button and lua bindings.
176. Feat: A chip can show up when you select your custom button.
177. Bugfix: Child nodes recolor correctly thanks to Alrysc.
178. New art provided by Enzan/DJ Rezzed: new battle reward skin.
179. Feat: psuedo-hot reloading.
     1. When launching client with `-d` debug flag you can now press `Num0` to reboot client.
     2. This feature is engine-wide and can also reboot directly into battle-only mode `-b`.
     3. This feature automatically reloads mods and is great for quick iterations.
180. Bugfix: Swoosh activity controller is now threadsafe.
181. Bugfix: Discovered and patched the sol memory leak.
182. Bugfix: Holding charge key down will recharge after flinch just like in the games.
183. Bugfix: Regression: key inputs can be read during time freeze.
184. Bugfix: Fixed the missing number `9` glyph in the megabytes draw.
185. Bugfix: Fixed the asterisk character `*` not displaying correctly in card preview.
186. Bugfix: Fixed the `Broken` tile state not working correctly when cracking tiles.
187. Bugfix: Correctly applied card type limits to the engine at boot.
188. Bugfix: Client replaces the boot, intro, and title scenes instead of pushing onto the stack which greatly reduces memory.
189. Adjusted `InputRepeater` valused to match the delay from the games. 
     1. Menu controls should feel a little more tighter and less sensitive.
190. Bugfix: Caos (aka Shoes) found some more utf8 pathing issues in the scripts which we fixed via the C++ side.
191. Feat: Added in-game rule checks and messages to the player when adding chips of each class and MB limit category.
192. Feat: In-game rules are dynamic.
     1. e.g. MegaChip limit could be increased via a Cust Part.
193. Feat: Rewrote the Folder Edit scene including the sort menu.
194. Clamped FPS subtitle string to 4 characters (no more very long FPS values).
195. Feat: Folders no longer disable and prevent you from editing them.
     1. Now only ill-formed folders prevent you from equipping them.
196. Feat: Web Launcher (ONBWEL) revealed.
197. Bugfix: Fixed time freeze crash when using chip during flinch.
198. Feat: Timestop chips correctly reposition player animation before and after.
199. Feat: A chip cannot be used while flinched - but they are correctly queued.
200. An attempt was made to perfect buster-canceling movement.
201. Feat: Chip charge mechanics.
202. Combo names (PAs) are not longer truncated to 9 characters.
203. Subsequent screen shakes are no longer queued from time freeze attacks.
204. `VirtualInputState` is now a property for all `Entity`s.
    1. This structure is now independant from the input polled by the OS.
205. Feat: Implemented REG and TAG chips.

	![](./images/tag_chip.png)

206. Feat: Implemented tile rules API.
	1. Tile rules allow for complex tile behavior and covers movement, sharing, and combat.
    
	<iframe src="https://youtube.com/embed/6t1qbGbjtOg" width="320" height="240" controls></iframe>

207. Client and Lua support for `unreserve_entity_by_id(ID)` added.
208. Feat: Implemented new anim file format rules.
    1. Engine still supports "legacy" format. 
209. Feat: Reg Tag chips show up first in the battle as expected.
210. Feat: Game accurate explosions.
211. More accurate battles: Battle doesn't end until last enemy's explosions are complete.
212. Feat: Client multiple language support.
    1. Default is `en` - english language.
    
	![](./images/i18n.png)

213. Feat: Server multiple language support.
214. Feat: replaced delta time with tick engine-wide for updates.
    1. This allows game to be frame-accurate.
215. Feat: Reduced mod loading prior to request.
216. Feat: New `TextboxTestScene` 
    1. Allows users to boot the client with mugshot and text to ensure dialog looks correct.
    2. `client.exe --chatbox --png path/to/a -anim path/to/b --dialog path/to/dialog.txt`
    3. `TextboxTestScreen` shows how many lines are left.
    4. New "Launch textbox" button can test in-game dialog from ONBWEL.
217. Feat: Lazy mod dependency resolution at bootup implemented.
218. Added `package:set_ability_name(str)` for Player mods to change the charge ability name in the Player Stat screen.
219. Feat: Player Stat screen is now complete. 
    1. Reg Mem, card limits, and installed blocks show up in the 2nd window area. 
    2. Stats update live after installing blocks too!
220. Navi cust screen is now more accurate to the game and fixed some bugs that were discovered.
221. Client support to manage player buffs during combat.
222. Feat: hold the `SHIFT` key to fast forward the game.

	!!! tip
		This will be disabled when visiting an online server.
 
223. Feat: <span class="hidden">SECRET</span>. 
224. Feat: Rewrite of card action life-cycle and abilities engine-wide:
    1. Introduced `Peek` event.
    2. Introduced `Ephemeral` card types.
225. Bugfix: Lua-provided charge time table bug (not updating) has been patched, finally.
226. Renamed the charge time table function to `charge_time_func` in lua to try and standardize the lua API.
227. Feat: Added an input focus feature in the engine that blocks debug keys from running while other widgets are using text input (e.g. textbox prompt).
228. `frametime` values can now be printed in lua e.g. `print(frametime.value)`.
229. Bugfix: Fixed horrible problem that has lingered in here for who knows how long: the engine correctly tags mods so you can load them in scripts (e.g. `CardFrom(uuid)`).
230. Engine provided special chars are now a lua property instead of a costly function.
231. Bugfix: Viewing an invalid chip in the folder no longer crashes.
232. Feat: If your folder is not adequate, the navi prompts a question to really leave.
    1. This routine runs if folder has less than the required chips. 
    2. If the folder rules are not met, instead of a question, the navi prompts the user with the reason and does not exit the screen.
233. FEAT: <span class="hidden">SECRET</span>.
234. Prototype new (buggy) interactive server to reboot and quit.
235. Server supports swapping OW costumes for "playable" operators.
236. Both client and server supports an additional button (R trigger) to be captured by scripts to do Jack-in or Hints like in single player games.
237. Server examples scripts now show how to create overworld segments.
238. Feat: Chip cust widget finally rewritten since the first ONB days to use the `SceneNode` system for drawing.
239. All chip cust art is now moved to `/resources/ui/cust_gui/` so new skin packs can be made.
240. Feat: <span class="hidden">SECRET</span>.
241. Feat: For the first time since v1.0, the chip cust drawn routine has been rewriten and even the cust itself can be animated! 
242. Feat: Rewrote all engine draw routines to use new plug-n-play renderer.
    1. Default renderer is "classic" (retro GBA-style) rendering.
243. Added `props.print_uncertain_damage = true` field to card props to print `???`.
244. Feat: Panels animate during pause and other sub-combat states where everything else normally pauses animation. This is more accurate.
245. Magma panels (lava) flicker out after accurate frames.
246. New server function to toggle Navi HUD for overworld games 
247. Feat: When the battle is over, combat can still simulate.
    1. This is like in the games: no one takes damage or status effects. 
248. Added `Battle.get_turn_count()` to lua which returns the turn count.
249. Added lua object anim function for `refresh()` and `tick()`.
    1. These functions don't explicitly need a sprite input anymore either.
250. Feat: <span class="hidden">SECRET</span>.
251. `PlayerMeta.set_charged_attack_level` -> renamed to just `set_charge_level`.
252. Added dark hole: `TileState.dark_hole`.
253. Feat: <span class="hidden">SECRET</span>.
254. Feat: Emotions are now modifiable.
    1.   `entity:mod_emotion_value(inc)`
    2.   Similarly taking damage can now put players in the `Anxious` state.
255. Feat: <span class="hidden">SECRET</span>.
256. Bugfix: Evil emotion no longer changes player's color.
257. Added `Battle.play_audio_for_player_or_default(entity, playerSound, otherSound, audioPriority)`
    1.   Will play `playerSound` if and only if the entity is the local player. Otherwise it will play `otherSound`.
258. Bugfix: Fixed reported animation problem with warping in. No more moon walk.
259. Feat: <span class="hidden">SECRET</span>.
260. Feat: <span class="hidden">SECRET</span>.
261. Feat: <span class="hidden">SECRET</span>.
262. Feat: ONBWEL will now AUTO-join the server launched from web links.
    1. A box informs you if a connection failed and returns to the PET home.
263. Bugfix: Fixed overworld teleportation bug.
    1. When converting seconds to frames, the check that someone was too far away was wrong causing pretty much everyone to teleport out constantly online.