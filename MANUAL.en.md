# DX Agent — User manual

*Version 0.3.3 · Made by Diego – LW5HR · Collaborators: Franco – LU2PWY, Juan Manuel – LU9HVR, Gustavo – LU7HW, Daniel – LU1HLH · [Versión en castellano](MANUAL.es.md)*

DX Agent watches the DX cluster and the Reverse Beacon Network (RBN), checks every station
spotted against your log, and tells you when one you still need is on the air: a new country,
a new band or mode, a DXpedition that adds something, or a station on your watchlist. It runs
on your PC and you use it from your web browser.

Contents:

1. [What you need](#1-what-you-need)
2. [Install and open it](#2-install-and-open-it)
3. [First setup](#3-first-setup)
4. [The screen](#4-the-screen)
5. [Alerts](#5-alerts)
6. [Tune the radio and pass the call to your logger](#6-tune-the-radio-and-pass-the-call-to-your-logger)
7. [Let DX Agent see your QSOs at once](#7-let-dx-agent-see-your-qsos-at-once)
8. [DXpeditions](#8-dxpeditions)
9. [Activations: POTA, SOTA, IOTA...](#9-activations-pota-sota-iota)
10. [The map](#10-the-map)
11. [LoTW](#11-lotw)
12. [Updating to a new version](#12-updating-to-a-new-version)
13. [Common problems](#13-common-problems)
14. [Help and contact](#14-help-and-contact)

---

## 1. What you need

- **Windows 10 or 11.** The zip brings everything it needs (Python included): nothing to
  install.
- **An internet connection**, for the spots and the DXpedition, country and LoTW lists.
- **Your log**, in one of two ways:
  - an **ADIF** file your logging program writes: Log4OM's backups, for example, or an ADIF
    export from any other logger;
  - or your **LoTW** account, if you have no ADIF file: DX Agent downloads every QSO you
    uploaded there.
- Optional: your **LoTW password**, so DX Agent knows what you have confirmed even when your
  logger isn't synced with LoTW.

## 2. Install and open it

1. Download the latest `dx-agent-<version>-windows.zip` from the
   [Releases](https://github.com/ddimunzio/dx-agent-download/releases/latest) page.
2. Unzip it into a folder of yours, for example `Documents\dx-agent` (not `Program Files`:
   DX Agent keeps its settings and data inside its own folder).
3. Double-click **start-dx-agent.bat**. The dashboard opens in your browser at
   **http://localhost:8073**.

If Windows shows a security warning the first time (the file came from the internet), choose
the option to run it (**Run**, or **More info → Run anyway**). To avoid it: before unzipping,
right-click the zip → **Properties** → tick **Unblock** → OK.

**How it runs:** without a window. While it runs, double-clicking start-dx-agent.bat again just
opens the dashboard. To close it, use the **Stop** button at the top right of the dashboard. To
see what it is doing, use **start-dx-agent-console.bat**, which runs it in a console window.

You can bookmark the dashboard: `http://localhost:8073`. Only your PC can open it; other
computers on your network can't.

## 3. First setup

The first time, the dashboard shows a yellow bar, because it doesn't know where your log is
yet. Click **Settings** (top right) and fill in:

1. **Station**
   - **Callsign:** yours. It is used to log in to the cluster and the RBN.
   - **Your log:** an ADIF file, or **LoTW only (no ADIF file)**.
   - **ADIF log:** click **Browse…** and pick the file. If you pick a dated Log4OM backup
     (`W1ABC_20261016_124613_backup.adi`), DX Agent turns it into `…/W1ABC_*_backup.adi`, so the
     newest backup is always used. **Test** shows which file it found and how many QSOs it has.
   - **Grid square:** yours (FN31, GF05pr...). Empty = taken from your log. It is used for the
     map, the beam heading and "near you".
2. **LoTW** (optional): your LoTW username and password, then **Test login**. The password is
   stored encrypted for your Windows user, never as plain text.
3. **Alerts:** which alert types you want, on which bands and modes. The defaults are a good
   start.
4. **Radio (Tune button):** the program that controls your radio (see section 6). If you don't
   want DX Agent to move your radio, leave it off.
5. Click **Save and apply**. No restart needed: within seconds it reads your log again and
   connects to the spot sources.

If something is wrong (an invalid callsign, no band picked...), a list appears in red next to
the button, and nothing is saved until it's fixed.

The dashboard follows your browser's language; the **ES/EN** button in the header switches it.

## 4. The screen

![The alert list, with Hide confirmed](images/list.en.png)

**Header:** your callsign; one pill per spot source (green = connected, amber = connecting,
red = down); your log's totals; the **Settings**, **About**, **ES/EN**, **Theme** (light / dark /
auto), **Sound** (a beep for the important alerts, off by default) and **Stop** buttons; the UTC
clock. **About** tells what DX Agent is, who made it and its collaborators, and has the links to
support it, to this manual and to report a problem.

**Filters**, above the list:

- buttons for **alert type**, **band** and **mode** (none ticked = everything shows);
- **Activity:** only POTA, SOTA, IOTA... activations (section 9);
- **Near you only:** only stations heard by someone near you;
- **Hide confirmed:** hides what you already have confirmed on that band and mode;
- a **search** box for a callsign or country;
- **Clear filters** shows everything again. Your browser remembers the filters.

**The alert list:** one row per alert, newest on top, coloured by type. Columns: UTC time,
alert type, callsign, frequency, band, mode, country, **what you need**, LoTW, **near you** and
who heard it. New rows flash, and fade after 10 and 30 minutes.

**On the right, the Station tab:** clicking a row (on the call or on the frequency) opens the
station:

- the band × mode grid of its country: green **C** = confirmed, amber **W** = worked, grey =
  missing;
- whether you worked that exact call, its LoTW status and, for a DXpedition, its details;
- the **beam heading**, large, with a compass, the long path and the distance;
- **who hears it**, the receivers nearest to you first, and **Check PSKReporter (FT8/FT4)**;
- **Show only** this call, **☆ Add to watchlist**, its latest spots, and links to QRZ.com,
  HamQTH and DXWatch.

![The Station tab, with a DXpedition on the air](images/station.en.png)

**The DXpeditions tab:** see section 8.

**List / Map:** to the left of the filters, switches between the list and the map (section 10).

## 5. Alerts

| On the dashboard | What it means | By default |
|------------------|---------------|------------|
| **ATNO** | A country you never worked | on, with a Windows pop-up |
| **DXPED NEED** | An active DXpedition that adds a country, band or mode | on |
| **DXPED CFM** | A DXpedition from a country worked but not confirmed on that band or mode | on |
| **DXPED HAVE** | A DXpedition you already have confirmed on that band and mode (grey) | on |
| **NEW BAND** | Country worked, but not on this band | on |
| **NEW MODE** | Country worked, but not in this mode (CW, phone, digital) | on |
| **WATCHLIST** | A call on your watchlist | on |
| **NEW SLOT** | Country not worked on this band and mode combination | off |
| **NEW CALL** | This exact call isn't in your log (lots of alerts) | off |

- **What you need** says exactly what the station adds: "new band", "17m DIGI never worked",
  "entity not confirmed", or "have it".
- **Confirmed** means LoTW or paper QSL, as in the ARRL's DXCC (eQSL doesn't count).
- **LoTW** says whether the station uploads to LoTW (and its last upload), whether the
  DXpedition announced LoTW, or "no LoTW".
- **Near you:** green **● 732 km · 7 dB** = heard by a receiver within your "near" radius (1,500
  km by default); in grey, the distance to the nearest receiver; **—** = nobody reported it in
  the last 15 minutes.
- The same alert isn't repeated for 15 minutes (change it in Settings).
- **Windows pop-ups** are shown for ATNO only by default; in Settings you can turn them on for
  more types.

**Watchlist:** calls you always want to hear about, even if they add nothing. Wildcards work,
for example `VP8*`. Edit it in Settings or with the ☆ in the Station tab. **Ignore** does the
opposite: those calls never alert.

## 6. Tune the radio and pass the call to your logger

With a radio set up (Settings → **Radio (Tune button)**), clicking an alert's **frequency** has
your program move the radio there, and the station opens on the right. The Station tab also
has a **Tune** button. A message at the top right says whether it worked.

| Program | What to do in the program | Mode too? |
|---------|---------------------------|-----------|
| **Log4OM 2** | Settings → Software integration → Connections → turn on **Remote control** (port 2241) | yes, when Log4OM uses OmniRig |
| **N1MM Logger+** | nothing | no |
| **OmniRig** | pick OmniRig's rig 1 or 2 in DX Agent | yes |
| **Hamlib (rigctld)**, **flrig**, **DXLab Commander** | have the program running | yes |

- **With Log4OM, the callsign is filled in too**, in its QSO window, ready to log. If you typed a
  call there yourself, Log4OM keeps yours.
- **With Log4OM, the mode** changes through OmniRig, the program Log4OM uses to control the
  radio: with **Also set the mode** ticked, DX Agent uses the OmniRig rig that is on-line and
  Log4OM follows it. If Log4OM controls your radio another way, only the frequency changes.
- **Radios that need their data mode for FT8** (an IC-9100 in USB-D, for example): put the
  radio in USB-D once. From then on, clicking FT8/FT4 spots moves only the frequency and leaves
  USB-D on. A phone spot switches to USB or LSB, and the next FT8 spot stays in USB: put USB-D
  back by hand. The same goes for CW: a radio in CW or CW-R keeps it.
- **WSJT-X:** tick **Also send the call to WSJT-X** in DX Agent, and in WSJT-X tick Settings →
  Reporting → **Accept UDP requests**. Clicking an FT8/FT4 spot puts the radio on the FT8/FT4
  frequency, fills in **DX Call** and **DX Grid**, and moves the audio to the station's
  frequency; you click **Enable Tx**.

## 7. Let DX Agent see your QSOs at once

DX Agent reads your log file at start and whenever it changes. But many loggers write that file
only now and then (Log4OM makes a backup every so often), and meanwhile a station you just
worked keeps alerting. To avoid that, your logger can tell DX Agent each QSO as you save it:

- **Log4OM 2: nothing to do.** DX Agent reads the new QSOs straight from Log4OM's database,
  every 10 seconds.
- **Other loggers, over UDP:** in your logger, add the destination `127.0.0.1`, port `12065`,
  next to the ones you already have:

| Logger | Where |
|--------|-------|
| **N1MM Logger+** | Config → Configure Ports… → **Broadcast Data**: tick **Contacts** and add `127.0.0.1:12065` |
| **DXLog.net** | Options → Configure network: QSO broadcast `127.0.0.1:12065`; Options → Broadcast: QSO and **Use N1MM QSO format** |
| **HRD Logbook** | Tools → Configure → QSO Forwarding → **UDP Send**: `127.0.0.1`, port `12065` (not tested with HRD yet) |
| **WSJT-X / JTDX** | usually not needed: your logger gets their QSOs and passes them on to DX Agent |

- **Loggers without UDP** (DXKeeper, Logger32, N3FJP...): if they write an ADIF file as you log
  (or export one automatically), add that file in Settings as a second log file, under the
  main one.

When a new QSO comes in, the alerts already listed update by themselves and stop saying "new
band" once you've worked it.

## 8. DXpeditions

![The DXpeditions tab](images/dxpeditions.en.png)

The **DXpeditions** tab shows the active ones and those starting in the next 30 days,
gathered from NG3K, 425 DX News and Club Log.

- Each card says what it adds: **ATNO**, **Not confirmed**, or how many bands you have (for
  example "3 of 10 bands").
- **ON THE AIR** (green frame) = spotted in the last 30 minutes, with its frequency (click to
  tune).
- **Order:** **On the air** (the ones transmitting first), **Most Wanted** (the most wanted
  country on Club Log first) or **Date**.
- **only needed** hides the ones you have on most bands (those on the air still show).
- When you open a DXpedition that uploads its log to Club Log, DX Agent looks for your call in
  their log: it shows the bands and modes you're in, and warns you if a QSO of yours isn't in
  theirs.

## 9. Activations: POTA, SOTA, IOTA...

![Activity: any activation, with new references](images/activity.en.png)

The **Activity** menu (next to the mode filters) shows only activations: **POTA, SOTA, IOTA,
WWFF, BOTA, COTA** (castles), **LOTA** (lighthouses), **TOTA, ROTA, JOTA, YOTA** and **EME**, with
how many stations are on in the last hour.

- With an activity picked, the list also shows its spots that made no alert (grey **SPOT**
  rows), such as a park in a country you already have.
- Each row shows the reference (`POTA US-5584`, `SOTA W7A/MN-001`...). Hover over it for the
  park's or summit's name, and for summits their altitude and points.
- **NEW REF:** with an activity picked, a summit, park, island or castle you haven't worked
  **on that band** is marked new, even if you have the country confirmed. DX Agent knows what
  you worked from the references in your log and from your QSOs with activators it saw on the
  air (since you started using it).
- On the map, summits are triangles and parks are squares, at their real location.

## 10. The map

![The map: alerts, DXpeditions, the night and your QTH](images/map.en.png)

Click **Map** (next to **List**). The filters work the same as in the list.

- **Dots:** alerts, coloured by type. **Diamonds:** active DXpeditions (green frame = on the
  air). **Triangles and squares:** summits and parks. **Blue dot:** your QTH.
- The shaded area is where it's night (its edge is the grey line).
- Picking a station draws the path from your QTH, with heading and distance, and the receivers
  that heard it.
- Mouse wheel or **+ / −** to zoom, drag to move, **⟲** for the whole world.
- **DXCC** (top left) paints each entity by your award standing: green = confirmed, amber =
  worked, red = missing. Pick Mixed, Phone, CW, Digital, one band, Satellite or the Challenge.
- **WAZ** paints the 40 CQ zones the same way, with its variants (by mode, by band, 5BWAZ...).

## 11. LoTW

- **Who uploads to LoTW:** DX Agent downloads the ARRL's public list every day and shows, in
  each alert, whether the station uploads to LoTW. No password needed.
- **Your confirmations:** with your LoTW username and password (Settings → LoTW → **Download my
  LoTW confirmations**), DX Agent downloads your confirmed QSLs every 12 hours, so "confirmed"
  stays current even if your logger isn't synced. The first download can take a few minutes.
- **LoTW as your log:** with no ADIF file, choose Settings → Station → **Your log** → **LoTW
  only (no ADIF file)**. Only what you uploaded to LoTW counts.

## 12. Updating to a new version

1. Close DX Agent (**Stop** button).
2. Download the new zip and unzip it **into the same folder**, replacing the files.
3. Open it as usual with **start-dx-agent.bat**.

Your settings (`config.toml`) and your data (the `data` folder) aren't in the zip, so they are
kept. If you'd rather use a new folder, copy `config.toml` and the `data` folder from the old
one into it.

## 13. Common problems

| What happens | What to do |
|--------------|------------|
| Yellow bar "No ADIF file found" | Settings → **Browse…** for your log file, **Test** until it finds it, then **Save and apply** |
| The dashboard doesn't open | Open `http://localhost:8073` yourself. If it says the port is taken, another program (or another copy of DX Agent) is using it |
| "Lost connection to DX Agent" | DX Agent closed: open it again with start-dx-agent.bat; the page reconnects by itself |
| A station I just worked keeps alerting | See section 7. Without your logger telling DX Agent, it updates when your logger writes its ADIF file |
| Too many alerts | In Settings → Alerts keep only **ATNO** and **DXpedition**, or untick bands you don't operate |
| Very few cluster alerts | Normal: the cluster sends a few spots a minute, and the RBN usually reports the same station first |
| No Windows pop-ups | Settings → **Show pop-ups**, and in Windows: Settings → System → Notifications (including "Windows PowerShell") |
| Tune doesn't move the radio | Check your program's remote control (section 6) and that it's running; the message at the top right says what failed |
| A settings change made things worse | `config.toml.bak` has the previous settings: copy it over `config.toml` and open DX Agent again |
| "saved password could not be read" | The LoTW password was saved by another Windows user or PC: type it again in Settings |

What DX Agent does is written to `data\dx-agent.log`, inside its folder. If you report a
problem, that file helps a lot.

## 14. Help and contact

![The About window](images/about.en.png)

- **Report a problem or suggest an improvement:** in this repository's
  [Issues](https://github.com/ddimunzio/dx-agent-download/issues). Say what you did, what you
  expected and what happened, and if you can, attach `data\dx-agent.log` (it doesn't contain
  your password).
- **Full technical documentation** (every setting in the configuration file, how it works
  inside): `DOCUMENTATION.md`, in the zip.
- **Support DX Agent:** [Ko-fi](https://ko-fi.com/lw5hr) · [PayPal](https://paypal.me/diegodimunzio1) (also in the dashboard's **About** window)
- **Made by** Diego – LW5HR, with Franco – LU2PWY, Juan Manuel – LU9HVR, Gustavo – LU7HW, Daniel – LU1HLH.
- **License:** free software, [MIT](LICENSE): anyone can use, copy and share it.

73, Diego – LW5HR
