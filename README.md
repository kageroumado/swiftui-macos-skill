# SwiftUI macOS Skill

An agent skill for writing and reviewing SwiftUI code on macOS — with runtime-level understanding of observation, concurrency, view identity, and platform integration.

Built from building a production macOS browser with SwiftUI and from reading the Swift runtime source. Every pattern here was validated against actual framework behavior, not inferred from documentation. Designed for frontier models: skips basics, focuses on the non-obvious traps and deep mechanisms that even experienced developers get wrong.

## What makes this different

Most SwiftUI guidance rewrites Apple's documentation or targets iOS beginners. This skill targets macOS, assumes the model already knows Swift and SwiftUI, and explains *why* things work the way they do — the kind of understanding you'd get from a Swift language contributor, not a tutorial author.

**Runtime-level depth:**
- **Observation internals** — how `shouldNotifyObservers` overloads resolve via the type system (Equatable vs non-Equatable vs AnyObject), why `_modify` always notifies but `set` checks equality, `Observations {}` for reactive async streams
- **Scheduling cost hierarchy** — actual allocation costs from reading the Swift runtime: `assumeIsolated` (~0B) vs `MainActor.run` (~0B) vs `DispatchQueue.main.async` (~64B) vs `Task` (~300-500B), and *when each is correct*
- **View identity graph** — how structural identity creates `_ConditionalContent`, why `.id()` with unstable values destroys state, how `Equatable` conformance short-circuits body evaluation

**Patterns from production:**
- Version counters to decouple observation from collection mutations
- `@ObservationIgnored` discipline for internal caches and callbacks
- `Observations {}` streams for reactive AppKit ↔ SwiftUI bridging
- Custom `Layout` protocol over `GeometryReader` for arrangement logic
- `NSHostingView` subclassing for embedding SwiftUI in AppKit hierarchies
- Debounced SwiftData persistence and background `@ModelActor` operations

**macOS-specific:**
- Multi-window state architecture with per-window vs global state separation
- `NSViewRepresentable` + `Equatable` to prevent unnecessary `updateNSView` calls
- AppKit bridging patterns: window chrome, drag and drop, system blur, menu bar extras
- `NSBackgroundActivityScheduler` over polling Task loops for periodic work

## Installing

### Claude Code (plugin)

Add the repo as a plugin marketplace, then install:

```
/plugin marketplace add kageroumado/swiftui-macos-skill
/plugin install swiftui-macos
```

### Claude Code (manual)

Clone the repo, then copy the skill folder to the appropriate scope:

**User-wide** (recommended — available across all projects):

```bash
git clone https://github.com/kageroumado/swiftui-macos-skill.git
mkdir -p ~/.claude/skills
cp -r swiftui-macos-skill/skills/swiftui-macos ~/.claude/skills/
```

**Per-project** (shared with collaborators via version control):

```bash
git clone https://github.com/kageroumado/swiftui-macos-skill.git
mkdir -p your-project/.claude/skills
cp -r swiftui-macos-skill/skills/swiftui-macos your-project/.claude/skills/
```

### Claude.ai

1. Download or clone this repo
2. Zip the `skills/swiftui-macos` folder
3. Upload via **Settings > Capabilities > Skills**

## Usage

In Claude Code:

```
/swiftui-macos
```

In Codex:

```
$swiftui-macos
```

The skill works for both writing and reviewing. Examples:

```
/swiftui-macos Review observation discipline in the sidebar views
/swiftui-macos I'm building a multi-window document editor — help me architect the state
/swiftui-macos Focus on concurrency patterns in the networking layer
/swiftui-macos Help me bridge this NSCollectionView into SwiftUI
```

## References

| File | Coverage |
|---|---|
| `references/observation.md` | Observation tracking: state capture, `_modify` vs `set`, version counters, scope narrowing, `Observations {}` streams, computed property propagation |
| `references/concurrency.md` | Scheduling cost hierarchy, actor isolation, SE-0461/0469/0420/0471/0472, `AsyncStream` bridging, `@concurrent`, `sending` |
| `references/views.md` | View decomposition, `.task(id:)`, preference keys, custom `Layout`, animation, navigation |
| `references/data.md` | Manager pattern, `@Bindable`, SwiftData, debounced saving, `@ModelActor`, environment injection |
| `references/platform.md` | `NSViewRepresentable`, `NSHostingView`, multi-window state, `openWindow`, AppKit bridging, macOS scenes |
| `references/performance.md` | View identity graph, `.id()` dangers, `Equatable` views, `@State` for gestures, `Canvas`, `TimelineView`, `drawingGroup()` |
| `references/api.md` | Deprecated API replacements, Swift modernisms, `@Previewable`, `@Entry`, Apple open-source Swift packages |
| `references/accessibility.md` | VoiceOver, Dynamic Type, keyboard navigation, accessibility rotors, `@AccessibilityFocusState`, drag-and-drop a11y, testing |

## Contributing

Contributions welcome. Every rule should earn its token cost — if a capable model already knows it, it doesn't belong here. Focus on things that are non-obvious, frequently wrong, or require runtime-level understanding to get right.

## License

MIT License. See [LICENSE](LICENSE).
