# The Old Mountain — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build the MVP iOS app — 30-turn text RPG with typewriter terminal, dice rolls, haptics, and hybrid LLM narrative engine.

**Architecture:** Scene-driven engine with event-stream presentation. GameEngine produces Scenes, NarrativeCompiler converts them to timed NarrativeEvent streams, TypewriterEngine plays them through TerminalView and HapticEngine in sync.

**Tech Stack:** Swift, SwiftUI, SwiftData, Core Haptics, Anthropic/OpenAI APIs, XCTest

**Test command pattern:**
```bash
xcodebuild test -project TheOldMountain/TheOldMountain.xcodeproj \
  -scheme TheOldMountain \
  -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest' \
  -only-testing:TheOldMountainTests/{TestClass}
```

---

## Project Directory Structure

```
TheOldMountain/
├── TheOldMountain.xcodeproj
├── TheOldMountain/
│   ├── App/
│   │   ├── TheOldMountainApp.swift
│   │   └── RootView.swift
│   ├── Models/
│   │   ├── GameState.swift
│   │   ├── Scene.swift
│   │   ├── NarrativeEvent.swift
│   │   ├── Choice.swift
│   │   └── DiceTypes.swift
│   ├── Engine/
│   │   ├── GameEngine.swift
│   │   ├── SceneResolver.swift
│   │   ├── DiceEngine.swift
│   │   └── NarrativeCompiler.swift
│   ├── Providers/
│   │   ├── NarrativeProvider.swift
│   │   ├── CachedSceneProvider.swift
│   │   ├── LLMSceneProvider.swift
│   │   ├── ClaudeProvider.swift
│   │   ├── OpenAIProvider.swift
│   │   └── SystemPromptBuilder.swift
│   ├── Presentation/
│   │   ├── TerminalGrid.swift
│   │   ├── ASCIIBoxRenderer.swift
│   │   ├── TypewriterEngine.swift
│   │   ├── CJKWidth.swift
│   │   ├── TerminalView.swift
│   │   ├── DiceRollView.swift
│   │   ├── ChoiceBoxView.swift
│   │   └── HUDBar.swift
│   ├── Haptics/
│   │   ├── HapticEngine.swift
│   │   └── HapticPattern.swift
│   ├── Persistence/
│   │   ├── RunRecord.swift
│   │   ├── TurnLogEntry.swift
│   │   ├── ActiveGameState.swift
│   │   ├── CachedSceneModel.swift
│   │   ├── GameStateRepository.swift
│   │   └── RunHistoryRepository.swift
│   ├── Security/
│   │   └── APIKeyStore.swift
│   ├── Screens/
│   │   ├── HomeScreen.swift
│   │   ├── GameScreen.swift
│   │   ├── GameViewModel.swift
│   │   ├── EndingScreen.swift
│   │   ├── ArchiveScreen.swift
│   │   ├── ReplayScreen.swift
│   │   ├── SettingsScreen.swift
│   │   └── PauseOverlay.swift
│   └── Resources/
│       ├── CachedScenes/
│       └── Assets.xcassets
├── TheOldMountainTests/
│   ├── DiceEngineTests.swift
│   ├── GameStateTests.swift
│   ├── TerminalGridTests.swift
│   ├── ASCIIBoxRendererTests.swift
│   ├── CJKWidthTests.swift
│   ├── NarrativeCompilerTests.swift
│   ├── SystemPromptBuilderTests.swift
│   ├── SceneResolverTests.swift
│   ├── GameEngineTests.swift
│   └── GameStateRepositoryTests.swift
```

---

## Task 1: Xcode Project Scaffold

**Files:**
- Create: `TheOldMountain/TheOldMountain.xcodeproj` (via Xcode CLI)
- Create: `TheOldMountain/TheOldMountain/App/TheOldMountainApp.swift`

**Step 1: Create Xcode project**

```bash
cd /Users/fhsu/Documents/app_dev/tquest
mkdir -p TheOldMountain
# Use xcodegen or create via Xcode. Minimal SwiftUI app target + test target.
# Deployment target: iOS 17.0 (SwiftData requires 17+)
```

If using `xcodegen`, create `TheOldMountain/project.yml`:

```yaml
name: TheOldMountain
options:
  bundleIdPrefix: com.tquest
  deploymentTarget:
    iOS: "17.0"
targets:
  TheOldMountain:
    type: application
    platform: iOS
    sources: [TheOldMountain]
    settings:
      PRODUCT_BUNDLE_IDENTIFIER: com.tquest.theoldmountain
      INFOPLIST_KEY_UILaunchScreen_Generation: true
      SWIFT_STRICT_CONCURRENCY: complete
  TheOldMountainTests:
    type: bundle.unit-test
    platform: iOS
    sources: [TheOldMountainTests]
    dependencies:
      - target: TheOldMountain
```

**Step 2: Create minimal app entry point**

`TheOldMountain/TheOldMountain/App/TheOldMountainApp.swift`:
```swift
import SwiftUI
import SwiftData

@main
struct TheOldMountainApp: App {
    var body: some Scene {
        WindowGroup {
            Text("The Old Mountain")
                .font(.custom("Menlo", size: 16))
                .foregroundStyle(.green)
                .frame(maxWidth: .infinity, maxHeight: .infinity)
                .background(.black)
                .preferredColorScheme(.dark)
        }
    }
}
```

**Step 3: Create directory structure**

```bash
cd /Users/fhsu/Documents/app_dev/tquest/TheOldMountain/TheOldMountain
mkdir -p App Models Engine Providers Presentation Haptics Persistence Security Screens Resources/CachedScenes
cd /Users/fhsu/Documents/app_dev/tquest/TheOldMountain
mkdir -p TheOldMountainTests
```

**Step 4: Build to verify**

```bash
xcodebuild build -project TheOldMountain.xcodeproj \
  -scheme TheOldMountain \
  -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest'
```
Expected: BUILD SUCCEEDED

**Step 5: Commit**

```bash
git add TheOldMountain/
git commit -m "feat: scaffold Xcode project with directory structure"
```

---

## Task 2: Core Data Models

**Files:**
- Create: `TheOldMountain/TheOldMountain/Models/GameState.swift`
- Create: `TheOldMountain/TheOldMountain/Models/Scene.swift`
- Create: `TheOldMountain/TheOldMountain/Models/NarrativeEvent.swift`
- Create: `TheOldMountain/TheOldMountain/Models/Choice.swift`
- Create: `TheOldMountain/TheOldMountain/Models/DiceTypes.swift`
- Test: `TheOldMountain/TheOldMountainTests/GameStateTests.swift`

**Step 1: Write failing tests for GameState**

