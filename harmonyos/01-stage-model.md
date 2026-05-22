# HarmonyOS Stage model

**Summary** — the Stage model replaces FA (Feature Ability) as the app
runtime; the unit of life-cycle is a `UIAbility`, the unit of process is
an HAP, and the resource pipeline is centralized.
**Why I'm reading this** — to know what is *the* current way of building
a Harmony app and what FA-era tutorials are now misleading.
**Status** — draft.

## What changed from FA → Stage

| Aspect             | FA (old)                      | Stage (current)                |
|--------------------|-------------------------------|---------------------------------|
| App entry          | `FeatureAbility`              | `UIAbility` + `ExtensionAbility`|
| Window provider    | Tightly coupled to ability    | `WindowStage` (separate)        |
| Module declaration | `config.json`                 | `module.json5`                  |
| Imports            | `@ohos.xxx`                   | `@kit.xxx` (kit-grouped)        |
| Process model      | One ability ≈ one process     | Multiple abilities can share    |
| Bundle             | `.hap`                        | `.hap` + `.hsp` (shared)        |

The shifts are mostly about separating concerns that FA had glued together
(window vs lifecycle, configuration vs code, ability vs process).

## The pieces, in scale order

```
┌──────────────────────────────────────────────────────┐
│ App (bundle, e.g. com.example.foo)                  │
│   ┌─────────────────────────────────────────────┐  │
│   │ Module (entry HAP, or feature HAP)          │  │
│   │   ┌─────────────────────────────────────┐  │  │
│   │   │ UIAbility (one task in recents)     │  │  │
│   │   │   ┌──────────────────────────────┐  │  │  │
│   │   │   │ WindowStage                  │  │  │  │
│   │   │   │   ┌────────────────────────┐ │  │  │  │
│   │   │   │   │ Page (ArkUI tree)      │ │  │  │  │
│   │   │   │   └────────────────────────┘ │  │  │  │
│   │   │   └──────────────────────────────┘  │  │  │
│   │   └─────────────────────────────────────┘  │  │
│   └─────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
```

`UIAbility` is what shows up in recents. `WindowStage` lets one ability
host several pages without spinning up another lifecycle. `Page` is the
ArkUI tree that gets rendered.

## Lifecycle

```
        Create
          │
          ▼
        Foreground ◄──── (window gains focus)
          │
          ▼
        Background ────► (window loses focus, but still alive)
          │
          ▼
        Destroy
```

The lifecycle is on **UIAbility**, not on individual pages. Pages are
just React-ish trees inside `WindowStage`. Routing between pages
(`router.pushUrl`) does not pass through the UIAbility lifecycle.

`Navigation` + `NavPathStack` is the newer alternative to `router.*` and is
what new code should use.

## What I got wrong the first time

Thinking `WindowStage` was a kind of overlay. It's just the container an
ability sets up for its window — analogous to setting up a `Surface` in
Android. Once `WindowStage.loadContent("pages/Index")` returns, you stop
thinking about it.

## References

- HarmonyOS NEXT app development guide — Stage model overview
- `module.json5` spec in the OpenHarmony docs
- API Versions 11 → 12 → 13 changelog (some kit boundaries shifted)
