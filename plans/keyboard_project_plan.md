---
title: Voice-Command Keyboard - Project Plan
class: CS 3651 (Prototyping Intelligent Devices)
tags:
  - project
  - cs3651
  - keyboard
  - esp32
  - ml
aliases:
  - intelliProj
---
# Voice-Command Keyboard - Project Plan
CS 3651 (Prototyping Intelligent Devices)

Related: [[Project Planning Conversation]] · [[eink_project_plan]]

## Goal
- 40% keyboard (~12x4 matrix, ~47 keys), hot-swap switches.
- Normal mode: works like a regular USB keyboard.
- Voice mode: press a button, say a command ("close tab"), keyboard sends the shortcut (Cmd+W).
- Speech recognition runs **on the microcontroller** (keyword spotting, fixed command list).
- Small OLED shows the recognized command, or WPM in normal mode.

## Why this matters for grading
- The on-device ML is what makes this an "intelligent device", not a dumb laptop interface.
- Protect that feature. Everything else is supporting it.
- Do **not** drift into "type anything I say" (full dictation). That can't run on an MCU.

## Core vs stretch
Decide this early and put it in the proposal.
- **Core:** matrix + USB HID + 10-15 voice commands + OLED showing command + on-device WPM.
- **Stretch:** system-usage (btop-style) screen fed by a laptop helper app.
- **Stretch:** more commands, custom wake word, layers/macros.
- **Stretch:** custom PCB instead of hand-wiring.

---

## Phase 0 - Setup and organization
*Do first, ~1 day.*
- [ ] Make a GitHub repo for the project.
  - Why: version control for firmware, CAD, and docs. Easy rollback when something breaks.
- [ ] Make folders: `/firmware` `/cad` `/docs` `/photos` `/ml`.
- [ ] Start a running "build log" file.
  - Write the date, what you tried, what worked, what failed.
  - Why: the final documentation needs "iterative process", "ideas that didn't pan out", and "lessons learned". This log **is** that content.
- [ ] Take photos as you go. Same reason.
- [ ] Put key dates on a calendar: proposal, in-person proposal meeting, mid-project review, demo, documentation.

---

## Phase 1 - Research
*~3-5 days. Goal: confirm every part of the plan is possible before buying anything.*

### 1.1 Keyboard basics
- [ ] Read how a key matrix works (rows, columns, diodes, ghosting).
- [ ] Read a hand-wiring guide (search "hand wired keyboard guide" / QMK hand-wire doc).
- Why: you need to understand diode direction and matrix scanning before designing anything.

### 1.2 Firmware approach
- Vanilla QMK/VIA does **not** run on ESP32. Don't plan around it.
- [ ] Look at MK32 and paul356/esp32_keyboard as references only.
- Plan: write your own firmware in ESP-IDF.
  - Matrix scan (your code).
  - USB keyboard output via TinyUSB (`esp_tinyusb` component).
- Why: easier to combine with speech recognition than porting someone else's big firmware. Also clearly your own work.

### 1.3 Speech recognition approach
*Most important research.*
- Option A: Espressif ESP-SR (MultiNet). Prebuilt command recognizer for ESP32-S3. You define the command phrases.
- Option B: Edge Impulse / TensorFlow Lite Micro keyword spotting. You record samples and train your own model.
- [ ] Read the official example for each: what board, what RAM, what mic format they need.
- [ ] Decide A or B (or A first, B as fallback).
- Why: this is the highest-risk element. Knowing the exact requirements decides which board and mic you buy.

### 1.4 Hardware compatibility check
- [ ] Confirm ESP32-S3 with PSRAM is needed (ESP-SR requires PSRAM).
- [ ] Confirm your mic is I2S (INMP441 or SPH0645).
- [ ] Confirm USB HID works on macOS with Cmd = "GUI" modifier.

### 1.5 Pin budget
- [ ] List every pin you need:
  - 12 columns + 4 rows = 16 pins.
  - I2S mic = 3 pins (BCLK, WS, DATA).
  - OLED I2C = 2 pins (SDA, SCL).
  - Voice-mode button = 1 pin. Status LED = 1 pin (optional).
  - Total ~23 pins.
- [ ] Check the DevKitC pinout and **avoid**:
  - GPIO 19/20 (native USB).
  - GPIO 35/36/37 (used by octal PSRAM on N8R8).
  - Strapping pins 0, 3, 45, 46.
- Why: picking the wrong pins is a common, painful bug. Do it on paper first.

> **Result of Phase 1:** a short "decisions" list (board, mic, ML approach, firmware approach, pin map). This feeds straight into the proposal.

---