```swift
// GameStateTests.swift
import XCTest
@testable import TheOldMountain

final class GameStateTests: XCTestCase {
    func testInitialState() {
        let state = GameState.newRun()
        XCTAssertEqual(state.turn, 1)
        XCTAssertEqual(state.act, .one)
        XCTAssertEqual(state.stats.innerPower, 0)
        XCTAssertEqual(state.stats.wisdom, 0)
        XCTAssertEqual(state.stats.honor, 0)
        XCTAssertEqual(state.stats.fate, 0)
        XCTAssertTrue(state.flags.isEmpty)
        XCTAssertNil(state.targetEnding)
    }

    func testActProgression() {
        XCTAssertEqual(GameState.act(forTurn: 1), .one)
        XCTAssertEqual(GameState.act(forTurn: 10), .one)
        XCTAssertEqual(GameState.act(forTurn: 11), .two)
        XCTAssertEqual(GameState.act(forTurn: 20), .two)
        XCTAssertEqual(GameState.act(forTurn: 21), .three)
        XCTAssertEqual(GameState.act(forTurn: 30), .three)
    }

    func testApplyStatEffects() {
        var state = GameState.newRun()
        let effects: [StatEffect] = [
            StatEffect(stat: .innerPower, modifier: 2),
            StatEffect(stat: .honor, modifier: -1)
        ]
        state.applyEffects(effects)
        XCTAssertEqual(state.stats.innerPower, 2)
        XCTAssertEqual(state.stats.honor, -1)
    }

    func testAdvanceTurn() {
        var state = GameState.newRun()
        state.advanceTurn()
        XCTAssertEqual(state.turn, 2)
        XCTAssertEqual(state.act, .one)
    }

    func testAdvanceTurnCrossesActBoundary() {
        var state = GameState.newRun()
        state.turn = 10
        state.advanceTurn()
        XCTAssertEqual(state.turn, 11)
        XCTAssertEqual(state.act, .two)
    }

    func testAddFlags() {
        var state = GameState.newRun()
        state.addFlags(["met_feng", "stole_pill"])
        XCTAssertTrue(state.flags.contains("met_feng"))
        XCTAssertTrue(state.flags.contains("stole_pill"))
    }
}
```

**Step 2: Run test to verify it fails**

Expected: FAIL — `GameState` not defined

**Step 3: Implement all model files**

`Models/GameState.swift`:
```swift
import Foundation

typealias LocalizedString = [String: String]

enum Act: Int, Codable, Equatable {
    case one = 1, two, three
}

enum Path: String, Codable, Equatable {
    case sword, alchemy, shadow, hidden
}

enum Ending: String, Codable, Equatable {
    case immortalSovereign, demonPatriarch, wanderingSage
    case ghostCultivator, hiddenImmortal
}

enum StatType: String, Codable, Equatable {
    case innerPower, wisdom, honor, fate
}

struct StatEffect: Equatable {
    let stat: StatType
    let modifier: Int
}

struct Stats: Equatable, Codable {
    var innerPower: Int = 0
    var wisdom: Int = 0
    var honor: Int = 0
    var fate: Int = 0

    mutating func apply(_ effect: StatEffect) {
        switch effect.stat {
        case .innerPower: innerPower += effect.modifier
        case .wisdom: wisdom += effect.modifier
        case .honor: honor += effect.modifier
        case .fate: fate += effect.modifier
        }
    }
}

struct OpenLoop: Codable, Equatable {
    let description: String
    let turnPlanted: Int
}

struct GameState: Equatable {
    var turn: Int
    var act: Act
    var path: Path
    var stats: Stats
    var flags: Set<String>
    var openLoops: [OpenLoop]
    var targetEnding: Ending?

    static func newRun() -> GameState {
        GameState(
            turn: 1, act: .one, path: .sword,
            stats: Stats(), flags: [],
            openLoops: [], targetEnding: nil
        )
    }

    static func act(forTurn turn: Int) -> Act {
        switch turn {
        case 1...10: return .one
        case 11...20: return .two
        default: return .three
        }
    }

    mutating func advanceTurn() {
        turn += 1
        act = Self.act(forTurn: turn)
    }

    mutating func applyEffects(_ effects: [StatEffect]) {
        for effect in effects { stats.apply(effect) }
    }

    mutating func addFlags(_ newFlags: [String]) {
        flags.formUnion(newFlags)
    }
}
```

`Models/DiceTypes.swift`:
```swift
import Foundation

struct DiceCheck: Equatable {
    let die: Int
    let dc: Int
    let modifier: Int
    let stat: StatType
}

struct DiceResult: Equatable {
    let raw: Int
    let modifier: Int
    let total: Int
    let dc: Int
    let passed: Bool
    let isNat20: Bool
    let isNat1: Bool
}
```

`Models/Choice.swift`:
```swift
import Foundation

struct Choice: Identifiable, Equatable {
    let id: String
    let text: LocalizedString
    let statEffects: [StatEffect]
    let flagsSet: [String]
    let diceCheck: DiceCheck?
}
```

`Models/NarrativeEvent.swift`:
```swift
import Foundation

enum HapticPatternType: Equatable {
    case keystroke, choiceAppear, choicePress, choiceRelease
    case diceBoxAppear, numberSlam, nat20, nat1
    case callbackLand, actTransition, endingSilence, endingFirstChar
}

enum NarrativeEvent: Equatable {
    case text(String, delay: Duration)
    case pause(Duration)
    case diceRoll(DiceCheck)
    case choicesReveal([Choice], stagger: Duration)
    case haptic(HapticPatternType)
}
```

`Models/Scene.swift`:
```swift
import Foundation

struct CallbackRef: Equatable, Codable {
    let id: String
    let targetTurn: Int?
}

struct SceneMetadata: Equatable {
    let peripheralDetail: String?
    let openLoopPlanted: String?
    let callbackSeed: CallbackRef?
    let callbackEcho: CallbackRef?
}

struct Scene: Equatable {
    let narrative: [NarrativeEvent]
    let choices: [Choice]
    let metadata: SceneMetadata
}
```

**Step 4: Run tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Models/ TheOldMountainTests/GameStateTests.swift
git commit -m "feat: add core data models with GameState tests"
```

---

## Task 3: DiceEngine

**Files:**
- Create: `TheOldMountain/TheOldMountain/Engine/DiceEngine.swift`
- Test: `TheOldMountain/TheOldMountainTests/DiceEngineTests.swift`

**Step 1: Write failing tests**

```swift
// DiceEngineTests.swift
import XCTest
@testable import TheOldMountain

final class DiceEngineTests: XCTestCase {
    let engine = DiceEngine()

    func testRollResultRange() {
        let check = DiceCheck(die: 20, dc: 15, modifier: 0, stat: .innerPower)
        for _ in 0..<100 {
            let result = engine.roll(check)
            XCTAssertGreaterThanOrEqual(result.raw, 1)
            XCTAssertLessThanOrEqual(result.raw, 20)
            XCTAssertEqual(result.total, result.raw + result.modifier)
        }
    }

    func testModifierApplied() {
        let check = DiceCheck(die: 20, dc: 10, modifier: 5, stat: .wisdom)
        let result = engine.roll(check)
        XCTAssertEqual(result.total, result.raw + 5)
        XCTAssertEqual(result.modifier, 5)
    }

    func testPassFailLogic() {
        let check = DiceCheck(die: 20, dc: 15, modifier: 0, stat: .honor)
        for _ in 0..<200 {
            let result = engine.roll(check)
            XCTAssertEqual(result.passed, result.total >= check.dc)
        }
    }

    func testNat20Detection() {
        // Use seeded engine for deterministic test
        let seeded = DiceEngine(seed: 42)
        var found20 = false
        for _ in 0..<1000 {
            let check = DiceCheck(die: 20, dc: 1, modifier: 0, stat: .fate)
            let result = seeded.roll(check)
            if result.raw == 20 {
                XCTAssertTrue(result.isNat20)
                XCTAssertFalse(result.isNat1)
                found20 = true
                break
            }
        }
        XCTAssertTrue(found20, "Should find a nat 20 in 1000 rolls")
    }

    func testNat1Detection() {
        let seeded = DiceEngine(seed: 7)
        var found1 = false
        for _ in 0..<1000 {
            let check = DiceCheck(die: 20, dc: 1, modifier: 0, stat: .fate)
            let result = seeded.roll(check)
            if result.raw == 1 {
                XCTAssertTrue(result.isNat1)
                XCTAssertFalse(result.isNat20)
                found1 = true
                break
            }
        }
        XCTAssertTrue(found1, "Should find a nat 1 in 1000 rolls")
    }

