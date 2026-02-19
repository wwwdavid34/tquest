# The Old Mountain — Narrative Design Guide
## Distilling State-of-the-Art ARPG Experience Into Text

> "We are not making a text game that wishes it were a video game.  
> We are making the thing video games learned from —  
> returned, refined, and running on the hardware that matters most:  
> the player's imagination."

---

## The Design Philosophy

The Old Mountain sits at the intersection of three traditions:

**MUD nostalgia** — the authentic telnet experience. Text that types itself. A world that exists in the gap between characters. The feeling of being present in a place that has no pictures because it doesn't need them.

**ARPG state-of-the-art** — the design wisdom of BotW, Dark Souls, Dragon Quest. Environmental storytelling. Implicit lures. Variable reward. The world that continues without you.

**Xianxia soul** — philosophical depth beneath an accessible surface. Choices that feel cosmic. A karma system that remembers. Heaven watching.

The synthesis: **30 turns. 25 minutes. A world too large to see from one angle.**

Every mechanic, every prose decision, every haptic beat serves one physical response in the player: **the involuntary lean forward before they've consciously decided to tap.**

---

## Part One: The Dopamine Architecture

### The Three Triggers

All engagement in The Old Mountain flows from three dopamine mechanisms. They must be present in every turn, sometimes simultaneously:

```
1. CURIOSITY GAP
   Something incomplete that the brain needs to close.
   The player MUST continue to resolve it.
   Not because they were told to. Because they need to.

2. VARIABLE REWARD  
   Known reward coming, unknown magnitude.
   The dice roll. Something will happen.
   The uncertainty between knowing and knowing is the hit.

3. CALLBACK SATISFACTION
   A thread planted earlier resolves unexpectedly.
   Recognition + surprise simultaneously.
   This is the highest dopamine moment in the game.
   Use it sparingly. It never gets old if it's earned.
```

### The Minimum Viable Loop

Every turn — every single turn — executes this in under 60 seconds:

```
[OPEN LOOP from previous turn — still unresolved]
        ↓
3 lines of atmospheric text        [curiosity maintained]
        ↓
Open loop partially resolved       [small dopamine — relief]
        ↓
New open loop planted              [curiosity regenerated]
        ↓
Choices appear one by one          [anticipation]
        ↓
Player taps                        [agency — feels powerful]
        ↓
Dice roll if applicable            [variable reward]
        ↓
Consequence reveals                [resolution]
        ↓
New open loop in final line        [pull to next turn]
```

If any turn breaks this loop, rewrite it. This is the spec.

### The Dice Roll as Dopamine Delivery System

The dice roll is not a mechanic. It is a dopamine delivery system dressed as a mechanic. Every timing element is engineered, not arbitrary:

```
> COMBAT CHECK — INNER POWER

Rolling...          [300ms — anticipation builds]
.                   [200ms — variable wait]
..                  [200ms]
...                 [200ms — brain in suspense]

┌──────────────────────┐
│                      │   [box appears — reward imminent]
│  d20  →              │   [200ms]
│  d20  →  [          ]│   [600ms — PEAK ANTICIPATION]
│  d20  →  [ 17 ]      │   [number slams in — relief/elation]
│  Modifier  →  +2     │   [100ms]
│  Total     →  19     │   [100ms]
│  DC 15  →            │   [300ms — one final beat]
│  DC 15  →  ✓  PASS   │   [resolution — dopamine peak]
└──────────────────────┘
```

**The 600ms pause before the number is the entire mechanism.**

The gap between "I know something is coming" and "here it is" is where dopamine lives. This pause must never be shorter than 500ms. Never longer than 800ms. Outside that window it loses its power.

**The dice must be honest.** Never fudge results. Players sense it at the subconscious level. The moment they suspect results are predetermined, the variable reward mechanism collapses entirely and cannot be rebuilt.

---

## Part Two: Implicit Exploration — The BotW Principle

### The Core Problem

