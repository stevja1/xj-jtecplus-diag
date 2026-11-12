# XJ Diag

A single static web page that reads live engine data from a Veepeak OBDCheck BLE dongle over Bluetooth and logs it for diagnosing power loss on a 2001 Jeep Cherokee (XJ) 4.6 stroker with a JTEC+ computer. It works with any 1996+ OBD2 vehicle.

No build step, no server code, no dependencies: everything is in `index.html`.

## What it does

- Connects to the dongle with Web Bluetooth and talks to it with ELM327 commands (auto protocol; the XJ uses SAE J1850 VPW).
- Detects which readings the engine computer supports and polls the important ones fastest: rpm, MAP, throttle, upstream O2 sensors, timing advance. Fuel trims, fuel loop status, speed, load, temperatures, downstream O2 and battery voltage are polled in rotation.
- Live gauges and a 60-second chart.
- Recording with markers, CSV export (download, or the iOS share sheet), copy to clipboard, and loading a CSV back in for review.
- Review chart of any channel vs rpm, optionally only at full throttle, with the 0.6 V lean line for O2 sensors.
- Read stored and pending trouble codes; clear codes (with confirmation).
- Keeps the screen awake while recording, autosaves the recording in the browser every 10 s, and reconnects automatically if the Bluetooth link drops.
- Simulator mode: a fake dongle and engine (idle, cruise, a full-throttle pull that goes lean above 2500 rpm) for trying the app without the Jeep.

It only reads data. Apart from clearing codes, it never writes to the engine computer.

## Which browser

| Device | Browser |
| --- | --- |
| Mac | Chrome (Safari has no Web Bluetooth) |
| iPhone | Bluefy (free on the App Store); Safari and Chrome on iOS won't work |

Web Bluetooth only works on `https://` pages or `http://localhost`. Don't pair the dongle in the iPhone's Bluetooth settings; the page connects to it directly.

## Run it on the Mac

```sh
cd xj-diag
python3 -m http.server 8000
```

Open http://localhost:8000 in Chrome. Click **Simulator** to try it without the Jeep, or **Connect dongle** with the dongle plugged in and the ignition on.

## Deploy to DigitalOcean (static site)

1. Push this folder to a GitHub repo.
2. In DigitalOcean App Platform, create an app from the repo and choose **Static Site**.
3. Leave the build command empty and set the output directory to `/`.
4. Open the app's `https://...ondigitalocean.app` URL in Bluefy on the iPhone and in Chrome on the Mac.

Any static host that serves HTTPS works the same way.

## Using it in the Jeep

1. Plug in the dongle, turn the ignition on, open the page, tap **Connect dongle** and choose VEEPEAK.
2. Wait for the status to show "Connected · SAE J1850 VPW".
3. Tap **Start recording** before you drive. Set up the phone or laptop so you never need to touch it while driving.
4. After a run, tap **Stop recording**, review the pull, and **Export CSV**.

Read rate on J1850 VPW is roughly 8–15 readings per second in total, so each recorded row is about half a second apart.

## Troubleshooting

- **Dongle doesn't appear:** open "Adapter details and raw log", tick "Show all Bluetooth devices", and connect again.
- **"No answer from the engine computer":** the ignition must be on (engine running is best).
- **Something looks wrong:** open "Adapter details and raw log", tap **Copy log**, and include it when asking for help.

## CSV columns

`t_s, rpm, map_kpa, tps_pct, timing_deg, stft_b1_pct, ltft_b1_pct, stft_b2_pct, ltft_b2_pct, fuel_status, speed_mph, load_pct, coolant_f, intake_air_f, battery_v, o2_b1s1_v ... , mark`

Only columns the vehicle supports are included. `fuel_status` is the raw OBD code: 1 open loop (warm-up), 2 closed loop, 4 open loop (load/decel), 8 open loop (fault), 16 closed loop (fault).

## Roadmap

- Guided tests (code scan, idle baseline, steady 2500 rpm, cruise trims, full-throttle pull) with on-screen steps and automatic markers.
- Phase 1 sensor box (ESP32 + ADS1115, fuel pressure) as a second Bluetooth device in the same log; wideband O2 in phase 2.