    func testDCField() {
        let check = DiceCheck(die: 20, dc: 18, modifier: 3, stat: .innerPower)
        let result = engine.roll(check)
        XCTAssertEqual(result.dc, 18)
    }
}
```

**Step 2: Run test to verify it fails**

Expected: FAIL — `DiceEngine` not defined

**Step 3: Implement DiceEngine**

```swift
// Engine/DiceEngine.swift
import Foundation

struct DiceEngine {
    private var rng: any RandomNumberGenerator

    init() {
        self.rng = SystemRandomNumberGenerator()
    }

    init(seed: UInt64) {
        self.rng = SeededRandomNumberGenerator(seed: seed)
    }

    mutating func roll(_ check: DiceCheck) -> DiceResult {
        let raw = Int.random(in: 1...check.die, using: &rng)
        let total = raw + check.modifier
        return DiceResult(
            raw: raw, modifier: check.modifier, total: total,
            dc: check.dc, passed: total >= check.dc,
            isNat20: raw == check.die, isNat1: raw == 1
        )
    }
}

// Deterministic RNG for testing
struct SeededRandomNumberGenerator: RandomNumberGenerator {
    private var state: UInt64
    init(seed: UInt64) { self.state = seed }
    mutating func next() -> UInt64 {
        state &+= 0x9e3779b97f4a7c15
        var z = state
        z = (z ^ (z >> 30)) &* 0xbf58476d1ce4e5b9
        z = (z ^ (z >> 27)) &* 0x94d049bb133111eb
        return z ^ (z >> 31)
    }
}
```

**Step 4: Run tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Engine/DiceEngine.swift TheOldMountainTests/DiceEngineTests.swift
git commit -m "feat: add DiceEngine with honest d20 rolls and seeded testing"
```

---

## Task 4: TerminalGrid + CJK Width

**Files:**
- Create: `TheOldMountain/TheOldMountain/Presentation/CJKWidth.swift`
- Create: `TheOldMountain/TheOldMountain/Presentation/TerminalGrid.swift`
- Test: `TheOldMountain/TheOldMountainTests/CJKWidthTests.swift`
- Test: `TheOldMountain/TheOldMountainTests/TerminalGridTests.swift`

**Step 1: Write failing tests**

```swift
// CJKWidthTests.swift
import XCTest
@testable import TheOldMountain

final class CJKWidthTests: XCTestCase {
    func testLatinCharWidth() {
        XCTAssertEqual(Character("A").terminalWidth, 1)
        XCTAssertEqual(Character("z").terminalWidth, 1)
        XCTAssertEqual(Character(" ").terminalWidth, 1)
    }

    func testCJKCharWidth() {
        XCTAssertEqual(Character("山").terminalWidth, 2)
        XCTAssertEqual(Character("内").terminalWidth, 2)
        XCTAssertEqual(Character("力").terminalWidth, 2)
    }

    func testStringTerminalWidth() {
        XCTAssertEqual("Hello".terminalWidth, 5)
        XCTAssertEqual("内力".terminalWidth, 4)
        XCTAssertEqual("Qi 内力".terminalWidth, 7) // Q=1 i=1 space=1 内=2 力=2
    }

    func testBoxDrawingCharsWidth() {
        XCTAssertEqual(Character("┌").terminalWidth, 1)
        XCTAssertEqual(Character("─").terminalWidth, 1)
        XCTAssertEqual(Character("│").terminalWidth, 1)
    }
}
```

```swift
// TerminalGridTests.swift
import XCTest
@testable import TheOldMountain

final class TerminalGridTests: XCTestCase {
    func testColumnCalculation() {
        // Menlo 14pt has ~8.4px char width on iOS
        // Screen 390px wide, 16px padding each side = 358px usable
        // 358 / 8.4 = 42 columns
        let grid = TerminalGrid.calculate(
            screenWidth: 390, charWidth: 8.4, horizontalPadding: 16
        )
        XCTAssertEqual(grid.columns, 42)
    }

    func testNarrowScreen() {
        // iPhone SE: 375px
        let grid = TerminalGrid.calculate(
            screenWidth: 375, charWidth: 8.4, horizontalPadding: 16
        )
        XCTAssertEqual(grid.columns, 40)
    }

    func testWideScreen() {
        // iPad landscape: ~1024px
        let grid = TerminalGrid.calculate(
            screenWidth: 1024, charWidth: 8.4, horizontalPadding: 16
        )
        XCTAssertEqual(grid.columns, 118)
    }

    func testWordWrap() {
        let result = TerminalGrid.wordWrap(
            "Step forward. Let him see your face.", width: 20
        )
        XCTAssertEqual(result, [
            "Step forward. Let",
            "him see your face."
        ])
    }

    func testWordWrapCJK() {
        // CJK chars take 2 columns each
        let result = TerminalGrid.wordWrap("上前一步。让她看清你的脸。", width: 20)
        // 上(2)前(2)一(2)步(2)。(1) = 9 cols
        // 让(2)她(2)看(2)清(2)你(2) = 10 cols → 19 total
        // 的(2)脸(2)。(1) = 5 cols
        XCTAssertTrue(result.count >= 2)
        for line in result {
            XCTAssertLessThanOrEqual(line.terminalWidth, 20)
        }
    }

    func testPadRight() {
        XCTAssertEqual("Hello".padRight(toTerminalWidth: 10), "Hello     ")
        XCTAssertEqual("内力".padRight(toTerminalWidth: 10), "内力      ")
    }
}
```

**Step 2: Run tests to verify they fail**

**Step 3: Implement CJKWidth.swift**

```swift
// Presentation/CJKWidth.swift
import Foundation

extension Character {
    var terminalWidth: Int {
        guard let scalar = unicodeScalars.first else { return 1 }
        let v = scalar.value
        // CJK Unified Ideographs
        if (0x4E00...0x9FFF).contains(v) { return 2 }
        // CJK Extension A
        if (0x3400...0x4DBF).contains(v) { return 2 }
        // CJK Compatibility Ideographs
        if (0xF900...0xFAFF).contains(v) { return 2 }
        // Fullwidth Forms
        if (0xFF01...0xFF60).contains(v) { return 2 }
        // CJK Symbols and Punctuation
        if (0x3000...0x303F).contains(v) { return 2 }
        // Hiragana / Katakana
        if (0x3040...0x30FF).contains(v) { return 2 }
        // Hangul Syllables
        if (0xAC00...0xD7AF).contains(v) { return 2 }
        return 1
    }
}

extension String {
    var terminalWidth: Int {
        reduce(0) { $0 + $1.terminalWidth }
    }

    func padRight(toTerminalWidth width: Int) -> String {
        let current = self.terminalWidth
        guard current < width else { return self }
        return self + String(repeating: " ", count: width - current)
    }
}
```

**Step 4: Implement TerminalGrid.swift**

