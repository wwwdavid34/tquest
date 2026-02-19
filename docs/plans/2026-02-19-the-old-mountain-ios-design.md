# The Old Mountain — iOS App Design Document

> MVP: Core game loop. 30 turns, typewriter terminal, dice rolls, haptics, hybrid narrative engine.

---

## Decisions

| Decision            | Choice                                         |
|---------------------|-------------------------------------------------|
| Platform            | iOS (native)                                    |
| UI Framework        | SwiftUI                                         |
| Narrative Engine    | Hybrid — cached key beats + live LLM generation |
| LLM Integration     | Abstracted provider layer (Claude, GPT, etc.)   |
| LLM Output Format   | JSON mode with defined schema                   |
| MVP Scope           | Core loop only — no ghost sightings, no sharing |
| Persistence         | SwiftData, local-only                           |
| Visual Aesthetic    | Terminal / MUD — monospace, dark, typewriter     |
| ASCII Layout        | Dynamic character grid — adapts to screen width  |
| Localization        | i18n-aware from start, multi-language JSON blobs |
| Navigation          | No NavigationStack — custom terminal transitions |

---

## Architecture Overview

Scene-Driven (Approach A) with Event-Stream presentation (Approach B).

The game engine thinks in **Scenes** — self-contained turn objects with narrative, choices, and metadata. The presentation layer consumes **NarrativeEvents** — a timed stream of characters, pauses, dice rolls, and haptic cues. The `NarrativeCompiler` bridges the two.

```
App
├── GameEngine (state machine)
│   ├── GameState (turn, stats, flags, open loops)
│   ├── SceneResolver (cached or LLM?)
│   ├── DiceEngine (honest d20, SystemRandomNumberGenerator)
│   └── NarrativeCompiler (Scene → [NarrativeEvent])
├── NarrativeProviders (protocol-based)
│   ├── CachedSceneProvider (JSON from bundle → SwiftData)
│   └── LLMSceneProvider (Claude/GPT via NarrativeProvider protocol)
├── Presentation
│   ├── TerminalGrid (adaptive character grid)
│   ├── TypewriterEngine (actor, AsyncStream<TypewriterTick>)
│   ├── TerminalView (scrolling monospace text)
│   ├── DiceRollView (timed ASCII box reveal)
│   ├── ChoiceBoxView (staggered fade-in)
│   ├── ASCIIBoxRenderer (dynamic-width box drawing)
│   └── HUDBar (minimal status line)
├── HapticEngine (Core Haptics, pre-compiled patterns)
├── Persistence (SwiftData)
│   ├── ActiveGameState (save/resume)
│   ├── RunRecord + TurnLogEntry (history)
│   └── CachedScene (bundled content)
└── APIKeyStore (Keychain)
```

---

## 1. Core Data Models

### GameState — In-Memory Runtime State

```swift
struct GameState {
    var turn: Int              // 1-30
    var act: Act               // .one, .two, .three
    var path: Path             // .sword, .alchemy, .shadow, .hidden
    var stats: Stats           // qi, wisdom, honor, fate
    var flags: Set<String>     // narrative flags ("met_feng", "stole_pill")
    var openLoops: [OpenLoop]  // active unresolved threads
    var targetEnding: Ending?  // determined ~turn 20
}

struct Stats {
    var innerPower: Int
    var wisdom: Int
    var honor: Int
    var fate: Int
}

enum Act: Int, Codable { case one = 1, two, three }
enum Path: String, Codable { case sword, alchemy, shadow, hidden }
enum Ending: String, Codable {
    case immortalSovereign, demonPatriarch, wanderingSage
    case ghostCultivator, hiddenImmortal
}
```

### Scene — The Engine's Output Unit

```swift
struct Scene {
    let narrative: [NarrativeEvent]
    let choices: [Choice]
    let metadata: SceneMetadata
}

struct SceneMetadata {
    let peripheralDetail: String?
    let openLoopPlanted: String?
    let callbackSeed: CallbackRef?
    let callbackEcho: CallbackRef?
}
```

### NarrativeEvent — The Presentation Stream

```swift
enum NarrativeEvent {
    case text(String, delay: Duration)
    case pause(Duration)
    case diceRoll(DiceCheck)
    case choicesReveal([Choice], stagger: Duration)
    case haptic(HapticPattern)
}
```

### Choice

```swift
struct Choice: Identifiable {
    let id: String
    let text: LocalizedString
    let statEffects: [StatEffect]
    let flagsSet: [String]
    let diceCheck: DiceCheck?
}
```

