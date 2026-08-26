# Schemes

A scheme is the behavior selected by an object's logic configuration. `[logic] active` names the initial section; the
part before `@` selects the implementation.

```ini
[logic]
active = walker@guard

[walker@guard]
path_walk = guard_walk
path_look = guard_look
on_info = {+alarm_started} walker@alarm

[walker@alarm]
path_walk = alarm_walk
def_state_moving = run
```

The scheme name is the part before the suffix. For example, `walker@guard` uses the `walker` implementation, and
`sr_idle@wait_for_actor` uses the `sr_idle` implementation.

Digits are ignored while resolving a scheme name, so numbered variants share the same implementation.

## Switching sections

Active sections can define switch rules. The engine evaluates rule groups in this fixed order: actor distance, signals,
info portions, real-time timers, game-time timers, actor-zone rules, then NPC-zone rules. Within a group, rules follow
their order in the section. Numbered variants such as `on_info1` add another rule of the same type.

### `on_info`, `on_info1`, ...

Value shape: condlist.

When the condlist picks a target section.

### `on_signal`, `on_signal1`, ...

Value shape: `signal | condlist`.

When a waypoint or manager sets the named signal.

### `on_timer`, `on_timer1`, ...

Value shape: `milliseconds | condlist`.

After the section has been active for the given real-time duration.

### `on_game_timer`, `on_game_timer1`, ...

Value shape: `seconds | condlist`.

After the section has been active for the given game-time duration.

### `on_actor_inside`

Value shape: condlist.

When the actor is inside the current restrictor object.

### `on_actor_outside`

Value shape: condlist.

When the actor is outside the current restrictor object.

### `on_actor_in_zone`

Value shape: `zone | condlist`.

When the actor is inside the named zone.

### `on_actor_not_in_zone`

Value shape: `zone | condlist`.

When the actor is outside the named zone.

### `on_npc_in_zone`

Value shape: `story_id | zone | condlist`.

When the NPC resolved by story id is inside the named zone.

### `on_npc_not_in_zone`

Value shape: `story_id | zone | condlist`.

When the NPC resolved by story id is outside the named zone.

### `on_actor_dist_le`

Value shape: `distance | condlist`.

When the object sees the actor and actor distance is less than or equal to the value.

### `on_actor_dist_le_nvis`

Value shape: `distance | condlist`.

Same distance check, without requiring actor visibility.

### `on_actor_dist_ge`

Value shape: `distance | condlist`.

When the object sees the actor and actor distance is greater than the value.

### `on_actor_dist_ge_nvis`

Value shape: `distance | condlist`.

Same distance check, without requiring actor visibility.

A condlist can also set info portions or run effects while selecting the next section:

```ini
on_info = {+actor_has_key} ph_door@open %=play_sound(door_unlock)%
```

An empty target, `nil`, or the current section does not switch. Timer baselines reset after a successful switch.

## Scheme families

XRF ships 55 schemes. Each one has its own page; the sidebar lists them alphabetically.

### Stalker

Primary schemes for stalker NPCs: movement, position, animation, and interaction.

| Scheme                          | Purpose                                                                  |
| ------------------------------- | ------------------------------------------------------------------------ |
| [`animpoint`](./animpoint.md)   | Move to a registered smart cover point and play an idle animation there. |
| [`camper`](./camper.md)         | Hold a combat position, scan look points, and fire from cover.           |
| [`companion`](./companion.md)   | Follow and assist the actor.                                             |
| [`cover`](./cover.md)           | Move to a cover point near a smart terrain and look while animating.     |
| [`patrol`](./patrol.md)         | Coordinate a group of stalkers moving as one unit around a commander.    |
| [`remark`](./remark.md)         | Play a short scripted animation, optionally aimed, with optional sound.  |
| [`sleeper`](./sleeper.md)       | Move to a sleeping patrol point and sleep or sit.                        |
| [`smartcover`](./smartcover.md) | Use a registered smart cover and update the cover target state.          |
| [`walker`](./walker.md)         | Follow a patrol path while no higher-priority planner state is active.   |

### Monster

Monster movement, territory, animations, and combat.

| Scheme                          | Purpose                                                          |
| ------------------------------- | ---------------------------------------------------------------- |
| [`mob_combat`](./mob_combat.md) | Generic monster combat switch scheme.                            |
| [`mob_death`](./mob_death.md)   | Handle monster death callbacks and record the killer id.         |
| [`mob_home`](./mob_home.md)     | Keep a monster within a home area and radius range.              |
| [`mob_jump`](./mob_jump.md)     | Turn a monster toward a point and force a jump.                  |
| [`mob_remark`](./mob_remark.md) | Play scripted monster animations and optional interaction state. |
| [`mob_walker`](./mob_walker.md) | Follow a patrol path, optionally stopping at look points.        |

### Restrictor

Zone triggers, timers, visual effects, and actor events driven from a restrictor.