```swift
// Presentation/TerminalGrid.swift
import Foundation

struct TerminalGrid: Equatable {
    let columns: Int
    let charWidth: CGFloat
    let padding: CGFloat

    static func calculate(
        screenWidth: CGFloat,
        charWidth: CGFloat,
        horizontalPadding: CGFloat
    ) -> TerminalGrid {
        let usableWidth = screenWidth - (horizontalPadding * 2)
        let columns = Int(floor(usableWidth / charWidth))
        return TerminalGrid(
            columns: columns, charWidth: charWidth, padding: horizontalPadding
        )
    }

    static func wordWrap(_ text: String, width: Int) -> [String] {
        var lines: [String] = []
        var currentLine = ""
        var currentWidth = 0

        for char in text {
            let cw = char.terminalWidth
            if currentWidth + cw > width && !currentLine.isEmpty {
                // Try to break at last space
                if let lastSpace = currentLine.lastIndex(of: " ") {
                    let before = String(currentLine[currentLine.startIndex...lastSpace]).trimmingCharacters(in: .whitespaces)
                    let after = String(currentLine[currentLine.index(after: lastSpace)...])
                    lines.append(before)
                    currentLine = after + String(char)
                    currentWidth = after.terminalWidth + cw
                } else {
                    lines.append(currentLine)
                    currentLine = String(char)
                    currentWidth = cw
                }
            } else {
                currentLine.append(char)
                currentWidth += cw
            }
        }
        if !currentLine.isEmpty {
            lines.append(currentLine)
        }
        return lines
    }
}
```

**Step 5: Run tests**

Expected: ALL PASS

**Step 6: Commit**

```bash
git add TheOldMountain/TheOldMountain/Presentation/CJKWidth.swift \
        TheOldMountain/TheOldMountain/Presentation/TerminalGrid.swift \
        TheOldMountainTests/CJKWidthTests.swift \
        TheOldMountainTests/TerminalGridTests.swift
git commit -m "feat: add TerminalGrid with adaptive columns and CJK width support"
```

---

## Task 5: ASCIIBoxRenderer

**Files:**
- Create: `TheOldMountain/TheOldMountain/Presentation/ASCIIBoxRenderer.swift`
- Test: `TheOldMountain/TheOldMountainTests/ASCIIBoxRendererTests.swift`

**Step 1: Write failing tests**

```swift
// ASCIIBoxRendererTests.swift
import XCTest
@testable import TheOldMountain

final class ASCIIBoxRendererTests: XCTestCase {
    func testDiceBoxWidth() {
        let renderer = ASCIIBoxRenderer(columns: 40)
        let result = DiceResult(
            raw: 17, modifier: 2, total: 19,
            dc: 15, passed: true, isNat20: false, isNat1: false
        )
        let lines = renderer.renderDiceBox(result)

        // Every line should be exactly 40 characters wide
        for line in lines {
            XCTAssertEqual(line.terminalWidth, 40, "Line: '\(line)'")
        }
        // First and last lines are borders
        XCTAssertTrue(lines.first!.hasPrefix("┌"))
        XCTAssertTrue(lines.last!.hasPrefix("└"))
    }

    func testDiceBoxContainsResult() {
        let renderer = ASCIIBoxRenderer(columns: 40)
        let result = DiceResult(
            raw: 17, modifier: 2, total: 19,
            dc: 15, passed: true, isNat20: false, isNat1: false
        )
        let joined = renderer.renderDiceBox(result).joined()
        XCTAssertTrue(joined.contains("17"))
        XCTAssertTrue(joined.contains("+2"))
        XCTAssertTrue(joined.contains("19"))
        XCTAssertTrue(joined.contains("PASS"))
    }

    func testDiceBoxShowsFail() {
        let renderer = ASCIIBoxRenderer(columns: 40)
        let result = DiceResult(
            raw: 3, modifier: 1, total: 4,
            dc: 15, passed: false, isNat20: false, isNat1: false
        )
        let joined = renderer.renderDiceBox(result).joined()
        XCTAssertTrue(joined.contains("FAIL"))
    }

    func testChoiceBoxWrapsLongText() {
        let renderer = ASCIIBoxRenderer(columns: 30)
        let choices = [
            Choice(
                id: "c1",
                text: ["en": "Step forward. Let him see your face."],
                statEffects: [], flagsSet: [], diceCheck: nil
            )
        ]
        let lines = renderer.renderChoiceBox(choices, locale: "en")
        for line in lines {
            XCTAssertLessThanOrEqual(
                line.terminalWidth, 30,
                "Line exceeded width: '\(line)'"
            )
        }
    }

    func testChoiceBoxDifferentWidths() {
        for cols in [30, 40, 60, 80] {
            let renderer = ASCIIBoxRenderer(columns: cols)
            let result = DiceResult(
                raw: 10, modifier: 0, total: 10,
                dc: 10, passed: true, isNat20: false, isNat1: false
            )
            let lines = renderer.renderDiceBox(result)
            for line in lines {
                XCTAssertEqual(line.terminalWidth, cols, "Cols=\(cols), line: '\(line)'")
            }
        }
    }
}
```

**Step 2: Run tests to verify they fail**

**Step 3: Implement ASCIIBoxRenderer**

```swift
// Presentation/ASCIIBoxRenderer.swift
import Foundation

struct ASCIIBoxRenderer {
    let columns: Int

    private var innerWidth: Int { columns - 4 } // "│ " + content + " │"

    func renderDiceBox(_ result: DiceResult) -> [String] {
        let passText = result.passed ? "✓  PASS" : "✗  FAIL"
        return [
            boxTop(),
            boxLine(" d20  →  [ \(result.raw) ]"),
            boxLine(" Modifier  →  \(result.modifier >= 0 ? "+" : "")\(result.modifier)"),
            boxLine(" Total     →  \(result.total)"),
            boxLine(" DC \(result.dc)  →  \(passText)"),
            boxBottom()
        ]
    }

    func renderChoiceBox(_ choices: [Choice], locale: String) -> [String] {
        var lines = [boxTop()]
        for (i, choice) in choices.enumerated() {
            let text = choice.text[locale] ?? choice.text["en"] ?? ""
            let prefix = " [\(i + 1)] "
            let continuation = String(repeating: " ", count: prefix.count)
            let wrapped = TerminalGrid.wordWrap(text, width: innerWidth - prefix.count)
            for (j, wLine) in wrapped.enumerated() {
                let leader = j == 0 ? prefix : continuation
                lines.append(boxLine(leader + wLine))
            }
            if i < choices.count - 1 {
                lines.append(boxLine(""))
            }
        }
        lines.append(boxBottom())
        return lines
    }

    private func boxTop() -> String {
        "┌" + String(repeating: "─", count: columns - 2) + "┐"
    }

    private func boxBottom() -> String {
        "└" + String(repeating: "─", count: columns - 2) + "┘"
    }

    private func boxLine(_ content: String) -> String {
        "│ " + content.padRight(toTerminalWidth: innerWidth) + " │"
    }
}
```

**Step 4: Run tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Presentation/ASCIIBoxRenderer.swift \
        TheOldMountainTests/ASCIIBoxRendererTests.swift
git commit -m "feat: add ASCIIBoxRenderer with dynamic-width dice and choice boxes"
```

---

## Task 6: HapticEngine + Patterns

**Files:**
- Create: `TheOldMountain/TheOldMountain/Haptics/HapticPattern.swift`
- Create: `TheOldMountain/TheOldMountain/Haptics/HapticEngine.swift`

**Step 1: Implement HapticPattern definitions**

```swift
// Haptics/HapticPattern.swift
import CoreHaptics

struct HapticPatternDefinition {
    let events: [CHHapticEvent]

