# AGENTS.md

## Project Context

This is a solo-dev Unity game that acts as a study companion and productivity tool. The main experience takes place in one room/scene with one companion character on screen, so changes should protect the calm study loop, the character's consistency, and long-session reliability.

The game is primarily menu driven. Treat menu code as core gameplay code: stable, reusable, easy to inspect, and safe across enable/disable, scene reload, and save/load flows.

This file applies to `Assets/Code`. Keep custom project code under `Custom`, keep third-party code under `ThirdParty`, and do not edit vendored/plugin code unless the task explicitly calls for it.

## Current Project Facts

- Unity version: 6000.4.4f1.
- Render pipeline: URP is installed.
- UI stack: Unity UI/uGUI and UI Toolkit modules are available; follow whichever the feature already uses.
- Input: Unity Input System is installed.
- Testing: Unity Test Framework is installed.
- Documented third-party tools: Animancer Pro v8, Beautify 3, DOTween Pro, Easy Save 3, FMOD, Text Animator for Unity 3, VolFX, Yarn Spinner+, Fimpossible Look Animator, and Fimpossible Eyes Animator.

When third-party details matter, read `Docs/ThirdPartyTools.md` and inspect the local package/plugin APIs before coding.

## Product Direction

- Build for a single-player, single-room study buddy experience.
- Prefer quiet, dependable interactions over flashy systems that add maintenance weight.
- Keep features appropriate for one developer to understand and maintain.
- Avoid general-purpose frameworks unless they remove real project complexity.
- Do not introduce ScriptableObject-based project architecture or new ScriptableObject asset workflows. Use MonoBehaviours, serializable plain C# data, prefab/scene wiring, and save data classes instead.
- Avoid assumptions that there are multiple companion characters or multiple gameplay scenes unless the user explicitly expands the design.

## Code Organization

- Put hand-written game code in `Assets/Code/Custom`.
- Prefer clear feature folders such as `Runtime/Menu`, `Runtime/Companion`, `Runtime/Study`, `Runtime/Saving`, `Runtime/Localization`, and matching test folders when needed.
- Keep one public type per file, with filenames matching type names.
- Use namespaces rooted at `StudyCompanion`, then the feature area, for example `StudyCompanion.Menu` or `StudyCompanion.Saving`.
- Keep editor-only code in `Editor` folders and editor assemblies.
- Keep tests separate from runtime code.
- Do not move or reformat third-party files for style cleanup.

## Assembly Definition Practice

- Prefer asmdefs for custom code instead of relying on `Assembly-CSharp`.
- Recommended custom assemblies when new code appears:
  - `StudyCompanion.Runtime` for runtime game code.
  - `StudyCompanion.Editor` for custom editor tooling, marked editor-only and referencing runtime only when needed.
  - `StudyCompanion.Tests.EditMode` for Edit Mode tests.
  - `StudyCompanion.Tests.PlayMode` for Play Mode tests.
- Keep assembly references minimal and directional. Runtime code must not reference editor assemblies.
- If a feature wraps a third-party package, consider a small adapter layer so the rest of the game is not tightly coupled to plugin APIs.
- Update asmdefs whenever adding files that depend on Unity packages or third-party assemblies.

## Unity Lifecycle Rules

- `Awake`: initialize owned state and cache local/sibling serialized dependencies. Avoid scene-wide lookups here unless the dependency is guaranteed and documented.
- `OnEnable`: subscribe to events, register with services, start listening for input, and resume UI/animation behavior.
- `Start`: perform work that requires other enabled objects to have completed `Awake`.
- `OnDisable`: unsubscribe and unregister symmetrically with `OnEnable`; stop coroutines, timers, tweens, and input listeners that should not survive disable.
- `OnDestroy`: release resources that outlive disable, kill remaining tweens, cancel async work, and guard against repeated cleanup.
- `OnSceneLoaded`: subscribe and unsubscribe symmetrically, guard against unexpected scenes, and avoid assuming scene objects exist before they are loaded.
- Do not leave empty Unity callbacks.
- Keep lookup timing explicit. If a serialized reference is required, validate it early and fail with a useful error.
- Avoid execution-order dependencies. If execution order matters, document it in a wiring comment and prefer explicit initialization methods.

## MonoBehaviour Wiring Comments

Every new or modified MonoBehaviour that needs inspector setup, sibling components, prefab structure, scene hierarchy, or execution-order assumptions must include a short wiring checklist comment near the serialized fields or class declaration.

Example:

```csharp
// Wiring:
// - Assign the companion Animator from the scene character root.
// - Assign menuRoot to the panel container under the main Canvas.
// - Requires StudySessionController on the same GameObject.
```

Keep comments specific enough that a solo developer can repair a prefab or scene without rereading the whole script.

## Menu and UI Guidance