### DiceCheck / DiceResult

```swift
struct DiceCheck {
    let die: Int          // 20
    let dc: Int
    let modifier: Int
    let stat: StatType
}

struct DiceResult {
    let raw: Int
    let modifier: Int
    let total: Int
    let dc: Int
    let passed: Bool
    let isNat20: Bool
    let isNat1: Bool
}
```

### Localization Type

```swift
typealias LocalizedString = [String: String]  // ["en": "...", "zh-Hans": "..."]
```

All narrative content (cached scenes, LLM responses, choice text, run quotes) uses `LocalizedString`. UI chrome uses Apple String Catalogs (`.xcstrings`).

---

## 2. Game Engine

### GameEngine

```swift
protocol GameEngineProtocol {
    var state: GameState { get }
    func startNewRun() async -> Scene
    func applyChoice(_ choice: Choice) async -> Scene
    func rollDice(_ check: DiceCheck) -> DiceResult
}

class GameEngine: GameEngineProtocol {
    private var state: GameState
    private let sceneResolver: SceneResolver
    private let diceEngine: DiceEngine
    private let stateRepository: GameStateRepository

    func applyChoice(_ choice: Choice) async -> Scene {
        // 1. Apply stat effects and flags to state
        // 2. Advance turn counter, update act if needed
        // 3. If turn >= 20, calculate targetEnding from stats/flags
        // 4. Resolve next scene via SceneResolver
        // 5. Persist state via GameStateRepository
        // 6. Return scene with compiled event stream
    }
}
```

### SceneResolver — Hybrid Decision Maker

```swift
class SceneResolver {
    private let cachedProvider: CachedSceneProvider
    private let llmProvider: LLMSceneProvider
    private let compiler: NarrativeCompiler

    func resolve(state: GameState, previousChoice: Choice, locale: String) async -> Scene {
        if let cached = cachedProvider.scene(for: state, locale: locale) {
            return compiler.compile(cached)
        }
        let raw = await llmProvider.generate(state: state, previousChoice: previousChoice)
        return compiler.compile(raw)
    }
}
```

**Cached vs. LLM-generated content:**

| Cached (hand-authored)          | LLM-generated                    |
|---------------------------------|----------------------------------|
| Turn 1 — Opening                | Mid-act exploration turns        |
| Act transitions (T10, T20)      | Combat encounters                |
| Prophecy planting (T5)          | NPC interactions on non-key turns|
| Prophecy payoffs                | Peripheral details               |
| All 5+ endings                  | Path-specific variations         |
| Tutorial/onboarding feel        | Connective tissue between beats  |

### DiceEngine — Honest Rolls

```swift
struct DiceEngine {
    func roll(_ check: DiceCheck) -> DiceResult {
        let raw = Int.random(in: 1...check.die)  // SystemRandomNumberGenerator
        let total = raw + check.modifier
        return DiceResult(
            raw: raw, modifier: check.modifier, total: total,
            dc: check.dc, passed: total >= check.dc,
            isNat20: raw == 20, isNat1: raw == 1
        )
    }
}
```

Never fudge results. Per the design doc: the moment players suspect results are predetermined, the variable reward mechanism collapses.

---

## 3. LLM Abstraction Layer

### NarrativeProvider Protocol

```swift
protocol NarrativeProvider {
    func generate(
        systemPrompt: String,
        gameState: GameState,
        previousChoice: Choice
    ) async throws -> RawNarrativeResponse
}

struct RawNarrativeResponse {
    let narrativeText: LocalizedString
    let choices: [RawChoice]
    let peripheralDetail: LocalizedString?
    let openLoopPlanted: LocalizedString?
}

struct RawChoice {
    let text: LocalizedString
    let impliedStatEffect: StatType
    let impliedFlag: String?
}
```

### Provider Implementations

```swift
class ClaudeProvider: NarrativeProvider { ... }
class OpenAIProvider: NarrativeProvider { ... }
```

Both use JSON mode to return structured output matching the `RawNarrativeResponse` schema.

### LLM JSON Schema

```json
{
  "narrative": { "en": "...", "zh-Hans": "..." },
  "peripheral": { "en": "...", "zh-Hans": "..." },
  "choices": [
    {
      "text": { "en": "...", "zh-Hans": "..." },
      "stat": "honor",
      "modifier": 1,
      "flag": "confronted_stranger"
    }
  ],
  "open_loop": { "en": "...", "zh-Hans": "..." }
}
```

### SystemPromptBuilder