    static let keystroke = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.3),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5)
            ],
            relativeTime: 0
        )
    ])

    static let choiceAppear = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.2),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.3)
            ],
            relativeTime: 0
        )
    ])

    static let numberSlam = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 1.0),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.8)
            ],
            relativeTime: 0
        )
    ])

    static let nat20 = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 1.0),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 1.0)
            ],
            relativeTime: 0
        ),
        CHHapticEvent(
            eventType: .hapticContinuous,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.7),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.2)
            ],
            relativeTime: 0.05,
            duration: 0.8
        )
    ])

    static let nat1 = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 1.0),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.6)
            ],
            relativeTime: 0
        ),
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.5),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.4)
            ],
            relativeTime: 0.15
        ),
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.3),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.3)
            ],
            relativeTime: 0.30
        )
    ])

    static let callbackLand = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.6),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5)
            ],
            relativeTime: 0
        ),
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.8),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.6)
            ],
            relativeTime: 0.4
        )
    ])

    static let actTransition = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticContinuous,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.5),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.1)
            ],
            relativeTime: 0,
            duration: 1.2
        )
    ])

    static let endingFirstChar = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.4),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.3)
            ],
            relativeTime: 0
        )
    ])

    static let diceBoxAppear = HapticPatternDefinition(events: [
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.3),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.3)
            ],
            relativeTime: 0
        ),
        CHHapticEvent(
            eventType: .hapticTransient,
            parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.3),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.3)
            ],
            relativeTime: 0.1
        )
    ])

    static func definition(for type: HapticPatternType) -> HapticPatternDefinition {
        switch type {
        case .keystroke: return .keystroke
        case .choiceAppear: return .choiceAppear
        case .choicePress: return .keystroke // reuse light tap
        case .choiceRelease: return .numberSlam // reuse sharp tap
        case .diceBoxAppear: return .diceBoxAppear
        case .numberSlam: return .numberSlam
        case .nat20: return .nat20
        case .nat1: return .nat1
        case .callbackLand: return .callbackLand
        case .actTransition: return .actTransition
        case .endingSilence: return HapticPatternDefinition(events: []) // no-op
        case .endingFirstChar: return .endingFirstChar
        }
    }
}
```

**Step 2: Implement HapticEngine**

```swift
// Haptics/HapticEngine.swift
import CoreHaptics
import Foundation

@Observable
class HapticEngine {
    private var engine: CHHapticEngine?
    private var compiledPatterns: [HapticPatternType: CHHapticPattern] = [:]
    private let supportsHaptics: Bool

    init() {
        self.supportsHaptics = CHHapticEngine.capabilitiesForHardware().supportsHaptics
        if supportsHaptics {
            prepareEngine()
        }
    }

    func play(_ type: HapticPatternType) {
        guard supportsHaptics, type != .endingSilence else { return }
        guard let pattern = compiledPatterns[type],
              let player = try? engine?.makePlayer(with: pattern)
        else { return }
        try? player.start(atTime: CHHapticTimeImmediate)
    }

    private func prepareEngine() {
        do {
            engine = try CHHapticEngine()
            engine?.isAutoShutdownEnabled = true
            engine?.resetHandler = { [weak self] in
                try? self?.engine?.start()
                self?.precompileAll()
            }
            try engine?.start()
            precompileAll()
        } catch {
            // Degrade gracefully — haptics unavailable
        }
    }

    private func precompileAll() {
        for type in HapticPatternType.allCases {
            let def = HapticPatternDefinition.definition(for: type)
            guard !def.events.isEmpty else { continue }
            if let pattern = try? CHHapticPattern(events: def.events, parameters: []) {
                compiledPatterns[type] = pattern
            }
        }
    }
}
```

Note: add `CaseIterable` conformance to `HapticPatternType` in `NarrativeEvent.swift`.

**Step 3: Build to verify (no unit tests for haptics — hardware dependent)**

```bash
xcodebuild build ...
```
Expected: BUILD SUCCEEDED

**Step 4: Commit**

```bash
git add TheOldMountain/TheOldMountain/Haptics/
git commit -m "feat: add HapticEngine with all pattern definitions from design doc"
```

---

## Task 7: NarrativeCompiler

**Files:**
- Create: `TheOldMountain/TheOldMountain/Engine/NarrativeCompiler.swift`
- Create: `TheOldMountain/TheOldMountain/Providers/NarrativeProvider.swift`
- Test: `TheOldMountain/TheOldMountainTests/NarrativeCompilerTests.swift`

**Step 1: Write failing tests**

```swift
// NarrativeCompilerTests.swift
import XCTest
@testable import TheOldMountain

final class NarrativeCompilerTests: XCTestCase {
    let compiler = NarrativeCompiler()

    func testCompilesTextIntoLineEvents() {
        let response = RawNarrativeResponse(
            narrativeText: ["en": "The wolf backs away.\nIt looks at you once more."],
            choices: [],
            peripheralDetail: nil,
            openLoopPlanted: nil
        )
        let scene = compiler.compile(response, diceCheck: nil, locale: "en")

        let textEvents = scene.narrative.filter {
            if case .text = $0 { return true }
            return false
        }
        XCTAssertEqual(textEvents.count, 2)
    }

    func testInsertsPausesBetweenLines() {
        let response = RawNarrativeResponse(
            narrativeText: ["en": "Line one.\nLine two.\nLine three."],
            choices: [],
            peripheralDetail: nil,
            openLoopPlanted: nil
        )
        let scene = compiler.compile(response, diceCheck: nil, locale: "en")

        let pauseEvents = scene.narrative.filter {
            if case .pause = $0 { return true }
            return false
        }
        XCTAssertGreaterThanOrEqual(pauseEvents.count, 2)
    }

    func testIncludesDiceRollWhenProvided() {
        let check = DiceCheck(die: 20, dc: 15, modifier: 2, stat: .innerPower)
        let response = RawNarrativeResponse(
            narrativeText: ["en": "You strike."],
            choices: [],
            peripheralDetail: nil,
            openLoopPlanted: nil
        )
        let scene = compiler.compile(response, diceCheck: check, locale: "en")

        let diceEvents = scene.narrative.filter {
            if case .diceRoll = $0 { return true }
            return false
        }
        XCTAssertEqual(diceEvents.count, 1)
    }

    func testAppendsChoicesReveal() {
        let response = RawNarrativeResponse(
            narrativeText: ["en": "The path forks."],
            choices: [
                RawChoice(text: ["en": "Go left"], impliedStatEffect: .wisdom, impliedFlag: nil),
                RawChoice(text: ["en": "Go right"], impliedStatEffect: .fate, impliedFlag: nil)
            ],
            peripheralDetail: ["en": "The gate is open."],
            openLoopPlanted: ["en": "It is never open."]
        )
        let scene = compiler.compile(response, diceCheck: nil, locale: "en")

        XCTAssertEqual(scene.choices.count, 2)
        let revealEvents = scene.narrative.filter {
            if case .choicesReveal = $0 { return true }
            return false
        }
        XCTAssertEqual(revealEvents.count, 1)
    }

    func testMetadataExtracted() {
        let response = RawNarrativeResponse(
            narrativeText: ["en": "Text."],
            choices: [],
            peripheralDetail: ["en": "The east gate is open."],
            openLoopPlanted: ["en": "It is never open."]
        )
        let scene = compiler.compile(response, diceCheck: nil, locale: "en")
        XCTAssertEqual(scene.metadata.peripheralDetail, "The east gate is open.")
        XCTAssertEqual(scene.metadata.openLoopPlanted, "It is never open.")
    }
}
```

**Step 2: Run tests to verify they fail**

**Step 3: Implement NarrativeProvider protocol + NarrativeCompiler**

`Providers/NarrativeProvider.swift`:
```swift
import Foundation

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

protocol NarrativeProvider {
    func generate(
        systemPrompt: String,
        gameState: GameState,
        previousChoice: Choice
    ) async throws -> RawNarrativeResponse
}
```

`Engine/NarrativeCompiler.swift`:
```swift
import Foundation