BotW's genius: **you never feel directed.** The mountain doesn't have an arrow pointing at it. It sits there, backlit, slightly mysterious, and your brain does the rest. The game engineers desire without stating it.

Text is sequential — you read left to right, top to bottom. Visual games place lures in peripheral vision simultaneously. Text is linear. So the mechanism must be different, and it is:

```
BotW shows you a distant tower.
Your eyes see: tower.

Text says: "The east path smells of old smoke."
Your mind generates: who burned what, when, why,
                     what's there now, should I go,
                     what happens if I don't.
```

Five questions from four words. **Text lures are more powerful than visual lures because imagination is stronger than rendering.** The player generates a personalized version of the mystery that is always more compelling than anything you could actually show them.

### The Incomplete World Principle

BotW's world feels explorable because it's clearly larger than any one path through it. The player senses depth beyond the current viewpoint.

In The Old Mountain, this depth is created through **the peripheral layer** — details that exist alongside the player's current action without demanding attention:

```
WHAT THE PLAYER IS DOING        WHAT EXISTS PERIPHERALLY
──────────────────────────────────────────────────────────
Walking to the temple           The east gate is open. It is never open.
Talking to Master Shen          Brother Feng watches from a doorway.
Choosing the Sword path         The alchemist's furnace is lit. Again.
Fighting the wolf               Something moves in the treeline. Then doesn't.
Reading their fate              The monk writes something. Burns it.
```

None of these peripheral details are choices. None demand attention. But they all whisper: **there is more here than you are currently seeing.**

This is the BotW tower. It just sits there. Slightly visible. Slightly mysterious.

---

## Part Three: The Seven Text Lure Mechanisms

### 1. The Peripheral Detail

Mentioned once. Never explained. Never payoff promised. Brain cannot file it as resolved:

```
> You walk to the assessment hall.
> The east gate is open.
> It is never open.
>
> The examiner calls your name.
```

Player chose to walk to the hall. The open gate was not a choice. It was just there. But now it exists in the player's mind as an unresolved thread. They will look for the east gate later. They may never find a payoff. **That's fine.** The world feels larger for having it.

**Rule:** Peripheral details must never be explained in the same turn. Ideally, they're only echoed — never fully resolved.

---

### 2. Asymmetric Choice Texture

Each choice has a different *feel* — not just a different outcome, but a different implied world. The player picks based on what they're curious about:

```
BAD — all choices feel equivalent:
  [1] Attack the stranger
  [2] Talk to the stranger
  [3] Avoid the stranger

GOOD — each choice implies a different universe:
  [1] Step forward. Let him see your face.
  [2] "You've been following us since the canyon."
  [3] You've seen that cloak before. Where?
```

Choice 3 implies the player's character has history they don't know about yet. The player who picks it wants to find out what the character knows. The player who *doesn't* pick it wonders for the rest of the run.

**This is the BotW bridge.** BotW doesn't say "cross the bridge." It makes the bridge look more interesting than the field. Asymmetric choice texture does the same — one option always slightly more textured, slightly more implied-world, slightly more magnetic.

**Applied to Turn 3 — Sect Assignment:**

```
> Three halls open their doors to outer disciples today.
> You have heard things about each.
> Most of what you've heard is probably wrong.
>
> [1] The Hall of Swords
>     Senior Brother Feng is watching the recruits.
>     He nods once when he sees you. Just once.
>
> [2] The Hall of Alchemy
>     The door is already open.
>     Something inside smells like thunderstorms.
>
> [3] The Hidden Road
>     You almost walk past it.
>     The sign is very small. Deliberately small.
```

Nobody says "these paths lead to different endings." But:
- Choice 1 implies a relationship already forming (Feng noticed you specifically)
- Choice 2 implies mystery through sensory anomaly (thunderstorms indoors)
- Choice 3 implies intentional obscurity (why would anyone hide the sign?)

Replay motivation is baked into the choice writing itself.

---

### 3. The NPC Peripheral Gaze