Assembles the system prompt from the narrative design guide (Part Eight) dynamically from game state. Injects turn/act/path/stats/flags, target_ending steering (turn >= 20), golden cache examples when available, and target locale for generation language.

### NarrativeCompiler — Raw Response to Event Stream

```swift
class NarrativeCompiler {
    func compile(_ response: RawNarrativeResponse, diceCheck: DiceCheck?, locale: String) -> Scene {
        // Split narrative into lines → .text events with typewriter delays
        // Insert .pause events between lines (200ms)
        // Insert dice sequence if applicable
        // Append .choicesReveal with 150ms stagger
        // Attach haptic events at appropriate points
    }
}
```

### API Key Management

Keys stored in Keychain via `APIKeyStore`. Never in bundle, UserDefaults, or source control.

---

## 4. Presentation Layer

### TerminalGrid — Adaptive Character Grid

Calculates available monospace columns from screen width at runtime. Recalculates on rotation and size class changes.

```swift
struct TerminalGrid {
    let columns: Int
    let font: UIFont
    let charWidth: CGFloat
    let padding: CGFloat

    static func calculate(screenWidth: CGFloat, font: UIFont, horizontalPadding: CGFloat) -> TerminalGrid {
        let charWidth = "W".size(withAttributes: [.font: font]).width
        let usableWidth = screenWidth - (horizontalPadding * 2)
        let columns = Int(floor(usableWidth / charWidth))
        return TerminalGrid(columns: columns, font: font, charWidth: charWidth, padding: horizontalPadding)
    }
}
```

### CJK Double-Width Support

CJK characters occupy 2 monospace columns. `Character.terminalWidth` and `String.terminalWidth` properties feed into word-wrap and padding calculations so box borders align correctly in all locales.

### ASCIIBoxRenderer

Renders dice boxes and choice boxes using box-drawing characters (`┌─┐│└─┘`). Box width stretches to fill `grid.columns`. Content left-aligned, padding fills remainder. Word-wraps choice text respecting word boundaries and CJK character widths.

### TypewriterEngine

An `actor` that emits `AsyncStream<TypewriterTick>`. Both the view and haptic engine subscribe to the same stream for perfect synchronization.

```swift
enum TypewriterTick {
    case character(Character)
    case newLine
    case pause(Duration)
    case diceSequence(DiceCheck)
    case revealChoice(Choice)
    case haptic(HapticPattern)
}
```

Character rate: ~30-40 per second. Slightly longer pauses on punctuation. Full stop at newlines.

### DiceRollView Timing

Per the design doc — all timing is load-bearing:

```
Rolling...              300ms
.                       200ms
..                      200ms
...                     200ms
Box appears             200ms
Empty bracket           500-800ms  ← THE PAUSE (peak anticipation)
Number slams in         instant    (hard haptic)
Modifier                100ms
Total                   100ms
DC → result             300ms
```

### Screen Layout

```
┌──────────────────────────────────────┐
│  [Scrolling narrative — typewriter]  │
│                                      │
│  [ASCII dice box when rolling]       │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ [Choices in bordered box]      │  │
│  └────────────────────────────────┘  │
│                                      │
│  T14 · II · Sword · ▮▮▮▮▮░░░░░      │  ← HUD bar
└──────────────────────────────────────┘
```

HUD is minimal: turn number, act (roman numeral), path name, block-character progress bar. Stats are intentionally hidden — the player feels them through narrative consequences.

---

## 5. Haptic System

Core Haptics with pre-compiled patterns loaded at launch for zero-latency playback.

### Pattern Catalog

| Moment                | Pattern                                        |
|-----------------------|------------------------------------------------|
| Keystroke (typewriter)| Single light tap. Intensity 0.3, sharpness 0.5 |
| Choice appearing      | Soft tap per choice, 150ms stagger             |
| Choice tap (press)    | Medium tap on finger down                      |
| Choice tap (release)  | Sharp tap on finger up                         |
| Dice box appearing    | Double soft tap                                |
| Number slam           | Single hard tap. Intensity 1.0, sharpness 0.8  |
| Nat 20                | Sharp tap + 800ms low rumble (0.7→0.0)         |
| Nat 1                 | Hard tap + 2 echo taps at 150ms/300ms          |
| Callback landing      | Double tap with 400ms pause between            |
| Act transition        | 1200ms swell rumble (0.0→0.5→0.0)             |
| Ending silence        | 800ms of NOTHING — no haptic                   |
| Ending first char     | Single distant tap. Intensity 0.4              |

### Integration

Haptic engine subscribes to the same `AsyncStream<TypewriterTick>` as the view. A `.character` tick renders AND taps. A `.haptic` tick fires the pattern only. Synchronization is guaranteed by shared stream ordering.

