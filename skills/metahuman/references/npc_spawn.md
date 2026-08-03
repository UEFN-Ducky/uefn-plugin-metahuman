---
description: "Wire assembled MetaHuman into NPCCharacterDefinition, AnimPreset, physics asset, Verse npc_behavior, and npc_spawner_device"
metadata:
  label: "MetaHuman NPC spawn"
  default_enabled: true
  load_condition: "Placing an assembled MetaHuman on a Fortnite island as an NPC with locomotion, custom clips, or Verse AI"
---

# MetaHuman → NPC Spawner (+ custom anims + Verse)

After **UEFN Export** assemble you have a character Blueprint + skeletal meshes.
Islands spawn NPCs through **`NPCCharacterDefinition`** + **`npc_spawner_device`**,
not by dropping the cinematic MH Blueprint alone into a published experience.

## Pipeline

```
Assembled MetaHuman (BP + meshes)
  → physics asset on skeletal mesh
  → retarget / author clips (SAME skeleton)
  → AnimPreset_BasicLocomotion (idle / run)
  → NPCCharacterDefinition (CharacterType_Custom + modifiers + VerseBehavior)
  → npc_spawner_device in level → Verse spawn manager
  → npc_behavior: MaintainFocus / NavigateTo / PlayAndAwait
```

Generic custom-NPC details (AnimPreset slots, modifiers):
`skill_read_subskill("animation", "npc_characters")`.
Verse AI loops: `skill_read_subskill("verse", "sys_npc_ai")`.

## Hard gotchas (read before PIE)

| Trap | Rule |
|------|------|
| **No physics asset** | Custom NPCs often spawn but **T-pose / slide without anim**. Assign a Physics Asset on the MH body skeletal mesh (create or duplicate a compatible one); save the mesh. |
| **AnimPreset mixed skeletons** | Every idle / run / MoveForward (and attack if on the same preset path) `AnimSequence` must use the **same Skeleton**. Mismatched skeletons → validation errors like “Invalid skeleton used in animation sequence for MoveForward”. Retarget all clips onto the MH skeleton first. |
| **Feet skate after retarget** | Rest-pose mismatch, not missing bones. Retarget with the Common `RTG_metahuman` / `IK_MetaHuman` assets when present, then correct the target retarget pose (`create_retarget_pose`, `set_retarget_pose_bone_rotation`, `set_retarget_pose_root_offset`) and re-bake — see `skill_read_subskill("animation", "retargeting")`. Never copy bones between skeletons. |
| **Unassembled Character asset** | Use meshes/BP under `Content/MetaHumans/<Name>/` from **UEFN Export**, not the raw Creator Character asset. |

## MetaHuman-specific setup

1. Use the **assembled** body (and face if required) skeletal meshes from
   `Content/MetaHumans/<Name>/`.
2. Confirm / add a **physics asset** on the body mesh (`get_skeletal_mesh_info`
   → inspect; fix in editor if missing).
3. **Locomotion clips**: MetaHuman body is Mannequin-compatible via Common
   `RTG_metahuman` / `IK_MetaHuman` — retarget Mannequin or FN clips with the
   animation skill (`retargeting`). Or author simple gestures with
   `create_anim_sequence` + `set_anim_bone_keys` (`anim_authoring`).
4. Build **`AnimPreset_BasicLocomotion`** from those clips (idle + run/walk).
   Enable `bSupportAnimPreset` on the definition.
5. **Attack / turn / act clips**: retarget or author onto the **same** MH
   skeleton; assign on `CharacterModifier_VerseBehavior` `@editable` slots
   (e.g. `AttackAnim`).
6. Face anim is separate (`animator_face`) — gameplay NPCs prefer short baked
   face loops; continuous Live Link is editor/previs, not island default.
7. Place **`npc_spawner_device`**, assign the definition, wire spawn events to
   your Verse manager (`sys_npc_ai`).

## Verse behavior (turns / chase / play clips)

`npc_behavior` subclass `OnBegin` loop typically uses (confirm names in digests):

- `GetFocusInterface[]` → `MaintainFocus` — look / turn toward the player
- `GetNavigatable[]` → `NavigateTo` — chase / strafe
- `GetPlayAnimationController[]` → `PlayAndAwait(Clip)` — play custom
  `animation_sequence` (attack, gesture, emote)

Full patterns: `skill_read_subskill("verse", "sys_npc_ai")`.

## Agent tooling honesty

Creating `NPCCharacterDefinition` / modifier / AnimPreset / physics assets is
primarily **editor asset authoring**. MCP can `search_assets`, `get_asset_info`,
`get_dependencies`, `get_skeletal_mesh_info`, retarget/author sequences,
`find_devices`, `wire_verse_device_ref` (**one wire per turn** —
`skill_read_subskill("uefn", "batch_commands")`), and verify — do not invent a
“create NPCDef from MetaHuman” tool unless capabilities prove it.

Prefer: duplicate a known-good definition → point mesh/preset/behavior slots at
MH assets → assign spawner → PIE.

## Checklist

1. `metahuman_get_info` on assembled BP — LODSync / Groom present.
2. Body skeletal mesh has a **physics asset**; save mesh.
3. All AnimPreset + Verse attack clips share the **same MH skeleton**.
4. Definition: custom mesh + AnimPreset + Health + VerseBehavior + anim slots.
5. Spawner label + definition set; Verse `npc_behavior` compiled clean.
6. `save_asset` / `save_current_level`.
7. PIE: spawn, idle/walk, **turn/look** (MaintainFocus), chase, PlayAndAwait
   attack/gesture, elimination if combat-relevant.