struct NarrativeCompiler {
    func compile(
        _ response: RawNarrativeResponse,
        diceCheck: DiceCheck?,
        locale: String
    ) -> Scene {
        var events: [NarrativeEvent] = []

        // Compile narrative text into line events with pauses
        let text = response.narrativeText[locale]
                ?? response.narrativeText["en"]
                ?? ""
        let lines = text.components(separatedBy: "\n").filter { !$0.isEmpty }
        for (i, line) in lines.enumerated() {
            events.append(.text(line, delay: .zero))
            if i < lines.count - 1 {
                events.append(.pause(.milliseconds(200)))
            }
        }

        // Insert dice roll if applicable
        if let check = diceCheck {
            events.append(.pause(.milliseconds(300)))
            events.append(.haptic(.diceBoxAppear))
            events.append(.diceRoll(check))
        }

        // Build choices from raw response
        let choices = response.choices.enumerated().map { i, raw in
            Choice(
                id: "choice_\(i)",
                text: raw.text,
                statEffects: [StatEffect(stat: raw.impliedStatEffect, modifier: 1)],
                flagsSet: [raw.impliedFlag].compactMap { $0 },
                diceCheck: nil
            )
        }

        // Append choices reveal
        if !choices.isEmpty {
            events.append(.pause(.milliseconds(300)))
            events.append(.choicesReveal(choices, stagger: .milliseconds(150)))
        }

        let metadata = SceneMetadata(
            peripheralDetail: response.peripheralDetail?[locale]
                ?? response.peripheralDetail?["en"],
            openLoopPlanted: response.openLoopPlanted?[locale]
                ?? response.openLoopPlanted?["en"],
            callbackSeed: nil,
            callbackEcho: nil
        )

        return Scene(narrative: events, choices: choices, metadata: metadata)
    }
}
```

**Step 4: Run tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Engine/NarrativeCompiler.swift \
        TheOldMountain/TheOldMountain/Providers/NarrativeProvider.swift \
        TheOldMountainTests/NarrativeCompilerTests.swift
git commit -m "feat: add NarrativeCompiler and NarrativeProvider protocol"
```

---

## Task 8: SystemPromptBuilder

**Files:**
- Create: `TheOldMountain/TheOldMountain/Providers/SystemPromptBuilder.swift`
- Test: `TheOldMountain/TheOldMountainTests/SystemPromptBuilderTests.swift`

**Step 1: Write failing tests**

```swift
// SystemPromptBuilderTests.swift
import XCTest
@testable import TheOldMountain

final class SystemPromptBuilderTests: XCTestCase {
    let builder = SystemPromptBuilder()

    func testIncludesTurnAndAct() {
        var state = GameState.newRun()
        state.turn = 5
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertTrue(prompt.contains("Turn: 5/30"))
        XCTAssertTrue(prompt.contains("Act: 1"))
    }

    func testIncludesStats() {
        var state = GameState.newRun()
        state.stats.innerPower = 7
        state.stats.wisdom = 3
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertTrue(prompt.contains("Inner Power: 7"))
        XCTAssertTrue(prompt.contains("Wisdom: 3"))
    }

    func testIncludesFlags() {
        var state = GameState.newRun()
        state.flags = ["met_feng", "stole_pill"]
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertTrue(prompt.contains("met_feng"))
        XCTAssertTrue(prompt.contains("stole_pill"))
    }

    func testEndingSteeringAfterTurn20() {
        var state = GameState.newRun()
        state.turn = 22
        state.targetEnding = .immortalSovereign
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertTrue(prompt.contains("immortalSovereign"))
        XCTAssertTrue(prompt.contains("Steer naturally toward"))
    }

    func testNoEndingSteeringBeforeTurn20() {
        var state = GameState.newRun()
        state.turn = 15
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertFalse(prompt.contains("Steer naturally toward"))
    }

    func testIncludesProseRules() {
        let state = GameState.newRun()
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertTrue(prompt.contains("Max 3 lines before a blank line"))
        XCTAssertTrue(prompt.contains("120 words"))
    }

    func testIncludesJSONOutputInstruction() {
        let state = GameState.newRun()
        let prompt = builder.build(from: state, locale: "en")
        XCTAssertTrue(prompt.contains("JSON"))
    }
}
```

**Step 2: Run tests to verify they fail**

**Step 3: Implement SystemPromptBuilder**

Build the system prompt template from the narrative design guide Part Eight. Inject game state variables. Include JSON output schema instruction.

The implementation should be a single `build(from:locale:)` method that interpolates state into the template string from the design doc. Include all prose rules, choice writing rules, and the JSON response schema.

**Step 4: Run tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Providers/SystemPromptBuilder.swift \
        TheOldMountainTests/SystemPromptBuilderTests.swift
git commit -m "feat: add SystemPromptBuilder with all narrative rules from design doc"
```

---

## Task 9: SwiftData Models + GameStateRepository

**Files:**
- Create: `TheOldMountain/TheOldMountain/Persistence/RunRecord.swift`
- Create: `TheOldMountain/TheOldMountain/Persistence/TurnLogEntry.swift`
- Create: `TheOldMountain/TheOldMountain/Persistence/ActiveGameState.swift`
- Create: `TheOldMountain/TheOldMountain/Persistence/CachedSceneModel.swift`
- Create: `TheOldMountain/TheOldMountain/Persistence/GameStateRepository.swift`
- Create: `TheOldMountain/TheOldMountain/Persistence/RunHistoryRepository.swift`
- Test: `TheOldMountain/TheOldMountainTests/GameStateRepositoryTests.swift`

**Step 1: Write failing tests**

```swift
// GameStateRepositoryTests.swift
import XCTest
import SwiftData
@testable import TheOldMountain

final class GameStateRepositoryTests: XCTestCase {
    var container: ModelContainer!
    var repo: GameStateRepository!

    override func setUp() {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        container = try! ModelContainer(
            for: RunRecordModel.self, ActiveGameStateModel.self, CachedSceneModel.self,
            configurations: config
        )
        repo = GameStateRepository(modelContext: ModelContext(container))
    }

    func testSaveAndLoadActiveState() async throws {
        var state = GameState.newRun()
        state.turn = 5
        state.stats.innerPower = 3
        state.flags = ["met_feng"]

        let runId = UUID()
        try repo.saveActive(state, runId: runId)
        let loaded = try repo.loadActive()

        XCTAssertNotNil(loaded)
        XCTAssertEqual(loaded?.turn, 5)
        XCTAssertEqual(loaded?.stats.innerPower, 3)
    }

    func testCompleteRunClearsActive() async throws {
        var state = GameState.newRun()
        state.turn = 30
        let runId = UUID()
        try repo.saveActive(state, runId: runId)
        try repo.completeRun(runId: runId, ending: .immortalSovereign, quote: ["en": "The wolf knelt."])

        let active = try repo.loadActive()
        XCTAssertNil(active)
    }

    func testRunHistoryRecordsCompletion() async throws {
        let runId = UUID()
        var state = GameState.newRun()
        state.turn = 30
        try repo.saveActive(state, runId: runId)
        try repo.completeRun(runId: runId, ending: .wanderingSage, quote: nil)

        let history = try repo.completedRuns()
        XCTAssertEqual(history.count, 1)
        XCTAssertEqual(history.first?.ending, "wanderingSage")
    }
}
```

**Step 2: Run tests to verify they fail**

**Step 3: Implement SwiftData models and repository**

Implement all `@Model` classes per the design doc. `GameStateRepository` handles CRUD for `ActiveGameStateModel` and `RunRecordModel`. Use `Codable` for nested types stored as `Data` (stats, flags, etc.).

**Step 4: Run tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Persistence/ TheOldMountainTests/GameStateRepositoryTests.swift
git commit -m "feat: add SwiftData models and GameStateRepository"
```

