# WhatsApp export format: locales, versions, and with-media exports

Research notes for wayfinder ticket
[#3](https://github.com/SoongGuanLeong/footprint-messaging/issues/3). Findings only, no
parser. Every claim is tagged:

- **VERIFIED**: I reproduced it myself on the emulator, or it is stated verbatim in a
  primary source I quote.
- **INFERRED**: consistent with the evidence, but I did not prove it.
- **UNVERIFIED**: community or third-party material only. Not trustworthy for design
  decisions on its own.

## Method and provenance

Two independent evidence sources, because neither alone is enough:

1. **This machine.** Emulator `emulator-5554`, Android 16 (API 36, x86_64, Google Play
   image), WhatsApp `2.26.37.73` (`versionCode=263707322`, `targetSdk=36`), linked as a
   companion device. I drove Export chat through the UI with `adb`, pulled the resulting
   zips with `adb pull`, and varied one device setting at a time. Nine exports were
   captured. All of them are listed in "Artifacts" at the end.
2. **Dated real exports from a public corpus.** `starkdmi/whats_json` test data
   (`test/data/android/*.txt` and 45 sampled files from `test/data/chats_unique/`),
   which are genuine WhatsApp Android and iOS exports collected by that project. These
   are real files with real bytes, but they are third-party, so provenance is weaker than
   something I produced. I use them for line shapes I could not produce myself, and I say
   so each time.

Primary documentation is thin. The official Help Center article on export is
[faq.whatsapp.com/1180414079177245](https://faq.whatsapp.com/1180414079177245), "How to
export your chat history". I retrieved the article body verbatim (the page is a
JavaScript app, so the text lives in an embedded RSC payload). **It documents the menu
path and nothing about the file format.** Quoted in full where it matters below.

## Read this first: on a linked device, the export contains no messages

**VERIFIED, reproduced three times on three different chats.** Every export I captured
from the companion-linked emulator contains exactly two system lines and zero actual
messages:

```
9/19/26, 9:52<U+202F>AM - Messages and calls are end-to-end encrypted. Only people in this chat can read, listen to, or share them. *Learn more*
9/25/26, 11:44<U+202F>PM - Messages and calls are now end-to-end encrypted. Only people in this chat can read, listen to, or share them. Learn more.
```

The chats themselves are not empty. The [chat A] chat shows, in the UI, ordinary text
messages, a multi-line message, a PDF attachment with its "7 pages / 148 kB / PDF"
header, and a "Yesterday" date divider. `[chat B]` has recent messages
stamped `1:32 AM`. Both produce message-less transcripts. Each export is a structurally
valid zip (`unzip -t` clean, correct end-of-central-directory), so this is not
truncation.

Why this matters for the downstream tickets: **the project premise is that Export chat
on a linked emulator yields the person's real history. On this device it does not.** Any
ticket that assumes a non-empty transcript is assuming something I could not reproduce.

Corroborating primary source, and the closest thing to an official explanation.
[faq.whatsapp.com/209942271778103](https://faq.whatsapp.com/209942271778103), "How to
transfer your chat history", states verbatim:

> It's not possible to transfer your chat history on linked devices such as WhatsApp on
> Web, WhatsApp on Mac, or WhatsApp on Windows. To transfer your chats, you'll need to
> use your phone

WhatsApp officially treats linked devices as second-class for history operations. It
does not say export is affected, and I have not proven the cause here.

**INFERRED, cause not established.** The most likely explanation is that Export chat
reads a message store that a companion device does not fully populate, but I could not
confirm this without reading private app storage, which is out of bounds for this
project. **Do not act on the cause.** Act on the observation.

**Recommended next step for the map, not for me:** before any parser work, someone should
decide whether Export chat on a companion device is acceptable at all, or whether the
architecture needs a primary phone. That is a grilling ticket, not a research ticket.

## 1. Line grammar

The Android transcript is line-oriented, UTF-8, no BOM, LF only, with a trailing
newline. **VERIFIED** on all nine of my exports and on all 50 corpus files.

Line shapes, from real files:

| Shape | Example | Source |
| --- | --- | --- |
| System / notice line, no sender | `23/03/2022, 13:31 - Messages and calls are end-to-end encrypted. No one outside of this chat, not even WhatsApp, can read or listen to them. Tap to learn more.` | corpus `WhatsApp Chat with [sender] EN Android.txt` line 1 |
| Group event line, no sender | `11/01/2023, 16:11 - [sender] created group "Group Chat"` | corpus `WhatsApp Chat with Group Chat XXX Android.txt` |
| Group event, actor is "You" | `11/01/2023, 16:14 - You changed the subject from "Group Chat" to "Group Chat XXX"` | same |
| Ordinary message | `18/03/2022, 14:56 - [sender]: Hello` | corpus EN |
| Sender is a phone number | `31/10/2021, 14:01 - [phone]: [message text]` | corpus `chats_unique` |
| Media present, export without media | `18/03/2022, 15:10 - [sender]: <Media omitted>` | corpus EN |
| Media present, export with media | `23/03/2022, 13:32 - Elon: IMG-20220323-WA0002.jpg (file attached)` | corpus `EN+Media` |
| Voice note | `18/03/2022, 15:12 - [sender]: PTT-20220323-WA0000.opus (file attached)` | corpus `EN+Media` |
| Sticker | `27/03/2022, 13:42 - [sender]: STK-20220327-WA0000.webp (file attached)` | corpus `EN+Media` |
| Document, original filename | `23/03/2022, 13:33 - Elon: Alex.vcf (file attached)` | corpus EN |
| Location share | `18/03/2022, 15:13 - [sender]: location: https://maps.google.com/?q=<lat>,<lon>` | corpus EN |
| Deleted, by the other party | `4/12/21, 21:20 - Adal: This message was deleted` | corpus `chats_unique` |
| Deleted, by you | `2/14/21, 19:09 - David Centurión: You deleted this message` | corpus `chats_unique` |
| Missed call, has a sender | `6/22/22, 9:02 PM - Charan Clg: Missed voice call` | corpus `chats_unique` |
| Missed group call | `9/10/22, 10:31 PM - Charan Clg: Missed group voice call` | corpus `chats_unique` |

Grammar, stated plainly:

```
transcript := line*
line       := timestamp " - " body
body       := sender ": " message      # ordinary message
            | system_text               # notice or group event, no sender
message    := first_line continuation*
continuation := any line with no timestamp prefix
```

Three traps in that grammar, all **VERIFIED** from real bytes:

**Multi-line bodies are bare continuation lines with no prefix at all.** From the corpus
EN export, a single shared location is three lines:

```
27/03/2022, 21:46 - [sender]: [message text]
London, United Kingdom
https://foursquare.com/v/4e89af95e5fa82ad4c0aa03b
```

Line counts confirm this: the EN file has 62 lines, of which 60 carry a timestamp
prefix and 2 do not. The `EN+Media` file has 73 lines, 60 with a prefix and 13 without.
A media message with a caption is a prefixed attachment line followed by bare caption
lines: `23/03/2022, 13:32 - Elon: IMG-20220323-WA0002.jpg (file attached)` then a line
holding only an emoji. **Consequence: a message body that itself contains a line looking
like a timestamp is indistinguishable from a new message. There is no escaping.** I found
no documented escaping mechanism in any source.

**The file is not globally sorted by time.** In the corpus group export the system lines
run `16:13, 16:11, 16:11, 16:12, 16:11, 16:12, 16:13, 16:13, 16:14, ...`. The encryption
notice, timestamped `16:13`, is emitted first regardless. System events appear to be
written in a separate pass from messages. **A parser must not assume monotonic time.**

**WhatsApp injects invisible Unicode into message lines.** **VERIFIED.** Immediately
before an attachment filename there is a LEFT-TO-RIGHT MARK:

```
23/03/2022, 13:33 - Elon: <U+200E>Alex.vcf (file attached)
```

Two occurrences per CZ and RU file, in exactly the attachment lines. The Russian system
line also contains a NO-BREAK SPACE (`не<U+00A0>могут`). The iOS form carries two LRM
characters: `[1/29/18, 23:46:03] Alice: <U+200E><U+200E><attached: <U+200E>PTT-20180129-WA0025.opus>`.
In the chat UI I also saw the system notice rendered with a leading U+200C ZERO WIDTH
NON-JOINER, though that is a UI artifact and I did not confirm it reaches the file.

**Deleted, edited, quoted, and call lines.** Deleted and call shapes above are
**VERIFIED**. For the rest:

- **UNVERIFIED: edited messages.** I found no "edited" marker in 50 real exports. The
  feature exists in the app, but I have no export evidence of how it is rendered, and no
  primary source describes it.
- **UNVERIFIED: quoted replies.** No reply marker found. The corpus contains lines whose
  *text* is the words "Reply" or "This message is reply to another", but those are people
  typing, not markup.
- **UNVERIFIED: call duration.** Zero hits for a `(mm:ss)` duration suffix across 45
  sampled exports. Community write-ups claim durations appear; I found none.
- **VERIFIED: no leading or trailing metadata.** The file starts with the first line of
  content and ends with a newline. No header, no footer, no version string, no chat id.
  Checked on all 50 corpus files and all 9 of mine.

## 2. What varies with device locale and clock setting

This is the part I could settle properly, because I could change the setting and re-run
the export. The headline: **the timestamp prefix has at least six independent variable
axes, and the date part alone has four of them.** A capture taken on a differently
configured device is genuinely not parseable by a hardcoded pattern.

**VERIFIED by experiment.** I changed one setting at a time on the emulator and captured
a fresh export each time. Same chat, same two system lines, underlying timestamps
identical throughout (`09:52` and `23:44` local):

| Device setting | Timestamp prefix produced |
| --- | --- |
| app locale `en-US`, `time_12_24` unset (VERIFIED) | `9/19/26, 9:52<U+202F>AM - ` |
| app locale `en-US`, `time_12_24=24` (VERIFIED) | `9/19/26, 09:52 - ` |
| app locale `en-GB`, `time_12_24` unset (VERIFIED) | `19/09/2026, 09:52 - ` |
| app locale `en-GB`, `time_12_24=24` (VERIFIED) | `19/09/2026, 09:52 - ` |
| app locale `en-IN`, `time_12_24` unset (VERIFIED) | `19/09/26, 9:52<U+202F>am - ` |

Read off that table:

1. **Date field order** follows the locale: `M/D` for en-US, `D/M` for en-GB and en-IN.
2. **Year width** follows the locale: 2 digits for en-US and en-IN, **4 digits** for
   en-GB. This is not a fixed-width field. Note en-US and en-IN disagree, so it is not
   simply "US is short".
3. **Day padding** follows the locale: en-US `9/19/26` is unpadded, en-GB and en-IN
   `19/...` are padded.
4. **Hour cycle** is 24h if the locale prefers it or if `Settings.System.TIME_12_24` is
   `"24"`. When 24h, the hour **is** zero-padded. When 12h, the hour is **not** padded
   and a meridiem follows.
5. **Meridiem case** follows the locale: `AM`/`PM` for en-US, `am`/`pm` for en-IN.
6. **The space before the meridiem is U+202F NARROW NO-BREAK SPACE**, not a plain space.
   This is the detail the original sample already flagged, and it holds on Android 16.

Independent confirmation from real files, across the 45 sampled corpus exports. I
classified every timestamp prefix by shape. The counts:

```
  24329  ('/', day-padded,   4-digit year, comma, ':', hour-padded)
  15835  ('/', day-padded,   2-digit year, comma, ':', hour-unpadded)
  10908  ('/', day-unpadded, 4-digit year, comma, ':', hour-unpadded)
  10267  ('/', day-unpadded, 2-digit year, comma, ':', hour-padded)
   8723  ('/', day-padded,   2-digit year, comma, ':', hour-padded)
   7364  ('/', day-unpadded, 2-digit year, comma, ':', hour-unpadded)
    788  ('/', day-unpadded, 4-digit year, comma, ':', hour-unpadded)
    757  ('/', day-unpadded, 2-digit year, NO comma, ':', hour-unpadded)
    309  ('/', day-unpadded, 2-digit year, NO comma, ':', hour-padded)
     95  ('.', day-padded,   2-digit year, comma, ':', hour-padded)
     66  ('/', day-padded,   2-digit year, NO comma, ':', hour-padded)
     60  ('.', day-padded,   2-digit year, NO comma, ':', hour-padded)
     60  ('.', day-padded,   4-digit year, comma, ':', hour-padded)
     12  ('/', day-unpadded, 2-digit year, NO comma, '.', hour-padded)
     10  ('/', day-padded,   2-digit year, NO comma, ':', hour-unpadded)
```

Two axes there I could not produce on this device and that you should still plan for:

- **The comma between date and time is optional.** The Czech corpus file has
  `23.03.22 13:31 - ` with a plain space and **no comma**, plus a 2-digit year. That is
  the same file where `<Media omitted>` becomes `<Média vynechány>`.
- **The time separator can be a period.** 12 occurrences of `HH.MM`, matching community
  reports of Turkish and Indonesian exports using `18.00`.

**Everything in the message text is localized too, not just the timestamp.** This is
underrated. From the same three real files:

| | English | Czech | Russian |
| --- | --- | --- | --- |
| encryption notice | `Messages and calls are end-to-end encrypted...` | `Zprávy a hovory jsou zabezpečeny...` | `Сообщения и звонки защищены...` |
| no-media placeholder | `<Media omitted>` | `<Média vynechány>` | `<Без медиафайлов>` |
| attachment marker | `(file attached)` | `(soubor byl přiložen)` | `(файл добавлен)` |
| location prefix | `location:` | `poloha:` | `Местоположение:` |
| filename prefix | `Chat WhatsApp s uživatelem ...` / `WhatsApp Chat with ...` | same | `Чат WhatsApp с контактом ...` |

**Consequence for the project: the transcript filename itself is localized.** "WhatsApp
Chat with [chat A]" is the en-US/en-GB form. A de-DE or ru-RU device produces a different
prefix. Any pipeline that pattern-matches the output filename is matching a localized
string.

**INFERRED: non-ASCII digits in some locales.** Several community parsers normalise
Arabic-Indic digits before matching. I have no real export with non-ASCII digits, so treat
this as likely but unproven.

## 3. What varies by WhatsApp version, and the compatibility floor

**The honest answer to the primary-source half of this question is that there is no
primary source.** Meta does not publish a per-version changelog for WhatsApp Android.
Third-party coverage of specific builds says so outright ("specific patch notes for every
minor iteration are not always published by Meta"), and Meta's own
[security advisories page](https://www.whatsapp.com/security/advisories) says "Due to the
policies and practices of app stores, we cannot always list security advisories within
app release notes." The Help Center articles I retrieved say nothing about the file
format at any point in their history. So version dependence has to come from dated real
exports plus the platform underneath, which is what follows.

**VERIFIED: the system-notice wording changed, and the change is datable.** Every corpus
export from 2018 through 2022 opens with the same sentence:

```
Messages and calls are end-to-end encrypted. No one outside of this chat, not even
WhatsApp, can read or listen to them. Tap to learn more.
```

My 2026 device emits something different, twice:

```
Messages and calls are end-to-end encrypted. Only people in this chat can read, listen
to, or share them. *Learn more*
Messages and calls are now end-to-end encrypted. Only people in this chat can read,
listen to, or share them. Learn more.
```

Three changes at once: the body of the notice was rewritten, the learn-more affordance
became a literal `*Learn more*` marker, and a second "now end-to-end encrypted" notice
was added as a separate line. Any parser that recognises system lines by string match
breaks on this, and did break between the corpus era and now.

**VERIFIED: the space before AM/PM is Android-version dependent, not WhatsApp
dependent.** In all 52,310 meridiem occurrences across the 45 sampled corpus exports, the
preceding character was U+0020 ASCII space. On my Android 16 device it is U+202F. The
mechanism is a CLDR change, and I can point at both ends of it:

- CLDR ticket CLDR-14032 changed the space between time and the AM/PM marker to NNBSP
  (U+202F) for Latin, Cyrillic and Greek scripts. It landed in **CLDR 42**.
  Source: <https://github.com/unicode-org/cldr/pull/2001>
- JDK 20 picked up CLDR 42 and the change broke parsing for a lot of software. OpenJDK
  bug JDK-8324308, "US DateTimeFormatter uses Narrow No-Break Space", states "Occurs in
  JDK 20, 21 and 22, but not in older ones like JDK 18 and 19". Source:
  <https://bugs.openjdk.org/browse/JDK-8324308>

Android tracks CLDR separately, but the direction and the character are the same. So a
capture from an older Android build will have U+0020 there and a capture from a current
one will have U+202F. **A parser must accept any Unicode space there, not one literal.**

I also checked whether WhatsApp uses Android's own `android.text.format.DateFormat`
helper, because AOSP applies a compatibility shim in that class. From
[core/java/android/text/format/DateFormat.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/format/DateFormat.java):

```java
private static String getCompatibleEnglishPattern(ULocale locale, String pattern) {
    if (pattern == null || locale == null || !"en".equals(locale.getLanguage())) return pattern;
    String region = locale.getRegion();
    if (region != null && !region.isEmpty() && !"US".equals(region)) return pattern;
    return pattern.replace('\u202f', ' ');
}
```

That shim rewrites U+202F back to a plain space for English locales outside the US. My
`en-IN` test produced U+202F, not a plain space. **VERIFIED: WhatsApp does not go
through that shim.** It formats the timestamp straight from CLDR/ICU. Practical upshot:
do not assume a plain space for en-GB, en-IN, or any other non-US English locale, even on
a current Android.

The same AOSP file also confirms the hour-cycle rule I measured, so the mechanism is
documented, not just observed:

```java
public static boolean is24HourFormat(Context context, int userHandle) {
    final String value = Settings.System.getStringForUser(context.getContentResolver(),
            Settings.System.TIME_12_24, userHandle);
    if (value != null) return value.equals("24");
    return is24HourLocale(context.getResources().getConfiguration().locale);
}
```

and

```java
String pattern = is24HourFormat(context, userHandle) ? dtpg.getBestPattern("Hm")
                                                      : dtpg.getBestPattern("hm");
```

`Settings.System.TIME_12_24` wins if set, otherwise the locale decides. That is exactly
the behaviour in my experiment table, and it confirms the pattern is generated by CLDR's
`DateTimePatternGenerator`, which is why every axis in section 2 moves together.

**A sensible compatibility floor.** For a read-only parser, the floor should be set by the
inputs you must accept, not by the WhatsApp version:

- **Timestamps: no floor. Accept all 50 corpus shapes plus both space characters.** The
  variation is CLDR's, it spans at least 2018 to 2026, and there is no way to bound it
  from WhatsApp's side. Parse the timestamp by pattern, not by a fixed format, and record
  the resolved format in provenance.
- **Body text: no floor, treat every literal as unrecognisable.** System notices,
  placeholders and attachment markers are all localized and all have changed wording
  inside the observed window.
- **WhatsApp build: pin the version you test against and record it.** The observed
  device is `2.26.37.73`. Be aware the version scheme itself changed in 2026, from
  `2.26.x.y` toward `26.x`, so a version string is not a stable key across time.
  Recording the version in the provenance sidecar (ticket #5) is what makes a future
  format change diagnosable rather than mysterious.
- **Practical floor if you must name one:** the oldest real export I have evidence for is
  from January 2018 (`[1/29/18, 23:46:03]`, iOS) and the oldest Android one is from 2018
  too. Nothing in the observed data suggests a format break before that. But treat 2018
  as "oldest tested", not "oldest supported".

## 4. With-media export layout

**This is the weakest section, and the honest reason is the blocking finding.** Because
every export from the linked device contained no messages, and because the chat I could
export had no media reaching the exporter, **I never produced a with-media export. I have
no verified data on the internal layout of one.** Marked accordingly below.

What I can state:

**VERIFIED: the transcript stays a single `.txt` inside the zip, and the zip is always
the container.** All nine of my exports, with and without any media reaching the
exporter, were a zip containing exactly one entry, `<title>.txt`. The MIME type WhatsApp
hands to the share sheet is `application/zip` (from logcat, see section 5). Community
sources describing a bare `.txt` on Android appear to describe older builds or the iOS
path; on this build it is always a zip.

**VERIFIED: the "Without media / Include media" choice did not appear.** I tapped Export
chat and went straight to the share sheet, on every one of the nine runs, via both entry
points I tried. The [official article](https://faq.whatsapp.com/1180414079177245) still
documents that choice, verbatim:

> Tap Without media or Include media.

**INFERRED, not established:** either the choice is conditioned on there being media to
include (and this account produced none, per the blocking finding), or it was removed in
`2.26.37.73`. I could not distinguish these. Also relevant: the chat-info screen in this
build has **no** Export chat entry at all. Scrolled to the bottom it offers Advanced chat
privacy, No groups in common, Create group, Add to groups, Add to Favorites, Add to list,
Clear chat, Block, Report. Export chat exists only under the chat's overflow menu, which
narrows what an unattended driver has to hit.

**VERIFIED: attachment filenames follow a documented convention, and the transcript
references them by bare filename.** From the real `EN+Media` export:

```
IMG-20220323-WA0002.jpg      images
VID-...mp4                   videos            (UNVERIFIED, no sample)
PTT-20220323-WA0000.opus     voice notes
STK-20220327-WA0000.webp     stickers
Alex.vcf                     documents, original filename
```

Pattern is `PREFIX-YYYYMMDD-WA<counter>.<ext>`, except documents, which keep their
original name. The counter is per-chat and per-type and starts low (`WA0000`). All
**UNVERIFIED** beyond that: whether filenames are sanitised against the chat title (they
are not derived from it at all, which answers half of the ticket question), and whether
they are stable across repeated exports of the same chat. The counter sequence looks
deterministic, but I did not export the same chat with media twice, so I cannot claim it.

**VERIFIED: the transcript can mix both forms in one export.** The `EN+Media` file
contains real `IMG-...(file attached)` lines *and* `<Media omitted>` lines side by side.
So "with media" does not mean every attachment resolved. **VERIFIED: the no-media
placeholder and the attachment marker are mutually exclusive forms of the same field**,
and which one appears is per-message, not per-export.

**UNVERIFIED: the internal directory structure of a with-media zip.** I found no primary
source and no real Android with-media zip. Community sources disagree with each other:
some describe a flat archive with all media at the root next to the transcript, others
describe a `media/` subfolder, and at least one describes the archive as folder-shaped.
None of these are trustworthy. **Do not design against any of them.** If the layout
matters, the way to settle it is one real export from a primary phone, which is a
prerequisite this project does not currently have.

**UNVERIFIED: export size.** My largest export was 354 bytes, because my exports were
empty. I have no measurement of a realistic export size and no verified size limit. See
the next section for the message-count limit, which is a different thing.

## 5. Failure and edge shapes

**VERIFIED: the export is not written to a file you can find. It is a content provider
stream, and the share sheet is mandatory.** From `adb logcat` during an export:

```
W/ChooserPreview: Could not read content://com.whatsapp.provider.media/export_chat_folder/[lid]/WhatsApp Chat with [chat A] stream types.
W/ChooserPreview: Could not read content://com.whatsapp.provider.media/export_chat_folder/[lid]/WhatsApp Chat with [chat A] metadata.
I/ActivityTaskManager: START u0 {act=android.intent.action.SEND typ=application/zip flg=0xb080001
  cmp=com.cxinventor.file.explorer/...SaveToActivity clip={application/zip {T(74) U(content)}}}
  from uid 10229 (com.whatsapp)
```

Several things fall out of this:

- The authority is `com.whatsapp.provider.media/export_chat_folder`, the path is
  `/<chat-jid>/<display name>`, and the stream is generated on read. There is no
  intermediate file in any directory `adb` can see. I checked: `/sdcard/Download`,
  `/sdcard/Documents`, and `/sdcard/Android/data/com.whatsapp/{files,cache}` contain
  nothing. `run-as` is refused because the package is not debuggable.
- **The export never lands in `/sdcard/Download` by itself.** It goes to the Android
  share sheet and a human, or a driver, has to choose a destination. On this device the
  working target was Cx File Explorer, which opened a save dialog already pointed at
  `/storage/emulated/0/Download`. The stock Files app was not registered as a share
  target here. **This is a real coupling between the export flow and a third-party file
  manager, and ticket #4 needs to reckon with it.**
- The zip is written as a stream: `zipinfo` reports "extended local header: yes", meaning
  a data descriptor, and the origin is recorded as MS-DOS/FAT with all-zero external
  attributes. Entry timestamps are DOS format.

**VERIFIED: the zip is byte-deterministic apart from the entry timestamp.** Two exports
of the same chat, same settings, minutes apart, differ in exactly **4 bytes**, at
offsets 11, 12, 238 and 239. Those are the DOS date and time fields, written twice: once
in the local file header and once in the central directory. Everything else, including
the deflate stream and the CRC, is identical. The `.txt` payload was byte-identical
(sha256 `8eaae168f5505051...` for both). So the answer to "does exporting twice change
anything other than timestamps" is a clean **no**, and the entry timestamp is simply the
moment of export, not of the message.

**VERIFIED: the filename is deterministic, so re-exporting collides.** Both runs produced
`WhatsApp Chat with [chat A].zip`. The file manager offered `OVERWRITE / RENAME / SKIP /
CANCEL` and I chose RENAME, which produced `WhatsApp Chat with [chat A] (2).zip`. The
collision handling is the file manager's, not WhatsApp's, but the collision itself is
WhatsApp's doing and any unattended capture has to handle it.

**VERIFIED: an earlier capture of the same chat did not reproduce, and the difference was
in the timestamps.** The pre-existing `WhatsApp Chat with [chat A].zip` (320 bytes, captured
09:37) has system lines stamped `9/19/26, 7:38 PM` and `9/26/26, 9:30 AM`. Every export I
took afterwards has `9/19/26, 9:52 AM` and `9/25/26, 11:44 PM`. Both timestamps moved
*backwards* relative to the original. I opened the chat in the UI between the two, so the
plausible reading is that the notices were re-stamped locally when the chat was rendered,
but **I did not establish the cause**, and the sample size is one. Treat it as a warning:
**the timestamps on system notices are not obviously stable over the life of a chat**,
so do not use them as an idempotency key.

**VERIFIED: message-count limits, and that WhatsApp has since stopped publishing them.**
The current Help Center article contains no numbers. The archived official article does.
From the [Wayback capture of 2022-01-21](https://web.archive.org/web/20220121035523/https://faq.whatsapp.com/android/chats/how-to-save-your-chat-history/?lang=en),
"How to save your chat history", verbatim:

> If you choose to attach media, the most recent media sent will be added as attachments.
>
> When exporting with media, you can send up to 10,000 latest messages. Without media, you
> can send 40,000 messages. These constraints are due to maximum email sizes.

Both sentences are still present in the live article in a reworded form
("the most recent media sent will be added as attachments to be sent from your
sharesheet"), but **the 10,000 / 40,000 figures have been dropped from the Help Center
with no replacement statement.** Three consequences:

1. If those limits still apply, an export is a **tail**, not the whole chat, and
   "complete archive" is not achievable through this surface. Given the rationale was
   email attachment size, and the modern flow is a share sheet rather than email, I would
   expect the limits to have changed. That is **INFERRED**.
2. **UNVERIFIED:** whether the limit still exists, and whether it applies to the
   share-sheet path at all. Nobody I found has tested it against a current build.
3. The old article also noted, verbatim, "If you're in Germany, you might have to update
   WhatsApp before you can use the export chat feature." Regional availability of this
   feature has been a real issue.

**Does export require network? Not established, and I did not test it.** Turning off
connectivity on someone's linked account was not a risk I was willing to take unasked.
Indirect evidence only, and it points both ways: the [transfer article](https://faq.whatsapp.com/209942271778103)
says for phone-to-phone transfer that "Both of your phones need to have WiFi enabled.
They don't need to be connected to a network", while the [backup article](https://faq.whatsapp.com/481135090640375)
requires "A strong and stable internet connection". Export is neither of those, so this
needs a deliberate experiment on a disposable setup. **If the media in a with-media
export has to be fetched from the primary device, export almost certainly needs the
network; if the emulator already holds the media, it may not.**

**Can the export be absent, or land somewhere else?** **VERIFIED for the "somewhere else"
half.** It goes wherever the share-sheet target puts it, and on this device the only
working path was a third-party file manager. It is not in `/sdcard/Download` until
something saves it there. Whether it can fail to appear at all, and what a failed export
looks like, I did not test. **UNVERIFIED.** Note also that the share sheet is a system UI
surface: a mis-tap can share the archive into a chat, a drive, or Bluetooth instead of
saving it, and nothing in the export itself records where it went.

**Per-platform and per-version differences in where the export is written.** iOS writes
a zip containing `_chat.txt`, and the transcript there is bracketed with seconds
(`[1/29/18, 23:46:03] Alice: ...`) rather than dash-separated without seconds. Verified
from corpus files, but iOS is out of scope for this project and I did not test it.

## 6. Identity inside the export

**VERIFIED: the display title in the filename is the only identity inside the archive.**
Checked directly with `zipinfo -v`:

- no zip file comment
- central directory entry has "length of extra field: 0 bytes"
- "length of file comment: 0 characters"
- the single entry is `WhatsApp Chat with [chat A].txt`, which is the chat title and nothing
  more
- the transcript's first line is a system notice, not a header, and contains no chat
  identifier

There is no chat id, no participant list, no JID, no timestamp-of-export, no version
string, no device metadata anywhere in the zip. For a one-to-one chat the title is the
only handle, and for a group it is the only handle, and it is **user-editable and
localized** ("WhatsApp Chat with" is the en form; ru gives "Чат WhatsApp с контактом").

**VERIFIED: a real chat identifier exists outside the archive, in the provider URI.** The
logcat URI path carries it:

```
content://com.whatsapp.provider.media/export_chat_folder/[lid]/WhatsApp Chat with [chat A]
content://com.whatsapp.provider.media/export_chat_folder/[lid]/WhatsApp Chat with [chat B]
```

Two different chats, two different ids, and both in WhatsApp's newer **LID** (linked id)
form rather than a phone-number JID. This is a stable per-chat handle. It is not in the
zip, and it is only visible in the intent, which means it is only obtainable by
intercepting the share intent, not from a saved file. **This is the one piece of good
news in this section**: if the project ever needs to bind a capture to a chat
unambiguously, the id exists at export time even though it does not survive into the
artifact.

**Enumerating available chats: the export format cannot help.** Nothing in the format
lists chats, and there is no batch export. **VERIFIED:** each export is one chat; the
Help Center describes only a per-chat action. Enumerating what is available means reading
the app's chat list UI, which is a separate surface with its own scraping problems
(archived chats, message requests, communities, business accounts, and so on, all of which
I saw present in the list on this device). That is a decision for the grilling tickets,
not something the research settles.

## What I could not establish

Listed plainly, because the point of this ticket was a known-unknowns list.

1. **Why the linked-device export is empty.** Observed three times, cause unknown.
2. **Any with-media export.** Never produced one. Internal zip layout, directory naming,
   filename sanitisation, filename stability across repeat exports, and realistic export
   sizes are all open.
3. **Whether the 10,000 / 40,000 message limits still apply** to the current share-sheet
   flow. Officially stated in 2022, since removed from the Help Center, never retested.
4. **Whether export needs the network.** Not tested; too risky unasked.
5. **Edited-message and quoted-reply rendering.** No example found in 50 real exports.
6. **Call duration rendering.** No example found in 45 sampled exports.
7. **Non-ASCII digit locales.** Community parsers normalise them; I have no real export.
8. **A failed or absent export.** Never induced. Every export I attempted succeeded.
9. **Whether the missing "Without media / Include media" dialog is conditional or
   removed.** Both readings fit the evidence.
10. **Whether the system-notice timestamp shift I saw is reproducible.** One occurrence.

## Out of bounds, mentioned only to exclude

While searching I ran into a lot of unofficial tooling: scrapers and bridges built on
reverse-engineered WhatsApp Web libraries (`whatsapp-web.js`, `whatsxapp`, Baileys and
similar), tools that read the app's private SQLite database with root, and a commercial
desktop tool that exports via an iOS backup. **All out of bounds for this project**, per
the official-surfaces-only constraint. I am naming them only so nobody downstream
re-discovers them and thinks they are options. The one genuinely useful thing from that
literature is the CLDR and AOSP material in section 3, which is primary documentation of
the platform and is cited as such.

## Artifacts

All on branch `research/whatsapp-export-format`. Nine exports pulled from
`emulator-5554` and left in place on the device in `/sdcard/Download` (nothing was
deleted). Host copies live in `/tmp/opencode/wa-exp` and are not committed; the hashes are
recorded here so the observations can be re-checked.

| file | device settings | bytes | sha256 (zip, first 16) |
| --- | --- | --- | --- |
| `WhatsApp Chat with [chat A].zip` | en-US, 12h, pre-existing | 320 | `31c8dfe6e0f6d7d4` |
| `WhatsApp Chat with [chat A] (2).zip` | en-US, 12h | 321 | `53b46f4cafc4c512` |
| `WhatsApp Chat with [chat A] (3).zip` | en-GB, `time_12_24=24` | 310 | `0b312bad0f755274` |
| `WhatsApp Chat with [chat A] (4).zip` | en-GB, `time_12_24=24` | 310 | `d258f072a56ec4e7` |
| `WhatsApp Chat with [chat A] (5).zip` | en-GB, 12h (locale default) | 310 | `7a5ee11b3c4d06db` |
| `WhatsApp Chat with [chat A] (6).zip` | en-US, `time_12_24=24` | 310 | `b6c682ecc83fec8e` |
| `WhatsApp Chat with [chat A] (7).zip` | en-US, 12h, settings restored | 321 | `3f01723c4545e565` |
| `WhatsApp Chat with [chat A] (8).zip` | en-IN, 12h | 320 | `d00c2a333ea22481` |
| `WhatsApp Chat with [chat B].zip` | en-US, 12h | 354 | `12a29fbcc5744d9e` |

Device settings were restored after the experiments: `Settings.System.TIME_12_24` is
unset, the WhatsApp per-app locale list is empty, and no message was ever sent. The
account was never logged out and nothing was uninstalled or deleted.
