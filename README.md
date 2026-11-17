# ULTRON Orb UI

An Iron Man–inspired holographic orb built with **Next.js**, **Three.js**, and **MediaPipe** hand tracking — control it with your bare hands through your webcam.

> 🔮 This is the open-source **interface** of [ULTRON](https://sagartamang.com/projects/ultron) — my AI that talks in real time and controls Android devices by itself. **[Read the write-up](https://sagartamang.com/projects/ultron)** or **[the X post](https://x.com/sagar_builds/status/2077277583646101921)**

> 📱 **[Watch the demo on Instagram](https://www.instagram.com/p/DayJ17OTwvx/)**

![ULTRON orb UI](docs/screenshot.png)

https://github.com/user-attachments/assets/91578a83-9a27-44e8-84b0-96defcfd7366

## Getting started

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## Windows desktop app (Electron)

Run the interface as a native desktop application:

```bash
npm run desktop:dev
```

### Single installer with bundled API keys

Set `OPENROUTER_API_KEY` and `GROQ_API_KEY` in `.env.local`, then run
`npm run desktop:bundled`. The setup in `dist/tek-setup/` installs its own
`.env` and the MediaPipe hand-tracking assets. The app loads that `.env` on
startup and calls the providers directly. Anyone with the installer can
retrieve the credentials it contains.

### Public GitHub release

For Raspberry Pi OS (64-bit), build the self-contained server installer with
`node scripts/build-pi-installer.cjs`. Copy `dist/ULTRON-Pi-Kurulum.sh` to the
Pi and run it with `bash ULTRON-Pi-Kurulum.sh`. The script prompts for the two
server keys, installs the gateway service, and tests it. Use `--tunnel-only`
later to connect an existing Cloudflare tunnel.

Use the hosted build for a release whose users should start without supplying API
keys. The installer installs `ULTRON-Connection.ultron` alongside the app.
ULTRON checks the file on launch and reads the public HTTPS gateway URL
automatically. The provider credentials stay in the gateway's hosting
environment, and the desktop key dialog is disabled in hosted builds.

1. On the hosting machine, install dependencies with `npm ci` and compile with
   `npm run build`.
2. Set `OPENROUTER_API_KEY`, `GROQ_API_KEY`, and optionally `OPENROUTER_MODEL` as
   private runtime variables. Run `npm run gateway:start` behind a public HTTPS
   reverse proxy. The gateway defaults to loopback port 8080 and exposes
   `GET /healthz` plus the five ULTRON cloud POST endpoints.
3. On Windows, run `npm run desktop:public -- https://api.your-domain.example`
   with the live gateway URL. The build checks `/healthz` and writes the
   shareable installer to `dist/public/`. Upload that installer as the release
   asset. It already includes the connection profile.

Run one gateway process with a persistent quota file. Set
`ULTRON_GATEWAY_QUOTA_FILE` to a writable path on persistent storage and
`ULTRON_GATEWAY_DAILY_LIMIT` to set the shared daily request limit (default
300, with additional route-specific limits). The gateway restricts body size,
requests per minute per address, and concurrent requests. Behind a reverse
proxy, set `ULTRON_GATEWAY_TRUST_PROXY=1` only when the gateway is bound to
loopback and the proxy overwrites `X-Real-IP` with the client's address.
Provider spending and account limits still apply. Cloud features need internet
access. Desktop actions and the local calendar, reminders, and notes run on
the user's computer.

### Private builds and key transfer

The first launch asks Windows for camera and microphone access when those
features are turned on. Create a distributable Windows installer with:

```bash
npm run desktop:build
```

The installer is written to `dist/`. On Windows, open **API Keys** to save your
OpenRouter and Groq keys. ULTRON stores the saved keys in Windows-protected
storage for the current account.

To move the keys to another PC:

1. Choose **API Keys → Export .ultron** and enter a strong, unique passphrase.
2. Copy the encrypted `.ultron` file and the installer to the other PC. Keep the
   passphrase separate from the file.
3. Install ULTRON, choose **API Keys → Import .ultron**, and enter the same
   passphrase. The keys are ready to use after import.

Existing `%APPDATA%\ULTRON\ultron.env` settings are migrated on first launch
when no protected keys exist. The old file still contains its original
contents. Keep it private while verifying the migration.

Chat can use either provider. Desktop speech recognition requires Groq.
Natural speech and source-backed research use OpenRouter and may consume
credits.

Electron uses Groq's free-tier speech-recognition API for voice input, rather
than Chromium's unreliable built-in speech-recognition service. It sends a
short audio clip for each turn. Toggle the conversation with **V** or the voice button.
Local desktop commands still work via the text input when a provider is
unavailable.

## Voice conversation (OpenRouter)

The orb can listen, think and talk back using any model on [OpenRouter](https://openrouter.ai).

1. Copy the example env file and add your key:

   ```bash
   cp .env.example .env.local
   ```

   | Variable | Purpose |
   | --- | --- |
   | `OPENROUTER_API_KEY` | Your key from <https://openrouter.ai/keys> (stays on the server) |
   | `OPENROUTER_MODEL` | Any model id, e.g. `openai/gpt-4o-mini` |

2. Restart `npm run dev`, press **VOICE OFF** (or `V`), allow the microphone and talk.
   You can also type in the box at the bottom. The orb pulses while listening,
   thinking and speaking.

Speech recognition uses the browser's Web Speech API, so use **Chrome, Edge or
Safari** (Firefox has no speech recognition; typing still works there).

## Controls

### Mouse / touch

| Input | Action |
| --- | --- |
| Drag | Spin the orb |
| Scroll / pinch | Zoom in & out |

### Hand gestures (webcam)

Click **GESTURES OFF** (or press `G`) and allow camera access, then:

| Gesture | Action |
| --- | --- |
| Pinch (thumb + index) one hand and move it | Spin the orb |
| Pinch with **both** hands, spread apart / bring together | Zoom in / out |

### Keyboard

| Key | Action |
| --- | --- |
| `V` | Toggle voice conversation |
| `Esc` | Interrupt a spoken reply |
| `G` | Toggle hand gestures |
| `R` | Reset the view |
| `+` / `−` | Zoom in / out |

## How it works

- **`lib/orbScene.ts`** — the Three.js scene: layered wireframe shells, a spiral
  inner core, floating code-text sprites, orbiting debris, dust particles, scan
  rings, and a bloom + chromatic-aberration post-processing stack.
- **`lib/handTracker.ts`** — MediaPipe HandLandmarker running on the webcam
  feed. Pinch detection with hysteresis: one pinched hand spins the orb, two
  pinched hands zoom by spreading apart or together.
- **`lib/voiceAgent.ts`** — the voice loop: speech recognition → `/api/chat` →
  spoken reply → listen again.
- **`app/api/chat/route.ts`** — server-side proxy to OpenRouter (keeps your API
  key out of the browser).
- **`components/JarvisOrb.tsx`** — the HUD and glue between the scene, the
  tracker, the voice agent and your inputs.

## Windows commands

ULTRON resolves application names against the Windows Start menu, installed shortcuts, and registered app paths. It also recognizes common aliases and asks for a more specific name if several apps match.

| Task | Example |
| --- | --- |
| Open an application | Open Google Chrome. Launch OBS. Open Visual Studio Code. |
| Close an application | Close Spotify. Quit Discord. |
| Control media | Turn the volume down. Pause music. Next track. |
| Capture the screen | Take a screenshot. |
| Check hardware | Show system status. |
| Open a folder | Open Downloads. Open Documents. |
| Search the web | Search on Wikipedia for Ada Lovelace. Look up electric cars on Google. Play ambient music on YouTube. |
| Organize the Desktop | Organize my desktop into folders without deleting anything. |
| Create a blank text file | Create a blank file called notes.txt. |
| Write and save a document | Write me an article about electric cars and save it as "EV Guide.md". |
| Edit a text file | Edit "notes.txt" to fix the grammar. |
| Append exact text | Append "Bring a charger." to notes.txt. |
| Use the clipboard | Read my clipboard. Clear the clipboard. |
| Write a note | Take a note: finish the build. |
| Open settings | Open Bluetooth settings. |
| Control power | Lock the computer. Put the computer to sleep. |

Desktop organization moves recognized files from the Desktop into category folders
under `ULTRON Organized`. Existing folders, shortcuts, and name collisions are
left in place. File commands support UTF-8 `.txt` and `.md` files on the Desktop.
Edits save a copy of the previous content beside the file. Drafting and
instruction-based revision use the configured chat service. Revision sends the
selected file's text to that service.

## Daily assistant

| Task | Example |
| --- | --- |
| Timer | Set a timer for five minutes. |
| Reminder | Remind me to call Alex tomorrow at 10 AM. |
| Calendar | Schedule a project review Friday at 3 PM. What's on my schedule today? What about tomorrow? |
| Notes | Take a note: the revised deadline is Friday. Find my notes about the deadline. |
| Tasks | Add send the invoice to my list. What's left? Mark the invoice as done. |
| Daily briefing | Give me my morning briefing. |
| Web research | What happened in tech today? Research the latest Node.js releases. |

The Windows app stores the calendar, tasks, notes, and reminders locally. Closing
its window keeps reminders running in the system tray and releases the microphone
and camera. Use **Quit ULTRON** in the tray to exit. Due reminders are checked
again on launch or when Windows wakes up. Calendar entries use the Windows local
time zone.

Web research uses the OpenRouter key and can use OpenRouter credits. ULTRON reads
the summary aloud and displays source links. While voice mode is on, say
**Wait** or **Hold on** to interrupt, then continue with your question. **Esc**
also stops playback.

## License

MIT