---

## Task 10: CachedSceneProvider + Sample Scenes

**Files:**
- Create: `TheOldMountain/TheOldMountain/Providers/CachedSceneProvider.swift`
- Create: `TheOldMountain/TheOldMountain/Resources/CachedScenes/turn1_opening.json`
- Create: `TheOldMountain/TheOldMountain/Resources/CachedScenes/turn30_immortal_sovereign.json`

**Step 1: Create Turn 1 opening scene JSON**

```json
{
  "turn": 1,
  "act": 1,
  "path": "all",
  "requiredFlags": [],
  "priority": 100,
  "narrative": {
    "en": "The mountain is older than its name.\nYou stand at the gate. The stone is warm.\nAbove you, a bell you cannot see rings once.",
    "zh-Hans": "这座山比它的名字更古老。\n你站在山门前。石头是温热的。\n头顶上方，一口看不见的钟敲了一声。"
  },
  "peripheral": {
    "en": "Someone has swept the steps recently. The broom is still leaning against the wall.",
    "zh-Hans": "有人最近扫过台阶。扫帚还靠在墙边。"
  },
  "choices": [
    {
      "text": {
        "en": "Bow to the gate. Enter properly.",
        "zh-Hans": "向山门鞠躬。正式进入。"
      },
      "stat": "honor",
      "modifier": 1,
      "flag": "entered_with_respect"
    },
    {
      "text": {
        "en": "Touch the warm stone. Feel what it knows.",
        "zh-Hans": "触摸温热的石头。感受它知道的一切。"
      },
      "stat": "wisdom",
      "modifier": 1,
      "flag": null
    },
    {
      "text": {
        "en": "Walk past without stopping. You didn't come here to wait.",
        "zh-Hans": "径直走过，不做停留。你来这里不是为了等待。"
      },
      "stat": "innerPower",
      "modifier": 1,
      "flag": "entered_boldly"
    }
  ],
  "open_loop": {
    "en": "The bell does not ring again. You expected it to.",
    "zh-Hans": "钟没有再响。你以为它会。"
  }
}
```

**Step 2: Implement CachedSceneProvider**

Reads JSON files from bundle, queries by turn/path/flags. Falls back to nil if no match (triggering LLM generation via SceneResolver).

**Step 3: Build and verify JSON loads correctly**

**Step 4: Commit**

```bash
git add TheOldMountain/TheOldMountain/Providers/CachedSceneProvider.swift \
        TheOldMountain/TheOldMountain/Resources/CachedScenes/
git commit -m "feat: add CachedSceneProvider with Turn 1 opening scene"
```

---

## Task 11: LLM Providers (Claude + OpenAI)

**Files:**
- Create: `TheOldMountain/TheOldMountain/Providers/ClaudeProvider.swift`
- Create: `TheOldMountain/TheOldMountain/Providers/OpenAIProvider.swift`
- Create: `TheOldMountain/TheOldMountain/Providers/LLMSceneProvider.swift`
- Create: `TheOldMountain/TheOldMountain/Security/APIKeyStore.swift`

**Step 1: Implement APIKeyStore** — Keychain wrapper for API keys.

**Step 2: Implement ClaudeProvider** — POST to `https://api.anthropic.com/v1/messages` with JSON mode. Parse response into `RawNarrativeResponse`.

**Step 3: Implement OpenAIProvider** — POST to `https://api.openai.com/v1/chat/completions` with JSON mode. Parse response into `RawNarrativeResponse`.

**Step 4: Implement LLMSceneProvider** — Wraps a `NarrativeProvider`, calls `SystemPromptBuilder` to assemble the prompt, delegates to the active provider.

**Step 5: Build to verify**

**Step 6: Commit**

```bash
git add TheOldMountain/TheOldMountain/Providers/ TheOldMountain/TheOldMountain/Security/
git commit -m "feat: add Claude and OpenAI providers with API key management"
```

---

## Task 12: SceneResolver + GameEngine

**Files:**
- Create: `TheOldMountain/TheOldMountain/Engine/SceneResolver.swift`
- Create: `TheOldMountain/TheOldMountain/Engine/GameEngine.swift`
- Test: `TheOldMountain/TheOldMountainTests/SceneResolverTests.swift`
- Test: `TheOldMountain/TheOldMountainTests/GameEngineTests.swift`

**Step 1: Write failing tests for SceneResolver**

Test that cached scenes are returned when available. Test that LLM provider is called when no cached scene matches. Use mock providers.

**Step 2: Write failing tests for GameEngine**

Test `startNewRun()` returns a scene and sets state to turn 1. Test `applyChoice()` advances turn, applies stats/flags, and returns next scene. Test act boundary transitions. Test ending calculation at turn 20+.

**Step 3: Implement SceneResolver and GameEngine**

SceneResolver: check cached first, fall back to LLM. GameEngine: the full `applyChoice` flow per the design doc.

**Step 4: Run all tests**

Expected: ALL PASS

**Step 5: Commit**

```bash
git add TheOldMountain/TheOldMountain/Engine/ TheOldMountainTests/SceneResolverTests.swift \
        TheOldMountainTests/GameEngineTests.swift
git commit -m "feat: add SceneResolver and GameEngine with full turn loop"
```

---

## Task 13: TypewriterEngine

**Files:**
- Create: `TheOldMountain/TheOldMountain/Presentation/TypewriterEngine.swift`

**Step 1: Implement TypewriterEngine actor**

```swift
actor TypewriterEngine {
    enum Tick {
        case character(Character)
        case newLine
        case pause(Duration)
        case diceSequence(DiceCheck)
        case revealChoice(Choice)
        case haptic(HapticPatternType)
    }

    private let charDelay: Duration  // ~30ms for 33 chars/sec

    init(speed: TextSpeed = .normal) {
        switch speed {
        case .slow: charDelay = .milliseconds(50)
        case .normal: charDelay = .milliseconds(30)
        case .fast: charDelay = .milliseconds(15)
        case .instant: charDelay = .zero
        }
    }

    func play(events: [NarrativeEvent]) -> AsyncStream<Tick> {
        AsyncStream { continuation in
            Task {
                for event in events {
                    switch event {
                    case .text(let text, _):
                        for char in text {
                            continuation.yield(.character(char))
                            continuation.yield(.haptic(.keystroke))
                            if char == "." || char == "," {
                                try? await Task.sleep(for: charDelay * 3)
                            } else {
                                try? await Task.sleep(for: charDelay)
                            }
                        }
                        continuation.yield(.newLine)

                    case .pause(let duration):
                        continuation.yield(.pause(duration))
                        try? await Task.sleep(for: duration)

                    case .diceRoll(let check):
                        continuation.yield(.diceSequence(check))

                    case .choicesReveal(let choices, let stagger):
                        for choice in choices {
                            continuation.yield(.revealChoice(choice))
                            continuation.yield(.haptic(.choiceAppear))
                            try? await Task.sleep(for: stagger)
                        }

                    case .haptic(let pattern):
                        continuation.yield(.haptic(pattern))
                    }
                }
                continuation.finish()
            }
        }
    }
}

enum TextSpeed: String, CaseIterable {
    case slow, normal, fast, instant
}
```

**Step 2: Build to verify**

**Step 3: Commit**

