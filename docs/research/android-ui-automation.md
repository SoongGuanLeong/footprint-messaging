# Which Android mechanisms can drive WhatsApp Export Chat unattended

Research findings for issue [#4](https://github.com/SoongGuanLeong/footprint-messaging/issues/4),
part of map [#1](https://github.com/SoongGuanLeong/footprint-messaging/issues/1).

Environment under test, all measured on the machine that will run this:

| Thing | Value | How established |
|---|---|---|
| Emulator | `emulator-5554`, AVD `small_phone` | VERIFIED |
| Image | `system-images/android-36.1/google_apis_playstore/x86_64`, `target=android-36.1` | VERIFIED (`config.ini`) |
| Device | Android 16, `ro.build.version.sdk=36`, `x86_64`, `sdk_gphone64_x86_64` | VERIFIED (`getprop`) |
| Emulator binary | 37.1.11.0 | VERIFIED (`emulator -version`) |
| WhatsApp | `com.whatsapp` 2.26.37.73, `minSdk=23`, **`targetSdk=36`** | VERIFIED (`dumpsys package`) |
| Root / debug | `ro.build.type=user`, `ro.secure=1`, no `su`, `run-as` refuses | VERIFIED |

Evidence labels used throughout:

- **VERIFIED** - I read it in a primary source (official docs, AOSP, the app's own
  listing, or the tool's own `--help`) or reproduced it myself on this machine.
- **INFERRED** - follows from a verified fact plus documented behaviour, but I did
  not run it.
- **UNVERIFIED** - I could not establish it. Stated as a gap, not smoothed over.

## Methodological caveat, read this first

This machine is shared. While I worked, a second agent session was driving the same
emulator concurrently. That turned out to be useful rather than merely annoying,
because it produced one of the most important findings here (the exclusive
`UiAutomation` connection, section 4). But it means some of my observations were
interleaved with someone else's taps. Where that happened I say so, and I re-ran
the observation in a quiet window before treating it as VERIFIED.

I did not log out of WhatsApp, reinstall WhatsApp, wipe the AVD, enable root, or send
any message. I did make one unintended change and corrected it immediately: while
probing whether the shell can write secure settings I set
`secure/accessibility_enabled` from `1` to `0` and restored it to `1` in the next
command. No accessibility service was bound at any point during this work
(`dumpsys accessibility` showed `Bound services:{}` and
`Enabled services:{}` throughout), so nothing was left in a different state.

## Headline conclusions

1. **There is no non-UI path. The UI is the only supported route.** Every provider
   WhatsApp exposes is either not exported or guarded by a WhatsApp-signed
   permission. `adb backup` is dead twice over. VERIFIED.
2. **`adb shell uiautomator dump` + `adb shell input tap` genuinely works** on this
   non-rooted, Google Play, API 36 image, and the WhatsApp chat list is fully
   legible in the dump, including exact chat names and pixel bounds. VERIFIED.
3. **The classic coordinate race is real and it fails silently.** I reproduced it:
   dumped the list, scrolled, tapped the stale coordinate, and a different chat
   opened. For an archive whose raw artifact is evidence, silent wrong-target
   capture is the worst possible failure mode. VERIFIED.
4. **`uiautomator dump` holds an exclusive system connection.** A second concurrent
   client makes it fail outright (3/10 succeeded under contention, 8/8 when quiet).
   Any unattended driver must be the only `UiAutomation` client, and must retry.
   VERIFIED.
5. **The with/without-media prompt is conditional, not universal.** It did not
   appear in any of four export runs, and the reason is that the chat's media
   screen reported "No media found". I could not test a chat that does contain
   media, so **the prompt's structure is UNVERIFIED**. This is the single biggest
   open gap for a with-media capture.
6. **Export does not put the zip in `/sdcard/Download` by itself.** The archive goes
   to app-private storage and is offered through the system share sheet. Landing it
   in Downloads needs a *third-party file manager's* UI, which is a separate
   fragility. VERIFIED.
7. **The accessibility route is credible but is not better than adb, and it costs
   strictly more.** It can do everything needed here, but consent is a manual
   Settings action by design, Android 14+ silently revokes it on every app update
   with no workaround, and it puts a second app in the maintenance path of an app
   that Google ships layout changes to weekly.
8. **A fully unattended capture is buildable, but I do not recommend shipping it as
   the default.** See "Bottom line".

## 1. Off-the-shelf UI automation on the emulator

### Automate

**The repository named in the ticket does not exist.** `github.com/taki9832/Automate`
returns 404, and so does the GitHub *user* `taki9832` (VERIFIED, `gh api`). The
app is real; the open-source repo premise is not. What exists:

- Play Store package `com.llamalab.automate`, by Llamalab. Closed source, Play Store
  only. VERIFIED.
- Play Store listing states: "This app uses the Accessibility API to provide features
  that interact with the UI, intercept key presses, take screenshots, read 'toast'
  messages, determine foreground app and capture fingerprint gestures." It also uses
  the Device Administrator permission. VERIFIED (listing text).
- "Updated on Aug 13, 2026", so it is actively maintained against current Android.
  VERIFIED (listing text).
- Free. VERIFIED, weakly: a MacroDroid Play review from Aug 23 2026 says "Downloaded
  Automate and it seems to work for free." UNVERIFIED as a price fact.

Against the four questions asked:

- **Does it work on API 36 / Android 16 x86_64?** UNVERIFIED. It is maintained and
  Play Store listings do not show a device exclusion, and its whole interaction model
  is the same public Accessibility API that I verified working on this image via
  `uiautomator`. I did not install it, so this is an inference, not a result.
- **Root / sideload / permission route?** No root needed. It is a normal Play Store
  install, and it needs **Accessibility** granted, which is the same human-in-Settings
  consent discussed in section 2. A manual Accessibility grant is therefore
  unavoidable for Automate. VERIFIED for the permission requirement (its own listing
  text and the Automate user group answer below); INFERRED for the no-root claim.
- **How does it hold up against a Play Store app that changes layout on update?**
  This is its real weakness. Automate selects UI elements the way the published
  community flow does: by **UI element text**, not by resource id. The Automate
  Community flow "Automatic Whatsapp Message" says: "To change recipient go to the
  1st interact click box and change UI element text to whatever contact you want."
  UNVERIFIED (community page, but it is Llamalab's own site). So the same fragility I
  measured against `adb` applies to Automate, plus a second risk: Automate flows are
  configured through a GUI, so a WhatsApp redesign breaks them invisibly and the
  breakage is discovered only when a run silently does the wrong thing.
- **Any published evidence of it driving WhatsApp's export flow?** No. The nearest
  published evidence is a flow that drives WhatsApp to *send a message*, from 2017.
  UNVERIFIED, and negative: I found no flow, gist, or issue that drives Export chat.

### Alternatives

| Tool | Route | Permission | Fragility | Licence |
|---|---|---|---|---|
| **Automate** | Play Store | Accessibility + Device Admin | Text-based selectors; GUI-configured flows break silently | Proprietary, free |
| **Tasker + AutoInput** | Play Store | Accessibility (AutoInput) | Most powerful and most scriptable; also most moving parts | Proprietary, paid one-time (~$4.49, 7-day trial), AutoInput free. VERIFIED via Play/Wikipedia |
| **MacroDroid** | Play Store | Accessibility for UI actions | Simplest UI; free tier is ad-supported and capped at 5 macros | Proprietary, freemium |
| **AutoJs6** | Build from source | Accessibility | Effectively abandoned | MPL-2.0, but 2 stars and a single "code first commit" in Jan 2025. VERIFIED via `gh api` |
| **DroidWright** | Build from source | Accessibility + overlay | Young, 23 stars, last push Nov 2025. MIT | VERIFIED via `gh api` |
| **AccessibilityAction** | Build from source | Accessibility | Text-based scripting, but needs Tasker or MacroDroid as host; **no licence file** | VERIFIED via `gh api` (17 stars, pushed 2026-09-25) |

None of these is a better deal than the `adb` route for this project, and one of them
is a genuinely useful negative result: **no maintained open-source project already
does this well enough that writing one is unjustified, and equally, none makes
writing one unnecessary.** The honest position is that Automate, Tasker and MacroDroid
are all *worse* than `adb` for this job, because they add an app that must be
configured and re-validated whenever WhatsApp changes, while `adb` needs no
on-device automation app at all and is already proven end to end for the pull.

## 2. The accessibility service route

### What a service can do

From the AOSP source for `AccessibilityService` (primary source, `frameworks/base`,
`core/java/android/accessibilityservice/AccessibilityService.java`):

- It receives `AccessibilityEvent` callbacks for state transitions, and can "request
  the capability for querying the content of the active window" by setting
  `canRetrieveWindowContent="true"`. VERIFIED.
- That window content arrives as an `AccessibilityNodeInfo` tree, which supports
  finding nodes by text, by view id, by class, and by traversal, plus acting on them.
  VERIFIED.
- It can dispatch gestures, subject to `canPerformGestures`, and can observe which app
  is in the foreground. VERIFIED.
- It requires the `android.permission.BIND_ACCESSIBILITY_SERVICE` permission "to
  ensure that only the system can bind to it". VERIFIED.

So structurally, a custom accessibility service **can** do this job: enumerate the
chat list, match a chat by name, tap it, walk the overflow menu, tap Export chat, and
answer the media prompt. Nothing about WhatsApp's chat list or export screens is
hidden from it.

### What it structurally cannot do

- **Secure windows.** `FLAG_SECURE` content is out of reach. The API even has a
  dedicated failure code, `ERROR_TAKE_SCREENSHOT_SECURE_WINDOW`, documented as "the
  status of taking screenshot is failure and the reason is the window contains secure
  content". VERIFIED (AOSP source and API reference).
- **Password or biometric entry.** Same class of protection. A service cannot enter
  credentials into a secure window.
- **Only the active window.** "For security purposes an accessibility service can
  retrieve only the content of the currently active window." VERIFIED (AOSP source).

**None of these limits bites in this project.** WhatsApp's chat list, its overflow
menus and its export screens are ordinary, non-secure, accessibility-visible windows.
I proved the same content is visible from the `uiautomator` side, which reads the
identical accessibility tree: the whole 140-node chat list, with every chat name and
every pixel bound, came out of one dump. VERIFIED.

### So is a custom service credible here?

Yes, technically. But it is **strictly worse than the `adb` route at the same job**,
for four reasons that are not about capability:

1. **It adds an app to the critical path.** The `adb` route needs no automation app
   installed on the emulator at all.
2. **It must be built, signed and installed by us.** VERIFIED as a real cost: a
   service needs a manifest `<service>` with the `BIND_ACCESSIBILITY_SERVICE`
   permission, an intent filter, and an `accessibilityservice` metadata XML.
3. **It must be maintained against a Play Store app that changes without notice.**
   WhatsApp is not a pinned dependency. It updated itself on this device
   (`lastUpdateTime=2026-09-25 23:33:16`) without us asking. Any selector we encode
   can break on any weekly update, and the failure is silent.
4. **It is harder to debug from the host.** With `adb` the whole state of the world is
   `adb shell uiautomator dump` plus logcat, and a driver script is a file in git. An
   on-device service is a second app to inspect through the UI it is automating.

### The consent problem, which is the real cost

**A one-time manual grant is unavoidable by design.** The AOSP source is explicit:

> The lifecycle of an accessibility service is managed exclusively by the system...
> **Starting an accessibility service is triggered exclusively by the user explicitly
> turning the service on in device settings.**

and the service stops "when the user turns it off in device settings or when it calls
`disableSelf()`". VERIFIED (AOSP `AccessibilityService.java`). The same is true in
practice for Automate: its user group records that the grant "has been enabled
manually by the user on the accessibility settings screen, by clicking 'Automate'
under the 'Downloaded apps' header". UNVERIFIED (forum), but consistent with the
source.

**However, the grant is bootstrap-able from the host, which changes the picture.** I
verified on the device that the `shell` UID holds `WRITE_SECURE_SETTINGS`:

```
$ adb shell dumpsys package com.android.shell | grep WRITE_SECURE
      android.permission.WRITE_SECURE_SETTINGS: granted=true
```

VERIFIED. And Key Mapper's own documentation gives the exact incantation:

```
adb shell settings put secure enabled_accessibility_services \
  <package>/<package>.<ServiceClass>
```

VERIFIED (Key Mapper docs, the project's own primary source). I confirmed the write
is *accepted* on this device (exit code 0), though my test component did not read back
because it is not installed, so whether a *real* service then binds without the
Settings dialog is **INFERRED, not VERIFIED**. The honest reading: the accessibility
grant is a one-time bootstrap cost, not a permanent blocker. It is INFERRED that it
can be done from the host; it is not something I would stake an unattended pipeline on
without testing it end to end.

**Survival across lifecycle events.** The grant lives in `Settings.Secure`, per user,
in `/data/system/users/0/settings_secure.xml`.

| Event | Does the grant survive? | Label |
|---|---|---|
| Emulator reboot | Yes. It is a Settings value, not process state, and it is read at boot by `AccessibilityManagerService`. | INFERRED (not tested: rebooting would have destroyed the concurrent session's in-flight work) |
| App update | **No, on Android 14+.** "the system will automatically disable all restricted permissions when an app is updated, including the Accessibility service API permission", and "there is no workaround". | VERIFIED-ish: official Google Play Developer Community, answered by a Product Expert pointing at `developer.android.com/about/versions/14/summary`. The forum is semi-official; I would call the underlying rule INFERRED-from-official-community rather than VERIFIED from the doc itself, because I did not read the Android 14 behaviour-change page directly. |
| App reinstall | No. Key Mapper's docs state of the related grant: "These permissions persist across reboots but need to be granted again if the app is reinstalled." | INFERRED (generalisation from a documented sibling permission) |
| Snapshot restore | Yes, if the snapshot includes userdata, which quick-boot snapshots do. | INFERRED |
| Emulator wipe | **No.** It is in userdata. | INFERRED |

**The Android 14 rule is the decisive one for maintenance.** An accessibility-based
tool that we install ourselves is re-granted by hand *every time we update it*. If we
never update it, we are running an unmaintained app against a WhatsApp that updates
weekly. That is a worse position than the `adb` route, which has no such coupling.

### Verdict on the accessibility route

A custom accessibility service is **credible but unjustified** for this project. It
can do the job, it is not blocked by any WhatsApp-side protection, and its consent
can probably be bootstrapped from the host. But it buys nothing over `adb`, costs an
app to build and maintain, and adds a grant that Android 14+ revokes on every update.
If someone wants to *evaluate* it, the honest framing is: it is an alternative
implementation of the same `UiAutomation`-class tree access that `uiautomator dump`
already gives us for free, and `adb` is the one with no on-device component.

## 3. The dependency-free route: `uiautomator dump` plus `input`

### Does it work on this image, and what is in the output?

Yes, on a non-rooted Google Play API 36 image. VERIFIED. A clean dump of the chat list
produced 140 nodes, all `package="com.whatsapp"`, 53535 bytes, and contained:

- `com.whatsapp:id/conversation_list_view_host` wrapping a `RecyclerView` at
  `android:id/list`
- one `com.whatsapp:id/contact_row_container` per chat, 152px tall, each containing
  `com.whatsapp:id/conversations_row_contact_name` with the **exact chat name as
  text**, plus `conversations_row_date` and a `single_msg_tv` preview
- `com.whatsapp:id/my_search_bar` with `search_text` = "Ask Meta AI or Search"
- `com.whatsapp:id/menuitem_overflow` at bounds `[640,56][720,152]`
- `com.whatsapp:id/status_list`, a *separate* horizontal list that also uses
  `contact_name` for status entries

That last one is a real trap: `contact_name` appears in the status row, so a naive
match on resource id can hit "My status" instead of a chat. VERIFIED.

The chat-list viewport held **5 rows** at a time (VERIFIED), so a chat that is not on
screen simply is not in the dump.

### Is coordinate-based input reliable, and what about the scroll race?

`adb shell input tap x y` works, and `input text`, `input swipe` and `input keyevent`
all work. VERIFIED.

**The race is real and I reproduced it.** VERIFIED, with numbers:

1. Dump. Target `[chat B]` at centre **(323, 459)**. Visible rows at
   that moment: `[chat B]`, `[chat A]`, `[chat C]`, `WhatsApp`,
   `[chat D]`.
2. `input swipe 360 900 360 400 250` to scroll.
3. Dump again. The viewport now holds a **completely different** set:
   `[chat D]`, `[chat E]`, `[chat F]`, `[chat G]`,
   `[chat H]`, `[chat I]`. No row within 40px of y=459.
4. Tap the **stale** (323, 459).
5. Result: **opened `[chat E]`**, a chat that was not even on screen at step 1.

Characterised honestly: this is not a crash and not an error. `input tap` dutifully
taps whatever is at those pixels. The failure is **silent and produces a valid-looking
capture of the wrong conversation**, and it does so precisely when the list is
scrolled between the dump and the tap, which is exactly what a "find a chat deep in
the list" implementation must do.

**The mitigation, which I did verify works, is to never use a stale coordinate.** My
driver re-dumped and re-resolved the target from the live hierarchy immediately before
every single tap, and verified the result by reading
`com.whatsapp:id/conversation_contact_name` after opening the chat. With that
discipline the flow ran repeatedly and correctly. The rule for the downstream design
is therefore: **re-dump before every tap, and assert the opened chat's name before
proceeding.** That converts a silent wrong-target into a loud failure, which is
acceptable for an archive.

### Can `input` dismiss the with-media / without-media dialog?

**I could not establish this, and the reason is itself a finding.** See section 5.

### How a chooser dialog should be targeted

The system share sheet is ordinary system UI and is fully visible in a dump. VERIFIED
by direct capture:

```
(360,640) FrameLayout  android:id/contentPanel
(136,589) TextView     com.android.intentresolver:id/headline                 'Sharing 1 file'
(278,701) TextView     com.android.intentresolver:id/content_preview_filename  'WhatsApp Chat with [chat A]'
(360,1046) GridView    com.android.intentresolver:id/resolver_list
(360,917)  LinearLayout com.android.intentresolver:id/item   -> 'My Drive'
...
(504,1180) TextView     android:id/text1                                   'WhatsApp'
```

So a chooser should be targeted by the **stable system resource id** of the item
container (`com.android.intentresolver:id/item`) scoped to the `resolver_list`, and
the resulting file should be asserted via `content_preview_filename` rather than by
assuming a name. Prefer `resolver_list` over `suggested_apps_container`: the latter
holds *contacts* and changes with recency, which is a moving target.

## 4. Non-UI paths

### Does WhatsApp expose an intent, provider or share target for export or reading chats?

**No. VERIFIED, by direct probing rather than by reading a manifest.**

Enumerating WhatsApp's registered content providers and trying to read each one as the
`shell` UID:

| Authority | Result |
|---|---|
| `com.whatsapp.orbitmessages` | Exported, but `SecurityException`: "requires com.whatsapp.orbit.permission.MESSAGES" |
| `com.whatsapp.securefileprovider` | "not exported from UID 10229" |
| `com.whatsapp.provider.migrate.ios` | "Could not find provider" from shell |

The `OrbitMessagesProvider` is the only exported one, and it is gated by a
**WhatsApp-signed permission**. Holding it would require signing as WhatsApp, which is
a patched or impersonated APK and is excluded by the project's constraints. Hard dead
end. VERIFIED.

Intent filters: `android.intent.action.SEND` and `SEND_MULTIPLE` resolve **only** to
`com.whatsapp.contact.ui.picker.ExternalShareAlias`, which is sharing content *into*
WhatsApp, not out. `SENDTO` is the telephony path. The one export-looking component,
`com.whatsapp.migration.export.ui.ExportMigrationActivity`, answers
`com.whatsapp.intent.action.migrate.ios.START`, which is the iOS device-migration
flow, not chat export, and takes no chat identifier. VERIFIED.

**So: WhatsApp exposes no official non-UI path to export or read chats. The supported
export path is the UI and nothing else.** That is the conclusion the downstream
decision depends on, and it is now established rather than assumed.

### `adb backup` on Android 16

**Dead, for two independent reasons.**

1. **Deprecated in the tool.** `adb backup` prints
   `WARNING: adb backup is deprecated and may be removed in a future release`, and then
   blocks on an on-device confirmation: "Now unlock your device and confirm the backup
   operation...". VERIFIED, run on this device. It never proceeded unattended.
2. **Functionally empty for any modern app.** The official Android 12 behaviour-change
   documentation states: "For apps that target Android 12 (API level 31) or higher,
   when a user runs the `adb backup` command, **app data is excluded** from any other
   system data that is exported from the device." VERIFIED (official docs). WhatsApp
   here is `targetSdk=36` (VERIFIED), so its data is excluded.

Note that `ALLOW_BACKUP` **is** in WhatsApp's package flags (VERIFIED), which is a
deliberate red herring: the flag being set does not help, because the API-level
exclusion overrides it. On a non-rooted device, `adb backup` could never have read app
private storage anyway.

For completeness: `allowBackup` itself is deprecated from Android 12 and may be
removed, and Google publicly warned in 2019 that `adb backup`/`restore` might be
removed entirely. UNVERIFIED (news reporting, not a doc I read directly), but the
deprecation warning I observed first-hand points the same way.

### Any other official mechanism

None found. WhatsApp's related official surfaces are "back up your chat history" and
"transfer your chat history", both of which are cloud or device-migration flows rather
than a host-readable export, and cloud relay is out of scope for this project by the
map's own decision. The third-party tools that do read WhatsApp databases (for example
the `whatsapp-chat-exporter` family) all read the private database or an Android
backup, both of which the project explicitly excludes.

## 5. The confirmation step

**This is the weakest link in the whole chain, and the reason is not what the ticket
assumed.**

WhatsApp's own Help Center documents the flow as four steps (VERIFIED, primary source,
`faq.whatsapp.com/1180414079177245`):

> 1. Open the chat.
> 2. Tap More options > More > Export chat.
> 3. **Tap Without media or Include media.**
> 4. Choose your desired method (ex. Message, Mail, Add to Notes) to export chat
>    history.

So step 3 is a real, documented step. **But I never observed it.** In four separate
export runs on this device, tapping `Export chat` went *straight* to the system share
sheet (`com.android.intentresolver`, "Sharing 1 file") with no media prompt at all.
VERIFIED, four times.

The reason is that **the prompt is conditional on the chat containing media**. I
opened the chat's `MediaGalleryActivity` and it reported, verbatim: **"No media
found."** (VERIFIED, tabs `All media` / `Docs` / `Links` / `Media`). The chat
contained a PDF document, which does not count as media. That explains all four runs.

Consequences, stated honestly:

- **For a without-media capture, the prompt may never appear.** A mechanism that
  *waits* for it would hang. A mechanism that *requires* it would fail. The correct
  design is to treat the prompt as optional: proceed if absent, and if present, assert
  you recognised it before tapping.
- **For a with-media capture, the prompt's node structure is UNVERIFIED.** I could not
  find any chat on this account containing photos or video to test against, and I was
  not willing to go hunting through the user's private conversations to manufacture
  one. The button labels are almost certainly "Without media" and "Include media" by
  symmetry with the Help Center text, but I will not claim a bounding box or a
  resource id I did not see.

**What happens if the answer is wrong, or the prompt never appears:**

- Wrong answer (choosing Include media when you meant Without media): you get a much
  larger archive, and per WhatsApp's own note "the most recent media sent will be
  added as attachments to be sent from your sharesheet" - the share sheet contents
  change shape. Not corrupting, but wrong, and it is exactly the sort of error a
  provenance sidecar would have to catch.
- Prompt never appears: the export proceeds with the default. Benign for us, provided
  the driver treats absence as normal rather than as failure.
- **A tap lands on nothing because the screen changed underneath:** this is the
  dangerous case, and it is the same silent-failure class as the coordinate race. It
  is mitigated only by asserting expected state after every step.

**Where the file actually goes.** Export does **not** write to `/sdcard/Download`.
After four exports, `find /sdcard -name '*.zip' -newermt '-30 minutes'` returned
nothing (VERIFIED). The archive is written to app-private storage, which `shell`
cannot read (`ls /data/data/com.whatsapp` -> "Permission denied", VERIFIED), and is
then offered to other apps through a `FileProvider` content URI. The only pre-existing
zip, `/sdcard/Download/WhatsApp Chat with [chat A].zip` at 320 bytes, arrived via a
chosen save destination. In the concurrent session I watched the route end-to-end:
share sheet -> **Cx File Explorer** -> `Downloads`. VERIFIED, observed.

So the real flow is five interactions across **three** apps, the last of which is a
third-party file manager:

1. search or scroll to the chat, tap it
2. overflow -> More -> Export chat
3. *(conditional)* Without media / Include media
4. share sheet -> a file manager -> `Downloads` -> confirm the save
5. `adb pull` (already proven)

Step 4 is a genuine fragility that the ticket did not anticipate: it depends on which
apps are installed and on a file manager's own UI, and this image does not appear to
offer a direct "save to Downloads" entry in the `ACTION_SEND` chooser.

## 6. Unattended emulator operation

| Capability | Status | Label |
|---|---|---|
| Boot with no display | `-no-window` is supported. The binary's own `--help` says "-no-window: disable graphical window display". Official docs: "This option is useful when running the emulator on servers that have no display. You can access the emulator through `adb` or the console." | VERIFIED that the option exists and is documented; **not** VERIFIED by experiment, because booting a second instance of the same AVD is refused ("Running multiple emulators with the same AVD is an experimental feature") |
| Faster modern route | `android emulator start [--cold] [--headless]`, documented as "Launches the specified virtual device. This command returns when the emulator is fully started and ready to use." | VERIFIED (official docs); not run |
| Restore from a snapshot | `fastboot.forceFastBoot=yes` in `config.ini`; `snapshots/default_boot/` exists with a 2.5 GB `ram.bin` written 2025-09-25. `-snapshot <name>`, `-no-snapshot-load`, `-no-snapshot-save`, `-no-snapshot` are all documented. | VERIFIED (config and files on disk). `emulator -snapshot-list` refused to run while another instance held the AVD, so the *list* is unconfirmed. |
| Snapshot load failure mode | `developer.android.com/studio/run/emulator-commandline.html`: "If the file isn't found, the emulator still launches, but without an SD card." So a bad snapshot degrades rather than bricks. | VERIFIED (official docs) |
| Left running across a host reboot | Needs a `systemd --user` unit. Nothing platform-specific blocks this. The current instance runs as an ordinary foreground process with 36 open sockets on `DISPLAY=:0`, which is *not* how it should be supervised. | INFERRED - **not tested**, because starting a competing emulator would have destroyed the running one |

A unit would need, at minimum: `ExecStart` with `-no-window -no-audio -no-boot-anim`,
`adb wait-for-device` before anything else, `RemainAfterExit=no` with the emulator as
the long-lived `main` process, and a `WantedBy=default.target` under
`graphical-session.target` avoided so it does not need X. The `nodraw on` emulator
console command exists for reducing host GPU load in headless runs.

**Honest status: the emulator is very likely fully manageable headless and across
reboots, but I verified none of it experimentally, because the only way to test it on
this machine was to kill the emulator another agent was using.** That experiment is
cheap and safe to run later when the device is free: stop the current instance, then
`systemd-run --user --scope emulator -no-window -no-audio @small_phone`, confirm
`adb devices` shows it, and reboot the host.

## Bottom line

**Is fully unattended capture achievable?** Mechanically, yes. Every step I could
test was driven successfully by `uiautomator dump` plus `input tap`, and the
accessibility route could do the same. Nothing about WhatsApp blocks it.

**Should it be the default?** No, and here is the evidence that drove that.

The binding constraints are not "can we tap" but "can we be sure we tapped the right
thing", and on that:

1. **The scroll race fails silently.** VERIFIED, reproduced: a stale coordinate
   opened a completely different chat. A capture pipeline that can silently archive the
   wrong conversation is worse than one that refuses to run. This is fixable, but only
   by making verification mandatory, which is a real design constraint on the
   acquisition command (issue #8), not a detail.
2. **`uiautomator dump` is an exclusive global resource.** VERIFIED: 3/10 under
   contention, 8/8 quiet. An unattended scheduler that collides with anything else
   touching the UI will fail in a way that looks like flakiness, not like a conflict.
3. **The media prompt's structure is unverified**, and it is conditional. A with-media
   capture would be built on a screen I have never seen.
4. **The save step depends on a third-party file manager** that WhatsApp does not
   control and we do not own.
5. **WhatsApp updates itself weekly** and every selector we encode is a guess about
   the future.

**Recommendation, in the order the project should take it:**

1. **Ship the human-tap-plus-one-command-pull shape now.** The human opens the chat and
   taps Export chat (and the save destination) in the GUI, exactly as they do today.
   The host command does what is already proven and adds the value: locate the
   resulting zip in `/sdcard/Download`, `adb pull` it, hash the content, write the
   provenance sidecar, and refuse to overwrite. This is honest, it is boring, it
   cannot silently capture the wrong chat, and it is the shape that survives a
   WhatsApp update. **A permanent human tap is the right answer for v1.**
2. **Then add an opt-in `--drive` mode** for the without-media case only, built on the
   re-dump-before-every-tap and assert-the-chat-name discipline from section 3. Gate it
   behind a flag, make it the only `UiAutomation` client, and make it fail loudly
   rather than fall back. Treat the media prompt as optional-and-unrecognised-is-fatal.
3. **Do not build a custom accessibility service, and do not adopt Automate, Tasker or
   MacroDroid.** They cost more than `adb` here, add an app to maintain, and add an
   accessibility grant that Android 14+ revokes on every app update with no workaround.
4. **Before promising with-media captures, run one experiment**: put a single photo in
   a scratch chat, then dump the media prompt and record its real structure. That is
   the cheapest way to close the one gap that actually blocks a decision.

**The seam for the future scheduler** (issue #9) should therefore be defined as
"acquire one named chat's export from a running emulator", with the driving mechanism
an implementation detail behind that contract. That way a human tap today and an
automated drive tomorrow are the same seam.

## Gaps I could not close

- Whether the with/without-media prompt appears, and its exact structure, for a chat
  that actually contains photos or video. **UNVERIFIED.** Needs a scratch chat with
  one photo.
- Whether writing `settings put secure enabled_accessibility_services` makes a real
  service bind without the Settings consent dialog. **INFERRED.** Needs a real service
  installed, which I declined to do.
- Whether Automate actually functions on this API 36 image. **INFERRED.** Needs an
  install, which I declined to do.
- Whether the accessibility grant survives an emulator reboot, in practice. **INFERRED.**
  Needs a reboot, which would have destroyed the concurrent session.
- Whether the emulator boots and restores unattended under systemd. **INFERRED.**
  Needs the current instance stopped.
- Whether any published Automate flow drives WhatsApp's **export** specifically.
  **UNVERIFIED, and I found no evidence that one exists** - only message-sending flows.
