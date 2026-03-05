# 👁️ SPARKS & EMBER — Robot Eyes for the Cheap Yellow Display

Two animated robot faces for the **ESP32-2432S028R** (Cheap Yellow Display). Each runs its own personality, palette, and name. When both are on the same WiFi they find each other automatically and react to each other like friends — not mirrors.

Flash `sparks.ino` to one board. Flash `ember.ino` to the other. Done.

---

## The Two Characters

| | SPARKS | EMBER |
|---|---|---|
| Personality | Sharp, bold, electric | Warm, soft, expressive |
| Resting eye colour | Cyan | Bubblegum pink |
| LED palette | Teal / red / green | Lilac / rose / coral |
| Halloween | Orange flicker | Purple flicker |
| Christmas | Red / green alternating | Pink / white alternating |
| Spooky mode (midnight tap) | Green eyes, RGB off | Pink-purple eyes, RGB off |

---

## Hardware

| Part | Detail |
|------|--------|
| Board | ESP32-2432S028R (Cheap Yellow Display) |
| Display | 2.8" ILI9341, 320×240, SPI (VSPI) |
| Touch | XPT2046 resistive, SPI (HSPI) |
| RGB LED | Onboard active-low, pins 4 / 16 / 17 |
| Backlight | Pin 21 PWM |

Tested on: **AITRIP 2 Pack ESP32-2432S028R** (Amazon).

---

## Dependencies

Install via **Arduino Library Manager**:

| Library | Author |
|---------|--------|
| `TFT_eSPI` | Bodmer |
| `XPT2046_Touchscreen` | Paul Stoffregen |

Included with the ESP32 Arduino core (no separate install):
`WiFi.h` · `WiFiUdp.h` · `time.h` · `EEPROM.h` · `SPI.h` · `math.h`

---

## TFT_eSPI Setup

Edit `TFT_eSPI/User_Setup.h` in your Arduino libraries folder:

```cpp
#define ILI9341_DRIVER
#define TFT_MISO 12
#define TFT_MOSI 13
#define TFT_SCLK 14
#define TFT_CS   15
#define TFT_DC    2
#define TFT_RST  -1
#define SPI_FREQUENCY  55000000
```

The touch controller runs on HSPI separately — no conflict.

---

## Configuration

At the top of each `.ino` file:

```cpp
#define WIFI_SSID  "your_network"
#define WIFI_PASS  "your_password"
#define TZ_OFFSET  (-6 * 3600)   // UTC offset in seconds
```

Both files must use the **same WiFi network** for the friend system to work.

**Timezone offsets:**

| Zone | Value |
|------|-------|
| PST (UTC-8) | `(-8 * 3600)` |
| MST (UTC-7) | `(-7 * 3600)` |
| CST (UTC-6) | `(-6 * 3600)` |
| EST (UTC-5) | `(-5 * 3600)` |
| UTC | `0` |
| CET (UTC+1) | `(1 * 3600)` |

---

## Screens

**Swipe left or right** anywhere to switch pages.

### Eyes Screen
Main face. All animations and touch interactions happen here.

### Clock Screen
- Large 12-hour time in current eye colour
- Full date
- **Your** name and pet counts (today / total)
- Divider line
- **Friend's** name, online status dot, and their pet counts
- Tap friend's name area to send a poke

---

## Expressions

Ten expressions per character. Each has unique eyelid shape, eye colour, and RGB LED colour. Transitions use staggered per-lid lerping — inner corners move first, outer follow. Eyes drift back toward centre during expression changes so there's no jarring position snap.

| Expression | Sparks eye | Ember eye | When |
|-----------|-----------|----------|------|
| NORMAL | Cyan | Bubblegum pink | Default |
| HAPPY | Warm yellow-white | Peach-pink | Weekends |
| SAD | Soft blue | Soft lavender | Random |
| ANGRY | Red | Deep rose | Before 9am |
| SURPRISED | Bright yellow | Coral pink | Random |
| SUSPICIOUS | Purple | Violet | Random |
| SLEEPY | Dim grey-cyan | Dusty mauve | After 10pm / before 6am |
| CONFUSED | Amber | Pale rose | Random |
| EXCITED | Magenta | Hot pink | Random |
| EMBARRASSED | Rose pink + blush | Warm blush | Random |