```bash
git add TheOldMountain/TheOldMountain/Presentation/TypewriterEngine.swift
git commit -m "feat: add TypewriterEngine actor with AsyncStream tick output"
```

---

## Task 14: Core Views (TerminalView, DiceRollView, ChoiceBoxView, HUDBar)

**Files:**
- Create: `TheOldMountain/TheOldMountain/Presentation/TerminalView.swift`
- Create: `TheOldMountain/TheOldMountain/Presentation/DiceRollView.swift`
- Create: `TheOldMountain/TheOldMountain/Presentation/ChoiceBoxView.swift`
- Create: `TheOldMountain/TheOldMountain/Presentation/HUDBar.swift`

**Step 1: Implement TerminalView** — Scrolling monospace text view. Appends characters from TypewriterTick stream. Auto-scrolls to bottom. Green/amber text on black. Blinking cursor at end.

**Step 2: Implement DiceRollView** — ASCII box rendered line by line with exact timing from design doc. 600ms pause before number reveal. Hard haptic on number slam.

**Step 3: Implement ChoiceBoxView** — ASCII-bordered choice box. Each choice fades in with 150ms stagger. Tap handler calls back to view model. Haptic on tap press/release.

**Step 4: Implement HUDBar** — Single line: `T{n} · {act} · {path} · {progress_bar}`. Block characters `▮` and `░`. Green at 50% opacity.

**Step 5: Build and test in Preview**

**Step 6: Commit**

```bash
git add TheOldMountain/TheOldMountain/Presentation/
git commit -m "feat: add core presentation views — terminal, dice, choices, HUD"
```

---

## Task 15: GameViewModel + GameScreen

**Files:**
- Create: `TheOldMountain/TheOldMountain/Screens/GameViewModel.swift`
- Create: `TheOldMountain/TheOldMountain/Screens/GameScreen.swift`

**Step 1: Implement GameViewModel** — Owns GameEngine, TypewriterEngine, HapticEngine. Exposes `@Published` display state (lines, choices, dice result, HUD data). `selectChoice()` calls engine, plays new scene through typewriter. Handles save/resume via GameStateRepository.

**Step 2: Implement GameScreen** — ZStack with black background. VStack: ScrollView with TerminalView, optional DiceRollView, optional ChoiceBoxView (pinned to bottom), HUDBar. Status bar hidden.

**Step 3: Build and run in simulator**

Test: tap through Turn 1 opening scene with typewriter effect, haptics, and choice selection.

**Step 4: Commit**

```bash
git add TheOldMountain/TheOldMountain/Screens/GameViewModel.swift \
        TheOldMountain/TheOldMountain/Screens/GameScreen.swift
git commit -m "feat: add GameScreen with full turn loop — typewriter, dice, choices"
```

---

## Task 16: Remaining Screens + Navigation

**Files:**
- Create: `TheOldMountain/TheOldMountain/Screens/HomeScreen.swift`
- Create: `TheOldMountain/TheOldMountain/Screens/EndingScreen.swift`
- Create: `TheOldMountain/TheOldMountain/Screens/ArchiveScreen.swift`
- Create: `TheOldMountain/TheOldMountain/Screens/ReplayScreen.swift`
- Create: `TheOldMountain/TheOldMountain/Screens/SettingsScreen.swift`
- Create: `TheOldMountain/TheOldMountain/Screens/PauseOverlay.swift`
- Create: `TheOldMountain/TheOldMountain/App/RootView.swift`
- Modify: `TheOldMountain/TheOldMountain/App/TheOldMountainApp.swift`

**Step 1: Implement HomeScreen** — Title types itself. Menu options stagger in. Resume option if active run exists.

**Step 2: Implement EndingScreen** — 800ms silence (no haptic). Screen clears. Single distant tap. Title types at half speed. Run quote fades in. Share/Archives/Begin Again options.

**Step 3: Implement ArchiveScreen** — Lists completed runs with quotes and endings. "Endings discovered: N/5+" counter.

**Step 4: Implement ReplayScreen** — Read-only scroll through TurnLogEntry chain. No typewriter, just text.

**Step 5: Implement SettingsScreen** — Terminal-styled settings: LLM provider, text speed, haptics toggle, language, CRT scanlines toggle.

**Step 6: Implement PauseOverlay** — Two-finger tap gesture. Translucent overlay. Resume / Abandon / Settings.

**Step 7: Implement RootView** — Custom ZStack navigation with `AppScreen` enum. Terminal-style transitions (clear → type-in). GeometryReader for TerminalGrid.

**Step 8: Update TheOldMountainApp.swift** — Wire SwiftData ModelContainer, HapticEngine, APIKeyStore into environment. Launch RootView.

**Step 9: Build and run full app flow in simulator**

Test: Home → Begin → play through turns → Ending → Archives → Settings

**Step 10: Commit**

```bash
git add TheOldMountain/TheOldMountain/Screens/ TheOldMountain/TheOldMountain/App/
git commit -m "feat: add all screens and custom terminal navigation"
```

---

## Task 17: Author Remaining Cached Scenes

**Files:**
- Modify: `TheOldMountain/TheOldMountain/Resources/CachedScenes/` (add JSON files)

Author the hand-written key beats per the design doc:
- Turn 1: Opening (done in Task 10)
- Turn 3: Sect assignment (path selection — sword/alchemy/shadow)
- Turn 5: Prophecy planting (per target ending)
- Turn 10: Act I → Act II transition
- Turn 20: Act II → Act III transition
- Turn 28-30: Ending scenes (5 endings × 1-3 scenes each)

Each JSON follows the schema from Task 10. Include `en` and `zh-Hans` for all `LocalizedString` fields.

**Commit after each batch of scenes.**

---

## Task 18: Integration Test — Full Playthrough

**Files:**
- Create: `TheOldMountain/TheOldMountainTests/IntegrationTests.swift`

**Step 1: Write integration test** — Use mock LLM provider that returns canned responses. Play through 30 turns programmatically via GameEngine. Verify: turn advances correctly, stats accumulate, act transitions fire, ending is reached, run is persisted.

**Step 2: Write edge case tests** — Nat 20 / Nat 1 handling. App kill and resume mid-turn. Empty flags. All 5 endings reachable.

**Step 3: Run full test suite**

```bash
xcodebuild test -project TheOldMountain/TheOldMountain.xcodeproj \
  -scheme TheOldMountain \
  -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest'
```

Expected: ALL PASS

**Step 4: Commit**

```bash
git add TheOldMountainTests/IntegrationTests.swift
git commit -m "test: add integration tests for full 30-turn playthrough"
```

---

## Dependency Graph

```
Task 1  (scaffold)
  └─→ Task 2  (models)
        ├─→ Task 3  (dice engine)
        ├─→ Task 4  (terminal grid + CJK)
        │     └─→ Task 5  (ASCII box renderer)
        ├─→ Task 6  (haptic engine)
        ├─→ Task 7  (narrative compiler)
        │     └─→ Task 8  (system prompt builder)
        │           └─→ Task 11 (LLM providers)
        └─→ Task 9  (SwiftData + repository)
              └─→ Task 10 (cached scene provider)

Tasks 3-11 ──→ Task 12 (scene resolver + game engine)
Task 6 + 7  ──→ Task 13 (typewriter engine)
Tasks 4-6,13 ─→ Task 14 (core views)
Tasks 12-14  ──→ Task 15 (game screen)
Task 15      ──→ Task 16 (all screens + navigation)
Task 10      ──→ Task 17 (author cached scenes)
Tasks 16-17  ──→ Task 18 (integration tests)
```