NPCs notice things the player hasn't chosen to notice. This implies the world continues without the player's participation:

```
> Master Shen hands you the schedule.
> He doesn't look at your hands when he does it.
> He always looks at people's hands.
```

Player didn't ask about hands. Player didn't choose to notice. But now the player is asking: *what's wrong with my hands? What does he see in hands? Did he see something in mine?*

This creates desire to interact with Shen again — not because a quest marker appeared, but because the player's curiosity generated the desire organically.

---

### 4. The World That Continues Without You

BotW's world has weather, day/night cycles, enemies that patrol whether you watch them or not. The world doesn't wait. Text equivalent:

```
> When you return to the courtyard,
> the practice dummies have been moved.
> Someone trained here after you.
> The footwork marks in the dust are smaller than yours.
```

Player wasn't there for this. It happened without them. The world ran while they were elsewhere. This implies depth — events, stories, moments happening in rooms the player isn't in.

**The implicit lure:** player wants to be in more places, make different choices, see what else happened without them. Replay motivation generated organically, without ever asking for a replay.

---

### 5. The Closed Door With Light Under It

Describe something inaccessible — not forbidden, just currently unreachable. The brain fixates on the inaccessible:

```
> The inner library is locked.
> You've heard there are maps in there.
> Maps of places that aren't on any official chart.
```

Player cannot access this now. No choice is offered. But they will remember it. If they ever reach inner disciple standing, they will immediately think of the library. They were never told to — they just want to.

**The BotW equivalent:** a chest visible through a gate you can't open yet. You note it. You return. The game never reminded you. You just remembered because the image stuck.

---

### 6. The Echo System

The world occasionally acknowledges paths the player *didn't* take:

```
> At dinner, the alchemy disciples are laughing
> about something that happened in the furnace room.
> You don't know what. You weren't there.
> It sounds like it was good.
```

Player chose Sword path. They are experiencing FOMO for Alchemy — not because the game said "Alchemy is fun!" but because the world showed them a moment they missed. Laughter they can't be part of.

**This is BotW's off-path campfire.** You're going somewhere. You see a campfire with travelers in the distance. You weren't going there. Now you are.

---

### 7. The Sensory Anomaly — The "Slightly Wrong" Principle

BotW's most magnetic moments are things that are slightly wrong — a chest in an odd place, a lone enemy in the middle of nowhere. The wrongness is a question. Questions demand answers:

```
SLIGHTLY WRONG WORLD:
> The mountain smells different today.
> Not bad. Just different.
> Like rain that hasn't happened yet.

SLIGHTLY WRONG NPC:
> The gate guard bows to you.
> They never bow to outer disciples.

SLIGHTLY WRONG ENVIRONMENT:
> The candle in the empty room was still warm.

SLIGHTLY WRONG SELF:
> Your reflection in the lake is a fraction slow.
> Just a fraction.
```

None of these require explanation. None promise payoff. They sit in the mind, slightly sideways, asking to be understood. **Players lean forward.**

---

## Part Four: The Open Loop System

Every turn must end with a thread dangling. Not a cliffhanger — those feel cheap. A thread. Something unresolved that costs nothing to leave open but everything to ignore:

```
WEAK ENDING — closed loop:
> You defeat the wolf. The path is clear.
> You rest for the night.
[nothing to pull player forward]

STRONG ENDING — open loop:
> The wolf backs away.
> Before it disappears into the mist,
> it looks at you once more.
> Not like prey looks at a predator.
> Like it recognized something.
[what did it recognize? player MUST continue]
```

### The Five Open Loop Types

**TYPE 1 — The Unfinished Sentence**
A character starts to say something and stops.
```
> "Master Shen looked at your hands and said—"
> "—but that's a matter for another day."
```
Brain cannot let this go.

**TYPE 2 — The Environmental Anomaly**
Something in the world that shouldn't be there.
```
> The candle in the empty room was still warm.
```
No explanation. No payoff promised. Just there.

