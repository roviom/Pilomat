# Pilomat III - Architecture Design

## Overview

A modern, event-driven timing system for archery competitions with full JSON configurability.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Pilomat III                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Master     │  │  Competition │  │   Display    │     │
│  │   Timer      │  │   Engine     │  │   Manager    │     │
│  │              │  │              │  │              │     │
│  │ • Tick 0.1s  │  │ • Lines      │  │ • Timers     │     │
│  │ • Count up   │  │ • Sequences  │  │ • Lights     │     │
│  │ • No reset   │  │ • Auto-adv   │  │ • Buzzers    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                 │              │
│         └─────────┬───────┴─────────┬───────┘              │
│                   │                 │                      │
│           ┌───────▼─────────────────▼───────┐              │
│           │     Event Dispatcher            │              │
│           │  • Time-based triggers          │              │
│           │  • Condition evaluation         │              │
│           │  • Action execution             │              │
│           └────────────┬────────────────────┘              │
│                        │                                   │
│                ┌───────▼────────┐                          │
│                │  Configuration │                          │
│                │  (JSON Driven) │                          │
│                └────────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Master Timer
**Purpose:** Single source of truth for time

**Responsibilities:**
- Count tenths of seconds (0.1s resolution)
- Never resets during competition
- Provides absolute time reference

**State:**
```javascript
{
  masterTicks: 0,           // Tenths of seconds since start
  running: true,            // Timer active
  lastUpdate: timestamp     // For sync
}
```

**Interface:**
```javascript
timer.start()
timer.pause()
timer.resume()
timer.getTime()  // Returns current ticks
```

---

### 2. Competition Engine
**Purpose:** Manages competition flow and line progression

**Responsibilities:**
- Track current line, end, round
- Auto-advance to next line
- Calculate time offsets for each phase
- Handle material faults
- Manage breaks

**State:**
```javascript
{
  // Current position
  currentRound: 1,
  currentEnd: 1,
  currentLine: 0,          // Index into config.lines array
  currentPhase: "idle",    // idle, prep, shooting, warning, retrieve
  
  // Timing
  phaseStartTime: 0,       // Master timer value when phase started
  phaseEndTime: 0,         // When phase should end
  
  // Line management
  linesInEnd: ["A", "B"],  // From config
  linesCompleted: 0,       // Count of completed lines in current end
  
  // Special states
  materialFault: {
    active: false,
    remainingArrows: 0,
    timePerArrow: 0
  },
  break: {
    active: false,
    savedTime: 0,
    savedPhase: null
  }
}
```

**Methods:**
```javascript
engine.startEnd()           // Begin new end
engine.startLine()          // Begin current line
engine.advanceToNextLine()  // Move to next line
engine.endLine()            // Complete current line
engine.calculatePhaseTime() // Get time for current phase
engine.handleMaterialFault(arrowsRemaining)
engine.handleBreak()
engine.resumeFromBreak()
```

---

### 3. Display Manager
**Purpose:** Control visual and audio outputs

**Responsibilities:**
- Update countdown/countup displays
- Control traffic lights
- Trigger buzzers
- Show current state info

**State:**
```javascript
{
  displays: {
    left: { value: 0, paused: false, countDown: true },
    center: { value: 0, paused: false, countDown: true },
    right: { value: 0, paused: false, countDown: true }
  },
  lights: {
    red: false,
    yellow: false,
    green: false
  },
  statusText: "Ready - LINE A"
}
```

**Methods:**
```javascript
display.setLight(color)
display.setTimer(timer, value, countDown)
display.pauseTimer(timer)
display.resumeTimer(timer)
display.buzz(pattern)
display.setStatus(text)
```

---

### 4. Event Dispatcher
**Purpose:** Execute actions at the right time

**Responsibilities:**
- Monitor master timer
- Evaluate trigger conditions
- Execute actions when conditions met
- Handle event priorities

**Event Types:**

