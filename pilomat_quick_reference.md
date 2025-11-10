# Pilomat III - Quick Reference Card

## Competition Day Setup

### 1. Power On
- Connect ESP32 to power
- Wait 10 seconds for WiFi connection
- Green LED indicates ready

### 2. Connect Tablets
- Connect to WiFi: `[Your WiFi Name]`
- Open browser (Safari/Chrome)
- Go to: `http://[IP Address shown on Serial Monitor]`
- Add to home screen for fullscreen

### 3. Load Configuration
- Click "📁 Load Configuration JSON"
- Select competition format
- Verify name appears at top

### 4. Test System
- Press "TEST SYSTEM" or equivalent
- Verify lights change
- Verify buzzer sounds
- Verify timer counts down

---

## WA Indoor 60 Arrows (AB-AB)

### Timing Sequence
| Time | Event | Signal | Light |
|------|-------|--------|-------|
| 0:00 | Preparation | - | 🔴 Red |
| 0:05 | 5s warning | 1 beep | 🔴 Red |
| 0:10 | Line A shoots | 2 beeps | 🟢 Green |
| 1:40 | 30s warning | 1 beep | 🟡 Yellow |
| 2:40 | Stop shooting | 3 beeps | 🔴 Red |
| 5:40 | 30s warning | 1 beep | 🔴 Red |
| 5:40 | Line B shoots | 2 beeps | 🟢 Green |
| 7:10 | 30s warning | 1 beep | 🟡 Yellow |
| 8:10 | Stop shooting | 3 beeps | 🔴 Red |
| 10:40 | 30s warning | 1 beep | 🔴 Red |

**Total cycle: 11 minutes 10 seconds**

### Manual Controls
- **START END (3+3)** - Begin new end
- **LINE A - SHOOT** - Manual start Line A
- **LINE B - SHOOT** - Manual start Line B
- **WARNING 30s** - Manual 30s warning
- **STOP SHOOTING** - Manual stop
- **RETRIEVE ARROWS** - Manual retrieve
- **EMERGENCY STOP** - Red light + pause

---

## WA Finals - Alternating Individual

### Timing
- **20 seconds per arrow**
- No automatic signals during shooting
- Manual control only

### Button Workflow

**Starting the Match:**
1. Press **START MATCH (LEFT)** or **START MATCH (RIGHT)**
2. First archer begins with 20s on their timer
3. Opponent's timer is paused

**During Match:**
1. Archer shoots arrow
2. Press **SWITCH TO LEFT** or **SWITCH TO RIGHT**
3. Timer resets to 20s for next archer
4. Previous timer pauses

**Common Situations:**

| Situation | Button | Result |
|-----------|--------|--------|
| Archer finishes | ◄ SWITCH or SWITCH ► | Switch to opponent |
| Equipment failure | ⏸ PAUSE MATCH | Both timers pause |
| Resume after pause | ▶ RESUME LEFT/RIGHT | Resume appropriate timer |
| Serious incident | ⚠ EMERGENCY STOP | Red light, all stop |
| Match complete | END MATCH | Red light, 3 beeps |
| Time adjustment | ADD 10s LEFT/RIGHT | Add time for delays |

---

## Traffic Light Meanings

| Color | Meaning | Action |
|-------|---------|--------|
| 🔴 **RED** | Do not shoot | Arrows on bows, wait |
| 🟢 **GREEN** | Shooting allowed | Shoot your arrows |
| 🟡 **YELLOW** | 30 seconds remaining | Finish current arrows |

---

## Buzzer Signals

| Signal | Meaning |
|--------|---------|
| 1 beep | Warning / Transition |
| 2 beeps | START shooting |
| 3 beeps | STOP shooting |
| 5+ beeps | EMERGENCY - all stop |

---

## Troubleshooting

### Display Not Updating
1. Check WiFi connection
2. Refresh browser (pull down)
3. Check "Mode: Synchronized" in status

### Wrong Configuration Loaded
1. Press "📁 Load Configuration JSON"
2. Select correct file
3. Press START to begin

### Timer Stuck or Wrong
1. Press EMERGENCY STOP
2. Press START END to reset
3. Manual buttons work anytime

### Multiple Tablets Out of Sync
- All tablets sync to ESP32 automatically
- If one is wrong, refresh that tablet
- All should show same time within 0.1s

### Emergency Situations

**Equipment Failure:**
1. Press EMERGENCY STOP
2. Resolve issue
3. Add time if needed (ADD 10s buttons)
4. Press RESUME or appropriate START

**Power Loss:**
- ESP32 auto-restarts
- Reconnect tablets to WiFi
- Load configuration
- Use manual buttons to resume

**Wrong Button Pressed:**
- Press correct button immediately
- System responds to most recent command
- EMERGENCY STOP if needed

---

## Pre-Competition Checklist

- [ ] ESP32 powered on and connected
- [ ] All tablets connected and displaying
- [ ] Correct configuration loaded
- [ ] Traffic lights working (TEST button)
- [ ] Buzzer working (TEST button)
- [ ] All tablets synchronized
- [ ] Backup power available
- [ ] Extra tablets on standby
- [ ] Spare ESP32 programmed (optional)

---

## End-of-Day Shutdown

1. Press EMERGENCY STOP
2. Disconnect tablets
3. Power off ESP32
4. Store equipment safely

---

## Contact Information

**Technical Support:**
- Email: [your email]
- Phone: [your phone]
- Emergency: [backup contact]

**Configuration Files:**
- Stored on ESP32 SPIFFS
- Backup copies: [location]
- Custom configs: [email to request]

---

*Pilomat III - Precision timing for champions* 🎯

**Quick Tips:**
- Green = Go
- Red = Stop
- Yellow = Hurry
- When in doubt, EMERGENCY STOP
- Manual buttons override everything
- Tablets auto-sync, no manual adjustment needed

**Version:** 3.0 | **Date:** 2024