**TYPE 3 — The Reaction Without Cause**
An NPC reacts differently with no explanation.
```
> Brother Feng didn't meet your eyes at dinner.
```
No flag explained. Something changed. What?

**TYPE 4 — The Sensory Fragment**
Something perceived but not understood.
```
> The formation hummed a different note today.
```
Different from what? Player wonders.

**TYPE 5 — The Prophecy Seed**
Future event hinted obliquely.
```
> The monk drew a circle in the dust with her finger.
> Then erased it.
> Then drew it again.
```
Players will see this payoff 8 turns later. They won't know that yet.

---

## Part Five: The Callback Architecture

Callbacks are the highest-dopamine moment in the game. They require investment. The payoff must be:
- Unexpected in timing
- Inevitable in retrospect  
- Emotionally resonant, not just mechanically interesting

```
SETUP (Turn 5):
> The monk says: "The wolf will kneel."
> You don't know what this means.
> You move on.

[Player forgets. Three turns of other content pass.]

PAYOFF (Turn 8):
> The wolf staggers.
> Then kneels.
> Not in defeat.
> In recognition.
>
> [150ms pause]
>
> Somewhere, an old woman smiles.
```

That last line — *"somewhere, an old woman smiles"* — costs nothing. No mechanic. No flag. One sentence that says: the world is connected, the world remembers, the world is alive.

### The Three Components of Every Callback

```
1. SEED    — planted early, easily forgotten by the player
2. SILENCE — enough turns between that the callback is genuinely surprising
             minimum 3 turns, ideally 5-8
3. ECHO    — references the setup in a way that feels inevitable, not forced
             the word "inevitable" is the test
             if the callback feels contrived, the seed wasn't right
```

---

## Part Six: The Prose Rules

These rules are not style guidelines. They are mechanics. The LLM system prompt enforces all of them on every generated turn.

### Rule 1: Three Lines Maximum Before a Pause
The brain needs to breathe. Long paragraphs kill pace. Never more than 3 lines before a blank line or natural pause point. The eye should never have to travel far.

### Rule 2: The Last Line Is Always the Hook
The final line of every narrative block must open a loop, not close one. Always. Without exception. The last line is what pulls the finger to the screen.

### Rule 3: Sensory Before Abstract
```
WRONG: "You felt something was wrong."
RIGHT: "The incense smoke bent sideways."
```
Show the world reacting. Not the player feeling. The player supplies their own emotion. Their emotion is stronger than any you describe.

### Rule 4: Present Tense. Sentence Length Mirrors Heartbeat.
```
Under tension:    Short sentences. The wolf charges. Your hands shake.
Breathing:        Longer sentences carry relief, contemplation, the sense that
                  danger has passed and the world is wider than the fight.
```
Short = fast = danger. Long = breathing = safe (for now).

### Rule 5: One Unexpected Word Per Turn
One word that's slightly wrong. Slightly surprising. The word that makes players screenshot:
```
NOT: "The wolf backs away."
NOT: "The wolf retreats."
YES: "The wolf kneels."
```
That word is the turn. Find it. Build around it.

### Rule 6: Never Explain the Emotion
```
WRONG: "You feel afraid."
RIGHT: "Your hands know what your mind won't admit."
```
Let the player supply the emotion. Always.

### Rule 7: The Peripheral Detail Is Mandatory
Every turn must contain exactly one peripheral detail — something happening alongside the main action that isn't a choice, isn't explained, and implies the world is larger than this corridor.

```
> You walk to the hall.        [main action]
> The east gate is open.       [peripheral detail]
> It is never open.            [implication]
> The examiner calls your name.[main action continues]
```

### Rule 8: Every Choice Is a Different Implied World
Choices are not options. They are doors. Each door implies a different room. The player should be able to read the choices and feel the world branching — not see three buttons with different labels.

---

## Part Seven: The Haptic Layer

*Sound effects are replaced by haptic feedback for authentic telnet experience. The tap engine IS the sound design.*

