# HR Recorder

Single-page web app that records heart rate from a Huawei watch (or any standard
Bluetooth heart-rate sensor) on an iPhone and exports a timestamped CSV.

```
Watch (HR Data Broadcast, standard BLE Heart Rate Service)
   └─BLE─> iPhone: Bluefy browser ─> index.html ─> IndexedDB (each sample saved on arrival)
                                                    └─> CSV ─> Share sheet (Mail / Google Drive)
```

No watch app, no Mac, no App Store build.

## 1. Check the watch

On the GT 6 Pro: open the app list, go to **Settings > HR Data Broadcasts** and turn it on
(Huawei support: "Heart rate broadcasts on your HUAWEI watch/band"). If that option is
missing, the watch model does not support it and this approach will not work.

Notes from Huawei's docs:
- Some watches only allow one connection and drop the Huawei Health app while broadcasting. That's fine.
- Starting a workout on the watch keeps the broadcast running.

## 2. Host the page (needs HTTPS)

Web Bluetooth only works on HTTPS pages. The free and simple way is GitHub Pages:

1. Create a GitHub repository, e.g. `hr-recorder`, and upload `index.html`.
2. Repository **Settings > Pages > Deploy from branch > main / root**.
3. The page will be at `https://<your-user>.github.io/hr-recorder/`.

## 3. Use it on the iPhone

Safari and Chrome on iOS do **not** support Web Bluetooth. Install the free
**Bluefy – Web BLE Browser** app from the App Store and open the page URL in it.

1. Enter the participant code.
2. Turn on HR Data Broadcasts on the watch, then tap **Connect watch** and pick the watch.
3. Tap **Start**, and **Stop** at the end.
4. Under *Recordings*, tap **Share CSV** and choose Mail or Google Drive.
   If the share sheet does not appear, use **Download** (saves to Files) or **Copy** (paste into an email).

Keep the screen on and Bluefy in the foreground while recording: iOS pauses Bluetooth for
background web pages. If the connection drops, the app reconnects automatically. Every sample
is stored on the phone as it arrives, so a dropped connection or closed tab does not lose what
was already recorded (the session shows as "interrupted (data kept)").

Try the interface without a watch by adding `?demo` to the URL (simulated heart rate).

## Live file for MATLAB (experiment PC, Chrome or Edge)

When HR Recorder runs in **Chrome or Edge on the experiment PC** (Web Bluetooth works there too), the
button **Live file for MATLAB** (under Start / Stop) writes every heart-rate value into a CSV file about
once per second. The MATLAB program `run_session.m` reads that file every second, shows the heart rate
live in its monitor and keeps a copy with the session (`raw\hr_live_<ID>_<arm>.csv`).

1. Connect the watch as usual.
2. Click **Live file for MATLAB** and save the file as `Desktop\Study_Data\hr_live.csv`
   (choosing the same file again starts it fresh). The status line shows `Live file: ON · N values written`.
3. Press **Start** / **Stop** as usual. The Recordings list and the CSV export work exactly as before;
   the export stays the main heart-rate file, the live file is extra.

The live file has the same columns as the export. Values are written from the moment the live file is
switched on (also before Start; `elapsed_s` is empty until Start). The button is hidden in browsers that
cannot write files (iPhone / Bluefy). The browser asks for the file again after the page is reloaded.
The code is in its own `<script>` block (also available as `hr-live-file.js`); the only other changes are
one `HRLive.push(...)` line in `onMeasurement()` and the button wiring at the end of the main script.

## CSV columns

| column | meaning |
|---|---|
| `timestamp_iso` | UTC time the sample reached the phone (ISO 8601, ms) |
| `unix_ms` | same, as Unix epoch milliseconds |
| `elapsed_s` | seconds since Start was pressed |
| `participant` | participant code |
| `bpm` | heart rate reported by the watch |
| `rr_ms` | RR intervals in ms, space-separated, if the watch sends them (often empty) |
| `sensor_contact` | 1/0 if the watch reports skin contact, else empty |

Sample rate is whatever the watch sends, typically about once per second.
