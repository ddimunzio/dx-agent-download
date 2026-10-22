# DX Agent — User Guide

*Created by Diego – LW5HR, with Franco – LU2PWY, Juan Pablo – LU9HVR, Gustavo – LU7HW, Daniel – LU1HLH, Javi – LU5FF. MIT License. Support DX Agent: [Ko-fi](https://ko-fi.com/lw5hr) · [PayPal](https://paypal.me/diegodimunzio1) · from Argentina, [Cafecito](https://cafecito.app/lw5hr).
Versión en español: [DOCUMENTACION.md](DOCUMENTACION.md).*

DX Agent watches the DX cluster and the Reverse Beacon Network (RBN), checks every
spotted station against your ADIF log, and alerts you when one you still need is on
the air. It runs on your PC with a dashboard in your browser, and needs only Python
3.11 or newer — nothing else to install.

It answers three questions in real time:

- **Is this station missing from my log?** A new country (ATNO), a new band, a new
  mode, or a call on your watchlist.
- **Is this an active DXpedition, and do I actually need it?** It tracks the NG3K,
  425 DX News and Club Log DXpedition lists and tells you whether working it adds something new, gets you
  a missing confirmation, or adds nothing.
- **Will I get a confirmation?** It shows whether the station uploads to LoTW, and
  can download your own LoTW confirmations.

Contents:

1. [Quick start](#1-quick-start)
2. [Web dashboard](#2-web-dashboard)
3. [Settings page](#3-settings-page)
4. [Commands](#4-commands)
5. [Reading the alerts](#5-reading-the-alerts)
6. [The configuration file](#6-the-configuration-file)
7. [LoTW integration](#7-lotw-integration)
8. [How it works](#8-how-it-works)
9. [Project files and data cache](#9-project-files-and-data-cache)
10. [Known limitations and troubleshooting](#10-known-limitations-and-troubleshooting)

---

## 1. Quick start

**Easiest:** unzip `dx-agent-<version>-windows.zip` anywhere and double-click
**start-dx-agent.bat**. That zip already includes Python, so there is nothing to
install. DX Agent runs without a window: the dashboard opens in your browser,
**Stop** (top right of the dashboard) closes it, and double-clicking again while it
runs just opens the dashboard. Starting a new version while an older one is still running
stops the old one first (as its Stop button would) and starts the new one. If it can't start, a Windows message says why; what it
prints goes to `data/dx-agent.log` in your DX Agent folder (see below), and **start-dx-agent-console.bat** runs it in a
window to watch it. (The source zip, `dx-agent-<version>-source.zip`, needs
[Python 3.11+](https://www.python.org/downloads/) installed with "Add python.exe to
PATH"; without it, start-dx-agent.bat says so and offers to install it with winget.)

**On Linux:** unzip `dx-agent-<version>-linux-x86_64.tar.gz` (it includes Python too) and run
its own launcher. It needs glibc 2.17 or newer, that is, any ordinary 64-bit distribution:

```sh
tar -xzf dx-agent-<version>-linux-x86_64.tar.gz
cd dx-agent
./start-dx-agent.sh
```

Everything works there except the Windows programs; see
[Known limitations](#known-limitations). With your own Python 3.11+ instead of the package,
use the source zip and `python3 -m dxagent run`.

**On a Mac:** unpack `dx-agent-<version>-macos.tar.gz` (double-click it in Finder; it includes
Python for Apple Silicon and for Intel Macs) and double-click **start-dx-agent.command** in the
`dx-agent` folder. Terminal opens for a moment and the dashboard opens in your browser. The
package isn't signed by Apple, so the first time macOS refuses to open it: open **System
Settings > Privacy & Security**, click **Open Anyway** next to the message about
start-dx-agent.command, and confirm (on macOS 14 and older, right-click the file > **Open**
works too). From then on a double-click is enough. Or, in Terminal:

```sh
cd ~/Downloads/dx-agent
xattr -dr com.apple.quarantine .
./start-dx-agent.command
```

On a Mac, too, everything works except the Windows programs ([Known limitations](#known-limitations)).

**Your settings and data** are kept outside the program's folder, in a folder of your own that
every version uses. They are `config.toml` and the `data` folder, which holds your alert
history, the saved LoTW password, and the log if you started from scratch:

| System | Folder |
|---|---|
| Windows | `%APPDATA%\DX Agent` |
| Mac | `~/Library/Application Support/DX Agent` |
| Linux | `~/.config/dx-agent` (or `$XDG_CONFIG_HOME/dx-agent`) |

To update, unzip the new version anywhere and start it; the old folder can be deleted.
**Settings** shows the exact path of `config.toml`. Up to 0.4.x they were kept in the
program's folder: the first time a newer version starts, it copies the settings and data from
there, or from the older version next to it (for example `dx-agent-0.4.3-windows\dx-agent`), and
says so on the dashboard. The older version is left as it was. When there is none, the first
start creates `config.toml` from `config.example.toml`; then set your callsign and log in
**Settings**. To use another `config.toml`: `python -m dxagent -c path/config.toml run` (its data
goes in `data`, next to it), or the `DXAGENT_HOME` environment variable with another folder.

From a terminal instead:

1. Open a terminal. In VS Code: **Terminal → New Terminal**.
2. Go to the project folder:

   ```powershell
   cd C:\path\to\dx-agent
   ```

3. Start it:

   ```powershell
   python -m dxagent run
   ```

   Your browser opens the dashboard at **http://localhost:8073**.
4. The first time, click **Settings** (top right) and check your callsign, the path
   to your ADIF log and, if you want, your LoTW password. **Save and apply**.
5. Press **Ctrl+C** in the terminal to stop (it prints `73!`).

It uses the `config.toml` in your DX Agent folder (above), wherever you run it from. To use
another one, pass its path (its data goes in `data` beside it):

```powershell
python -m dxagent -c C:\path\to\config.toml run
```

On Linux the commands are the same with `python3` and `/path/to/dx-agent`.

A normal start looks like this in the terminal, followed by one line per needed
station:

```text
[log] 23817 QSOs, 271 DXCC entities (248 confirmed) from N0CALL_20261013_175723_backup.adi (1.6s)
[lotw] no LoTW password set - using confirmations from the ADIF log only
[dxped] NG3K 69, 425 DX News 71, Club Log 3 -> 106 DXpeditions
[dxped] 36 active now
[agent] alerting on: atno, dxpedition, new_band, new_mode, watchlist | bands: 160m ... 6m | modes: CW PHONE DIGI
[ui] dashboard at http://localhost:8073
[rbn-cw] connected to telnet.reversebeacon.net:7000
[rbn-ft8] connected to telnet.reversebeacon.net:7001
[cluster] connected to w3lpl.net:7373
[cluster] logged in as N0CALL
```

Every 10 minutes it prints a status line with the number of spots processed and
alerts raised.

**Pop-ups:** alerts also appear as Windows notifications (Windows 10/11), and in a Mac's
Notification Center, with nothing extra to install. On Windows, clicking one opens the
dashboard at that station (its grid, its row in view). On Linux they need `plyer`
(`pip install plyer`); without it the alerts still appear in the dashboard.

---

## 2. Web dashboard

`run` starts the dashboard and opens it in your browser at **http://localhost:8073**.
The console keeps working as before; the dashboard is another view of the same agent
and closes when you stop `run`.

```powershell
python -m dxagent run               # console + dashboard, opens the browser
python -m dxagent run --no-browser  # dashboard running, but don't open a tab
python -m dxagent run --no-ui       # console only
```

### What's on the screen

| Area | What it shows |
|------|---------------|
| **Header** | Your call; one pill per feed (green = online, amber = connecting, red = down) with its spot count (hover for the server and last spot time); a **DXpeditions** pill next to them: how many are within their dates and how many of them were spotted in the last 15 minutes ("48 · 6 on the air", its dot green while one is; a click opens the DXpeditions tab); your log totals; icon buttons (hover for what each does): **About** ⓘ (what DX Agent is, who made it, the collaborators, how to support it, and help: the website, the user manual and the issues page), **Settings** ⚙ (on the Settings page, a list icon back to the alerts), **EN/ES/PT** (language), **Theme** (half circle = Auto, sun = Light, moon = Dark), **Browser sound** (speaker, crossed out when off) and **Stop** (red); UTC clock. `#about` opens the About window |
| **Filters** | Three rows, a line between them: alert type (with **List** / **Map** and **Clear list**); band, mode and the others; continent and a search box for call or country. **Continent** (AF, AN, AS, EU, NA, OC, SA) leaves the stations from the continents picked, to keep to where your antenna points; your own call spotted always shows. Only the alert types turned on in Settings get a chip. No chip selected = show everything. An alert counts for each of its reasons, so a DXpedition on your watchlist shows under both **DXPED NEED** and **WATCHLIST**; **WATCHLIST** also lists your watchlist calls' spots that made no alert (grey **SPOT** rows), so you see when they are on the air. Several types picked show any of them, except **WATCHLIST**: with another type, only your watchlist calls that are also that type (**WATCHLIST** + **NEW SLOT**: the calls you follow on a band and mode you still need; their SPOT rows by what they add). SPOT rows are one per station, band and mode: the latest spot, with **×N** for how many (a CQ is heard by several RBN skimmers at once; hover for who heard it); those before an alert of the same station, band and mode are that alert's. A SPOT row that still adds to your log (it made no alert of its own because the same alert isn't repeated within the cooldown) is coloured like what it adds, not grey. **Hide confirmed** leaves out stations whose entity you already have confirmed on that band and mode (an alert for a new call, a SPOT row): only what adds to your log stays. **Only LoTW** leaves the stations that uploaded to LoTW in the last 12 months and the DXpeditions that announced LoTW: the ones likely to confirm there. Your filters are remembered in the browser. **Clear list** (after confirming) empties the alert list, in every open dashboard and after a reload, and starts the cooldown again: whatever is still on the air alerts again at its next spot. The alert history (`data/dxagent.db`) and the Station tab's latest spots are kept |
| **Alert table** | One row per alert, newest on top, coloured by type: UTC, alert, call, kHz, band, mode, country, what you need, LoTW, **near you** (see below), who heard it. Click a column header to sort by it (kHz, band, mode grouped CW / phone / digital, country...; **Alert** and **What you need** put the most wanted first, **LoTW** the latest uploads first); click again for the other way round, a third time for newest first again. The arrow shows the order; rows with nothing in that column go last, ties stay newest first, new alerts slot into place, and the order is remembered in the browser. New rows flash. A newer spot of the station on the same band and mode (one the cooldown kept from alerting again) updates its row: time, frequency, mode, who heard it (×N) — so the row shows where the station is now, and the Tune button goes there. Rows fade 10 minutes after the station's last spot and fade more after 30 (hover a row for its age and the alert's own time); with **Take out of the list after** set in Settings, they leave the list after that many minutes |
| **Station tab** | Click any alert, or type a call in the box, to see that country's band × mode grid (green C = confirmed, amber W = worked, grey = not worked; a green ring marks the bands and modes where the station was spotted in the last 15 minutes, the latest one blinking, and a dot sits over the band), whether you worked this exact call, LoTW status, DXpedition details, who hears it nearest you, a **Check PSKReporter** button (or, live, the receivers reporting it as they arrive), **Show only** this call, **☆ watchlist** and **Spot it** buttons, its latest spots, and links to QRZ.com, HamQTH and DXWatch |
| **DXpeditions tab** | Active DXpeditions and those starting in the next 30 days. Badge: **ATNO**, **Not confirmed**, or how many bands you have (e.g. "1 of 10 bands", highlighted when fewer than half). Tick **only needed** to hide ones you already have on most bands. Sorted by what's **on the air** (spotted in the last 15 minutes, the latest on top), by Club Log's **Most Wanted** rank (most wanted country first) or by start date: **Order** at the top. Each card shows its sources and Club Log's **Most Wanted** rank; one spotted in the last 15 minutes is framed in green as **ON THE AIR**. Click one to open its grid; the **☆** adds it to your watchlist |

**Which station was it?** Hover over a **C** or **W** cell of the Station tab's grid and a
card lists the QSOs that made it — call, date, mode, and a **✓** when confirmed — newest
first, up to eight (then "+N more"): the fastest way to tell whether the 20 m QSO with São Tomé
was the current DXpedition or another station years ago. The cell keeps showing that and, when
the station is on the air, its latest spot.

**On the air, in the Station tab's grid.** The band × mode grid also shows where the station
is transmitting now:

- **Green ring** on each band and mode where it was spotted (cluster or RBN) in the last 15
  minutes, and a **dot ●** over that band. The ring is set apart from the cell, so it shows
  on a confirmed **C**, a worked **W** or an empty cell alike; on an empty one, it's on the
  air on a band you still need.
- **Blinking:** only the cell of the **latest spot**, where the station is now. The others
  keep a steady ring. When a spot comes in on another band, the blinking moves there.
- **On several bands at once** (a DXpedition with several stations, S9R say): every band
  spotted in the last 15 minutes has its ring, and the latest one blinks.
- **Hover** a ringed cell: when it was spotted, on what frequency and mode, and by whom.
- **Click** a ringed cell (with a radio set up, see Tune the radio): Tune goes to that band
  and mode's latest spot, and the Station tab's Tune button keeps it.
- A spot that doesn't say its mode marks the whole band. With no spot for 15 minutes, the
  ring and the dot go. With Windows' animations off, the latest one has a thicker ring
  instead of blinking.

**Band colours.** Each HF band and 6 m has its own colour, used for the band tag in
the table, the band filter chips, the station grid and the Settings page: 160 amber,
80 violet, 60 red, 40 blue, 30 olive, 20 pink, 17 green, 15 teal, 12 brown, 10 purple,
6 orange (4 m, 2 m and 70 cm share grey). The order was checked with a colour-blindness
simulator so that neighbouring bands stay distinct, and the band name is always shown
in the tag, so colour is never the only cue.

**Mode colours.** The mode tag in the table (and in a station's spots) takes its group's
colour: CW blue, phone (SSB, AM, FM) amber, digital (FT8, FT4, RTTY...) pink; the mode
filter chips too. A mode DX Agent guessed has a dashed border, an unknown one is grey.

**EN/ES/PT** switches the dashboard between English, Spanish and Portuguese (Brazil): each
click goes to the next one. The first time it follows your browser's language; after that
it remembers your choice. A link can force one: `http://localhost:8073/?lang=es` (or
`?lang=pt`). In Portuguese, the About window's website and manual links open the English
ones. Country names and DXpedition notes stay in
English, as they come from their sources; the terminal and the alerts kept in
`data/dxagent.db` are always in English.

**Theme** switches between **Auto** (follows Windows light/dark mode), **Light** and
**Dark**. Your choice is remembered in the browser. A link can force one:
`http://localhost:8073/?theme=light`.

**Browser sound** beeps in the browser tab for ATNO and priority-2 alerts. It is off
by default; the agent itself makes no sound.

**Links you can bookmark:** `http://localhost:8073/#settings` opens Settings,
`#call=VP8PJ` opens that station, `#dxped` opens the DXpeditions tab.

**When something is wrong** — for example the log file can't be found — the agent
keeps the dashboard running and shows a yellow banner with the reason and a link to
Settings, so you can fix it without the terminal.

### Tune the radio

With a radio target set (Settings → **Radio (Tune button)**), clicking an alert's
frequency asks your logger or rig-control program to move the radio there, and opens the
station on the right, as clicking the row does. The
Station tab also gets a **Tune** button: for the row you clicked in the list (highlighted
more strongly than the call's other rows), or else the station's latest spot. A message in
the top-right corner says whether it worked.

| Target | How DX Agent talks to it | Default port | Mode too? |
|--------|--------------------------|--------------|-----------|
| Log4OM 2 | UDP remote control (`SetTxFrequency`, `SetCallsign`); the mode through OmniRig | 2241 | yes, through OmniRig (Log4OM's own `SetMode` changes only its screen) |
| N1MM Logger+ | UDP `radio_setfrequency` | 13064 | no (N1MM has no mode field) |
| DXLog.net | UDP `radio_setfrequency` (as N1MM); in Options → Configure network add the port to the UDP broadcast listener, and turn on Options → Broadcast → Receive broadcasts | 13064 | no (only the frequency) |
| OmniRig | Windows COM, through PowerShell; Rig 1 or Rig 2 | — | yes |
| Hamlib rigctld | TCP `F <Hz>` / `M <mode> -1` | 4532 | yes |
| flrig | XML-RPC `rig.set_frequency` / `rig.set_mode` | 12345 | yes |
| DXLab Commander | TCP `CmdSetFreqMode` | 52002 | yes |
| TCI (AetherSDR, ExpertSDR, Thetis) | WebSocket `vfo` / `modulation`, on a connection DX Agent keeps open; it also puts the alerts on the panadapter | 50001 | yes |

- **Mode chosen:** CW for CW spots; USB or LSB for phone (LSB below 10 MHz, except
  60 m); RTTY for RTTY; for FT8/FT4, USB through OmniRig (how WSJT-X runs the radio)
  or the data mode through Hamlib, flrig and DXLab. Untick **Also set the mode** to
  change only the frequency.
- **A mode that already suits the spot is kept (OmniRig, and Log4OM through it):** for an
  FT8/FT4 spot, a radio in USB or in its data mode only moves frequency; for CW, one in CW or
  CW-R. So a radio that needs its data mode for FT8 (an IC-9100 in USB-D) keeps it from one
  FT8 spot to the next: put it in USB-D once. Phone spots always set USB or LSB, so a data
  mode is never left on with the microphone muted.
- **CW on Icom radios (OmniRig):** OmniRig's Icom files call the radio's CW-R "CW upper" and
  its plain CW "CW lower", so on an Icom DX Agent asks for "CW lower" (tested: IC-9100 in CW,
  not CW-R); on other makes plain CW is "CW upper".
- **Log4OM and the mode:** Log4OM's remote control accepts `SetMode`, but it changes only
  the mode on Log4OM's screen, not the radio's (tested: Log4OM 2.42 with an IC-9100). Log4OM
  drives the radio through OmniRig, so with **Also set the mode** on, DX Agent sets frequency
  and mode through OmniRig itself, on the OmniRig rig that is on-line (the **OmniRig radio**
  setting is tried first), and Log4OM follows the radio as usual; the call still goes to
  Log4OM. Without OmniRig, or with no rig on-line, Tune moves the frequency through Log4OM
  and the message says the mode didn't change.
- **Log4OM:** turn on Settings → Software integration → Connections → **Remote
  control** (port 2241). Log4OM's own document spells the
  command `SetTxFrequeny`; DX Agent uses the spelling reported to work,
  `SetTxFrequency`. It also sends the spot's call (`SetCallsign`), so Log4OM's QSO window
  is ready to log it; a call you typed there yourself is left alone (Log4OM's rule).
- **Only from the dashboard:** like saving settings, Tune only works from the
  dashboard page itself, so other websites can't move your radio.
- **Tested:** every target against stand-in programs; Log4OM 2.42 with an IC-9100 on
  OmniRig: frequency, mode (through OmniRig) and call.

**FlexRadio with AetherSDR (TCI).** AetherSDR, the FlexRadio client, has a TCI server
(as ExpertSDR for SunSDR and Thetis do). In AetherSDR turn on Settings → **Autostart TCI
with AetherSDR**, or **Enable** in its TCI applet (port 50001); in DX Agent, Settings →
Radio → Tune through **TCI**. DX Agent then keeps one connection open (the header shows a
**TCI** pill: green while connected, with the number of spots on the panadapter), and:

- **Tune** moves the active slice (receiver) to the spot, and sets the mode (CW, USB/LSB,
  RTTY, DIGU for FT8/FT4); a slice in CW-R keeps it for a CW spot. An FT8 spot always sets DIGU (a
  Flex sends DAX audio in DIGU), even from USB.
- **The alerts go on the panadapter** (**Show the alerts on the radio's panadapter**, on by
  default): each station of the alert list in its colour (ATNO red, new band orange, new
  mode cyan...), with its headline and country. It moves when a newer spot puts it on
  another frequency, and is taken off when it is no longer spotted (after **Take out of the
  list after**, or 15 minutes), when the list is cleared, or when your log says you no
  longer need it. Your own call's spots are not shown. AetherSDR's own DX cluster and RBN
  spots stay as they are: DX Agent only adds, and takes off, its own.
- **A click on one of those spots** tunes the radio there (AetherSDR does it), and opens the
  station in DX Agent's Station tab.
- If AetherSDR is closed or restarted, DX Agent connects again by itself and puts the spots
  back. AetherSDR on another computer: its address in **Address**.

**WSJT-X.** Settings → Radio → tick **Also send the call to WSJT-X**, and in WSJT-X tick
Settings → Reporting → **Accept UDP requests**. Then Tune on an FT8/FT4 spot puts the
radio on the FT8/FT4 dial frequency (14074, not the spot's 14075.6) and fills WSJT-X's
**DX Call** and **DX Grid** (when RBN reported it), sets FT8 or FT4 and moves the Rx
frequency to the station's audio offset (1600 Hz for 14075.6), then generates the
standard messages: you click **Enable Tx**. It works with any radio control above, or
alone (radio target none). DX Agent finds WSJT-X by itself (no port to set) and doesn't
take over WSJT-X's UDP port 2237, so Log4OM, JTAlert or GridTracker keep working.

### Follow one station

In the Station tab (click an alert, or a DXpedition in the DXpeditions tab):

- **Show only VP8PJ** limits the list and the map to that call. The list also shows its
  spots that made no alert (grey **SPOT** rows, e.g. a band you already have), so a
  DXpedition on the air never shows an empty list. A **Only VP8PJ ✕** chip
  appears above the list; click it (or **Show all calls**, or **Clear filters**) to see
  every call again. This filter is not saved, so a reload shows everything.
- **☆ Add to watchlist / ★ Remove from watchlist** adds the call to your watchlist or
  takes it off. It's saved in `config.toml` and takes effect at once, without a restart.
  A call covered by a wildcard such as `VP8*` shows **★ In the watchlist (VP8*)**;
  wildcards are edited on the Settings page. The DXpeditions tab has the same star on
  each card.
- **Spot it** posts a DX spot of the station on the DX cluster DX Agent is logged in to
  (Settings → Spot sources), as your callsign: `DX 14025.0 3Y0K up 2`. It opens a small form
  with the frequency of its latest spot (you can change it) and a comment (at most 30 plain
  characters, no accents: what DXSpider and AR-Cluster keep); after you confirm, the spot
  goes out: **posted** when the cluster sends it back to you (most do), **sent** when it doesn't
  (W3LPL, for one: check it on DXSummit), and an error with what the cluster answered when it
  refuses it within 5 seconds (some nodes only take spots from registered users). Every station connected to that cluster sees
  it, and it goes on to the other clusters: spotting yourself is frowned upon, and DX Agent
  asks again if you try. At most one spot every 30 seconds. With the cluster off or down the
  button is greyed out; the RBN only receives.
- **Latest spots** lists the call's spots from the last 60 minutes, from the cluster
  and RBN, including ones that didn't make an alert (for example a band you already
  have). It's kept for DXpeditions, watchlist calls and calls that alerted, from the
  moment the agent started. With radio control on, click a frequency to tune to it.

### Activations: POTA, SOTA, IOTA...

The **Activity** menu (after the mode chips) shows only activations: **POTA, SOTA, IOTA,
WWFF, BOTA, COTA (WCA castles), LOTA (lighthouses), TOTA, ROTA, JOTA, YOTA** and **EME**, or
*any activation* (*none*, the first option, is no Activity filter: everything shows). Next to each, how many stations are on it in the last hour.

- The list shows their alerts and also their spots that made no alert (grey **SPOT**
  rows: a park in a country you already have still shows), from the last hour.
- Each row says the activation under *What you need*, with its reference: `POTA US-5584`,
  `IOTA EU-038`, `WWFF KFF-6494`...
- **The References tab** (it shows while an Activity is picked in the menu, and opens on that program): every summit, park, island and castle you worked (POTA, SOTA, WWFF,
  IOTA, WCA), program by program: how many worked, confirmed and confirmed by QSL, and each one
  with its name and place (POTA's and SOTA's lists), the bands you worked it on, **C** confirmed /
  **W** worked, and your latest QSO with it. Search, and show all, confirmed, by QSL (a paper QSL)
  or not confirmed; click a row for your QSOs with it (**✓** confirmed in LoTW, **✓ QSL** by a
  card), and a POTA or SOTA row for its card too. From your log (QSOs that say the reference: POTA_REF, SOTA_REF,
  WWFF_REF, IOTA, SIG_INFO, Log4OM's award references) and your QSOs with activators DX Agent saw
  on the air. *Confirmed* means a QSO with it is confirmed in LoTW or by QSL; each program's own
  credit is on its site (pota.app, sotadata, wwff.co, iota-world.org). **IOTA** credits an island
  by QSL card, or through LoTW when the LoTW confirmation says the island (the station put it in
  its LoTW location) or the country is a single island group, so not every island confirmed here
  counts there: upload your log at iota-world.org to see the ones it matches. **By QSL** shows
  those with a paper QSL (`QSL_RCVD` in your log; in a log downloaded from LoTW it is LoTW's own
  confirmation, so it doesn't count as a card). POTA's parks list
  (`data/pota-parks.csv`, ~10 MB) is downloaded again when a month old.
- **Your parks and summits on the map:** the **References** menu next to DXCC / WAZ shows the
  POTA parks and/or SOTA summits you worked, green confirmed, amber worked. Click one to see it in
  the Station tab: its pota.app or SOTA card and your QSOs with it (click one to open that
  station), and on the map the path, beam heading and distance from your QTH.
- **The park or summit in the Station tab (POTA, SOTA):** click the row and a card shows what
  pota.app or SOTA say about it: its name, type and where it is (grid square), whether **you**
  worked it and on which bands (or "New for you"), how many activations and QSOs it has, who
  activated it first, its latest activations (call, date, QSOs by mode) and links to its page
  on pota.app or SOTA and to the park's own website. Asked from api.pota.app and
  api2.sota.org.uk (public, no login) when you open it, and kept 30 minutes.
- **POTA and WWFF:** DX Agent also reads their own lists of who is activating now (pota.app
  and wwff.co, every 2 minutes). Any spot of those stations gets its park, even an RBN FT8
  spot (RBN drops the "POTA" of "CQ POTA"), and activators spotted only on those sites show
  too ("(POTA)" in *Heard by*). SOTA's list of who is on the air needs registration with
  SOTA and approval, so SOTA is found from comments only.
- **SOTA summits:** DX Agent keeps SOTA's list of summits (storage.sota.org.uk, fetched again
  when a month old). A summit reference must be a valid summit to count, unless the spotter
  wrote "SOTA" before it (a summit newer than our copy). Hover over the activation to see the
  summit's name, altitude and points; POTA and WWFF parks show their name there too.
- **On the map:** activations sit at their summit (a mountain with a jagged peak) or park (a pine tree), from SOTA's
  list and POTA's and WWFF's; coloured by where you stand with that reference, as the DXCC and
  References layers: red not worked, amber worked, green confirmed (hover for which). Drawn so
  only with an Activity picked: with "none", a station in a park or on a summit is a dot like
  the others, still at its place.
- **New summits and parks:** with an Activity picked, a summit, park, island (IOTA) or castle
  you haven't worked **on that band** is marked *new*, even in a country you have confirmed: a
  spot that made no alert shows as **NEW REF** ("reference not worked on 20m"), the reference
  says *new*, the map paints it like NEW BAND and *Hide confirmed* keeps it. With no Activity
  picked nothing changes. What you have worked comes from:
  - your log, when the QSO says the reference: `SOTA_REF`, `POTA_REF` (a two-fer counts for each
    park), `WWFF_REF`, `IOTA`, `SIG` + `SIG_INFO`, and Log4OM's award references. A POTA
    reference from before POTA's 2024 prefixes is read as it is now, using the QSO's country:
    Canada's VE- is CA- now (VE-1596 is CA-1596), the USA's K- is US-, with the same numbers. An
    old one that is another country's park now (VE- is Venezuela's) and has no known new
    reference keeps the log's reference, with no place on the map, and counts for no park;
  - what DX Agent saw on the air: it remembers where each activator was and when (the last 32
    days, in `data/dxagent.db`). A QSO in your log with that station within 45 minutes of
    one of its spots counts for the summit or park it was on, even if your logger didn't write
    it (`/P` or not). Those QSOs are kept for good, in the same file. This only
    knows activations DX Agent saw, so it fills in from now on.
- Clusters have no field for this: DX Agent reads the spot's comment. It recognises the
  program's name (`POTA`, `WWFF`, `WCA`, `ILLW`, `JOTA`, `EME`...) and references by their
  format (`US-5584`, `KFF-6494`, `EU-038`, `LA/VF-010`, `DL-01121`). A spot whose spotter
  wrote neither isn't seen as an activation. Stations with YOTA in their call (GB23YOTA)
  count as YOTA.
- Like the other filters it is saved; **Clear filters** turns it off.

### CQ DX Marathon

The [CQ DX Marathon](https://dxmarathon.com) scores one point for each country and one for
each CQ zone you work: in the year (January 1 to December 31), or in the **Sprint**, ten days
in October (2026: October 17 to 26). Turn it on in Settings → **CQ DX Marathon**, pick the
year or the Sprint (and its dates) and the modes your entry counts. Then:

- A **Marathon** tab next to DXpeditions shows the period, your score (countries + zones),
  the 40 CQ zones (green = worked) and the countries: **Worked**, with the QSO that counts
  (the first one in the period: call, date, band, mode; click it to open the station), or
  **Missing**, the ones still to work. The search box finds a country by name or prefix.
  The tab's number is your score.
- A station that would add a country or a zone alerts as **MARATHON** (purple), even in a
  country you have confirmed for DXCC, and the "What you need" column says what it adds
  ("Marathon: new country (Sicily), zone 15"). Each kind can be turned off in Settings. They
  cool down per country (or zone), band and mode: one Sicilian station is enough. The
  **MARATHON** filter chip and the tab only show while the Marathon is followed.
- The Station tab says, for any call, its Marathon country and zone, and whether and when
  you worked them in the period.
- Everything follows your log: a QSO you log (file, Log4OM's database, UDP) counts at once.

What counts, as in the rules:

- **Countries:** the CQ DX Countries List, the DXCC entities plus six WAE countries:
  Sicily (IT9), African Italy (IG9/IH9), Shetland (GM/s), European Turkey (TA1), Bear Island
  (JW/b) and the Vienna International Centre (4U1VIC). A QSO's country is the ADIF `REGION`
  when your logger wrote one, else the `DXCC` field, made a WAE country when the call is in
  one (from AD1C's country file).
- **Zones:** the QSO's `CQZ`, else the call's zone in AD1C's file (which knows that W0 is
  zone 4 and VE7 zone 3). A US, Canadian or Russian station can be in another zone than its
  call says: for a spot, the zone shown is the call's.
- **Callsigns:** your main one, or the ones you tick in the card (at most 2, see Several
  callsigns).
- **Bands and modes:** 160 to 6 m; no satellites, repeaters or internet links; no /MM or /AM.
  Modes: all, or only CW, phone or digital, or the Sprint's **Classic** class (CW and phone).
- The annual Marathon is the current UTC year; until January 5th (the log deadline) it is
  still last year's. DX Agent doesn't submit your entry: upload your ADIF at
  [entry.dxmarathon.com](https://entry.dxmarathon.com).

### World map

Click **Map** (next to **List**, left of the filter chips) to see the same alerts on a
world map. The filters apply to both views.

- **Dots** are alerts, coloured like their type. A station's position comes from the
  grid square RBN reports for FT8/FT4 spots; otherwise it's the country's centre
  (marked "approx." in the tooltip). Hover a dot for the details and how long ago the
  station was spotted, click it to select it. As in the list, a dot fades 10 minutes after
  the station's last spot on that band and mode, and more after 30: a faded dot is where it
  was, not where it is.
- **Click a country** (anywhere on it, or near a small island) to see it in the Station tab,
  as a lookup shows a call's country: its card, beam headings to its centre, status, CQ DX
  Marathon box and band × mode grid.
- **Diamonds** are active DXpeditions: red = ATNO, amber = not confirmed, teal = you have the
  country but on fewer than half of your bands, grey = you have it. With an alert in the last
  15 minutes the diamond takes the alert's colour. A **green frame** means it was spotted in
  the last 15 minutes (on the air). The legend in the bottom-left corner sums this up. Several at
  the same place (four at Koror, Palau, say) sit on a small ring around it, each with its own
  click, the ones on the air on top. Which ones show: the **DXpeditions** menu (next to DXCC /
  WAZ / References): those **on the air** (the default), those **needed** (as the tab's "only
  needed"), **all** the active ones, or **no**; the station open in the Station tab always shows,
  and your browser remembers it.
- **Blue dot:** your QTH, from your grid square.
- **Shaded area:** where it is night right now (the greyline is its edge), updated every minute.
- **Selected station:** a dashed great-circle path from your QTH, the **beam heading**
  (and long path) and distance, and small dots for every receiver that heard it in the
  last 15 minutes (green = near you).
- **Zoom** with the mouse wheel or the **+ / −** buttons, **drag** to move,
  **⟲** to see the whole world again. Call signs of all stations appear once you zoom in;
  where they would overlap, the most important ones are shown (hover a dot for its call).
- The Station tab also shows both **beam headings**, big, each with a compass pointing that
  way and its distance: the short path in blue, the long path (the other way round the
  Earth) in orange, in either view.
- The borders are **DXCC** borders, not just countries': European and Asiatic Russia,
  England, Scotland and Wales or Mount Athos are split too. Hover anywhere on the map to see
  which entity it is. Each entity shows its prefix once it is big enough on screen (zoom in
  to see the small ones).
- Coasts and country borders come from Natural Earth 1:50m (public domain); the DXCC
  entities and the borders countries don't have, from GridTracker's DXCC map (BSD licence).
  Both are downloaded the first time you open the map and kept in
  `data/world-countries-50m.json` and `data/dxcc-shapes.json`.
- A link straight to the map: `http://localhost:8073/?view=map`.

#### Your DXCC on the map

The **DXCC** menu in the map's top-left corner paints every entity by where you stand for
an award: **green** = confirmed, **amber** = worked but not confirmed, **red** = not worked.
Next to it: how many you have confirmed and worked, out of the current entities. Hover an
entity to read its status. Pick the award in the first menu and, for Mixed, Phone, CW or
Digital, a band in the second:

- **Mixed**, **Phone**, **CW**, **Digital** with *all bands*: those awards;
- **Mixed** and a band: that band's award (160 to 6 m always; 2 m, 70 cm and up once you
  have worked them);
- **Phone**, **CW** or **Digital** and a band, say **CW** and **40 m**: your slots on that
  band and mode. That isn't an ARRL award, but it is how you chase slots;
- **Satellite**;
- **It follows the filters above the list:** with *all bands* (or **Mixed**) here and a single
  band (or mode) picked in the filters, say 12 m and DIGI, the layer paints that band and mode,
  as the dots do; the menus then say "12 m (filter)", "Digital (filter)". A band or mode picked
  here wins. The WAZ layer follows the band the same way;
- **Challenge (160–6 m)**: the more bands confirmed, the greener (0 to 10); the total
  counts band-entities, as the DXCC Challenge does.

It follows the ARRL's DXCC rules (arrl.org/dxcc-rules):

- Confirmed means **LoTW or a paper QSL** received (QSL_RCVD); eQSL doesn't count. A card
  you have received isn't credited until ARRL or a card checker has checked it, so your
  ARRL totals can be lower.
- Satellite QSOs (PROP_MODE SAT) count only for Satellite; repeater and internet QSOs
  (RPT, INTERNET, ECH, IRL), 60 m QSOs and stations at sea or in the air (/MM, /AM) don't
  count at all; QSOs before 15 Nov 1945 don't count, and CW before 1975 counts for Mixed only.
- Only the current DXCC list counts: deleted entities are left out.
- It doesn't check where you operated from or under which call: every QSO in your log counts.

Small entities (islands, microstates, enclaves like Vatican City or Guantanamo Bay) also get a
halo in their colour, so you can see them without zooming in.

#### Your WAZ on the map

Pick **WAZ** in the first menu (instead of DXCC) to paint the 40 **CQ zones** by where you
stand for CQ's Worked All Zones: the same green / amber / red, each zone with its number and
its borders. Hover a zone to read its name and status. Pick the award:

- **Mixed, SSB, CW, RTTY, AM**; **SSTV** (QSOs from 1973); **Digital** (from 2000, without
  RTTY, which has its award);
- a mode and a band, say **CW** and **40 m** (80-10 m from 1973; 30, 17 and 12 m from 1991),
  or **Mixed** and a band (160 m from 1975, 30 zones to apply; 6 m, 25 zones); 2 m and up
  count EME QSOs only;
- **Satellite** (from 1989, 25 zones) and **EME** (from 1973, 25 zones);
- **5BWAZ**: the 40 zones on each of 80, 40, 20, 15 and 10 m, any mode, QSOs from 1979: the
  more bands confirmed, the greener; the total counts band-zones (200; the basic award is 150).

It follows CQ's rules (n4baa.com/9_Rules.pdf, August 2026):

- Confirmed means **LoTW, a paper QSL or eQSL** (WAZ takes eQSL; DXCC doesn't).
- The zone is the one your logger saved with the QSO (ADIF CQZ), else the entity's zone from
  cty.csv. In countries with several zones (USA, Canada, Russia, Brazil, China, Australia...)
  check that your logger fills it in.
- Ships and aircraft (/MM, /AM), repeaters and internet links never count; QSOs before
  14 Nov 1945 don't either.
- Not checked: that you operated from one DXCC entity, and the Traditional / Remote categories.

**CQ zones** (the box in the menu) shows the zones' borders and numbers in any view, DXCC
or none.

### Safety

- The dashboard only listens on `127.0.0.1`, so other computers can't reach it.
- It only answers requests addressed to `localhost` (protection against
  DNS-rebinding tricks from websites).
- Settings can only be saved from the dashboard page itself. Other websites you visit
  can't send a request that changes them, and can't show the dashboard inside their
  own page.
- Text from spots (cluster comments come from anyone on the internet) is always shown
  as plain text, never run as code.

---

## 3. Settings page

Click **Settings** in the header (or open `http://localhost:8073/#settings`). Change
what you need and click **Save and apply**: the agent rewrites `config.toml`,
reloads your log and reconnects the feeds within a few seconds. You don't need to
restart it.

| Section | What you can set |
|---------|------------------|
| **Station** | Your callsign; **Your log**: an ADIF file from your logger, **LoTW only** (see LoTW as your log), or **Start from scratch** (see A new log); the path to your ADIF log (one per line; `*` picks the newest file, e.g. `…/CALL_*_backup.adi` for Log4OM backups). **Browse…** opens the Windows file window to pick the file (if you pick a dated Log4OM backup, it becomes `…/CALL_*_backup.adi` so the newest is always used). **Test** shows which file it found, its size, date and QSO count; your grid square (empty = taken from your log); **Your other callsigns**, each with its own log (see Several callsigns) |
| **LoTW** | Turn the confirmation download on or off; LoTW username (empty = your call); LoTW password; how often to download. **Test login** checks the username and password with LoTW |
| **Alerts** | Which alert types you want (and **Your own call**: who hears you); bands; modes; when a DXpedition or watchlist call counts as worked; watchlist; ignore list; how long before the same alert repeats and **Keep only the latest row of each station** (same call, frequency and mode: when it shows up again, its older rows leave the list); **Take out of the list after** N minutes (stations not spotted on that band and mode for that long leave the list; 0 = never, the default); the "near you" radius and **Only alert when heard near you**; Windows pop-ups and for which alerts |
| **Spot sources** | DX cluster on/off, its servers (`host:port`, tried in order) and login commands; **Recent spots on connecting** (minutes, 0 = off); which continents' cluster spotters to use; RBN on/off, CW/RTTY and FT8/FT4 feeds; which continents' skimmers count |
| **Radio (Tune button)** | Which program tunes the radio (Off, Log4OM 2, N1MM Logger+, DXLog.net, OmniRig, Hamlib rigctld, flrig, DXLab Commander, TCI), its address and port (0 = default), whether to set the mode too, which OmniRig radio, and with TCI whether the alerts go on the panadapter |
| **CQ DX Marathon** | Follow it or not; the year or the Sprint, and the Sprint's dates; the modes that count (all, CW, phone, digital, or Classic: CW and phone); alerts for a new Marathon country and for a new zone |
| **DXpeditions, log and dashboard** | Show DXpeditions you already have; DXpedition sources and how often to re-read them; how often to check the log file; dashboard port; open the browser on start |

How it behaves:

- **Checks before saving.** A callsign that doesn't look valid, an empty log path, no
  bands, a cluster server without `:port` and similar mistakes are listed in red, and
  nothing is saved until they are fixed.
- **Backup.** The previous file is kept as `config.toml.bak`.
- **Your LoTW password** is stored encrypted for your Windows user in
  `data/lotw.secret` — never in `config.toml` — and is never shown again. The box
  says whether a password is saved. Leave it empty to keep the saved one; tick
  **Forget the saved password** to remove it.
- **Dashboard port** changes take effect the next time you start DX Agent.
- **Comments.** Saving rewrites `config.toml` with its own short comments; comments
  you added by hand are not kept.

---

## 4. Commands

All commands run from the project folder as `python -m dxagent <command>`. `run` is
the default.

| Command                 | What it does                                                                     |
|-------------------------|----------------------------------------------------------------------------------|
| `run`                   | Watches cluster and RBN, serves the dashboard, alerts on needed stations until Ctrl+C |
| `dxpeditions --days 30` | Lists active DXpeditions and those starting in the next N days, with your status and LoTW for each |
| `check CALL`            | Shows the worked/confirmed grid for that call's country, by band and mode       |
| `stats`                 | Counts countries worked and confirmed per band and per mode                     |

Run the tests with:

```powershell
python -m unittest discover -s tests -t .
```

### `check` example

`C` = confirmed (LoTW or paper QSL), `W` = worked but not confirmed, `.` = not worked.

```text
VP2MAW: Montserrat (DXCC 96, NA, CQ 8) - no LoTW
          160m   80m   40m   30m   20m   17m   15m   12m   10m    6m
      CW     .     .     .     .     C     C     W     .     C     .
   PHONE     .     .     .     .     C     .     .     .     .     .
    DIGI     .     .     .     .     .     .     C     .     .     .
  C = confirmed (LoTW/QSL)   W = worked, not confirmed   . = not worked
  this call: not in log
```

### `dxpeditions` status column

- `NEED: ATNO` — country never worked.
- `NEED: not confirmed` — worked, but no confirmation on any band.
- `have: 4 bnd, 2 cfm` — worked on 4 bands, confirmed on 2. `(wkd 15,20)` means you
  already worked this DXpedition call on those bands.

The LOTW column shows the station's last upload date, `LoTW announced` (the DXpedition
listing says QSL via LoTW but nothing is uploaded yet) or `no LoTW`.

### `stats` example

```text
        worked   cfm
  160m      58    41
   80m     121    97
   40m     203   181
   ...
    CW     247   226
 PHONE     219   190
  DIGI     232   208
 total     271   248
```

---

## 5. Reading the alerts

Each alert is a row on the dashboard and one line in the terminal: time (UTC),
headline, call, frequency (kHz), band, mode, country, then details separated by `|`.

```text
14:56Z DXPED NEED  VP2MAW   18095.0   17m FT8   Montserrat   NEED: 17m DIGI never worked | until 10-20 | no LoTW | rbn NG7M -14dB
14:55Z DXPED HAVE  VP2MAW   14030.0   20m CW    Montserrat   have it: 20m CW confirmed | until 10-20 | no LoTW | rbn WE9V 7dB 27wpm
14:55Z NEW BAND    SV2GNJ   18100.0   17m FT8   Greece       LoTW 2022-05-10 | rbn WZ7I -8dB
```

- **Mode** comes from RBN or from the cluster comment. When it has to be guessed
  from the band plan it is shown with a `?`, e.g. `CW?`.
- **Last detail:** `rbn SPOTTER SNR WPM` for skimmer spots, or
  `de SPOTTER: comment` for cluster spots.
- **Square brackets** at the end list every reason when there is more than one,
  e.g. `[new_band, new_mode]`.

### Alert types

| Headline   | Meaning                                                                      | Default |
|------------|------------------------------------------------------------------------------|---------|
| ATNO       | Country never worked                                                         | on      |
| DXPED NEED | Active DXpedition that adds a new country, band, mode or band+mode for you   | on      |
| DXPED CFM  | Active DXpedition for a country you worked but haven't confirmed on this band or mode | on |
| DXPED HAVE | Active DXpedition you already have confirmed on this band and mode (grey, no pop-up) | on |
| NEW BAND   | Country worked, but not on this band                                         | on      |
| NEW MODE   | Country worked, but not in this mode group (CW, PHONE, DIGI)                 | on      |
| WATCHLIST  | A call from your watchlist, not yet worked                                   | on      |
| NEW SLOT   | Country not worked on this band+mode combination                             | off     |
| NEW CALL   | This exact callsign isn't in your log (very noisy)                           | off     |
| MARATHON   | A country or CQ zone you haven't worked yet in the CQ DX Marathon (see CQ DX Marathon) | on, while the Marathon is followed |
| YOU'RE HEARD | Your own callsign spotted (any of yours, on any band and mode): who hears you, and how far away, in What you need ("heard 8,912 km away, in United States"); the row is highlighted in pink and follows the latest spot (×N skimmers). Click it for every receiver that heard you, nearest first, and on the map | on (`alerts.self_spots`) |

"Default" refers to **Alert me for** on the Settings page (`alert_on` in the file).

The list and the map keep the last 500 alerts. When the list is full, the least important go
first: the oldest NEW CALL and DXPED HAVE, or any alert over an hour old. The others (ATNO, a
new band, mode or slot, a DXpedition, the watchlist, the Marathon) stay at least an hour, so
NEW CALL, which can bring a hundred alerts a minute, doesn't push them out.

When your log changes (you work the station), the alerts already listed follow: a reason that
no longer holds leaves (a NEW SLOT you then worked isn't one any more, nor under that filter),
and the headline, colour and "What you need" with it. A DXpedition stays one: DXPED NEED, CFM or
HAVE by what it still adds. An alert with no reason left shows as **WORKED** (grey).

### Where the DXpeditions come from

Three public sources are merged by call sign (choose them in Settings →
**DXpedition sources**; all three are on by default):

| Source | What it adds |
|--------|--------------|
| **NG3K ADXO** | The best-known calendar of announced operations, with dates, QSL route and notes |
| **425 DX News** | The Italian weekly bulletin's calendar: many operations NG3K doesn't list yet. Special-event and anniversary calls are left out |
| **Club Log** | Expeditions uploading their log to Club Log right now (a log updated in the last 3 days counts as on the air), a link to search the log and request an OQRS card, and the **DXCC Most Wanted** rank |

When the same call is in more than one source, the dates, QSL route and notes come
from NG3K first, then 425 DX News; Club Log adds its link and, because its dates are
real QSOs, can stretch the dates. The same call on two separate trips (now and next
year) stays two entries. Each card in the DXpeditions tab shows its sources and the
Most Wanted rank; the Station tab links each source and the Club Log log.

**Order.** **On the air** puts first the DXpeditions spotted in the last 15 minutes, the latest spot on top, and the rest by Most Wanted; one that goes 15 minutes without a spot drops back. **Most Wanted** (the default) puts the most wanted countries first, by Club Log's rank, and those without a rank last; **Date** sorts by the announced start date. The browser remembers your choice. 425 DX News sometimes
gives only an end ("till 31/10"); the start is then read from the bulletin text
("from 20 September to…", "on 8-11 October", "between 7 and 21 October"). When the
text doesn't say either, the card shows **until 10-31** and goes first, since it is
already on.

**On the air.** A DXpedition spotted (cluster or RBN) in the last 15 minutes gets a
green frame and an **ON THE AIR** line with the frequency (click it to tune), band,
mode, how long ago and who spotted it. The DXpeditions tab shows how many are on the
air (**● 4**) even while you look at the Station tab, the map rings their diamonds,
and a message pops up in the dashboard the moment one comes on. This happens for every
active DXpedition, needed or not and on your watchlist or not; **only needed** never
hides one that is on the air.

### Are you in their log?

When you open an active DXpedition in the Station tab and its log is on Club Log, the
DXpedition box checks Club Log's log search for your callsign: the band slots where you
are in their log (✓), the ones they have been on the air that you still need (·), and two
warnings: **in your log but not in theirs** (a busted call, a QSO they didn't log, or a
day not uploaded yet: the last upload date is shown) and **in their log but not in
yours** (a QSO missing from your log). It asks only when you open that station and keeps
the answer for 10 minutes; it says when it asked. Just worked them? **↻ Refresh** asks
Club Log again at once (your QSO shows there once they upload it). Many DXpeditions don't use Club Log: then it shows the web
addresses from their announcement (NG3K, 425 DX News), where their own log search or
OQRS usually is.

### Do I need this DXpedition?

The text after `NEED:` says exactly what working it adds, most important first:

| Text                                                   | Meaning                                                    |
|--------------------------------------------------------|------------------------------------------------------------|
| `ATNO`, `new band`, `new mode`                         | Something you have never worked                            |
| `17m DIGI never worked`                                | You have the band and the mode, but not together           |
| `entity not confirmed`, `band not confirmed`, `mode not confirmed` | Worked, but no LoTW or paper QSL (eQSL doesn't count) |
| `have it: 20m CW confirmed`                            | Nothing new — work it only if you want to                  |

The headline summarises it: **DXPED NEED** when something is new, **DXPED CFM** when
the gain is a confirmation, **DXPED HAVE** when there is nothing to gain.

### Colours and pop-ups

| Priority | Colour        | Pop-up | Alerts                                                              |
|----------|---------------|--------|---------------------------------------------------------------------|
| 1        | white on red  | yes    | ATNO                                                                |
| 2        | yellow        | no     | NEW BAND; DXPED NEED/CFM for a new band or mode, or an unconfirmed country |
| 3        | cyan          | no     | NEW MODE, WATCHLIST; DXPED for a new band+mode or an unconfirmed band/mode |
| 4        | green         | no     | NEW SLOT; DXPED for an unconfirmed band+mode                        |
| 5        | grey          | no     | NEW CALL, DXPED HAVE                                                |

By default only ATNO pops up (`notify_max_priority = 1`). To get pop-ups for
priority 2 or 3 alerts too, change it under
**Windows pop-ups** on the Settings page. Every alert is also
kept in `data/dxagent.db` (the last `keep_days` days).

### Near you: is the station reaching your area?

A strong signal on another continent means little if it isn't reaching you. DX Agent
records **every** RBN and cluster spot (not only the ones that alert) together with
where the receiving station is, and for each alert shows the nearest one that heard
it in the last 15 minutes on that band:

| Shown | Meaning |
|-------|---------|
| **● 732 km · 7 dB** (green) | Heard near you: a station within your "near" radius (default 1,500 km) heard it, at that SNR |
| **8,176 km** (grey) | Only heard further away; this is the nearest receiver |
| **—** | No RBN or cluster reports in the last 15 minutes |
| **~** before a distance | Approximate: a human cluster spotter, placed at their country's centre |

- **Where receivers are:** RBN skimmers come from RBN's own skimmer list, with
  exact grid squares, downloaded daily. Cluster spotters are placed at their
  country's centre from `cty.csv`.
- **Your location:** the grid square from Settings. If it's empty, it is taken from
  `MY_GRIDSQUARE` in your log.
- **Updates:** the column refreshes every 20 seconds, because nearby receivers often
  report a station a little after the first spot.
- **Filter:** the **Near you only** chip shows only alerts heard near you. To get
  *alerts* only for those, tick **Only alert when heard near you** in Settings; a
  station then alerts as soon as someone near you hears it.
- **Station tab:** **Who hears it** lists the receivers nearest you first, with band,
  SNR and age. **Check PSKReporter (FT8/FT4)** asks PSKReporter, which has many more
  FT8 receivers in South America than RBN has skimmers. Its receptions are added to
  "near you", so the summary and the table's column update too. The answer is cached
  for 5 minutes per call. PSKReporter asks for **at most one query every 5 minutes**
  (for all stations together) and blocks connections that query more often (HTTP
  429), so DX Agent enforces that limit, shows when the next query is allowed, and
  after a block waits at least 15 minutes before trying again.
- **PSKReporter live:** in Settings → Spot sources, **PSKReporter** can be **Live
  (MQTT)** instead of **When you ask**. DX Agent then connects to
  mqtt.pskreporter.info, which passes on every report sent to PSKReporter as it
  arrives, with no account and no query limit, and follows:
  - the station open in the Station tab, from the moment you open it, for 15
    minutes after you last looked at it;
  - the digital (FT8, FT4…) alerts in the list spotted in the last 15 minutes;
  - with **Only alert when heard near you** on, the digital stations it kept from
    alerting, so a nearby receiver lets their next spot alert.

  It follows at most 100 stations, by call: never a whole country (receivers in the
  USA alone send about 300 reports a second). Their receivers count for "near you" on
  their own, with no button. **Who hears it** lists them as PSK, and the Station tab
  says since when the station is followed and how many receivers have reported it.
  Only new reports arrive: right after you open a station the list is empty, and
  receivers show up as their programs send them, usually within a few minutes. A
  **PSKReporter** pill next to the feeds shows the connection and the reports
  received. Live needs your grid square.

How many receivers are near you depends on where you are: RBN skimmers are dense in
North America and Europe and sparse elsewhere, while PSKReporter usually has many
more FT8 receivers.

### Repeat alerts (cooldown)

The same alert isn't repeated within the cooldown (default 15 minutes):

- **Country alerts** (ATNO, NEW BAND, NEW MODE, NEW SLOT) are limited per
  country + band + mode. One US station on 30 m CW is enough; you won't get an alert
  for every other US station on 30 m CW for the next 15 minutes.
- **Call alerts** (DXPEDITION, WATCHLIST, NEW CALL) are limited per call + band + mode,
  so the DXpedition alerts again when it moves to another band or mode.

---

## 6. The configuration file

Everything on the Settings page is stored in `config.toml` in your DX Agent folder (see
[Quick start](#1-quick-start); **Settings** shows its exact path).
You can also edit it by hand; restart `run` afterwards.

| Section     | Setting                     | Default                                           | What it does |
|-------------|-----------------------------|---------------------------------------------------|--------------|
| (top)       | `data_dir`                  | `data` (default)                                  | Where downloads, the alert history and the saved LoTW password are stored |
| station     | `callsign`                  | (your call)                                       | Used to log in to the cluster and RBN |
| station     | `log_source`                | `adif`                                            | `adif` = a file from your logger; `lotw` = every QSO you uploaded to LoTW (no ADIF file); `new` = a new log DX Agent keeps, `data/<CALL>.adi` (starting from scratch) |
| station     | `adif`                      | `C:/Users/you/AppData/Roaming/Log4OM2/backup/CALL_*_backup.adi` | Path or glob of your log. With a glob the newest file is used. A list of paths also works |
| station     | `grid`                      | empty (= taken from the log)                      | Your grid square, for "near you" |
| station     | `others`                    | `[]`                                              | Your other callsigns, each with its log: `[{call = "N1CALL", adif = ["C:/…/N1CALL.SQLite"]}]` (see Several callsigns) |
| station     | `active`                    | empty (= all your callsigns)                      | The callsign alerts, DXCC and WAZ follow; set by the header's picker |
| alerts      | `alert_on`                  | `atno, dxpedition, new_band, new_mode, watchlist` | Which alert types fire |
| alerts      | `call_scope`                | `band_mode`                                       | For DXpedition and watchlist calls: `call` = any QSO counts, `band` = per band, `band_mode` = per band and mode |
| alerts      | `bands`                     | 160m–6m (no 60m)                                  | Spots on other bands are ignored |
| alerts      | `modes`                     | `CW, PHONE, DIGI`                                 | Mode groups to alert on |
| alerts      | `watchlist`                 | empty                                             | Calls to always look for; wildcards allowed, e.g. `"VP8*"` |
| alerts      | `ignore`                    | empty                                             | Calls never to alert on; wildcards allowed |
| alerts      | `cooldown_minutes`          | `15`                                              | Minimum time before repeating the same alert |
| alerts      | `desktop`                   | `true`                                            | Windows pop-up notifications (built into Windows 10/11) |
| alerts      | `notify_max_priority`       | `1`                                               | Pop-ups for: 1 = ATNO only; 3 = also new mode and watchlist |
| alerts      | `near_km`                   | `1500`                                            | "Near you" = heard by a station within this many km |
| alerts      | `near_only`                 | `false`                                           | Only alert when the station is heard near you |
| alerts      | `latest_only`               | `false`                                           | Dashboard list: keep only the latest row of each station on the same frequency and mode |
| alerts      | `self_spots`                | `true`                                            | List the spots of your own callsign (YOU'RE HEARD): who hears you, and how far away |
| alerts      | `expire_minutes`            | `0`                                               | Dashboard list: take out stations not spotted on that band and mode for this many minutes (0 = never) |
| alerts      | `keep_days`                 | `7`                                               | Days of alerts kept in `data/dxagent.db` (0 = none). With RBN on, about 25,000 alerts a day, some 15 MB |
| ui          | `enabled`                   | `true`                                            | Start the web dashboard with `run` |
| ui          | `port`                      | `8073`                                            | Dashboard address: `http://localhost:8073` |
| ui          | `open_browser`              | `true`                                            | Open the dashboard automatically when `run` starts |
| ui          | `check_updates`             | `true`                                            | Say on the dashboard (a pill next to the version) when a newer DX Agent is out: asks GitHub every hour for the latest release of the download repository. Nothing of yours is sent, nothing is installed |
| logbook     | `reload_seconds`            | `60`                                              | How often to check the ADIF file for changes |
| logbook     | `live`                      | `true`                                            | Also read the QSOs Log4OM 2 saved since its last backup, every 10 seconds, from its database |
| logbook     | `udp_port`                  | `12065`                                           | QSOs your logger sends over UDP as you log them (N1MM, DXLog.net, Log4OM, WSJT-X; see "Your QSOs as you log them"); 0 = off |
| logbook     | `udp_group`                 | `""`                                              | A multicast group to join on that port (WSJT-X's UDP Server, e.g. `239.255.0.1`) |
| dxpeditions | `refresh_hours`             | `6`                                               | How often to re-read the DXpedition sources |
| dxpeditions | `sources`                   | `["ng3k", "425dxn", "clublog"]`         | Where to find DXpeditions: NG3K ADXO, 425 DX News, Club Log |
| dxpeditions | `show_have`                 | `true`                                            | Show DXPED HAVE lines (grey, no pop-up) |
| lotw        | `enabled`                   | `true`                                            | Download your LoTW confirmations |
| lotw        | `username`                  | empty (= your call)                               | LoTW login (empty = your call) |
| lotw        | `password_env`              | `LOTW_PASSWORD`                                   | Name of an environment variable to read the password from (optional, see LoTW) |
| lotw        | `refresh_hours`             | `12`                                              | How often to download new confirmations |
| cluster     | `enabled`                   | `true`                                            | Use the DX cluster |
| cluster     | `hosts`                     | `w3lpl.net:7373`, `dxspider.co.uk:7300`, `dx.k3lr.com:23` | DX clusters, tried in order. If one fails or sends nothing for 5 minutes, the next is used |
| cluster     | `commands`                  | empty                                             | Sent after login, e.g. `["set/ft8"]` |
| cluster     | `history_minutes`           | `30`                                              | On connecting, ask the cluster for its latest spots (`sh/dx 100`) and keep those from the last N minutes; `0` = off |
| cluster     | `spotter_continents`        | empty (all)                                       | Only use spots from spotters on these continents |
| rbn         | `enabled`                   | `true`                                            | Use the Reverse Beacon Network |
| rbn         | `cw` / `digital`            | `true` / `true`                                   | CW+RTTY feed (port 7000) and FT8/FT4 feed (port 7001) |
| rbn         | `spotter_continents`        | `SA, NA`                                          | Only alert when a skimmer on these continents hears the station |
| rig         | `target`                    | `none`                                            | Tune button: `none`, `log4om`, `n1mm`, `dxlog`, `omnirig`, `hamlib`, `flrig` or `dxlab` |
| rig         | `host` / `port`             | `127.0.0.1` / `0`                                 | Where that program listens; 0 = its default port |
| rig         | `set_mode`                  | `true`                                            | Also set the mode (CW, USB/LSB, data/RTTY) |
| rig         | `omnirig_rig`               | `1`                                               | OmniRig radio 1 or 2 |
| marathon    | `enabled`                   | `false`                                           | Follow the CQ DX Marathon (a Marathon tab and MARATHON alerts) |
| marathon    | `event`                     | `year`                                            | `year` (January 1 to December 31) or `sprint` |
| marathon    | `sprint_start` / `sprint_end` | `2026-10-17` / `2026-10-26`                     | The Sprint's first and last day (UTC) |
| marathon    | `category`                  | `all`                                             | Modes that count: `all`, `CW`, `PHONE`, `DIGI` or `classic` (CW and phone) |
| marathon    | `alert_countries` / `alert_zones` | `true` / `true`                             | Alert on a country / a CQ zone not worked yet in the Marathon |
| marathon    | `calls`                     | `[]` (= your main callsign)                       | Your callsigns whose QSOs count, at most 2 |

The LoTW password is never stored in this file (see below).

### Tuning tips

- **Too many alerts.** On bands where your log has few countries, NEW BAND fires
  often. Keep only **ATNO** and **DXpedition**
  ticked, or untick bands you don't operate.
- **Only stations you can probably hear.** Under **Only skimmers in**, select only
  your own continent. That is a better sign the path to you is open, but where
  skimmers are few you will miss some spots.
- **Hide DXpeditions you already have.** Untick **Still show them**.
- **Chase specific stations.** Add them to the **Watchlist**, e.g. `3Y0K` or `VP8*`.
- **Repeat alerts sooner or later.** Change **Don't repeat for**.

### Several callsigns

Add your other callsigns (a contest call, say) in Settings → Station → **Your other
callsigns**, each with its own log: an ADIF file (`*` picks the newest), or Log4OM's
database itself (`.SQLite`, e.g. `C:/Users/you/Log4OM/N1CALL.SQLite`: nothing to export).
Then:

- **Alerts, DXCC and WAZ count all your callsigns together**: a country worked with any of
  them is worked, a QSO confirmed with any of them is confirmed. The header's callsign
  becomes a picker: **All** (the default), or one callsign alone, to see what that one still
  needs. The choice applies at once and is kept for the next start.
- **The CQ DX Marathon** counts the callsigns you tick in its card, at most 2: the rules allow
  two in one entry, used from the same location.
- **Which QSO is whose:** each log's QSOs are its callsign's. A QSO whose `STATION_CALLSIGN`
  is another of your calls goes to that one (a logger with every call in one log works too).
  QSOs logged over UDP or in Log4OM's database go to the call they say (WSJT-X, N1MM and
  Log4OM say it), else to your main callsign.
- **LoTW:** one LoTW account with every callsign's certificate is enough: the confirmations are
  downloaded with the callsign each QSO was made with, and each callsign takes its own. A
  confirmations file from before this version is downloaded again once, in the background
  (the old one is used meanwhile).
- Your own spots, from any of your callsigns, never alert. The cluster and RBN log in with
  your main callsign; **Are you in their log?** asks Club Log for every callsign (with All)
  or for the one picked.

### A new log (starting from scratch)

With no log yet, choose Settings → Station → **Your log** → **Start from scratch (a new log)**.
DX Agent creates an empty ADIF log, `data/<CALL>.adi` (a call with a `/` gets `_`), and uses
it: at first every station counts as new. It never overwrites one that's already there.

The QSOs your programs send over UDP as you log them (WSJT-X, N1MM Logger+, DXLog.net,
Log4OM...: see "Your QSOs as you log them") are written to it, once each, so the log grows by
itself. Nothing else writes to it, and DX Agent never writes to your logger's file. It's a
normal ADIF file: when you start using a logger, import it there and choose **ADIF file**.

### Your QSOs as you log them

DX Agent reads your log file (the ADIF backup) at start and whenever it changes. A logger
that tells it each QSO as you save it keeps it current in between: a station you just worked
stops alerting at once, the alerts already listed say what they need now, and the
DXpeditions, activations and the map's slots refresh. The QSOs go on top of the file until
the next backup has them. Those that came over UDP are also kept in `data/dxagent.db`, so
closing DX Agent before your logger writes its file doesn't lose them: at the next start
they count again, and each one goes once the log read again has it (at most 2,000, for 30
days).

**Log4OM 2** needs nothing: DX Agent reads the QSOs since the backup straight from Log4OM's
database, every 10 seconds (`logbook.live`).

**Over UDP** (`logbook.udp_port`, 12065 by default, on 127.0.0.1): one port takes N1MM's
contact XML, ADIF text and WSJT-X's messages. In the logger, add `127.0.0.1` port `12065` as a
destination, next to the ones you already have:

| Logger | Where | Tested |
|--------|-------|--------|
| **N1MM Logger+** | Config → Configure Ports… → **Broadcast Data**: tick **Contacts**, add `127.0.0.1:12065` to the addresses | its documented format |
| **DXLog.net** | Options → Configure network: QSO broadcast `127.0.0.1:12065` (up to three, separated by spaces); Options → Broadcast: QSO, and **Use N1MM QSO format** | its format (as N1MM's) |
| **Log4OM 2** (without the database) | Settings → Software integration → Connections → UDP outbound, type **ADIF_MESSAGE**, `127.0.0.1` port `12065` | ADIF format |
| **WSJT-X / JTDX** | Usually not needed: your logger gets their QSOs and tells DX Agent. If they log on their own: Settings → Reporting → UDP Server. A multicast group (`239.255.0.1`, port `2237`) lets DX Agent (`udp_group = "239.255.0.1"`, `udp_port = 2237`) and a program that joins groups (GridTracker, JTAlert) hear it at once; a plain address goes to one program only | WSJT-X's Logged ADIF message |
| **HRD Logbook** | Tools → Configure → QSO Forwarding → **UDP Send**: `127.0.0.1`, port `12065` | not tested: HRD doesn't say its format; it works if it sends ADIF or N1MM's XML |

Without UDP (DXKeeper, Logger32, N3FJP, others): a logger that writes an ADIF file as it
goes, or exports one automatically, can be added as a second log file in Settings, next
to the backup; WSJT-X's own `wsjtx_log.adi` is one. UDP has no delivery check: a lost
packet only waits for the next backup.

---

## 7. LoTW integration

LoTW is used in two ways.

### Who uploads to LoTW (public, no login)

ARRL publishes the last upload date of about 236,000 callsigns. DX Agent downloads
that list once a day and adds the station's status to each alert:

| Shown             | Meaning |
|-------------------|---------|
| `LoTW 2026-09-22` | The station uploads to LoTW; this is its last upload |
| `LoTW announced`  | No uploads yet, but the DXpedition listing says QSL via LoTW. New DXpedition calls often upload only after the trip |
| `no LoTW`         | Not a LoTW user and no LoTW announced — a confirmation will need a paper QSL or OQRS |

### LoTW as your log (no ADIF file)

If you don't have an ADIF file (or your logger can't export one), DX Agent can use
LoTW itself as your log: Settings → Station → **Your log** → **LoTW only (no ADIF
file)**, then fill in your LoTW username and password in the LoTW section and click
**Save and apply**.

- It downloads every QSO you uploaded to LoTW, confirmed or not, into
  `data/lotw_log.adi`. The first time it asks LoTW for one year at a time (one query for
  a big log can keep LoTW busy so long that the connection drops), with a progress bar
  on the dashboard. If LoTW is busy or doesn't answer, it tries again by itself every 10
  minutes and continues from the last finished year. After that it only asks for QSOs
  uploaded since the last download, every time it checks LoTW (12 hours by default).
- The confirmations download works the same way, in the background: the agent starts
  watching spots meanwhile.
- Confirmations come from the same download plus the normal confirmations download.
- Your grid square for "near you" is taken from the QSOs (MY_GRIDSQUARE), unless you
  set it in Settings.
- **Limit:** only what you uploaded to LoTW counts. A QSO you logged but never
  uploaded looks "not worked".

### Your confirmations (needs your LoTW password)

DX Agent downloads your confirmed QSLs from LoTW every 12 hours and merges them into
the log index. That keeps "confirmed" current even when Log4OM hasn't synced with
LoTW. The first download fetches everything and can take a few minutes; later ones
only fetch QSLs received since the last download.

**To turn it on:** Settings → LoTW → tick **Download my LoTW confirmations**, type
your LoTW website password, click **Test login**, then **Save and apply**.

Where the password lives:

- It is encrypted with Windows' own protection for your user account (DPAPI) and
  stored in `data/lotw.secret`. Only your Windows account on this PC can decrypt it.
  It is never written to `config.toml`, so the config file is safe to share.
- Alternatively you can set an environment variable once in PowerShell and open a new
  terminal: `setx LOTW_PASSWORD "your-lotw-website-password"`. If both exist, the
  environment variable wins, and the Settings page says so.

What happens without it, or with a wrong one:

- Without a password, DX Agent uses the confirmations already in your ADIF file and
  prints `no LoTW password set` at start.
- With a wrong password, **Test login** says `LoTW rejected this username or password`;
  while running it prints `LoTW did not return ADIF - check username / password` and
  falls back the same way.

### What counts as confirmed

A QSO is confirmed when `LOTW_QSL_RCVD` or `QSL_RCVD` is `Y` or `V` in the log, or
when it comes from the LoTW download. eQSL doesn't count, matching DXCC rules.

---

## 8. How it works

```text
 DX cluster (telnet, w3lpl.net:7373) ─────┐
 RBN CW     (telnet.reversebeacon.net:7000)┼──> parse "DX de ..." line
 RBN FT8/FT4 (telnet.reversebeacon.net:7001)┘          │
                                                       v
                                  band + mode (RBN, comment, or band plan)
                                                       │
                                                       v
                    filters: beacons, own call, bands, modes, ignore list,
                             spotter continent
                                                       │
                                                       v
                         callsign ──> DXCC country (cty.csv)
                                                       │
                                                       v
   ADIF log + LoTW QSLs ──> worked / confirmed index ──> compare ──> reasons
   NG3K/425/Club Log ────> active DXpeditions ───────┘      │
                                                            v
                                         filter by alert_on, cooldown
                                                            │
                                                            v
              web dashboard + console line + pop-up + data/dxagent.db
```

### Step by step

1. **Feeds.** Three telnet connections run at the same time: the DX cluster, RBN CW
   and RBN FT8/FT4. Each one answers the login prompt with your callsign (or sends it
   after 10 s if no prompt is recognised). If a connection drops, or a server sends
   nothing for 5 minutes, it moves to the next host in the list; once every host has
   failed it waits longer between tries, up to 5 minutes.
   **Recent spots:** right after logging in to the cluster, the agent sends `sh/dx 100`
   (DXSpider and AR-Cluster answer with their last ~100 spots, about the last 30
   minutes) and processes those from the last `history_minutes`, oldest first, so the
   list isn't empty at the start. They alert with their own time, never pop up or
   beep, and a spot already seen is not repeated after a reconnect. The RBN doesn't
   answer `sh/dx`, so it starts empty.
2. **Parsing.** Every `DX de SPOTTER: FREQ CALL comment TIME` line becomes a spot.
   Skimmer suffixes like `-#` are removed from the spotter's call.
3. **Band and mode.** The band comes from the frequency. The mode comes from RBN
   (it reports CW, RTTY, FT8, FT4), else from the cluster comment (`cw`, `ssb`,
   `ft8`...), else from what RBN reported for that call within 2 kHz in the last 30
   minutes, else from the frequency: FT8/FT4 frequencies (14074, 10136...) are FT8/FT4,
   the CW and phone parts of the band plan are CW and SSB. Only a spot in a part used by
   several modes (the digital part, 160m and 6m phone) is shown with `?`. Modes are grouped like DXCC:
   **CW**, **PHONE** (SSB, AM, FM) and **DIGI** (FT8, FT4, RTTY, PSK, MFSK...).
4. **Filters.** Beacons, your own call, bands and modes you don't want, calls in
   `ignore`, and (if set) spotters outside your chosen continents are dropped.
5. **Country lookup.** The call is matched to a DXCC entity with AD1C's `cty.csv`:
   exact calls first, then the longest matching prefix. Portable calls such as
   `TF/PD9DX` or `W9HT/VP9` use the location prefix; `/P`, `/M` and `/QRP` are
   ignored; `/MM` and `/AM` have no country.
6. **Your log.** The ADIF file is read into sets of what you have: countries,
   country+band, country+mode, country+band+mode, and calls — each one both
   "worked" and "confirmed". For logged QSOs the ADIF `DXCC` field is used when
   present. The file is checked every 60 seconds and re-read when it changes (about
   3 seconds for 49,000 QSOs). LoTW confirmations are merged in the same way.
7. **Comparison.** The spot is checked against those sets to find the reasons
   (ATNO, new band, new mode, new slot, DXpedition, watchlist, new call). For
   DXpeditions it also works out what working it would add, including confirmations.
8. **Alert.** Reasons not in `alert_on` are dropped, the cooldown is applied, and the
   alert is shown on the dashboard, printed and saved.

When you save settings, the agent stops the feeds, reloads the configuration, the log
and LoTW, and starts the feeds again. The dashboard and the alerts already on screen
stay.

### Data sources

| Source | Used for | Address |
|--------|----------|---------|
| DX cluster (W3LPL, with DXSpider UK and K3LR as fallbacks) | Human spots | `w3lpl.net:7373`, `dxspider.co.uk:7300`, `dx.k3lr.com:23` |
| Reverse Beacon Network | Skimmer spots, CW/RTTY and FT8/FT4 | `telnet.reversebeacon.net:7000` and `:7001` |
| AD1C country files | Callsign → DXCC country | https://www.country-files.com/ |
| NG3K ADXO calendar | Announced DXpeditions | https://www.ng3k.com/Misc/adxo.html |
| 425 DX News calendar | Announced DXpeditions | https://www.425dxn.org/index.php?op=wcal |
| Club Log | DXpeditions uploading now, their logs and OQRS, DXCC Most Wanted | https://clublog.org/expeditions.php |
| ARRL LoTW user activity | Who uploads to LoTW | https://lotw.arrl.org/lotw-user-activity.csv |
| ARRL LoTW report | Your confirmations | https://lotw.arrl.org/lotwuser/lotwreport.adi |
| RBN skimmer list | Where each skimmer is (grid square) | https://www.reversebeacon.net/nodes/ |
| PSKReporter | Who received a station on FT8/FT4 (on request only) | https://pskreporter.info/ |
| PSKReporter live (MQTT) | The same reports as they arrive, for the stations followed (PSKReporter → Live) | https://mqtt.pskreporter.info/ |
| POTA, WWFF spots | Who is activating a park now (every 2 minutes) | https://api.pota.app/spot/activator, https://spots.wwff.co/static/spots.json |
| Natural Earth (world-atlas) | Coasts and country borders for the map | https://www.naturalearthdata.com/ |
| GridTracker DXCC map shapes | DXCC entities, DXCC-only borders and CQ zones for the map | https://gitlab.com/gridtracker.org/gridtracker2 |

---

## 9. Project files and data cache

The code is in `dxagent/`, one module per job. `config.toml`, `config.toml.bak` and `data/`
are in your DX Agent folder, not the program's ([Quick start](#1-quick-start)). Everything
downloaded goes to `data/` and can be deleted at any time; it is downloaded again when needed.

| File                        | Job |
|-----------------------------|-----|
| `config.toml`               | Your settings (written by the Settings page; personal, never shared) |
| `config.example.toml`       | The starting settings for a new installation |
| `start-dx-agent.bat`        | Double-click to start DX Agent on Windows, without a window |
| `start-dx-agent-console.bat` | The same, in a console window that shows everything it prints |
| `start-dx-agent.sh`         | The same two launchers for Linux and macOS (`-console.sh` shows what it prints); they use the package's Python if it brings one (`python/`, or on a Mac `python-arm64/` / `python-x86_64/`) |
| `start-dx-agent.command`    | macOS: double-click in Finder to start it; takes the folder out of quarantine so the bundled Python can run |
| `tools/make_package.py`     | Builds the packages to share: the Windows zip and the Linux and macOS tarballs (Python included, program compiled), plus the source zip with `--source`; never your personal data |
| `config.toml.bak`           | The previous settings, kept on every save |
| `README.md`                 | Short overview |
| `DOCUMENTATION.md`          | This guide |
| `DOCUMENTACION.md`          | This guide in Spanish |
| `dxagent/__main__.py`       | Command line: `run`, `dxpeditions`, `check`, `stats` |
| `dxagent/agent.py`          | Main loop: starts the feeds, filters spots, reloads the log, refreshes DXpeditions and LoTW, applies saved settings |
| `dxagent/config.py`         | Reads `config.toml` with defaults; validates and writes it for the Settings page |
| `dxagent/home.py`           | Where your settings and data live (your DX Agent folder); copies them there from an older version's folder on the first start |
| `dxagent/credentials.py`    | Stores the LoTW password encrypted for your Windows user (on Linux and a Mac, readable only by you) |
| `dxagent/telnet.py`         | Connects to the cluster and RBN, logs in, reconnects |
| `dxagent/spots.py`          | Parses `DX de ...` lines into call, frequency, band, mode, SNR, WPM |
| `dxagent/bands.py`          | Band from frequency, mode groups, band-plan mode guess |
| `dxagent/cty.py`            | Callsign → DXCC country, using `cty.csv` |
| `dxagent/adif.py`           | Reads ADIF files, including UTF-8 names |
| `dxagent/logbook.py`        | Builds the worked and confirmed index from the log and LoTW |
| `dxagent/matcher.py`        | Decides the alert reasons and what a DXpedition would add |
| `dxagent/dxpeditions.py`    | Reads and merges the NG3K, 425 DX News and Club Log DXpedition lists |
| `dxagent/lotw.py`           | LoTW user list, login test and confirmation download |
| `dxagent/alerts.py`         | Console line, colours, pop-up, cooldown, alert history |
| `dxagent/queries.py`        | Station grid, DXpedition list and DXCC standing, shared by the commands and the dashboard |
| `dxagent/reach.py`          | "Near you": who heard each station and how far from you; RBN skimmer list; PSKReporter |
| `dxagent/pskmqtt.py`        | PSKReporter live: an MQTT client for mqtt.pskreporter.info, and the stations followed |
| `dxagent/refinfo.py`        | A POTA park's or SOTA summit's page, for the Station tab |
| `dxagent/geo.py`            | Grid squares and distances |
| `dxagent/onlinelog.py`      | Checks a DXpedition's log on Club Log |
| `dxagent/wsjtx.py`          | Sends a call to WSJT-X (UDP Configure message) |
| `dxagent/activities.py`    | Activations (POTA, SOTA, IOTA...) found in the spots' comments |
| `dxagent/hunting.py`       | Summits, parks, islands... you have worked, band by band |
| `dxagent/livelog.py`       | The QSOs Log4OM saved since its last backup, from its database |
| `dxagent/udplog.py`        | QSOs other programs send over UDP as they log them (N1MM / DXLog.net XML, ADIF, WSJT-X) |
| `dxagent/store.py`         | `data/dxagent.db` (SQLite): DX Agent's own records |
| `dxagent/worldmap.py`       | Land, DXCC borders and DXCC entities for the map view |
| `dxagent/marathon.py`       | CQ DX Marathon: which QSOs count, the countries and zones worked in the period, what a spot would add |
| `dxagent/rig.py`            | Tune: talks to Log4OM, N1MM, DXLog.net, OmniRig, Hamlib, flrig or DXLab |
| `dxagent/tci.py`            | TCI (AetherSDR, ExpertSDR, Thetis): tunes the radio and keeps the alerts on its panadapter |
| `dxagent/web.py`            | Dashboard web server: page, JSON API, live event stream, settings |
| `dxagent/static/index.html` | The dashboard and Settings page (HTML, CSS and JavaScript in one file) |
| `dxagent/static/logo.svg`  | The DX Agent logo (dashboard header and browser tab icon) |
| `tests/test_dxagent.py`     | 117 unit tests (`python -m unittest tests.test_dxagent`) |
| `tests/test_filters.js`     | The alert list's filters, every combination (run by the tests above when Node.js is installed) |

| Data file                                     | Source             | Refreshed      |
|-----------------------------------------------|--------------------|----------------|
| `data/cty.csv`                                | country-files.com  | daily (checked every 6 hours, also while DX Agent runs) |
| `data/dxped-ng3k.json`, `dxped-425dxn.json`, `dxped-clublog.json` | NG3K, 425 DX News, Club Log | every 6 hours |
| `data/clublog-mostwanted.json`                | Club Log Most Wanted | daily        |
| `data/lotw-user-activity.csv`                 | ARRL LoTW          | daily          |
| `data/lotw_qsls.adi` + `data/lotw_state.json` | your LoTW account  | every 12 hours |
| `data/lotw.secret`                            | Settings page      | when you save a password |
| `data/rbn-skimmers.json`                      | RBN skimmer list   | daily          |
| `data/sota-summits.csv`                       | SOTA summits list  | monthly        |
| `data/pota-parks.csv`                         | POTA parks list (the References tab) | monthly        |
| `data/world-countries-50m.json`               | Natural Earth (via world-atlas) | once |
| `data/dxcc-shapes.json`, `data/cq-zones.json` | GridTracker                     | once |
| `data/world-map.json`                         | made from the two above (the finished map) | when they change |
| `data/dxagent.db`                             | DX Agent: alerts (last `keep_days` days), where activators were (last 32 days), your QSOs with them, QSOs over UDP not in your log yet | while it runs |
| `data/cache/log-<CALL>.pickle`                | DX Agent: each callsign's log indexes as last read, so a big log isn't read again at each start. Used only when the log files, the country file, your callsigns, the year and DX Agent's version are all the same; the QSOs since (your logger's database, UDP) are added on top as before. Safe to delete | when the log changes |
| `data/update.json`                            | DX Agent: the latest version on GitHub, asked every hour (`ui.check_updates`) | every hour |
| `data/dx-agent.log`                           | DX Agent (without a window) | while it runs; kept under 5 MB |

### data/dxagent.db

DX Agent's own records are one SQLite file (Python has SQLite built in; nothing to
install). Everything downloaded from elsewhere stays the file it came as: it is replaced
whole and read whole. The tables:

| Table        | What                                                   | Kept |
|--------------|--------------------------------------------------------|------|
| `alerts`     | every alert: time, call, band and the alert as JSON    | `alerts.keep_days` days (7) |
| `activators` | where each activator was (call, time, references)      | 32 days |
| `hunted`     | your QSOs with activators seen on the air (call, program, reference, band, time) | for good |
| `udp_qsos`   | QSOs your programs sent over UDP that your log file doesn't have yet (their ADIF fields) | until the log has them; at most 2,000, 30 days |

Any SQLite tool opens it (DB Browser for SQLite, `sqlite3`), even while DX Agent runs.
Earlier versions wrote `alerts.jsonl` (and `activations*.jsonl`): the first start reads
them in and renames them `*.migrated`; delete those when you like. An alert's `data`, for
example (shortened):

```json
{"utc": "2026-10-13T14:55:04+00:00", "call": "VP2MAW", "freq_khz": 14030.0, "band": "20m",
 "mode": "CW", "mode_group": "CW", "entity": "Montserrat", "dxcc": 96, "reasons": ["dxpedition"],
 "headline": "DXPED HAVE", "need": [], "need_text": "have it: 20m CW confirmed", "lotw": "no LoTW",
 "source": "rbn", "spotter": "WE9V", "snr": 7, "wpm": 27,
 "dxpedition": {"entity": "Montserrat", "end": "2026-10-20", "qsl": "?", "info": "By DK8YY ..."}}
```

---

## 10. Known limitations and troubleshooting

### Known limitations

- **Windows-only features.** Everything else works on Linux with the `-linux` package, and on a Mac with the `-macos` one (pop-ups included, in Notification Center). Not
  available there: Log4OM's live log, and the Log4OM / N1MM / DXLog.net / OmniRig / DXLab "Tune" targets
  (Hamlib, flrig and TCI do work); sending the call to WSJT-X; and the Windows message box when it
  can't start (there the reason goes to `data/dx-agent.log`). On Linux, pop-ups need `plyer`, and
  the Settings "Browse…" button needs tkinter (`python3-tk`); a Mac uses its own Choose file
  window. The LoTW password is encrypted with Windows DPAPI; on Linux and a Mac it is only
  obfuscated, with file permissions `0600`.
- **A QSO you just saved, with Log4OM 2:** DX Agent reads it straight from Log4OM's
  database within about 10 seconds, without waiting for the next ADIF backup
  (`logbook.live`, on by default). It finds the database in Log4OM's `l4ong.status`, opens
  it read-only, and takes only the QSOs after the backup's newest. The alerts already listed
  say what they need now; the DXpeditions, activations and the map's slots refresh too.
  The backup is still the log: it is read whole at start and whenever it changes. Not read:
  Log4OM on MySQL, or a Log4OM log that isn't the one in Settings (the database must hold
  the backup's newest QSO). Confirmations Log4OM gets for older QSOs still wait for the
  backup (or DX Agent's own LoTW download).
- **A station you just worked keeps alerting, with other loggers.** Unless the logger tells
  DX Agent each QSO (over UDP: N1MM Logger+, DXLog.net, WSJT-X...) or writes its ADIF as it
  goes, a new QSO counts only once the log file has it. See "Your QSOs as you log them".
- **LoTW download not yet tested with a real account.** The public LoTW list works;
  the confirmation download and **Test login** follow the ARRL report format but
  haven't been run with a real password yet.
- **Mode from the frequency on cluster spots.** When neither the comment nor RBN
  says the mode, it comes from the frequency (hover the mode to see so). A station
  outside the usual segments (SSB in the CW part, a DXpedition's FT8 on its own
  frequency) can be misread; the digital part is shown with `?`.
- **Announced dates are approximate.** A DXpedition alerts from one day before its start
  date. When its announced end has passed, DX Agent reads its sources (NG3K, 425 DX News, Club
  Log) again right away, before it leaves the list: if one extended it, it stays with the new
  date; if they still say it's over, it leaves the DXpeditions tab and the alerts. While the
  sources can't be read, it stays (they are tried again every 30 minutes), at most two days
  past its end. A new UTC day works out again which ones are on.
- **KG4 calls.** The country file gives the whole KG4 prefix to Guantanamo Bay, but only KG4
  calls with a two-letter suffix (KG4AA) are there: KG4 + one or three letters (KG4OJT) is a
  US call, and DX Agent takes it as such. Guantanamo stations the file lists by name count
  as Guantanamo.
- **The country file** (AD1C's `cty.csv`) is downloaded again when a day old, checked every
  6 hours while DX Agent runs; when it changed, spots use the new one and your log is
  indexed again with it.
- **Portable calls.** Calls like `TF/PD9DX` resolve to the shorter prefix. Rare cases,
  such as `3D2/R` for Rotuma, can resolve to the wrong country.
- **Log4OM's mode goes through OmniRig.** Log4OM's remote control can't change the
  radio's mode, so DX Agent sets it through OmniRig (see Tune the radio): a Log4OM that
  controls the radio another way (Hamlib, its own CAT) gets the frequency only.
- **60 m is off by default.** Select it under **Bands** to watch it; there is no
  band-plan mode guess for 60 m.

### Troubleshooting

| Symptom | Fix |
|---------|-----|
| Yellow banner "DX Agent can't start: No ADIF file found" | Open Settings, fix the log path and use **Test** until it finds the file, then save |
| `config.toml` not found | It is in your DX Agent folder ([Quick start](#1-quick-start)); with `-c`, give the full path |
| Settings won't save, red list under the button | Fix each item listed; nothing is saved until they are all fixed |
| A setting change made things worse | `config.toml.bak` holds the previous settings: copy it over `config.toml` and restart |
| "saved password could not be read" | The password was saved by another Windows user or PC; type it again in Settings |
| `[cluster] ... dropped ...; trying ...` | Normal; it moves to the next cluster in the list by itself |
| Few alerts from the cluster | Normal; the cluster sends a few spots a minute against several per second from RBN, and RBN often reports the same station first |
| No pop-ups for some alerts | Check **Windows pop-ups** in Settings; DXPED HAVE, NEW SLOT and NEW CALL never pop up by default |
| No pop-ups at all | Tick **Show pop-ups** in Settings, and check that Windows notifications are on (Windows Settings → System → Notifications, including "Windows PowerShell", which shows them) |
| Colours show as `[1;93m` in the terminal | Use Windows Terminal or the VS Code terminal |
| `ImportError: bad magic number in 'dxagent'` | You are running the Windows zip on Linux: it is compiled for its own Python. Use the `-linux` package, or the source zip with your own Python |
| `Permission denied` running `./start-dx-agent.sh` | `chmod +x start-dx-agent.sh start-dx-agent-console.sh` (not needed with the `-linux` package) |
| No pop-ups on Linux | `pip install plyer`; the alerts still appear in the dashboard without it |
| Mac: "start-dx-agent.command cannot be opened" / "Apple could not verify…" | It isn't signed by Apple: System Settings > Privacy & Security > **Open Anyway**, or `xattr -dr com.apple.quarantine .` in the dx-agent folder (see Quick start) |
| Wrong or old country data | Delete `data/cty.csv`; it downloads again on the next start |
| `[ui] could not start on port 8073` | Another program (or a second copy of DX Agent) uses that port; stop it or change the dashboard port |
| Dashboard shows "Lost connection to the agent" | `run` stopped or crashed; start it again and the page reconnects by itself |
| Dashboard is empty after a restart | It fills with the cluster's spots from the last 30 minutes as soon as the cluster connects (unless **Recent spots on connecting** is 0). Older alerts are in `data/dxagent.db` (table `alerts`) |
| DXpedition list looks old | Delete the `data/dxped-*.json` files |
| LoTW confirmations look wrong | Delete `data/lotw_qsls.adi` and `data/lotw_state.json` to force a full download |