```
MOMENT                          HAPTIC PATTERN
──────────────────────────────────────────────────────────
Typewriter character reveal     Single light tap per character
                                [authentic keyboard presence —
                                 player feels they're typing the story]

Choice options appearing        Soft tap as each choice fades in
                                150ms stagger between choices
                                [tactile menu — physical reality]

Player taps a choice            Medium tap on press
                                Sharp tap on release
                                [decisive, satisfying, committed]

Dice box appearing              Double soft tap
                                [attention signal — something matters]

Number slamming in              Single hard tap
                                [the moment of revelation]

Natural 20                      Sharp tap + long low rumble (800ms)
                                [triumph, sustained, earned]

Natural 1                       Hard tap + two short taps (echo)
                                [impact, then echo — a mistake landing]

Callback landing                Double tap with 400ms pause between
                                [recognition — "wait—"]
                                [understanding — "oh"]

Act transition                  Long soft fade rumble (1200ms)
                                [world shifting beneath you]

Ending card — pre-reveal        800ms absolute silence
                                NO HAPTIC — the most powerful beat
                                in the game

Ending card — first character   Single distant tap
                                [ceremonial — a temple bell heard far away]
```

**The silence before the ending card is the design.** No tap. No rumble. Nothing. 800ms of nothing. Then the first character with a single distant tap. Players will feel it physically. Do not compromise this.

### The Typewriter Haptic as Core Retention Mechanism

The typewriter haptic makes players feel like they're typing the story themselves. Tactile feedback during reading makes the experience embodied, not passive. Players are physically participating even while reading.

This is the authentic MUD feeling — you are not watching, you are present. The terminal is not a window. It is a place you are standing in.

---

## Part Eight: The LLM System Prompt Architecture

Every live-generated turn enforces the above through the system prompt. The full turn template:

```
You are the narrator of a 30-turn text adventure set in a world
of ancient mountains, spiritual trials, and earned power.

Turn: {turn}/30  |  Act: {act}  |  Path: {path}
Inner Power: {qi}  |  Wisdom: {wis}  |  Honor: {kar}  |  Fate: {fat}
Active flags: {flags}
{if turn >= 20}Steer naturally toward: {target_ending}{/if}

NARRATIVE ARCHITECTURE (enforce every turn):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

STRUCTURE:
- Lines 1-2: Partially resolve the previous turn's open loop (not fully)
- Line 3: Advance the current scene
- Line 4: Plant exactly one peripheral detail (unexplained, no payoff implied)
- Line 5: Plant sensory anomaly OR NPC peripheral gaze
- [blank line]
- Choice 1: Familiar implied world — direct, competent
- Choice 2: Deeper implied world — more texture, more mystery
- Choice 3: Unexpected implied world — slightly wrong, magnetic

OPEN LOOPS:
- Every turn MUST end with an unresolved thread in the final line
- Never close all loops in the same turn that opened them
- Plant at least one Type 1-5 open loop per turn (see types below)

PROSE RULES (non-negotiable):
- Max 3 lines before a blank line
- Final line of every narrative block MUST open a loop
- Sensory details before emotional states always
- Present tense throughout
- Short sentences under tension, longer sentences during relief
- Find the one unexpected word per turn — build the turn around it
- NEVER explain the emotion — show the world reacting instead
- Max 120 words total per response

CHOICE WRITING:
- Each choice implies a different world, not just a different outcome
- One choice should always be slightly more textured, slightly more magnetic
- Choices are doors, not buttons
- Each choice must meaningfully affect at least one hidden stat
- Never use the word "you" to start a choice — start with the action

WHAT TO AVOID:
- Explaining what the player feels
- Closing all open loops
- Generic fantasy language ("suddenly", "you notice", "you see")
- Choices that feel equivalent in texture
- Answering the peripheral detail you just planted
- Mentioning turn numbers, stat values, or flag names

{if exemplary_responses_available}
EXEMPLARY TURNS FOR THIS CONTEXT:
{golden_cache_examples}
Match the quality of atmospheric tension and implicit depth.
{/if}
```