**EMBARRASSED** draws soft pink blush ellipses below each eye on the display.  
**CONFUSED** renders one eye higher than the other — asymmetric height offset.  
**EXCITED** oscillates both eyes vertically with a rapid bounce and strobes the RGB LED.

---

## Idle Animations

15 animations fire randomly during idle. Each expression has a weighted pool so context feels right. Behaviour chains link related animations.

| Animation | What it does | Mood bias |
|-----------|-------------|-----------|
| Squint | One eye slowly closes to 70%, holds, pops open | ANGRY |
| Startled | Both eyes snap wide | SURPRISED |
| Sneeze | 3 rapid scrunches + screen shake | — |
| Dizzy | Eyes orbit in opposite directions | — |
| Yawn | Lids droop, mouth shape opens, slow recovery | SLEEPY ×3 |
| Think | Eyes drift up-right, one squints | NORMAL, SUSPICIOUS |
| Wink | One eye closes, other perfectly still | HAPPY ×2 |
| Eye roll | Eyes sweep sideways and return | SAD, SUSPICIOUS |
| Smug | Eyes drift sideways, one squints, long hold | SUSPICIOUS |
| Exasperated blink | 3× slower blink | SAD, ANGRY |
| Side-eye | Hard snap to edge, long hold, snap back | SUSPICIOUS |
| Excited bounce | Fast vertical oscillation | HAPPY ×3 |
| Sleepy droop | Three rounds of drooping and jolting awake | SLEEPY ×3 |
| Startled blink | Reflex snap-close in ~30ms | ANGRY, SURPRISED |
| Pet joy | Touch-only — eyes swell, crinkle, bounce | Touch only |

**Behaviour chains:** yawn → sleepy droop · startled blink → startled · sleepy droop → yawn · think → side-eye or smug · startled → startled blink

---

## Touch Interactions

All interactions fire on the eyes screen only.

| Gesture | Reaction |
|---------|----------|
| **Tap** | 2–4 hearts float up, face goes HAPPY, pet joy animation, pet counter saves |
| **Double-tap** (2 taps < 350ms) | Eyes snap wide, SURPRISED, 6 hearts burst |
| **Triple-tap** (3 taps < 600ms) | 10 star particles explode radially, CONFUSED + dizzy |
| **Tap while ANGRY** | Red-orange zigzag sparks shoot up, eyes narrow further |
| **Hold (0–1.5s)** | Eyes widen progressively up to 25%, SURPRISED anticipation at 800ms |
| **Hold then release (1.5s+)** | Relief — randomly sneezes or startled jump, settles to NORMAL |
| **5 double-taps in a row** | OVERSTIMULATED — hearts + sparks + stars fire everywhere, all touch blocked for 30s |
| **Drag** | Eyes track finger, drift damped while touching |
| **Swipe** | Switch between eyes and clock screens |
| **Tap friend name (clock screen)** | Sends a poke to the other device |

---

## Particle Effects

All particles are drawn inside the sprite — zero screen artifacts.

### Hearts
Pink hearts spawn at tap position, float upward, shrink and fade over ~1.5s. Up to 6 simultaneous.

### Angry Sparks
8 red-orange particles with zigzag horizontal movement and gravity. Fade red → orange → yellow.

### Star Burst
10 yellow-white stars radiate outward on triple-tap or overstimulated. Arc with gravity.

### Confetti
20 multi-colour rectangles rain downward on milestone celebrations and New Year's midnight. Each piece tumbles and fades.

### Sleep ZZZs
After SLEEPY holds for 5+ seconds, small Z characters drift up from the right eye every 1.5–3 seconds. Soft blue-green, sway gently, stop when expression changes.

---

## Easter Eggs