**A) Time-Based Events**
```javascript
{
  type: "absolute",         // Trigger at specific master time
  time: 100,               // Seconds
  actions: [...]
}
```

**B) Phase-Relative Events**
```javascript
{
  type: "phaseRelative",   // Relative to phase start
  phase: "shooting",
  offset: 90,              // 90s into shooting phase
  actions: [...]
}
```

**C) Phase-End-Relative Events**
```javascript
{
  type: "beforePhaseEnd",  // Before phase ends
  phase: "shooting",
  offset: 30,              // 30s before end
  actions: [...]
}
```

**D) Condition-Based Events**
```javascript
{
  type: "condition",
  condition: "lineComplete",
  actions: [...]
}
```

**Methods:**
```javascript
dispatcher.checkEvents(currentTime, engineState)
dispatcher.executeAction(action)
dispatcher.clearEventQueue()
```

---

## Configuration Structure

### Modern JSON Format

```json
{
  "name": "WA Indoor 60 Arrows",
  "version": "3.0",
  
  "competition": {
    "type": "indoor",
    "lines": ["A", "B"],
    "arrowsPerLine": 3,
    "autoAdvanceLines": true,
    "pauseAfterLastLine": true
  },
  
  "timing": {
    "prepTime": 10,
    "shootingTime": 120,
    "warningTime": 30,
    "retrieveTime": 0
  },
  
  "phases": [
    {
      "name": "preparation",
      "duration": "timing.prepTime",
      "light": "red",
      "displayMode": "countDown",
      "onEnter": {
        "buzzer": { "pattern": "double" },
        "display": { "value": "phase.duration" }
      }
    },
    {
      "name": "shooting",
      "duration": "timing.shootingTime",
      "light": "green",
      "displayMode": "countDown",
      "onEnter": {
        "buzzer": { "pattern": "single" },
        "display": { "value": "phase.duration" }
      },
      "during": [
        {
          "when": "timing.warningTime",
          "type": "beforeEnd",
          "actions": {
            "light": "yellow"
          }
        }
      ],
      "onExit": {
        "condition": "isLastLine",
        "then": {
          "buzzer": { "pattern": "triple" },
          "pauseCompetition": true
        },
        "else": {
          "buzzer": { "pattern": "double" },
          "advanceToNextLine": true
        }
      }
    }
  ],
  
  "materialFault": {
    "enabled": true,
    "timePerArrow": "timing.shootingTime / competition.arrowsPerLine"
  },
  
  "display": {
    "mode": "center",
    "showLineInfo": true,
    "showEndInfo": true
  }
}
```

---

## Data Flow

### Normal Competition Flow

```
1. START pressed
   ↓
2. Engine: Load config, set currentLine=0
   ↓
3. Engine: Calculate phase times
   ↓
4. Engine: Enter "preparation" phase
   ↓
5. Dispatcher: Execute onEnter actions
   - Display: Set timer to 10s countdown
   - Display: Red light
   - Display: 2 beeps
   ↓
6. Timer: Count (master continues)
   ↓
7. Dispatcher: At prep end → transition to "shooting"
   ↓
8. Engine: Enter "shooting" phase
   ↓
9. Dispatcher: Execute onEnter actions
   - Display: Set timer to 120s countdown
   - Display: Green light
   - Display: 1 beep
   ↓
10. Timer: Count
    ↓
11. Dispatcher: At 90s → Execute "during" action
    - Display: Yellow light
    ↓
12. Timer: At 120s → Phase ends
    ↓
13. Engine: Check if more lines
    - If yes: currentLine++, goto step 3
    - If no: Execute onExit with pauseCompetition
    ↓
14. Display: 3 beeps, pause, wait for START
```

### Material Fault Flow