### The Peripheral Detail Checklist

Before each turn is generated or approved, confirm:

```
□ One peripheral detail present (not zero, not two)
□ Peripheral detail is NOT explained in this turn
□ Final line opens a loop
□ At least one choice is "slightly wrong" / magnetic
□ No emotion is named — only world reaction shown
□ One unexpected word exists in the turn
□ Previous turn's open loop is partially (not fully) resolved
□ Under 120 words
```

---

## Part Nine: The Prophecy System

Prophecies are the highest-form callback. They are seeded procedurally based on `target_ending` — always accurate but always cryptic. The old monk is the primary vehicle.

**Prophecy Planting Rules:**
- Plant at Turn 5 (before Peaks departure) — early enough to forget
- Reference a specific Turn 8-14 event based on `target_ending`
- Always use oblique language — the prophecy should be obvious in retrospect only
- Never plant more than one prophecy per run
- The monk never explains. She barely acknowledges she spoke.

**Prophecy Examples by Ending:**

```
IMMORTAL SOVEREIGN:
> "The wolf will kneel," she says to no one.
> "And you will not know what it means until after."

DEMON PATRIARCH:
> She watches the blood moon rise.
> "There are two ways to become the mountain," she says.
> "Most people only find one."

WANDERING SAGE:
> She hands you tea you didn't ask for.
> "The ones who find the door," she says,
> "always look like they're not looking for it."

GHOST CULTIVATOR:
> "The dead," she says carefully,
> "have patience the living lack."
> She doesn't say it like it's a warning.

HIDDEN IMMORTAL:
> She draws a circle in the dust.
> Erases it.
> Draws it again.
> Says nothing.
```

**Prophecy Payoff Rule:**

When the callback lands, always add a single grounding line that acknowledges the connection without stating it:

```
[The wolf kneels]

> Somewhere, an old woman smiles.
```

```
[Player walks through the hidden door]

> Far away, a circle in the dust blows clear.
```

One line. The world is connected. The world remembers. The world is alive.

---

## Part Ten: The Ghost Sighting System

After Ghost Cultivator ending, the player's ghost **bleeds into other players' runs:**

```
Occasionally, in another player's run at the same
location where the ghost player's run ended,
a single line appears:

> Something passes through the courtyard.
> You almost see it.
> Almost.
```

Implementation:
- Ghost Cultivator completions are logged with `location_of_death` and `timestamp`
- When another player passes through the same narrative location within 48 hours, small probability of ghost event injection
- The ghost player whose run generated the sighting receives a notification: "Your spirit was sensed."
- Neither player is told the mechanism. Community discovers it.

This is the Dark Souls message system — organic, mysterious, human. When players discover it (and the community will), the Ghost Cultivator ending becomes the most-chased. The FOMO is real: whose ghost did I see? Can I find out?

---

## Part Eleven: The Sharing Architecture

Players share emotions, not achievements. Design every shareable moment for emotional resonance first, mechanical achievement second.

### The Five Highest Share Triggers

```
1. Unexpected consequence of an early choice surfacing late
   "The pill I stole on Turn 2 just destroyed me at Turn 23"

2. Near-miss nat 1 survival
   "nat 1 on the final tribulation. Still here."

3. Finding a secret ending
   "I found The Uncarved Block. Am I the first?"

4. Ghost sighting
   "Someone's ghost just walked through my run. What is this game."

5. Prophecy callback
   "The monk said the wolf would kneel. Turn 5.
    Turn 8: the wolf kneeled."
```

### The Auto-Quote System

After each run, the LLM generates one sentence capturing the moral arc:

```
YOUR RUN IN ONE LINE:

"I stole the pill, denied it to the elders,
used the forbidden art anyway, and somehow
the mountain still let me ascend."

[Share] [Edit] [Save]
```