| Scheme                                    | Purpose                                                        |
| ----------------------------------------- | -------------------------------------------------------------- |
| [`sr_crow_spawner`](./sr_crow_spawner.md) | Spawn crows at configured patrol paths up to a total limit.    |
| [`sr_cutscene`](./sr_cutscene.md)         | Teleport the actor, disable game UI, and play camera effects.  |
| [`sr_deimos`](./sr_deimos.md)             | Drive a disorientation effect based on actor movement speed.   |
| [`sr_idle`](./sr_idle.md)                 | Wait and evaluate switch conditions without running an effect. |
| [`sr_light`](./sr_light.md)               | Register the restrictor as a light-control zone for stalkers.  |
| [`sr_monster`](./sr_monster.md)           | Stage a monster ambush while the actor is inside the zone.     |
| [`sr_no_weapon`](./sr_no_weapon.md)       | Track whether the actor is inside a weapons-disabled zone.     |
| [`sr_particle`](./sr_particle.md)         | Play particle effects, optionally following a path.            |
| [`sr_postprocess`](./sr_postprocess.md)   | Apply a gray/noise postprocess while the actor is inside.      |
| [`sr_psy_antenna`](./sr_psy_antenna.md)   | Apply psy-zone effects while the actor is inside.              |
| [`sr_silence`](./sr_silence.md)           | Mark the restrictor as a silence zone.                         |
| [`sr_teleport`](./sr_teleport.md)         | Teleport the actor after entry once a timeout elapses.         |
| [`sr_timer`](./sr_timer.md)               | Show a HUD timer and switch sections when it reaches a value.  |

### Physical

Usable and reactive world objects.

| Scheme                              | Purpose                                                                 |
| ----------------------------------- | ----------------------------------------------------------------------- |
| [`ph_button`](./ph_button.md)       | Play a button animation and switch sections when used.                  |
| [`ph_code`](./ph_code.md)           | Open a numeric input window and evaluate condlists for entered codes.   |
| [`ph_door`](./ph_door.md)           | Control door open/closed and lock state, NPC locking, tips, and sounds. |
| [`ph_force`](./ph_force.md)         | Apply a constant force toward a patrol point.                           |
| [`ph_hit`](./ph_hit.md)             | Apply a scripted hit when the section activates.                        |
| [`ph_idle`](./ph_idle.md)           | Neutral physical-object scheme controlling usability and tips.          |
| [`ph_minigun`](./ph_minigun.md)     | Aim and fire a minigun at a patrol point, the actor, or a story object. |
| [`ph_on_death`](./ph_on_death.md)   | Switch sections when the object receives a death callback.              |
| [`ph_on_hit`](./ph_on_hit.md)       | Switch sections when the object receives a hit callback.                |
| [`ph_oscillate`](./ph_oscillate.md) | Apply alternating constant force to a physical object joint.            |

### Helicopter

Scripted flight and weapons.

| Scheme                        | Purpose                                                                    |
| ----------------------------- | -------------------------------------------------------------------------- |
| [`heli_move`](./heli_move.md) | Move a helicopter along a patrol path and configure targeting and weapons. |

### Generic

Behavior attached alongside an active scheme rather than replacing it.

| Scheme                                      | Purpose                                                                 |
| ------------------------------------------- | ----------------------------------------------------------------------- |
| [`abuse`](./abuse.md)                       | React when the actor abuses an NPC repeatedly.                          |
| [`combat`](./combat.md)                     | Select a scripted combat style for stalkers through a condlist.         |
| [`combat_camper`](./combat_camper.md)       | Internal `combat` helper: hide and watch the last known enemy position. |
| [`combat_ignore`](./combat_ignore.md)       | Control whether a stalker accepts an enemy while scripted logic runs.   |
| [`combat_zombied`](./combat_zombied.md)     | Internal `combat` helper: simplified zombied combat actions.            |
| [`corpse_detection`](./corpse_detection.md) | Find and loot nearby corpses.                                           |
| [`danger`](./danger.md)                     | Replace the default danger evaluator and track heard hostile sounds.    |
| [`death`](./death.md)                       | Run configured condlists on death and store the killer id.              |
| [`gather_items`](./gather_items.md)         | Control whether a stalker may use the base item-pickup evaluator.       |
| [`hear`](./hear.md)                         | Switch sections from `on_sound` rules when a matching sound is heard.   |
| [`help_wounded`](./help_wounded.md)         | Help nearby wounded friendly stalkers.                                  |
| [`hit`](./hit.md)                           | Switch sections on a hit callback and record hit metadata.              |
| [`meet`](./meet.md)                         | Control greetings, idle animation, and dialog at interaction distance.  |
| [`post_combat_idle`](./post_combat_idle.md) | Wait briefly after combat before returning to alife behavior.           |
| [`reach_task`](./reach_task.md)             | Drive squad members toward their assigned simulation target.            |
| [`wounded`](./wounded.md)                   | Capture a stalker into a wounded state at health or psy breakpoints.    |

## Patrol names

Several schemes read patrol path fields such as `path_walk` and `path_look`. When an object is running under a smart
terrain, relative path names are resolved against the smart terrain name. For example, `path_walk = guard_walk` in smart
terrain `zat_b40_smart_terrain` resolves to `zat_b40_smart_terrain_guard_walk`.

Use full path names when the path does not belong to the active smart terrain.

## Before testing a section

- Supply every required field for the selected scheme.
- Keep `path_walk` and `path_look` distinct where both are used.
- Use `sr_idle` for a state that only waits for switch rules.
