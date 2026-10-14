# SwiftUI macOS Skill

An agent skill for macOS SwiftUI work, centered on observation internals, actor scheduling, view identity and AppKit integration. Its reference files explain framework behavior behind common failures, including redundant updates, lost view state and incorrect actor hops.

```text
/plugin marketplace add kageroumado/swiftui-macos-skill
/plugin install swiftui-macos
```

## Highlights

- Trace common SwiftUI failures to observation, scheduling, and view-identity behavior.
- Use the same skill for writing and reviewing macOS code.
- Consult focused references for AppKit integration, persistence, performance, and accessibility.

<details>
<summary>Full feature list</summary>

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

</details>

## Notes

- Invoke `/swiftui-macos` in Claude Code or `$swiftui-macos` in Codex.
- Manual installation: copy `skills/swiftui-macos` into the agent’s user or project skills directory.
- Contributions should explain recurring errors or framework behavior and include supporting evidence.
