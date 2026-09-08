# Unity Editor Map

The editor is six windows and a menu bar. Everything else is a variation. Learn these on Day 1 and the intimidation mostly evaporates.

## The six windows

| Window | What it is | Backend analogy |
|---|---|---|
| **Hierarchy** | Every object in the currently open scene, as a tree | The object graph, live |
| **Scene** | The 3D/2D viewport you edit in | Your IDE's design surface |
| **Game** | What the player's camera actually renders | The running app |
| **Inspector** | Properties of whatever is selected | A property editor bound to the selected instance |
| **Project** | Files on disk under `Assets/` | Solution Explorer / the filesystem |
| **Console** | Logs, warnings, errors | stdout + stderr |

**The one that will confuse you:** Scene vs Game. Scene is the editor's god-view. Game is the camera's view. If something looks right in Scene and wrong in Game, it is a camera, culling, layer, or UI-canvas problem — not a logic problem.

## Play Mode

The single most important editor fact:

> **Changes you make while in Play Mode are discarded when you exit.**

Everyone loses work to this exactly once. Unity tints the editor while playing — go into `Edit > Preferences > Colors` and set the Play Mode tint to something aggressive. Do it on Day 1.

Corollary: Play Mode is *superb* for tuning. Change values while playing, find the number that feels right, write it down, exit, apply. That workflow is a genuine superpower and it's why the tint matters.

## Layout

Default layout is fine. Two changes worth making early:

1. **Console docked visible at all times.** An error you don't see is an hour you don't get back.
2. **Wide Inspector.** You will live in it.

Save your layout: `Window > Layouts > Save Layout`.

## Menus you'll actually use

- `File > Build Profiles` (Unity 6; older versions: `Build Settings`) — what ships, which scenes are included
- `Edit > Project Settings` — physics, input, tags & layers, quality, player
- `Window > Package Manager` — adding Cinemachine, Input System, etc.
- `Window > Analysis > Profiler` — M15
- `Component > ...` and the Inspector's **Add Component** button — same thing
- `GameObject > ...` — the same list you get right-clicking in the Hierarchy

> **Menu paths move between Unity versions.** If your agent gives you a path that doesn't exist, that's a stale-training-data problem, not a you problem. Tell it, and have it check the docs.

## Folder conventions

Unity only enforces a few magic folder names (`Resources`, `Editor`, `StreamingAssets`, `Plugins`). Everything else is yours. Use:

```
Assets/
  _Project/          # underscore keeps YOUR stuff at the top, above imported packages
    Art/
    Audio/
    Code/
      Core/          # asmdef: Hearthfall.Core — NO UnityEngine
      Unity/         # asmdef: Hearthfall.Unity
      Editor/        # asmdef: Hearthfall.Editor
    Content/         # dialogue, quests — data, not code
    Prefabs/
    Scenes/
    Settings/
```

Set this up on Day 15 and never reorganise again mid-project — Unity's `.meta` files make moving things safe, but it still eats a session you'd rather spend building.

## Things that are not obvious and cost people hours

- **`.meta` files are load-bearing.** They hold the GUID that every reference points at. Commit them. Never delete them. Never `.gitignore` them.
- **`Library/` is a build cache.** Gitignore it. Deleting it is safe and is a legitimate fix — it just costs a long re-import.
- **The Inspector shows public fields and `[SerializeField] private` fields.** Nothing else.
- **Renaming a serialized field disconnects it** from everything set in the Inspector. `[FormerlySerializedAs("oldName")]` prevents that.
- **A prefab asset and a prefab instance are different things.** Editing the instance doesn't change the asset unless you apply it.
- **Tags, layers, and sorting layers are three unrelated systems** that all sound like the same system.
- **Selecting an object in the Hierarchy while in Play Mode** lets you watch its state change live. Do this constantly; it's the best debugger Unity has.