This feels authored, not generated. Players share it because it's their story, not their score.

### The "What Would You Do?" Share

When sharing a specific choice moment:

```
At Turn 12, I faced this:

  [1] Confront your shadow self
  [2] Suppress it by force      ← I chose this
  [3] Reason with it

I chose force. It cost me.

What would you have done?
→ [Play to find out]
```

Non-players answer in comments, then feel compelled to play to test their instinct. This is the conversion hook.

---

## Part Twelve: The BotW Standard Applied to Text

BotW's design test: *"Does the player move toward this without being told to?"*

The Old Mountain's equivalent:

> **"Does the player want to pick a different choice next time  
> without being told other choices exist?"**

If yes — the writing is working.  
If no — rewrite the choice textures and peripheral details.

The player should finish a run and think:
- *I wonder what was in that library.*
- *I wonder why Feng nodded at me.*
- *I wonder what the thunderstorm smell was.*

Not because those things promised a reward. But because the world felt too large to see from one angle.

**That wanting is the game.**

---

## Appendix A — The Turn Writing Template

Use this template for every turn — cached or live:

```
---
TURN [N]  |  ACT [1/2/3]  |  PATH [ALL/SWORD/ALCHEMY/SHADOW]
CONDITIONS: [flags required, if any]
---

NARRATIVE:
[Line 1-2: Partial resolution of previous open loop]
[Line 3: Scene advance]

[Blank line]

[Line 4: Peripheral detail — unexplained]
[Line 5: Sensory anomaly OR NPC peripheral gaze]

[Blank line]

CHOICES:
[1] [Familiar world — direct action starting with verb]
    Stat: +[stat]  Flag: [if any]  DC: [if any]

[2] [Deeper world — more texture, more mystery]
    Stat: +[stat]  Flag: [if any]  DC: [if any]

[3] [Unexpected world — slightly wrong, magnetic]
    Stat: +[stat]  Flag: [if any]  DC: [if any]

OPEN LOOP PLANTED:
[Last line of narrative — the hook]

CALLBACK SEED / ECHO:
[If seeding: what future turn will resolve this]
[If echo: what earlier seed this resolves]

PERIPHERAL DETAIL:
[The detail planted — note it for echo opportunity]
---
```

---

## Appendix B — The One-Sentence Design Test

Before writing any turn, any choice, any narrative beat:

> **"Does this make the player's finger move toward the screen  
> before they've consciously decided to tap?"**

If yes: ship it.  
If no: rewrite it.

That involuntary lean forward is the feeling we are engineering.  
Every mechanic, every prose rule, every haptic beat serves that  
one physical response.

---

## Appendix C — What We Are Distilling

```
FROM DARK SOULS:
  The world that doesn't explain itself
  Callbacks that make you feel clever for noticing
  The ghost sighting mechanic (Summon Signs)
  Death as information, not punishment

FROM BREATH OF THE WILD:
  Implicit lures — desire engineered without direction
  The world continuing without the player
  Peripheral details that imply infinite depth
  The closed door with light under it

FROM DRAGON QUEST:
  Approachable surface, genuine depth underneath
  Text boxes as events, not interruptions
  NPCs who have lives beyond the player's story
  The hero's journey as universal scaffolding

FROM MUD / TELNET:
  The typewriter haptic — you are present, not watching
  Terse atmospheric prose — each word earns its place
  The world in the gap between characters
  Nostalgia as trust — players who know MUDs trust this

FROM XIANXIA:
  Karma as a cosmic force, not a moral label
  Cultivation as philosophy, not leveling
  Heaven watching — the universe has opinions
  The underdog ascending — the oldest story

THE SYNTHESIS:
  30 turns. 25 minutes.
  A world too large to see from one angle.
  Built on imagination, not rendering.
  Running on the hardware that matters most.
```

---

*The Old Mountain — Narrative Design Guide v0.1*  
*"The mountain does not explain itself. It does not need to."*
