# Jackdaw Inputs

Live controller overlay for streaming. The agent reads your gamepad in the background, even while a
fullscreen game has focus, and serves a browser page that draws the pad as you press it. Point an OBS
browser source at that page and your inputs are on stream.

![The overlay showing a DualSense with the face buttons and left stick active](docs/images/hero-overlay.png)

Windows 10 and 11, x64.

## Downloads

Everything is on the [latest release](../../releases/latest).

| File | Use it when |
| --- | --- |
| `JackdawInputs.appinstaller` | You want it installed and kept up to date. This is the one most people want. |
| `JackdawInputs-x64.msix` | You want a one-off install with no update channel. |
| `JackdawInputs-<version>-win-x64.zip` | You want to run it from a folder without installing anything. |
| `checksums.txt` | You want to check the download by hand. See below. |

## Installing

### With the update channel

Download `JackdawInputs.appinstaller` and open it. Windows shows you who published it before anything
runs, and installs the app once you confirm.

![The App Installer window naming The Omen Den L.L.C. as the publisher](docs/images/install-appinstaller.png)

This route registers an update channel. On later launches Windows checks whether a newer version has
been published and fetches it in the background, and Jackdaw Inputs tells you when one is waiting.

### Without it

`JackdawInputs-x64.msix` installs the same application and skips the update channel entirely. You get
the Start Menu entry, the firewall rule and the uninstall entry, but nothing will tell you when a new
version exists.

### From the zip

Unpack it anywhere and run `JackdawInputs.exe`. Nothing is written outside the folder except the
connection key and a log directory.

Two things the installed versions do that the zip does not. There is no Start Menu entry, so you
launch it from wherever you unpacked it. More importantly there is no firewall rule, so the first
time you run it Windows asks whether to allow it through the firewall. Answer that prompt. If you
dismiss it, the agent still runs and still works on the machine it is on, but a second PC on your
network will not be able to reach it, and nothing will say why.

## Checking your download

Every binary is signed by The Omen Den L.L.C. through [Azure Artifact Signing](https://learn.microsoft.com/en-us/azure/artifact-signing/overview), and Windows checks the
signature for you. The installer names the publisher before anything executes, and it refuses a
package whose signature does not validate. There is no warning to click through.

To check by hand:

```powershell
Get-AuthenticodeSignature .\JackdawInputs-x64.msix | Format-List Status, SignerCertificate
```

`Status` should read `Valid` and the subject should name The Omen Den L.L.C.

`checksums.txt` carries SHA-256 hashes for each file. Treat it as a convenience rather than as the
thing keeping you safe: anyone able to tamper with a download could tamper with the hashes sitting
next to it. The signature is the part that cannot be forged.

```powershell
Get-FileHash -Algorithm SHA256 .\JackdawInputs-x64.msix
```

## First run

Start Jackdaw Inputs. It opens a terminal window and prints three things: the address of the overlay
page, the connection key, and where it is writing logs.

![The agent console after startup, showing the overlay URL and the connection key](docs/images/first-run-console.png)

Leave that window open. Closing it stops the agent.

Open the overlay address in a browser. You land on the harness page, which shows every controller the
agent can currently see and updates as you press buttons. If your pad shows up here, the capture side
is working and everything that follows is about getting the picture into OBS.

![The harness page showing a connected controller responding to input](docs/images/harness-page.png)

The connection key is stored in Windows Credential Manager and stays the same between runs, so the
URL you set up in OBS today still works tomorrow. Run the agent with `--new-key` if you ever want to
change it, which also means updating the URL in OBS.

## Putting it on stream

Go to the Overlay setup page in the sidebar. Paste in the key from the console, choose whether you
want an outline and how the pad should scale, and the page builds the URL for you.

![The Overlay setup page with a key entered and the browser source URL built](docs/images/overlay-setup.png)

Copy that URL. In OBS, add a Browser source and paste it into the URL field. Set the width and height
to suit the pad you use, and tick "Shutdown source when not visible" if you want the agent to stop
sending frames while the source is hidden.

![The OBS browser source properties dialog with the overlay URL pasted in](docs/images/obs-browser-source.png)

The overlay draws on a transparent background, so it composites straight over game footage with no
chroma key.

![The overlay composited over gameplay in the OBS preview](docs/images/obs-composited.png)

## Two-PC setups

If you game on one machine and stream from another, run Jackdaw Inputs on the gaming PC, where the
controller is, and point OBS on the streaming PC at the gaming PC's address.

The agent prints that address on startup. Use it rather than `localhost`, which from the streaming
PC would mean the streaming PC. The Overlay setup page warns you when it thinks you are about to make
that mistake.

This is the case the firewall rule exists for. The installed versions declare it; the zip relies on
you answering the Windows prompt on first run.

## Command line

Running the application with no arguments is the same as `--serve` with defaults, which is what the
Start Menu entry does.

| Flag | Effect |
| --- | --- |
| `--serve` | Serve the overlay and the frame stream. The default. |
| `--port <n>` | Listen on a different port. Defaults to 8787. |
| `--bind <ip>` | Listen on one interface instead of all of them. |
| `--key <k>` | Use a specific key rather than the stored one. |
| `--new-key` | Replace the stored key with a fresh one. |
| `--allow-origin <origin>` | Accept a browser page served from somewhere other than the agent. |
| `--cert <pkcs12>` | Serve over https and wss using this certificate. |
| `--version` | Print the version and exit. |
| `--help` | Print usage and exit. |

If you change the port, note that the firewall rule the installer creates names 8787 specifically. A
different port needs a rule of your own.

## Controllers

The overlay draws artwork matched to the pad it sees: PlayStation 4, PlayStation 5, Xbox, Nintendo
Switch, and Steam Deck. Anything it does not recognise falls back to a generic pad rather than
failing, so an unusual controller still shows its inputs.

Xbox 360, Xbox One and Xbox Series pads all draw as one family. SDL reports the same type for all
three, so there is nothing to tell them apart by.

## Documentation

Full documentation is at [jackdaw.corvid.online](https://jackdaw.corvid.online).

## Reporting a problem

Open an issue on this repository. Two things make a report much easier to act on: the version, which
`JackdawInputs --version` prints, and the log file, whose location the agent prints on startup. The
log opens with the version and carries the startup sequence, so it usually answers both questions at
once.

## About this repository

This repository holds released artifacts only. It carries no source code, and its releases are
published by the build that compiled and signed them.

Artwork credits ship inside the application and are shown on its Credits page.
