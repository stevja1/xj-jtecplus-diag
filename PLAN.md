# XJ Power-Loss Diagnostic App — Plan

> Snapshot of the plan doc exported Nov 12, 2026. The live doc is the source of truth: https://claude.ai/artifact/C9QMu6RAxNQKwBsVX4XWZY

## Problem & use case

Power fades gradually above about 2500 rpm on a freshly built stroker (stroker crank, Clegg stage 2 cam, .040 over, Hesco head) running the factory JTEC+ calibration. No codes, no surging, popping or smell. The app should log live data plus air-fuel ratio and fuel pressure, and narrow the cause before parts are bought.

**Suspects, in the order to rule out:**

| Suspect | Why it fits a gradual fade | Ruled in or out by |
| --- | --- | --- |
| Fuel mismatch: stroker on stock fuel tables | The ECU's wide-open-throttle fueling assumes a 4.0. Stroker builders on JTEC+ pair bigger injectors with modified fuel tables ([jeepstrokers](https://jeepstrokers.com/forum/viewtopic.php?p=55580)); one 4.6 on stock 19 lb injectors bogged around 3000 rpm ([jeepstrokers](https://www.jeepstrokers.com/forum/viewtopic.php?p=54406)) | Wideband air-fuel ratio during a pull |
| Restricted exhaust: two original pre-cats + main cat at 158k | A bigger-breathing engine pushes more flow through an old cat | MAP rising at a steady 2500 rpm |
| Fuel delivery: pump, filter/regulator | Stock pump feeding a larger engine; pressure can sag under demand | Fuel pressure logged during a pull |
| Cam timing: install or degree error | Installed dot-to-dot, not degreed; a few degrees off moves the powerband | Idle vacuum, cam degree check |
| Ignition timing: stock spark tables | Factory advance may not suit the new combo; knock retard possible | Timing advance during a pull |
| Injectors over-fueling | 240 cc injectors on factory tables could land rich or lean; the wideband decides | Wideband shows rich |

**Primary use:** one person drives (or a helper rides along) while the phone records. Afterwards the log is reviewed on the Mac.

## Vehicle & ECU

A 2001 XJ 4.0 runs speed-density fueling (MAP sensor, no MAF) on a JTEC+ computer that talks SAE J1850 VPW on the OBD2 port.