| Trigger | What happens |
|---------|-------------|
| **5 double-taps in a row** | OVERSTIMULATED — chaos mode, all particles fire at once, 30s rest, all touch blocked |
| **Triple-tap** | Dizzy stars radiate from tap point, CONFUSED expression |
| **Tap at exactly midnight (00:00)** | SPOOKY MODE — eyes change colour (Sparks: green, Ember: pink-purple), RGB off, lasts 5 minutes |

---

## Milestone Celebrations

Every milestone fires **exactly once** and is saved to EEPROM — it never repeats even after power-off.

| Pets total | Celebration |
|-----------|-------------|
| **10** | Pet joy animation + heart shower across the screen |
| **50** | EXCITED expression, star bursts from both eye positions simultaneously |
| **100** | Full confetti explosion, hearts everywhere, rainbow RGB cycle for 5 seconds |
| **500** | Everything at once — confetti, stars, hearts, sparks, RGB white flash then rainbow for 8 seconds |

When a milestone fires, the **friend device** also celebrates with a small sympathetic burst — hearts, stars, excited bounce — after a short random delay.

---

## Seasonal Events

Checked every 60 seconds via NTP. Require WiFi.

| Date | Sparks | Ember |
|------|--------|-------|
| **Halloween** (Oct 31) | Orange eyes, flickering orange RGB | Purple eyes, flickering purple RGB |
| **Christmas** (Dec 25) | Eyes alternate red/green every 1.5s | Eyes alternate pink/white every 1.5s |
| **New Year's Eve** (Dec 31, 11:55pm+) | Gold eyes, fast RGB rainbow cycle | Gold eyes, fast RGB rainbow cycle |
| **New Year's midnight** | Confetti + stars + hearts explosion | Confetti + stars + hearts explosion |

---

## Friend System

Both devices broadcast state packets over **UDP on port 4242** using subnet broadcast. No server, no pairing, no configuration — just both on the same WiFi.

### How they find each other
Both boot up broadcasting `PKT_HELLO` packets for 5 seconds. When one hears the other they exchange state immediately. When first seen, both do an excited greeting — EXCITED expression, heart shower, bounce animation.

### Friendship reactions

| Event on one device | Reaction on the other (with natural delay) |
|---------------------|-------------------------------------------|
| Gets petted | Side-eye glance, then happy wink |
| Goes ANGRY | SUSPICIOUS + side-eye — "uh oh" |
| Goes SLEEPY | Sympathetic yawn after 1–3s |
| Goes SURPRISED | Startled blink reflex |
| Hits a milestone | Small celebration — hearts, stars, excited bounce |
| Comes back online | Both go EXCITED, heart shower greeting |
| Goes offline | Goes SAD |

All reactions have randomised delays so they feel natural, not instant.

### Poking
On the clock screen, tap the friend's name area (the row with their name and pet count). Their board gets startled, then goes happy with hearts. Their screen flashes magenta briefly to confirm the poke was received.

### Clock screen status
- **Green dot** = friend online, showing their name and pet counts
- **Dim red dot** = friend offline or not yet seen

### Packet types

| Type | Sent when |
|------|-----------|
| `PKT_HELLO` | Boot, and every 500ms during 5s discovery window |
| `PKT_STATE` | Every 1 second during normal operation |
| `PKT_POKE` | When friend name is tapped on clock screen |
| `PKT_MILESTONE` | When a pet milestone is reached |

---

## Pet Counter

Every tap-pet increments two EEPROM counters:

- **petsToday** — resets at midnight (checked via `tm_yday` on boot)
- **petsTotal** — lifetime, never resets

Both shown on clock screen for each device.

**EEPROM layout:**

| Address | Data | Size |
|---------|------|------|
| 0 | petsToday | int (4 bytes) |
| 4 | petsTotal | int (4 bytes) |
| 8 | lastDay (tm_yday) | int (4 bytes) |
| 12 | robotName | char[20] |
| 32 | milestoneFlags | byte (bitfield) |
| 33 | deviceId | byte |

Total: 34 bytes of 128-byte allocation.

---

## Fluency System

**Ease-out cubic lerp** — all movement decelerates into targets.

