# macOS Scene & Window API Catalog

What exists, when it arrived, and only the behavior you would not guess from the name.
`platform.md` covers `NSViewRepresentable`, `NSHostingView`, AppKit bridging, and
multi-window *state* — this file is the scene and window-chrome API surface.

Availability verified against the Xcode 26.6 `SwiftUI.swiftinterface`.

## Scenes

| API | Since | macOS-only |
|---|---|:--:|
| `WindowGroup` | 11.0 | no |
| `Window` | 13.0 | no |
| `UtilityWindow` | 15.0 | **yes** |
| `Settings` | 11.0 | **yes** |
| `MenuBarExtra` | 13.0 | **yes** |
| `DocumentGroup` | 11.0 | no |

- **`WindowGroup` vs `Window`**: a group supports many instances, window tabbing, and the
  automatic Window menu. `Window` is a singleton the system keeps unique. The behavioral
  difference that bites: **`WindowGroup` keeps the app alive after its last window closes;
  a `Window` that is the app's sole scene quits the app when closed.** Prefer `WindowGroup`
  for the primary scene, `Window` for supplementary singletons.
- **`UtilityWindow`** is a floating palette/inspector. It receives `FocusedValues` from
  whichever main window is focused (the same mechanism menu commands use), floats at
  `.floating` level, hides when the app deactivates, takes focus only when explicitly
  clicked, dismisses on Escape, is not minimizable, and **adds its own show/hide item to the
  View menu** — remove that with `.commandsRemoved()` and place a `WindowVisibilityToggle`
  yourself if you want it elsewhere.

```swift
UtilityWindow("Photo Info", id: "photo-info") { PhotoInfoViewer() }

struct PhotoInfoViewer: View {
    @FocusedValue(PhotoSelection.self) private var selection   // tracks the focused window
    var body: some View { Text(selection.map { "\($0.count) selected" } ?? "No selection") }
}
```

- **`Settings`** supplies the Cmd+, item and window lifecycle for free. Reach it in code with
  `SettingsLink` (macOS 14) or `@Environment(\.openSettings)` (macOS 14).
- **`DocumentGroup`** brings the whole File menu (New/Open/Save/Revert) and multiple document
  windows. Document type conforms to `FileDocument` (value) or `ReferenceFileDocument`
  (reference); `DocumentGroup(viewing:)` for read-only formats.

Gate macOS-only scenes with `#if os(macOS)` in multiplatform apps.

## MenuBarExtra

```swift
MenuBarExtra("Status", systemImage: "chart.bar") { DashboardView().frame(width: 240) }
    .menuBarExtraStyle(.window)     // .menu (default) or .window
```

- `.menu` is a standard dropdown; `.window` is a popover panel hosting arbitrary SwiftUI.
- `isInserted:` takes a binding (typically `@AppStorage`) so users can show/hide the item.
- **Menu-bar-only app**: `MenuBarExtra` as the sole scene plus `LSUIElement = true` in
  Info.plist hides the Dock icon. The app auto-terminates if the user drags the extra out of
  the menu bar — decide whether that is what you want.

## Window chrome

| API | Since | macOS-only |
|---|---|:--:|
| `windowStyle(_:)` | 11.0 | no |
| `windowToolbarStyle(_:)` | 11.0 | **yes** |
| `defaultSize`, `defaultPosition`, `windowResizability` | 13.0 | no |
| `windowIdealPlacement(_:)` | 15.0 | no |

- `windowToolbarStyle`: `.automatic`, `.unified`, `.unifiedCompact`, `.expanded`, and
  `.unified(showsTitle: false)`. `.expanded` puts the title bar above the toolbar and is
  only worth it when you have many toolbar items.
- `windowStyle(.hiddenTitleBar)` for chromeless windows (media players, custom chrome).
- **Sizing is a three-part contract**: set `minWidth`/`minHeight` via `.frame()` on the
  *content*, then `.windowResizability(.contentMinSize)` to enforce them, then
  `.defaultSize()` for the opening size. `.contentSize` fixes the window and disables the
  zoom button; `.automatic` lets the system decide.
- `windowIdealPlacement` gives a closure with display geometry for precise placement.

## Navigation chrome

- `NavigationSplitView` on macOS always shows columns side by side — never overlaid — with a
  translucent sidebar and user-resizable columns. Size them with
  `.navigationSplitViewColumnWidth(min:ideal:max:)`.
- `Inspector` (macOS 14) is the trailing-edge panel, sized with
  `.inspectorColumnWidth(min:ideal:max:)`.
- Use `NavigationSplitView` for sidebar-driven navigation. `HSplitView` / `VSplitView`
  (macOS 10.15, macOS-only) are for IDE-style layouts where the panes are equal peers, not
  a sidebar driving a detail view.

## Commands

```swift
.commands {
    CommandMenu("Tools") {
        Button("Run Analysis") { }.keyboardShortcut("r", modifiers: [.command, .shift])
    }
    CommandGroup(after: .newItem) { Button("New From Template…") { } }
}
```

`CommandGroup` placement: `.replacing(_:)`, `.before(_:)`, `.after(_:)`, against system
groups such as `.newItem`, `.saveItem`, `.help`, `.toolbar`, `.sidebar`. On macOS these
populate the menu bar and shortcuts appear next to menu items and in button tooltips.

## Views and file operations

| API | Since | macOS-only | Note |
|---|---|:--:|---|
| `Table` | 12.0 | no | `.tableStyle(.bordered(alternatesRowBackgrounds:))`, `.inset`; `.tableColumnHeaders(.hidden)` |
| `PasteButton` | 10.15 | no | **does not auto-validate the pasteboard on macOS**, unlike iOS |
| `CopyButton` | 15.0 | **yes** | copies `Transferable` content |
| `fileImporter` / `fileExporter` / `fileMover` | 11.0 | no | native NSOpenPanel/NSSavePanel |
| `fileDialogMessage`, `fileDialogConfirmationLabel`, `fileExporterFilenameLabel` | 13.0 | **yes** | customize dialog text |

**URLs from `fileImporter` are security-scoped.** Access fails without bracketing:

```swift
guard url.startAccessingSecurityScopedResource() else { return }
defer { url.stopAccessingSecurityScopedResource() }
```

Drag and drop on macOS crosses application boundaries — a `draggable()` item can land in
Finder or Mail, and `dropDestination(for:)` accepts drops from any app. Prefer `Transferable`
over `NSItemProvider`; the latter is legacy compatibility only.