```
1. MATERIAL FAULT pressed during shooting
   ↓
2. Engine: Calculate remaining time
   remainingTime = (remainingArrows × timePerArrow)
   ↓
3. Engine: Save fault state
   materialFault.active = true
   materialFault.remainingArrows = X
   ↓
4. Engine: End current line early
   ↓
5. Engine: Continue with next lines normally
   ↓
6. When last line ends:
   ↓
7. Engine: Check if material fault active
   ↓
8. Engine: Enter special "makeup" phase
   - Display: Show "MATERIAL FAULT - Skytt X"
   - Display: Set timer to remainingTime
   - Light: Red → Green
   ↓
9. Timer: Count down makeup time
   ↓
10. Timer: Ends → Complete end normally
```

---

## Key Design Patterns

### 1. Event-Condition-Action (ECA)
All behavior follows: **When X happens → If Y condition → Do Z action**

```javascript
{
  when: "phaseTime >= 90",
  if: "phase === 'shooting'",
  do: { light: "yellow" }
}
```

### 2. Configuration Inheritance
Timing values can reference other config values:
```json
"timePerArrow": "timing.shootingTime / competition.arrowsPerLine"
```

### 3. State Machine with Guards
Phase transitions have conditions:
```javascript
transition(from: "shooting", to: "preparation") {
  guard: isNotLastLine()
  action: advanceToNextLine()
}

transition(from: "shooting", to: "retrieve") {
  guard: isLastLine()
  action: pauseForRetrieve()
}
```

### 4. Command Pattern for Actions
All actions are objects that can be queued, logged, and undone:
```javascript
{
  type: "SetLight",
  params: { color: "yellow" },
  timestamp: 1234567890,
  execute: function() { ... },
  undo: function() { ... }
}
```

---

## Extensibility Points

### Adding New Competition Types

1. Define in config:
```json
"competition": {
  "type": "finals_alternating",
  "customBehavior": {
    "switchOnArrow": true,
    "manualSwitching": true
  }
}
```

2. Engine recognizes type and adjusts behavior

### Adding New Display Modes

```json
"display": {
  "mode": "triple",  // left, center, right
  "leftTimer": "countUp",
  "centerTimer": "countDown",
  "rightTimer": "pause"
}
```

### Adding New Actions

```javascript
// Register new action type
actionRegistry.register("playSound", (params) => {
  audio.play(params.soundFile);
});

// Use in config
{
  "actions": {
    "playSound": { "soundFile": "whistle.mp3" }
  }
}
```

---

## Error Handling

### Graceful Degradation
- If server unavailable → Local mode continues
- If config invalid → Load default safe config
- If timer desync → Resync on next START

### Recovery Mechanisms
- Emergency stop → Save state, allow resume
- Power loss → State saved to SPIFFS every 10s
- Button mispress → Undo buffer (last 3 actions)

---

## Testing Strategy

### Unit Tests
- Timer accuracy (±0.1s over 10 minutes)
- Phase calculations
- Line progression logic
- Material fault calculations

### Integration Tests
- Full end cycles
- Multi-line scenarios
- Material fault flows
- Break/resume flows

### Configuration Tests
- Valid config validation
- Invalid config rejection
- Edge cases (1 line, 6 lines, etc.)

---

## Performance Targets

- **Timing Accuracy:** ±0.1s over 30 minutes
- **Display Update:** < 50ms lag
- **Sync Tolerance:** All clients within 0.1s
- **Button Response:** < 100ms
- **Config Load:** < 500ms

---

## Migration from Current System

### Phase 1: Core Rewrite
- Implement new engine
- Keep old JSON format compatible
- Test extensively

### Phase 2: New Config Format
- Support both old and new formats
- Provide conversion tool
- Documentation

### Phase 3: Deprecation
- Mark old format as deprecated
- Remove after 6 months

---

## Future Enhancements

1. **Remote Control API** - Control from mobile app
2. **Statistics Tracking** - Times, arrows, scores
3. **Multi-Range Sync** - Sync multiple ranges
4. **Voice Announcements** - TTS for phase changes
5. **Visual Themes** - Customizable colors/layouts
6. **Replay Mode** - Review competition timeline

---

*This architecture prioritizes clarity, maintainability, and extensibility while honoring the proven concepts from Pilomat II.*