| Item | Detail |
| --- | --- |
| Vehicle | 2001 Jeep Cherokee XJ, 4.0L inline-6, OHV |
| ECU | Chrysler JTEC+ |
| OBD2 protocol | SAE J1850 VPW (slow: about 10.4 kbit/s) |
| Fueling | Speed-density: MAP + IAT + rpm; narrowband O2 sensors |
| Fuel system | Returnless; regulator sits on the pump module in the tank, not vacuum-referenced ([cherokeeforum](https://www.cherokeeforum.com/f2/cherokee-96-4-0l-fuel-system-pressure-12346/)) |
| Fuel filter | Combined filter/regulator on top of the pump module; reached without pulling the whole assembly ([cherokeeforum](https://www.cherokeeforum.com/f2/fuel-pressure-question-110175/)) |
| Fuel pressure spec | 49.2 psi ±5; should hold above 30 psi for 5 min after shutoff ([cherokeeforum](https://www.cherokeeforum.com/f2/fuel-pressure-question-110175/)) |
| Test port | Schrader valve on the fuel rail ([NAXJA](https://naxja.org/threads/low-fuel-pressure-wont-start-2001-xj.1069909)) |
| Ignition | Distributorless coil rail; timing set by the ECU, not adjustable |
| Injectors | 12-hole EV6, 240 cc/min (see engine build) |
| Cam timing | Fixed, chain-driven; no variable valve timing |

**Engine build and history (owner-reported):**

| Item | Detail |
| --- | --- |
| Mileage | 158,225 |
| Displacement | 4.6L stroker: stroker crank, cylinders bored .040 over |
| Cam | Clegg stage 2, installed at factory (dot-to-dot) timing; not degreed |
| Head | New Hesco cylinder head, new head and manifold gaskets |
| Injectors | [K Suspension remanufactured 12-hole EV6](https://www.ksuspensionfab.com/store/p22/Remanufactured%253A_12_Port_Upgraded_Injectors.html), 240 cc/min (~23 lb/hr; rated pressure not listed) |
| Ignition | New spark plugs |
| Intake | New air filter |
| Exhaust | Original 2-piece manifold with two integrated pre-cats, plus the under-car cat ([go-parts](https://www.go-parts.com/garage/ps-1987-2001-jeep-cherokee-exhaust-manifold)); no audible leak |
| Transmission | AW4 automatic; no felt slip |
| ECU calibration | Factory JTEC+, not reflashed; HP Tuners no longer sells the hardware to flash it |
| Codes | None stored |
| Symptom | Gradual fade above ~2500 rpm; no surging, popping or smell |

**Barometric pressure:** it varies with elevation and weather. Wide-open-throttle MAP can only reach about barometric, so judge MAP against a key-on, engine-off MAP reading, not a sea-level number. Naturally aspirated power also drops with altitude, so some loss against sea-level ratings is normal and isn't the fade.

## Hardware & platforms

The dongle supports the Jeep's protocol, so a web page using Web Bluetooth can run on both the Mac (Chrome) and the iPhone 15 (Bluefy browser).

| Resource | Role | Notes |
| --- | --- | --- |
| [Veepeak OBDCheck BLE](https://manuals.plus/veepeak/obdcheck-ble-bluetooth-obd-ii-scanner-adapter-dongle-manual) (ASIN B073XKQQQW) | OBD2 to Bluetooth bridge, ELM327-style commands | Bluetooth 4.0 LE; supports J1850 VPW ([premium-diagnostics](https://premium-diagnostics.com.au/products/veepeak-obd2-bluetooth-scanner)). Don't pair it from the iOS Bluetooth menu; the app connects ([manuals.plus](https://manuals.plus/m/b40a4e719645e9c8cff1ffbac8a107e69e71fc6e22769ef49618756efead4213)) |
| iPhone 15 | In-car logger and live display | Safari has no Web Bluetooth; use [Bluefy](https://apps.apple.com/app/id1492822055), a free iOS browser that adds it. Page must be served over HTTPS |
| Mac (macOS) | Development, bench testing, log review | Chrome supports Web Bluetooth; Safari doesn't |
| Fuel pressure gauge or sensor | Fills the one data gap OBD2 can't | See Sensor add-ons |

**Architecture:** dongle (BLE serial service) → web page (ELM327 commands, PID polling, charts) → CSV log export. The dongle's BLE service and characteristic IDs get discovered at connect time; ELM327-style BLE adapters commonly use a FFF0 service with FFF1 notify / FFF2 write ([OBDLink notes](https://support.obdlink.com/support/solutions/articles/43000746707-obdlink-cx-adapter-notes)).

**Logging plan (driver alone):** either device can log. The Mac in Chrome on the passenger seat is the most reliable logger. The iPhone in Bluefy works too, but likely stops getting Bluetooth data if the screen locks or the app goes to the background (approximate; test it), so the app keeps the screen awake while recording.

## Available data

Generic OBD2 on JTEC+ gives about a dozen useful readings, but no fuel pressure and no MAF. The slow J1850 bus limits total polling to roughly 5–10 readings per second (approximate; measure on first connect).

| PID | Reading | Use in this diagnosis | Priority |
| --- | --- | --- | --- |
| 0C | Engine rpm | X-axis for every test | Core |
| 0B | MAP (kPa) | Load; exhaust restriction shows as MAP climbing at steady rpm | Core |
| 11 | Throttle position (%) | Detects wide-open throttle pulls | Core |
| 14 | Upstream O2 voltages, front and rear manifold halves (PID 13 shows which PIDs map to which) | Lean vs rich at WOT, when trims are frozen; a gap between halves points to one pre-cat or one group of injectors | Core |
| 06 / 07 | Short / long-term fuel trim (%) | Lean or rich correction in closed loop | Core |
| 03 | Fuel system status | Shows open vs closed loop | Core |
| 0E | Timing advance (°) | Sudden drops point to knock retard | Core |
| 0D | Vehicle speed | Context for road tests | Secondary |
| 04 | Calculated load (%) | Cross-check against MAP | Secondary |
| 05 / 0F | Coolant / intake air temp | Confirms warm engine; heat-soak context | Secondary |
| 15 | Downstream O2 voltage | Cat health (should be steadier than upstream) | Secondary |
| Mode 03 / 07 | Stored and pending trouble codes | Read before every session | Core |

**Gaps:** fuel pressure, misfire counts per cylinder, and knock sensor activity aren't available on generic OBD2. Supported PIDs get confirmed from the PID 00/20 bitmaps at first connect.

## Sensor add-ons: fuel pressure and wideband

Build one small sensor box that reads a fuel pressure transducer and a wideband O2 controller, and sends both over Bluetooth to the same app. On a stroker running stock fuel tables, the wideband air-fuel ratio is the single most useful reading; fuel pressure tells you why it's lean or rich.

**Sensor box parts:**

| Part | What to buy | Approx. cost | Notes |
| --- | --- | --- | --- |
| Fuel pressure transducer | 0–100 psi, 5 V supply, 0.5–4.5 V output, 1/8" NPT, stainless | $20–40 | Teed at the rail's Schrader port with an adapter that holds the valve core open; keep it away from exhaust heat |
| Wideband O2 controller + Bosch LSU 4.9 sensor | e.g. [AEM X-Series inline](https://www.jegs.com/i/aem-electronics-017-30-0310-x-series-inline-wideband-uego-afr-o2-controller) or [14Point7 Spartan 3 Lite](https://ampenginemanagement.com/products/spartan-3-lite-v2-wideband) | ~$200–220 for the AEM | Both have a 0–5 V analog output to read. Needs a weld-in exhaust bung |
| Microcontroller | ESP32 dev board (Bluetooth LE built in) | $10 | Acts as a second BLE device the app connects to |
| ADC | ADS1115 16-bit board | $5–10 | The ESP32's own ADC is too nonlinear for this |
| Power | 12 V to 5 V buck converter on a switched fuse tap | $5–10 | Also powers the pressure transducer |

The [PE Wideband](https://pe-ltd.com/product/wideband/) has its own Bluetooth and iPhone app, but it logs separately from the OBD data; its 0–5 V output could still feed the sensor box. Costs other than the AEM are rough estimates, not looked up.

**Wideband placement:** in the downpipe after the two pre-cats merge, before the under-car cat, so it reads all six cylinders. Mount it on the top half of the pipe, tilted at least ~10° above horizontal so condensation drains off the sensor (approximate; follow the controller's manual). Decision: weld-in bung.

**Quick check before the box exists:** a $20–40 mechanical gauge on the Schrader port, taped outside the windshield and filmed during a pull. Never route the fuel hose into the cabin.

**Pass criteria:** 49.2 psi ±5 at idle and held through a loaded pull to ~4000 rpm; above 30 psi five minutes after shutoff ([cherokeeforum](https://www.cherokeeforum.com/f2/fuel-pressure-question-110175/)). Target AFR at wide-open throttle is roughly 12.5–13.0:1 (approximate, typical for naturally aspirated gasoline).

## Requirements

The MVP is a single HTTPS web page that connects to the dongle, logs the core PIDs, and runs guided tests. The fuel pressure sensor and automated verdicts come later.

| ID | Requirement | Phase |
| --- | --- | --- |
| F1 | Connect to the Veepeak over Web Bluetooth; auto-discover its serial service | MVP |
| F2 | Initialize the ELM327 (reset, echo off, headers off, protocol auto or VPW) and report the detected protocol | MVP |
| F3 | Read supported-PID bitmaps; poll only supported core PIDs, fastest first | MVP |
| F4 | Read and clear trouble codes (stored and pending) | MVP |
| F5 | Live gauges: rpm, MAP, TPS, O2 voltage, trims, timing, loop status | MVP |
| F6 | Record a session with timestamps; export CSV | MVP |
| F7 | Guided tests (see Diagnostic test plan) with on-screen steps and markers in the log | MVP |
| F8 | Charts of a recorded pull: O2, MAP, timing vs rpm | MVP |
| F9 | Read the sensor box (fuel pressure + wideband AFR) as a second BLE device into the same log | Later |
| F10 | Automatic pass/fail per test against the thresholds below | Later |
| N1 | Runs in Bluefy on iPhone 15 and Chrome on macOS; code in git, deployed as a static HTTPS site on DigitalOcean | MVP |
| N2 | Large, glanceable display usable in a moving vehicle; driver never needs to touch it during a pull | MVP |
| N3 | Logs survive a dropped connection (auto-reconnect, keep data) | MVP |
| N4 | Read-only to the ECU: no writes, resets, or tuning | MVP |

## Diagnostic test plan

Run six tests in order; the first failure usually names the cause. Thresholds marked ~ are typical values from experience, not from a cited spec.

1. **Code scan.** Read stored and pending codes. Any lean, misfire, O2 or cat code steers the rest.
2. **Warm idle baseline (engine at operating temp).** Expect MAP ~30–38 kPa (a big cam raises it), trims within ±10%, closed loop.
3. **Steady 2500 rpm in park, 60 s.** MAP should hold or fall slightly. MAP that keeps climbing points to a restricted exhaust (cat).
4. **Closed-loop cruise, 2000–2500 rpm.** Combined STFT + LTFT above ~+15% means lean: fuel delivery, a vacuum leak, or a cracked exhaust manifold fooling the O2 sensor.
5. **Wide-open throttle pull, 2nd gear, ~1500→4500 rpm (safe, legal road).** MAP should plateau near the key-on barometric reading. The AW4 kicks down to 1st shortly after tip-in and shifts back to 2nd (owner-reported; a 2nd-gear pull can't be forced), so expect a rpm and O2 disturbance a second or two in; ignore those rows. Upstream O2 should rise and hold above ~0.8 V; with the wideband, AFR should sit near 12.5–13.0:1. Sustained below ~0.6 V above 2500 = lean, likely fuel pressure. A sudden drop in timing advance = knock retard (ignition, fuel or heat).
6. **Fuel pressure under load.** Repeat test 5 with a gauge or sensor; pass if it holds 49 ±5 psi.

**Reading the results:**

| Pattern | Likely cause | Next step |
| --- | --- | --- |
| Test 3 fails, O2 fine | Clogged pre-cats or main cat | Inspect or replace the cats |
| Tests 4–5 lean, test 6 sags under load | Fuel pump (or filter/regulator) | Replace the pump: Walbro GSS342 or AEM 50-1220 (see Fuel pump and regulator upgrades) |
| Tests 4–5 lean, test 6 holds 49 psi | Not enough fuel for the stroker at stock pressure and tables, a vacuum leak, or an exhaust leak before the O2 | Rule out leaks, then add the wideband and consider the regulator options (Hesco adjustable, or G-Force delete + external regulator); change pressure only with the wideband installed |
| O2 rich at WOT, power still low | Ignition breakdown or exhaust restriction | Recheck test 3; inspect coil rail, plugs and wires |
| Everything passes | Mechanical | Compression test, degree the cam, check for transmission slip |

## Open questions & decisions

- [ ] Sensor box: build it, fuel pressure first, wideband added later (decided).
- [ ] Wideband bung: weld-in (decided).
- [ ] If fuel and exhaust check out: degree the cam (timing cover off)?
- [ ] Any other way to retune the JTEC+ now that HP Tuners hardware is unavailable? (not yet researched)

## Research questions

### HP Tuners options

HP Tuners supports the 2001 Cherokee 4.0 for 1 credit, but only on MPVI2, MPVI2+ and MPVI3; the current MPVI4 doesn't support JTEC+ yet ([HP Tuners](https://www.hptuners.com/vehicles/stellantis-tuning/), [MPVI3 JTEC note](https://www.hptuners.com/articles/jtec-is-now-supported-by-mpvi3/)).

- [ ] Can a used MPVI3 or MPVI2+ be bought and licensed for this Jeep? How do credits transfer?
- [ ] Is MPVI4 JTEC support on HP Tuners' roadmap with a date?
- [ ] Which independent tuners still flash JTEC+ by mail or bench flash ([JeepStrokers thread](https://jeepstrokers.com/forum/viewtopic.php?p=36385))?
- [ ] Without a reflash, does an adjustable fuel pressure regulator fit the returnless 2001 system? Yes, with a return line: see Fuel pump and regulator upgrades.

### Fuel pressure gauges

Decision: no PLX. The logged Bluetooth fuel pressure reading comes from the DIY sensor box (phase 1), so nothing has to be read while driving and it lines up with the OBD log. A mechanical gauge covers the driveway checks (idle pressure, hold after shutoff).

- [ ] Which pressure transducer and Schrader adapter fit the XJ rail's test port?
- [ ] Mechanical gauge for driveway checks: borrow or buy?

### Sensor box

A DIY ESP32 + ADS1115 box reads a 0–100 psi transducer and the wideband's 0–5 V output and logs both in our app; roughly a weekend build with ~10 solder joints. It also works on other cars: only the fuel fitting and exhaust bung change.

**Phasing:** phase 1 is fuel pressure only. The ADS1115 has four inputs, so the box is built with a spare connector for the wideband and the firmware already reads that channel; phase 2 plugs the wideband in with no rebuild.

- [x] Build the box (owner solders; no PLX)
- [ ] Check spare Arduino hardware: it needs Bluetooth LE (ESP32 or an Arduino Nano ESP32 / Nano 33 BLE; an Uno has no Bluetooth)
- [ ] Exact parts list and wiring diagram
- [ ] Enclosure and mounting location under the hood

### Wideband

Keep the wideband through diagnosis and tuning; afterwards keep it as a gauge or plug the bung. If left installed, it must be powered whenever the engine runs (approximate; confirm in the controller manual).

- [ ] Controller: AEM X-Series or 14Point7 Spartan 3 Lite?
- [x] Bung: weld-in, in the downpipe after the pre-cats merge (decided)
- [ ] Plug thread for removal (commonly M18×1.5; confirm with the bung)

### Fuel pump and regulator upgrades

XJ and TJ stroker owners upgrade the in-tank pump, the regulator, or both; all options below fit the returnless module used in 1997–2001 XJs and 1997–2004 TJs. The same filter/regulator (Omix 17718.05) fits both ([Quadratec](https://www.quadratec.com/products/917718_05.htm)). The 2000–01-only parts (two-piece manifold with pre-cats, coil rail) don't change the fuel system. Grand Cherokee parts are excluded.

Measure before buying: forum math puts the stock 90 lph pump at about 300 crank hp ([JeepStrokers](https://www.jeepstrokers.com/forum/viewtopic.php?p=40975)), so a worn 158k-mile pump is more likely than an undersized one. Experienced XJ stroker owners also warn against changing pressure without a wideband ([JeepStrokers](https://www.jeepstrokers.com/forum/viewtopic.php?p=46674)).

| Option | What it does | Used on | Source |
| --- | --- | --- | --- |
| Walbro GSS342 255 lph + 400-869 kit | Drop-in pump for the stock module; keeps the 49 psi regulator | Recommended by a 2000 XJ 4.6 stroker owner | [JeepStrokers](https://www.jeepstrokers.com/forum/viewtopic.php?p=43610) |
| AEM 50-1220 320 lph | Drop-in pump for 97+ XJ module | 2000 XJ stroker; fixed fuel delivery after the build | [NAXJA](https://naxja.org/threads/aem-50-1220-320lph-e85-compatible-fuel-pump.1150756) |
| Hesco adjustable regulator | Stock regulator top with a Hesco base that collects bypass fuel into a return line to the tank | Late (97+) XJ module; price around $400 per one forum post | [NAXJA](https://naxja.org/threads/fuel-pumps-for-an-xj-buggy.78329), [JeepStrokers](https://jeepstrokers.com/forum/viewtopic.php?p=17762) |
| G-Force GF-JFRD-V2 regulator delete + external regulator | Removes the module's regulator so a 58 psi (C5 Corvette) or adjustable regulator can mount outside the tank | Listed for 97–04 TJ and 97–01 XJ; not early TJs with the Dakota-style module | [Crossmembers](https://www.crossmembers.com/products/jeep-wrangler-tj-fuel-pressure-regulator-delete-gf-jfrd-v2) |
| Return-style rail + adjustable regulator under the hood | Converts to a return system; pressure set at the engine | A 2000 XJ owner and a 2001 XJ owner | [JeepStrokers](https://jeepstrokers.com/forum/viewtopic.php?p=15075), [JeepStrokers](https://www.jeepstrokers.com/forum/viewtopic.php?p=49883) |

Raising pressure from 49 to 58 psi adds ~9% injector flow (flow scales with √pressure). Crossmembers' listing states stock pressure is 40 psi, which conflicts with the 49 psi XJ spec; trust the gauge.

- [ ] Logged pressure test first: does pressure sag under load? (pump) Or hold at 49 psi while AFR goes lean? (injectors/pressure/tune)
- [ ] If the pump is weak: Walbro GSS342 or AEM 50-1220?
- [ ] If more pressure is wanted: Hesco adjustable vs G-Force delete + external regulator (needs return plumbing to the tank)
- [ ] Any pressure change waits for the wideband

## Sources

- [Cherokee Forum: 4.0 fuel system pressure (FSM excerpt)](https://www.cherokeeforum.com/f2/cherokee-96-4-0l-fuel-system-pressure-12346/)
- [Cherokee Forum: fuel pressure test, 95.5–2001 XJ](https://www.cherokeeforum.com/f2/fuel-pressure-question-110175/)
- [NAXJA: low fuel pressure, 2001 XJ](https://naxja.org/threads/low-fuel-pressure-wont-start-2001-xj.1069909)
- [Veepeak OBDCheck BLE user manual](https://manuals.plus/veepeak/obdcheck-ble-bluetooth-obd-ii-scanner-adapter-dongle-manual)
- [Veepeak OBDCheck BLE specs](https://premium-diagnostics.com.au/products/veepeak-obd2-bluetooth-scanner)
- [Bluefy – Web BLE Browser (App Store)](https://apps.apple.com/app/id1492822055)
- [OBDLink CX BLE adapter notes](https://support.obdlink.com/support/solutions/articles/43000746707-obdlink-cx-adapter-notes)
- [JeepStrokers: JTEC+ injector and fuel table notes](https://jeepstrokers.com/forum/viewtopic.php?p=55580)
- [JeepStrokers: 4.6 stroker on 19 lb injectors bogging near 3k](https://www.jeepstrokers.com/forum/viewtopic.php?p=54406)
- [AEM X-Series inline wideband (JEGS)](https://www.jegs.com/i/aem-electronics-017-30-0310-x-series-inline-wideband-uego-afr-o2-controller)
- [14Point7 Spartan 3 Lite V2](https://ampenginemanagement.com/products/spartan-3-lite-v2-wideband)
- [PE Wideband O2 controller](https://pe-ltd.com/product/wideband/)
- [K Suspension 12-port remanufactured injectors](https://www.ksuspensionfab.com/store/p22/Remanufactured%253A_12_Port_Upgraded_Injectors.html)
- [go-parts: 1987–2001 Cherokee exhaust manifold (2000–01 pre-cats)](https://www.go-parts.com/garage/ps-1987-2001-jeep-cherokee-exhaust-manifold)
- [JeepStrokers: injector sizing rule of thumb](https://jeepstrokers.com/forum/viewtopic.php?p=53554)