### Fallback

Devices without Taptic Engine (older iPads): haptic calls become no-ops via `CHHapticEngine.capabilitiesForHardware()`. No substitute audio — silence is better than fake.

---

## 6. SwiftData Persistence

### RunRecord — Completed Run History

```swift
@Model class RunRecord {
    var id: UUID
    var startedAt: Date
    var completedAt: Date?
    var isActive: Bool
    var finalTurn: Int
    var path: String
    var ending: String?
    var finalStats: StatsSnapshot
    var turnLog: [TurnLogEntry]
    var runQuote: LocalizedString?
}
```

### TurnLogEntry — Per-Turn Record

```swift
@Model class TurnLogEntry {
    var turn: Int
    var act: Int
    var narrative: String
    var choiceMade: String
    var diceResult: DiceResultSnapshot?
    var flagsAfter: [String]
    var statsAfter: StatsSnapshot
    var timestamp: Date
}
```

### ActiveGameState — Save & Resume

```swift
@Model class ActiveGameState {
    var runId: UUID
    var turn: Int
    var act: Int
    var path: String
    var stats: StatsSnapshot
    var flags: [String]
    var openLoops: [String]
    var targetEnding: String?
    var lastSceneEvents: Data?  // encoded [NarrativeEvent] for redisplay on resume
}
```

Engine saves after every choice. On resume, `lastSceneEvents` is decoded and displayed instantly (no typewriter replay, no re-querying the LLM).

### CachedScene — Bundled Content

```swift
@Model class CachedScene {
    var turn: Int
    var act: Int
    var path: String            // "all", "sword", "alchemy", "shadow"
    var requiredFlags: [String]
    var priority: Int
    var content: Data           // encoded SceneContent with LocalizedString fields
}
```

Loaded from JSON files in the app bundle on first launch. Versioned — re-imported only when bundle version changes.

### GameStateRepository

Handles save, load-active, and complete-run operations. Single source of truth for persistence.

### Schema

```
RunRecord 1──< TurnLogEntry
RunRecord 1──1 ActiveGameState (while active)
CachedScene (independent, loaded from bundle)
```

---

## 7. App Structure & Navigation

### Screens

| Screen        | Purpose                                         |
|---------------|-------------------------------------------------|
| HomeScreen    | Title types itself, menu options fade in         |
| GameScreen    | Core loop — scrolling terminal, 95% of time here|
| EndingScreen  | 800ms silence → ending title → run quote → share|
| ArchiveScreen | Run history with quotes, endings discovered      |
| ReplayScreen  | Read-only scroll through a completed run's log   |
| SettingsScreen| LLM provider, text speed, haptics, language, CRT |
| PauseOverlay  | Translucent overlay on frozen terminal           |

### Navigation

No `NavigationStack`. Custom `ZStack` with terminal-style transitions (clear → type-in). Every screen change feels like a terminal reloading, not an iOS app navigating.

```swift
enum AppScreen {
    case home
    case game(UUID)
    case ending(Ending, LocalizedString)
    case archive
    case replay(UUID)
    case settings
}
```

### Launch Flow

1. App launches → check for `ActiveGameState`
2. If active run exists → `GameScreen` (resume, display last scene instantly)
3. If no active run → `HomeScreen`

### Pause

Two-finger tap or swipe-down. Translucent overlay. Options: Resume, Abandon Run, Settings.

### Ending Flow

1. Final narrative turn types
2. 800ms absolute silence (no haptic)
3. Screen clears to black
4. Single distant haptic tap
5. Ending title types at half speed
6. Run quote fades in
7. Share / Archives / Begin Again options appear

---

## Future (Post-MVP)

Not in scope for v1, but the architecture accommodates:

- **Ghost Sighting System** — requires backend (Firebase/Supabase). `RunRecord` already stores `ending` and turn data needed to generate ghost events.
- **Sharing Architecture** — auto-quote already generated and stored. Share sheets and "What Would You Do?" cards are UI-only additions.
- **Prophecy Callbacks** — the `CallbackRef` in `SceneMetadata` and `CachedScene` already tracks seeds and echoes. Wiring them to the LLM system prompt is prompt engineering, not architecture.
- **Cloud Sync** — SwiftData supports CloudKit integration. Models are already `@Model`.
- **Additional Locales** — `LocalizedString` dictionary pattern extends to any number of locales without schema changes.

---

*The Old Mountain — iOS Design Document v1.0*
*2026-02-19*
