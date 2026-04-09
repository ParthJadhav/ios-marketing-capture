[![bloom-banner-01-light-tags-1500x500](https://github.com/user-attachments/assets/31139b9d-1b89-44e8-b563-5bb7ba150b7b)](https://bloom.parthjadhav.com)

# iOS Marketing Capture

A skill for AI-powered coding agents (Claude Code, Cursor, Windsurf, etc.) that automates marketing screenshot capture for SwiftUI iOS apps. It builds an in-app capture system, seeds demo data, snapshots every screen and UI element, and loops across all your locales automatically.

#### Built for and used by Bloom - https://apps.apple.com/us/app/bloom-coffee-shelf-recipe/id6759914524

## What it does

- Adds a `#if DEBUG`-gated capture system to your app target — zero production footprint
- Seeds deterministic demo data so every screenshot looks populated and polished
- Navigates to each screen programmatically via a step-based coordinator
- Snapshots the full window including status bar, safe area, and presented sheets
- Renders isolated elements (cards, widgets, charts) via `ImageRenderer` at 3x with transparency
- Loops every locale automatically — one build, N relaunches with `-AppleLanguages`
- Works with any SwiftUI navigation pattern: `TabView`, `NavigationStack`, `NavigationSplitView`

## Install

### Using npx skills (recommended)

```bash
npx skills add ParthJadhav/ios-marketing-capture
```

This works with Claude Code, Cursor, Windsurf, OpenCode, Codex, and [40+ other agents](https://github.com/vercel-labs/skills#available-agents).

Install globally (available across all projects):

```bash
npx skills add ParthJadhav/ios-marketing-capture -g
```

Install for a specific agent:

```bash
npx skills add ParthJadhav/ios-marketing-capture -a claude-code
```

### Manual (git clone)

```bash
git clone https://github.com/ParthJadhav/ios-marketing-capture ~/.claude/skills/ios-marketing-capture
```

## Usage

Once installed, the skill triggers automatically when you ask your agent to:

- Capture marketing screenshots for my iOS app
- Generate locale screenshots across all languages
- Render my widgets as isolated PNGs
- Automate App Store screenshot capture

Or just tell the agent what you need:

```
> Capture marketing screenshots for my app across all locales
```

The agent will ask you about your screens, elements, locales, device, appearance, and seed data before writing any code.

## Example prompts

These are good starting prompts because they provide context while still leaving room for the skill to guide the process.

### Coffee app

```text
Capture marketing screenshots for my coffee tracking app.
I want Home, Shelf, Coffee Detail, Brew Timer, and Settings.
Also render coffee cards and all my widgets as isolated elements.
Capture across en, de, es, fr, ja.
```

### Habit tracker

```text
Generate locale screenshots for my habit tracker app.
I need the dashboard, habit detail, streak view, and settings.
Light mode only, iPhone 17 simulator.
All 5 locales in my Localizable.xcstrings.
```

### Finance app

```text
Capture marketing assets for my finance app.
I want the overview, transaction list, budget detail, and charts.
Render the spending chart and category cards as isolated elements.
English and German only.
```

### Fitness app with widgets

```text
Automate App Store screenshot capture for my workout app.
Capture the main dashboard, workout detail mid-session, and history.
Render all my WidgetKit widgets (small, medium, lock screen) as isolated PNGs.
```

## Better prompt tips

- List the exact screens you want captured by tab name or navigation path
- Mention any components you want rendered independently (cards, widgets, charts)
- Specify locales explicitly or say "all locales in my xcstrings"
- Say which simulator and iOS version you want
- Mention if any screen needs to be captured in a non-default state (e.g. timer mid-countdown)
- Say light only, dark only, or both

## How it works

### In-app capture mode, not XCUITest

The skill uses an in-app capture approach instead of XCUITest / Fastlane:

- **No test target surgery** — many projects have none, and adding one means fragile pbxproj edits
- **Direct access to everything** — ViewModels, SwiftData, `ImageRenderer`, `UIWindow.drawHierarchy`
- **Faster** — `xcodebuild build` once, then `simctl launch` per locale (no test-bundle overhead)
- **Element renders require it** — `ImageRenderer` on widget views must run inside the app process

### Step-based coordinator

Each screenshot is a self-contained `CaptureStep`:

```swift
struct CaptureStep {
    let name: String                         // "01-home"
    let navigate: @MainActor () -> Void      // put the app in the right state
    let settle: Duration                     // wait for animations
    let cleanup: (@MainActor () -> Void)?    // tear down before next step
}
```

The coordinator is a simple loop — no hardcoded screen sequences. The agent composes steps for your specific navigation architecture.

### Navigation patterns

The skill covers three navigation architectures:

| Pattern | How steps drive it |
|---------|-------------------|
| `TabView(selection:)` | `setTab(index)` |
| `NavigationStack` + router | `router.push(.route)` / `router.popToRoot()` |
| `NavigationSplitView` | Set sidebar + detail selection bindings |

### Element rendering

Isolated components are rendered via `ImageRenderer` at 3x scale with natural background inside rounded corners and transparency outside:

```swift
MarketingElementHarness.renderElement(
    name: "card-morning-blend",
    width: 380,
    cornerRadius: 20,
    background: theme.background
) {
    CoffeeCard(coffee: coffee, theme: theme)
        .padding(.horizontal, 16)
        .padding(.vertical, 12)
}
```

Widget rendering handles the quirks of rendering WidgetKit views outside the widget process (missing `containerBackground`, missing padding, `ProgressView` rendering bugs).

## What gets generated

The skill guides the agent to create:

```
YourApp/
├── Debug/
│   └── MarketingCapture.swift      # Capture system (DEBUG-only)
├── ContentView.swift               # Modified — DEBUG hook for seed + coordinator
├── Views/.../TimerView.swift       # Modified — primed state hooks (if needed)
scripts/
└── capture-marketing.sh            # Build + install + per-locale loop
```

### Output layout

```
marketing/
    en/
        01-home.png
        02-detail.png
        03-settings.png
        elements/
            card-morning-blend.png
            widget-pulse-small.png
            chart-cupping.png
    de/
        ...
    es/
        ...
```

## Known gotchas

The skill documents 11 real bugs discovered during development. These are all baked into the skill's guidance so the agent avoids them automatically:

| # | Gotcha | What happens |
|---|--------|--------------|
| 1 | Live Activities persist across launches | Next locale crashes on stale SwiftData references |
| 2 | Re-seeding per locale | CloudKit sync churn causes crashes |
| 3 | VMs setup before seed | Hold stale empty snapshots |
| 4 | Setting trigger binding to nil | Doesn't dismiss fullScreenCover — wrong screenshot |
| 5 | NavigationPath can't be popped externally | Must capture clean stack before pushed detail |
| 6 | `membershipExceptions` is an INCLUSION list | Widget target membership goes backwards |
| 7 | `ImageRenderer` + `ProgressView` | Renders as prohibited symbol without explicit style |
| 8 | `.containerBackground` outside WidgetKit | No-op — widget renders have no background |
| 9 | iPhone 8 Plus gone on iOS 26 | Legacy 6.5" simulator unavailable |
| 10 | Locale launch argument format | Parens are mandatory: `(xx)` not `xx` |
| 11 | SwiftUI animations in ImageRenderer | Captures frame 0, not the animated state |

## Pairs well with

Use [app-store-screenshots](https://github.com/ParthJadhav/app-store-screenshots) as a post-processing step to composite the captured PNGs into Apple-style marketing pages with device mockups, headlines, and gradients.

## Requirements

- Xcode 16+ (synchronized folder groups support)
- iOS 17+ deployment target (for `ImageRenderer`, `@Observable`)
- A simulator runtime matching your target iOS version
- Python 3 (used by the shell script for JSON parsing)

## Contributing

Contributions are welcome, especially around:

- Support for additional navigation patterns
- New gotcha documentation from real projects
- Cross-agent compatibility improvements
- Clearer docs and onboarding

## License

MIT