## Phase 2 - Parts list and ordering
*~2-3 days. Order early.*

Starting list (confirm stock + links before ordering):
- ESP32-S3-DevKitC-1-N8R8 (8MB flash / 8MB PSRAM) - main MCU.
- INMP441 or Adafruit SPH0645 I2S MEMS mic - voice input.
- SSD1306 128x64 I2C OLED - display.
- ~47 mechanical switches (MX-style) + a few spares.
- Hot-swap: Amoeba King single-key PCBs (hand-wire friendly) **or** Kailh hot-swap sockets + custom PCB.
- 1N4148 diodes (only if your hot-swap boards don't already include them) - get 100.
- 1x 2u plate-mount stabilizer (for spacebar, if layout has one).
- Keycaps for a 40% layout (or blanks).
- USB-C cable, hookup wire (thin solid core for rows, enameled or insulated for columns).
- Push button for voice mode, LED + resistor (optional).
- PLA/PETG filament for case and plate.
- M2/M3 screws + heat-set inserts for closing the case.

Steps:
- [ ] Make a spreadsheet: part, purpose, link, price, qty, status (ordered / arrived / tested).
- [ ] Order the MCU and mic **first**. They decide whether the ML works.
- [ ] Order 1-2 spare mics and a spare OLED. They're cheap and fail/break often.
- Why: course says order early to leave time to re-order. Shipping delays kill projects.

> **Result:** everything on order, and a parts spreadsheet you can paste into the proposal.

---

## Phase 3 - Proposal
*Uses Phase 1-2 output.*
- [ ] Written description of the goal.
- [ ] Links to every part and resource.
- [ ] Block diagram: Mic → ESP32-S3 (ML core) → USB HID → Computer; Matrix → ESP32-S3; ESP32-S3 → OLED.
- [ ] "A" feature list (core) + stretch goals.
- [ ] Hard mid-project deliverable. Suggested: "ESP32-S3 recognizes at least 5 spoken commands on-device and prints them; matrix of a few test keys sends keystrokes over USB."
- [ ] **Must** discuss in person with instructor/TA, or it's a 0.

---

## Phase 4 - Breadboard prototype
*Before any case or soldering. Goal: prove each risky piece works alone, then together.*

### 4.1 Board bring-up
- [ ] Install ESP-IDF. Flash "hello world". Blink an LED.
- Why: confirms the toolchain works before debugging real code.

### 4.2 Mic test
- [ ] Wire I2S mic. Read raw audio. Check levels change when you speak.
- Why: bad wiring or wrong I2S config looks exactly like "ML doesn't work". Rule it out first.

### 4.3 Speech recognition test
*Highest risk, do early.*
- [ ] Run the ESP-SR (or Edge Impulse) example as-is.
- [ ] Swap in your own command list: "close tab", "new tab", "spotlight", "quit app", "undo", "copy", "paste", etc.
- [ ] Test in a quiet room, then with background noise. Write down accuracy.
- [ ] Tune: fewer/more distinct phrases, mic gain, distance.
- Why: if this fails, you need to know **now** so you can adjust scope.

### 4.4 USB HID test
- [ ] TinyUSB keyboard example. Send "hello" and Cmd+Space to your Mac.
- Why: confirms the output side and macOS modifier mapping.

### 4.5 Mini matrix test
- [ ] Breadboard 4 switches in a 2x2 matrix with diodes.
- [ ] Write scan code + debounce. Confirm each key sends the right keycode.
- [ ] Press multiple keys at once. Confirm no ghosting.
- Why: debug matrix logic on 4 keys, not 47 soldered ones.

### 4.6 OLED test
- [ ] Display text on SSD1306 over I2C.

### 4.7 Integration
- [ ] Combine all of the above in one firmware using FreeRTOS tasks:
  - Task 1 (core 0): audio capture + speech recognition.
  - Task 2 (core 1): matrix scan + USB HID.
  - Task 3: OLED updates.
- [ ] Voice button toggles voice mode. Recognized command → HID shortcut + OLED text.
- Why: proves everything can run at the same time without starving each other.

> **Result:** a working breadboard keyboard-with-voice. This is a great mid-project review deliverable.

---

## Checkpoint - Mid-project review
*Around the end of Phase 4 / start of Phase 5.*

Course expects by this point:
- [ ] All physical parts in hand (or first pass).
- [ ] Firmware compiles, flashes, prints output.
- [ ] Input working (mic + some keys).
- [ ] Output working (HID + OLED).
- [ ] Online submission + in-person meeting.

Bring: updated block diagram, photos, test results, problems you need help with.
- Why: if speech recognition is struggling, this is when instructors can help re-scope.

---

## Phase 5 - Layout and CAD
*Can overlap with Phase 4.*

### 5.1 Layout
- [ ] Design layout in keyboard-layout-editor.com (KLE).
- [ ] Decide: staggered or ortholinear, spacebar size, where the voice button goes.
- [ ] Export the layout JSON.
- Why: everything else (plate, case, matrix wiring, keycaps) depends on this.

### 5.2 Plate
- [ ] Generate plate from KLE JSON (ai03 plate generator or swillkb builder).
- Switch cutouts 14mm, spacing 19.05mm. Plate ~1.5mm thick so switches clip in.
- [ ] Print a small test section first (3-4 keys). Check switch fit and stabilizer fit.
- Why: printers aren't perfectly accurate. Tune tolerance on a small print, not the full plate.

### 5.3 Case
- [ ] Design in Fusion 360 / Onshape. Import plate outline.
- Include:
  - Mounting for plate (screws or tray mount).
  - Space under plate for wiring + hot-swap boards (measure height).
  - ESP32 mount with USB-C cutout aligned to the board's native USB port.
  - Mic opening (small hole + mesh) away from where hands rest, to reduce key noise.
  - OLED window.
  - Voice-mode button hole.
  - Heat-set insert holes for closing the case.
  - Slight typing angle (optional).
- [ ] Print a test fit with electronics before the final print.
- Why: test fits catch wrong USB/OLED alignment before wasting a long print.

> **Result:** printed plate + case that fit the real parts.

---

## Phase 6 - Soldering and hand-wiring
- [ ] Clip switches into the plate. Add stabilizer first.
- [ ] Attach hot-swap boards (Amoeba) to each switch, or sockets to PCB.
- [ ] Wire rows: connect diode ends along each row. Watch diode direction (stripe = cathode, match your firmware COL2ROW / ROW2COL).
- [ ] Wire columns: connect switch pins down each column. Keep row and column wires insulated from each other.
- [ ] Run rows/columns to the ESP32 pins from your pin map.
- [ ] Test with a multimeter for shorts and continuity **before** powering on.
- [ ] Test each key in firmware one by one. Fix cold joints.
- [ ] Mount mic, OLED, button. Keep mic wires short.
- Why: careful continuity testing saves hours of "why does this key not work".

> **Result:** fully wired keyboard in the case.

---

## Phase 7 - Full firmware
- [ ] Full 12x4 keymap with a function layer (40% needs layers for numbers/symbols).
- [ ] Debounce tuned for your switches.
- [ ] Voice mode:
  - Button press = start listening (or hold-to-talk).
  - Command → shortcut lookup table (easy to add commands).
  - Confirmation on OLED. Timeout back to normal mode.
- [ ] WPM (on-device processing):
  - Count characters typed. WPM = (chars / 5) per minute over a rolling window.
  - Show on OLED in normal mode.
- [ ] Optional: system-usage screen via a small laptop script over USB serial (clearly marked as host-fed, not on-device processing).
- [ ] Comment the code as you go.

> **Result:** final firmware with all core features.

---

## Phase 8 - Testing and tuning
- [ ] Speech accuracy test: say each command 20 times, record hit rate. Quiet room vs noisy room.
- [ ] False-trigger test: talk normally in voice mode, count wrong commands.
- [ ] Typing test: every key, every layer, fast typing, multiple keys at once.
- [ ] Latency: time from end of command to shortcut firing.
- [ ] Fix the worst issue, re-test, log results.
- Why: real numbers make the documentation and demo far stronger.

---

## Phase 9 - Demo and documentation
- [ ] Record the 2-minute demo video **before** the live demo (backup if live demo fails).
- [ ] 5 slides: title + photo, motivation, implementation, challenges, live demo.
- [ ] Documentation (5-10 pages, PDF):
  - Hardware components, libraries used, what you designed yourself.
  - Skills learned beyond skill demos (ESP-IDF, TinyUSB, ESP-SR/ML, CAD).
  - Iteration: what failed, what changed (from your build log).
  - Test results (accuracy numbers).
  - Photos.
  - Value added beyond tutorials/libraries.
- [ ] Zip: code, schematic/wiring diagram, CAD files.

---

## Biggest risks
| Risk | Where it's handled |
|---|---|
| Speech recognition accuracy | Phase 4.3, as early as possible |
| Wrong pins (PSRAM/USB/strapping) | Phase 1.5 |
| 3D print tolerances | Test prints before full prints (Phase 5) |
| Hand-wiring mistakes | Continuity test before power (Phase 6) |
| Scope creep | Lock core features in the proposal. Everything else is stretch |