**Momentum + drag** — eyes have velocity and mass. Spring pulls toward drift target, drag bleeds energy. Drag varies by mood: ANGRY snaps fast, SLEEPY crawls.

**Overshoot + settle** — on fast large moves, a small kick past the target then spring back. Threshold tuned so it only fires on genuinely fast movements.

**Micro-jitter** — ±0.3px noise refreshed every 75ms removes the "too perfect" quality of pure lerp.

**Attention variance** — after 8+ seconds of stillness, next drift target is 1.5–1.6× larger than normal.

**Per-lid staggered lerp** — inner eyelid corners move faster than outer. Anatomically correct.

**Transition neutral pause** — 90ms hold at neutral when crossing between extreme expressions. Eyes drift back toward centre during this pause.

**Asymmetric blink** — close and open speeds separate per mood.

**Lid overshoot on open** — eyes go fractionally wider after a blink opens, then settle.

**Breath-linked blink rate** — slower breathing (SLEEPY) = rarer blinks.

**Eye breathing** — continuous sine wave adds ±2px to eye height.

---

## Wake Animation

On every boot:

1. Screen fades in from black over ~600ms, eyes shut
2. Three REM-style flickers — half-open and close like dreaming
3. Groggy half-open at ~45%, holds 500ms
4. Closes again — resisting waking up
5. Opens fully with small lid overshoot
6. RGB LED fades in to correct expression colour over 800ms

---

## Rendering

A single `320×140` sprite (~87KB RAM) covers the eye region. Both eyes, all lid animations, hearts, and mouth are drawn into the sprite each frame, then pushed to the display in one atomic call. Zero flicker — the display never sees a partial frame.

Particles outside the sprite band (sparks, confetti, ZZZs, stars) draw on the raw TFT but handle their own per-pixel erase each frame.

---

## Touch Wiring

XPT2046 on HSPI — separate from display VSPI.

| Signal | GPIO |
|--------|------|
| CLK | 25 |
| MISO | 39 |
| MOSI | 32 |
| CS | 33 |
| IRQ | 36 |

Touch is polled directly rather than using the IRQ pin, which is unreliable on some board revisions.

---

## Serial Monitor

Baud: **115200**

```
WiFi.... OK
Heap: 187432
I am SPARKS (device A)
Friend online: EMBER
```

---

## File Structure

```
sparks/
└── sparks.ino     — flash to Sparks board (~2200 lines)

ember/
└── ember.ino      — flash to Ember board (~2200 lines)

README.md
ENCLOSURE_DESIGN.md
```

Each `.ino` must be inside a folder of the same name for Arduino IDE to accept it.

---

## Troubleshooting

**Blank screen on boot** — sprite allocation failed. Free heap below ~95KB. Check Serial Monitor for the heap value printed at boot.

**Friend never shows as online** — both boards must be on the same WiFi subnet. Check that your router doesn't block UDP broadcast between clients (some guest networks do). Verify both use the same `FRIEND_PORT` (4242).

**Wrong names on clock screen** — EEPROM from a previous sketch may have stale data. The name is hardcoded in each file's globals and writes to EEPROM on first boot. If it still shows wrong, add `EEPROM.write(EEPROM_NAME_ADDR, 0); EEPROM.commit();` to setup temporarily to force a reset.

**Pet counts wrong after reflash** — EEPROM persists across flashes. This is intentional. To reset counts, temporarily add `EEPROM.put(0, 0); EEPROM.put(4, 0); EEPROM.commit();` to setup.

**Milestone fired but friend didn't celebrate** — the friend must be online at the moment the milestone packet is sent. If they're offline it won't queue, it just misses.

**Touch not responding** — confirm `XPT2046_Touchscreen` by Paul Stoffregen is installed. Board must be specifically the ESP32-2432S028R.

**Seasonal colours not showing** — requires NTP sync (WiFi). Seasonal check runs every 60 seconds after boot, so may take up to a minute to activate.

---

## License

MIT — build whatever you want, credit appreciated.

---

*Sparks & Ember. Two little faces that miss each other when the WiFi goes down.*