- Treat menus as reusable stateful screens with explicit open, close, refresh, and selection/focus behavior.
- Separate state decisions from view binding when practical. Small presenter/controller classes are preferred over large all-in-one UI scripts.
- Make menu transitions idempotent: calling open on an already-open menu or close on an already-closed menu should not corrupt state.
- Keep subscriptions, input actions, coroutines, and tweens symmetric across enable/disable.
- Disable or guard interactions while a menu is transitioning, saving, loading, or otherwise busy.
- Avoid static mutable UI state unless it represents a deliberate app-level service.
- Player-facing text must use localization keys, not hard-coded visible strings.
- Refresh visible text when language changes or localization tables reload.
- Prefer clear serialized references over runtime hierarchy searches for menu parts.

## Companion Character Guidance

- The companion should feel consistent, supportive, and non-disruptive to study flow.
- Use character animation tools intentionally: Animancer for animation flow, Look Animator/Eyes Animator for attention and presence, Text Animator/Yarn Spinner for dialogue where appropriate.
- Keep animation and dialogue triggers resilient to menu pauses, scene reloads, and save/load restoration.
- Avoid character behavior that depends on hidden global state without a clear owner.

## Saving and Runtime State

- Easy Save 3 is available. Use stable explicit save keys; do not derive save identity from mutable GameObject names or hierarchy paths.
- Persist only state that should survive sessions. Keep temporary UI state transient unless the feature requires restoration.
- Save/load paths must handle missing keys, version changes, and invalid data gracefully.
- After load, ensure the scene reaches a valid post-load state before UI becomes interactive.
- If introducing saveable scene objects, use generated or otherwise stable save IDs and document the wiring.
- Track dirty state deliberately so saving is not spammed.
- Restore Busy/Occupied or equivalent interaction locks safely after load.

## Third-Party Usage Notes

- DOTween Pro: kill or complete owned tweens on disable/destroy; avoid orphaned sequences; prefer linking tweens to GameObjects when appropriate.
- FMOD: use event references intentionally, release instances correctly, and avoid leaking emitters/listeners across scene reloads.
- Yarn Spinner+ and Text Animator: keep dialogue line IDs, localization, and text effects stable; do not hard-code player-facing dialogue in UI scripts.
- Animancer, Look Animator, and Eyes Animator: keep character animation wiring explicit and avoid burying required references in runtime searches.
- Beautify, VolFX, and other visual effects should support the study companion mood without hurting readability or focus.

## Performance Guidelines

- Avoid unnecessary `Update` polling. Prefer events, input callbacks, coroutines with clear lifetimes, or scheduled timers.
- Avoid hot-path allocations: LINQ, closures, string concatenation, boxing, temporary lists, and allocating physics APIs in recurring checks.
- Cache components used repeatedly. Avoid `FindObjectOfType`, `Camera.main`, and uncached `GetComponent` in hot paths.
- Do not add empty callbacks just for future use.
- Keep menu rebuilds and layout refreshes targeted so opening one panel does not rebuild unrelated UI.
- Profile or add focused tests when changing recurring timers, animation loops, save/load loops, or menu refresh paths.

## Localization

- Any new or changed player-facing text must use UIStringTable keys.
- Update locale files when adding or changing visible text.
- UI that remains open must refresh its visible text when language reloads.
- Do not bake English strings into menu controllers, companion dialogue presenters, notifications, or save/load error UI.

## Testing and Verification

- For pure menu state, timers, save data, and localization key resolution, prefer Edit Mode tests where possible.
- For lifecycle-heavy UI, scene object wiring, and save/load restoration, add Play Mode tests when the behavior is risky enough to justify it.
- If Unity cannot be run locally, explicitly say so and tell the user what compile/test confirmation remains.
- Do not mark a task complete if compile errors are visible in IDE diagnostics, Unity logs, or test output.

## End of Task Checklist

Before marking any code task complete, verify all 8 items:

1. Compile status: confirm the project compiles without errors through Unity/IDE diagnostics, Unity Test Runner, or an explicit note that the user must confirm because local tooling is unavailable.
2. Wiring comments: every new or modified MonoBehaviour that requires inspector setup, sibling dependencies, prefab wiring, or scene hierarchy setup has a short wiring checklist comment.
3. Assembly and namespace hygiene: categorize new code into the proper .asmdef files.
4. Unity lifecycle safety: review modified Awake, OnEnable, Start, OnDisable, OnDestroy, and 'OnSceneLoaded' methods for safe lookup timing, symmetric subscription/registration, scene-load assumptions, and necessary execution order. This is extremely important.
5. Save and state impact: decide whether modified runtime state must persist; ensure saveables use stable SaveKeys, generated scene saveIds when needed, complete Save()/Load() coverage, correct dirty tracking, valid post-load state, and safe Busy/Occupied restoration.
6. Unity performance: avoid unnecessary Update() polling, hot-path allocations, LINQ/closures/string churn in per-frame code, FindObjectOfType/Camera.main/uncached GetComponent in hot paths, empty Unity callbacks, and allocating physics APIs in recurring checks.
7. Localization: any new or changed player-facing text uses UIStringTable keys, locale files are updated as appropriate, and visible text refreshes on language reload.
8. Scope fit: confirm the result is appropriate for a study companion productivity app, meant to be managed by 1 dev.
