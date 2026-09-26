# Does a linked companion device receive every message sent after linking?

Research notes for wayfinder ticket
[#15](https://github.com/SoongGuanLeong/footprint-messaging/issues/15), child of map
[#1](https://github.com/SoongGuanLeong/footprint-messaging/issues/1). Findings only, no
parser, no code. Every claim is tagged:

- **VERIFIED**: read directly from a primary source I quote, or from an artifact/protocol file
  I quote, or observed in the project's own exports.
- **INFERRED**: consistent with the evidence, but I did not prove it.
- **UNVERIFIED**: asserted by a third party, or observed once without reproduction, or simply
  not established. Not safe to design against on its own.

The project owner has ruled history irrelevant: the archive is **forward-looking from link
time**. So this file answers one question — whether *new* messages are delivered — and it
keeps that strictly apart from the **link-time history copy**, which is a different mechanism
([#10](https://github.com/SoongGuanLeong/footprint-messaging/issues/10),
[#14](https://github.com/SoongGuanLeong/footprint-messaging/issues/14)). Evidence about one
must not be used to answer the other.

**Client caveat, stated once and repeated where it matters.** The well-documented clients are
WhatsApp Web and Desktop. This project uses the **Android app in companion mode**. WhatsApp
says Desktop syncs more *history* than Web, so Web/Desktop history numbers are not this
device's. For **new-message fan-out** the protocol is the same across client types (see §2),
so Web/Desktop observations about *live and offline delivery* are strong platform evidence —
but they are still not a measurement of the Android companion. Each claim below says which
client it rests on.

---

## Headline

**Forward delivery is real, and it is the reliable half of the companion story — but it is
not guaranteed, and it cannot be verified from the export.**

1. **New messages are pushed to the companion, not fetched.** WhatsApp uses **client-fanout**:
   the sending client encrypts the message once per destination device and sends it to every
   device in the account's device list. A linked companion is in that list from the moment it
   is linked, so it receives new 1:1 and group traffic — including messages the owner sends
   from the primary, because a client fans out to *all devices of its own account* too.
   **VERIFIED** (protocol, §3). The phone does **not** need to be online. **VERIFIED** (§3).
2. **The Help Center's "not all messages and chats are synced" caveat is about history, not
   new messages.** The sentence is "synced to linked devices **from your phone**", and it sits
   in the paragraph about the link-time history copy. New messages are not synced *from your
   phone*; they are fanned out by the sender. **INFERRED** (strong; §4) — WhatsApp never says
   this in as many words, and that residual ambiguity is the one place a contrary reading is
   possible.
3. **Offline gaps are bridged by the server — for up to 30 days.** A message sent while the
   companion is off is queued encrypted on WhatsApp's servers and delivered when the companion
   reconnects. **VERIFIED** (privacy policy) and **VERIFIED** as observed behaviour (a
   companion offline ~8.5 h received "85 offline messages/notifications" on reconnect; §5).
   But the same server deletes anything still undelivered after 30 days, **and** disconnects a
   linked device after 30 days of inactivity. An emulator off for more than 30 days therefore
   loses both the queued messages and the link itself. **VERIFIED** for the two 30-day rules;
   **INFERRED** that they compound.
4. **Even inside the window, offline delivery can fail silently.** Offline-flush decryption
   can lose a message (a per-message decryption failure is not retried by the server), and a
   group message can be permanently undecryptable if the companion was offline during a
   sender-key / identity change. **VERIFIED** as third-party observations (§5.4); the official
   client has the same class of healing path but no public guarantee.
5. **Media is not all there.** The message and caption arrive; the media blob is fetched
   per-device on demand, and the export carries only what is physically on the device.
   **VERIFIED** that a companion can hold and export media; **UNVERIFIED** that all media
   arrives without being opened.
6. **The export cannot tell you it missed anything.** There is no completeness metadata, no
   gap marker, no count. **VERIFIED** by absence (§9). An archive built this way is
   trustworthy only by external reasoning, never by inspection of the artifact.

**Confidence.** High that the fan-out mechanism delivers new messages to a linked companion
and queues them while it is offline. Medium-high that, in normal daily operation with a device
that reconnects within days, the companion receives essentially all new messages. Low that it
is *guaranteed* — nothing official says so, and there are at least three silent-loss paths and
two hard 30-day bounds.

---

## 1. Method, and what I did not do

Evidence classes, in order of trust:

1. **Official primary sources** — WhatsApp Help Center articles, WhatsApp/Meta's multi-device
   engineering blog, the WhatsApp Security Whitepaper, and the WhatsApp Privacy Policy, all
   retrieved 2026-09-26, quoted verbatim.
2. **The protocol** — the whitepaper is official; the recovered protobuf schemas carried by
   whatsmeow and Baileys are the platform's wire format and are used only as corroboration.
3. **Third-party ecosystem observation** — GitHub issues from whatsmeow and Baileys, and a
   known export-tool limitation. Their *method* (unofficial client) is out of bounds for this
   project; their *observations* are evidence about platform behaviour, cited as third-party.
4. **This project's own artifacts** — the exports already on this machine, cited from
   [#10](https://github.com/SoongGuanLeong/footprint-messaging/issues/10) and
   [#3](https://github.com/SoongGuanLeong/footprint-messaging/issues/3), re-read here where
   useful.

**I did not boot or drive the emulator**, per the ticket. adb/emulator were not used; no
message was sent; no device setting was changed; no unofficial client was run or installed.
The exports on disk were read only.

The bias this question is exposed to, stated plainly: **a companion looks self-consistent.**
You only notice a missing new message if you compare against the phone, and almost nobody
does. So "it works for me" is a report of non-detection, not of completeness. The evidence
below is chosen for what it can *falsify*, not for agreement.

---

## 2. Two mechanisms, and why the trap is real

| | Link-time history copy | Live fan-out of new messages |
| --- | --- | --- |
| When | once, right after linking | continuously, from link time on |
| Who sends it | the **primary phone** ("your primary phone sends an end-to-end encrypted copy of your most recent message history") | the **sending client**, whoever it is — a contact's phone, or the owner's own primary |
| Path | phone → companion, as an encrypted bundle | sender → WhatsApp server → each device, as one ciphertext per device |
| Scope | **partial by design** ("most recent", "recent chats") | the message that was just sent |
| Export relevance | source of the backfill the project has ruled irrelevant | **the thing this ticket is about** |

Sources: Help Center [653480766448040](https://faq.whatsapp.com/653480766448040) and the
engineering blog (both quoted in §3.1 / §4); the whitepaper's *Message History Syncing* and
*Exchanging Messages* sections (quoted in §3.2 / §5.1). **VERIFIED** that these are two
distinct mechanisms. The whole of the #14 partiality finding belongs in the left column; none
of it is evidence about the right column.

---

## 3. Live delivery — does the companion get new messages?

### 3.1 The phone is not in the path

Help Center [378279804439436](https://faq.whatsapp.com/378279804439436), *About linked
devices*. Verbatim:

> **Benefits of linked devices**
>
> - Use WhatsApp on your computer even when your phone is off.
> ...
> Each linked device connects to WhatsApp independently, maintaining the same expected level
> of privacy and security through end-to-end encryption.

And the engineering blog (Meta, 2021-07-14, *How WhatsApp enables multi-device capability*),
verbatim:

> Each companion device will connect to your WhatsApp independently ...
> WhatsApp multi-device uses a client-fanout approach, where the WhatsApp client sending the
> message encrypts and transmits it N number of times to N number of different devices — those
> in the sender and receiver's device lists. Each message is individually encrypted using the
> established pairwise encryption session with each device.

**VERIFIED**, verbatim. Two facts: the companion receives new traffic independently of the
phone, and delivery is a **push to every device in the device list**, not a pull.

### 3.2 Every device, including the owner's own companion

WhatsApp Security Whitepaper (v7, updated 2026-02-25), *Exchanging Messages* and *Sender Side
Backfill*. Verbatim:

> The client uses client-fanout for all the exchanged messages, which means each message is
> encrypted for each device with the corresponding pairwise session.

> Each client maintains a list of verified companion devices for WhatsApp accounts the user
> communicates with, **as well as all other devices associated with its own account**, and uses
> this list to specify the destination devices at the sending time.

**VERIFIED**, verbatim. The second sentence is the one that answers the ticket's "and those the
owner sends from the primary": the primary's own client includes **its own companion devices**
in the fan-out, so a message the owner sends from the phone is also encrypted for the
companion. **VERIFIED** for the mechanism.

For groups the whitepaper says the same scalable Sender Key scheme is used, with the server
doing fan-out to participants; the companion receives group traffic by the same push.
**VERIFIED** for the scheme.

### 3.3 What is *not* claimed anywhere

No official source says "a linked device receives **all** new messages", and none says the
opposite. The Help Center's only caveat is the history sentence (§4), and the whitepaper's only
new-message caveat is the device-list staleness handled by backfill (§5.5). **VERIFIED** by
absence across every linked-device article and the whitepaper.

**Answer to the ticket's first bullet.** In steady state, yes: a linked companion receives new
1:1 and group messages, incoming and owner-sent-from-primary, pushed at send time and
independent of the phone. **INFERRED** from a **VERIFIED** mechanism — the mechanism is
documented, a completeness guarantee is not.

---

## 4. The Help Center ambiguity, resolved

The sentence, from [653480766448040](https://faq.whatsapp.com/653480766448040), verbatim:

> **Note**:
>
> - Not all messages and chats are synced to linked devices from your phone. WhatsApp Desktop
>   syncs more message history than WhatsApp Web. To see or search your full history, check
>   your phone.
> - When you delete a message for yourself or for everyone in a chat, the deletion will sync
>   across all online devices where you are logged into WhatsApp.

Read in context, it is about the **history copy**, for four independent reasons:

1. **"synced ... from your phone."** The qualifier names the phone as the source. New messages
   are not synced *from your phone*; they are fanned out by the sender (§3.2). The sentence's
   own wording excludes the live path.
2. **The paragraph it sits in** is entirely about the link-time transfer: the sentence
   immediately before is "your primary phone sends an end-to-end encrypted copy of your most
   recent message history to your newly linked device, where it's stored locally."
3. **The next sentence** — "To see or search your **full history**, check your phone" — states
   the consequence, and it is about history, not about new messages.
4. **"WhatsApp Desktop syncs more message history than WhatsApp Web"** is a statement about
   history volume; it cannot be about new messages, which are not "more" or "fewer".

**INFERRED**, high confidence: the caveat is the history copy. The residual ambiguity is that
WhatsApp never writes "this does not apply to new messages", so a reader determined to read it
as a blanket warning has a grammatically possible reading. The mechanism decides it: history
comes from the phone and is partial; new messages come from the sender and are fanned out to
all devices. **VERIFIED** for every quote; **INFERRED** for the resolution.

---

## 5. Offline gaps — the crux

This is the normal case for this project: an emulator on a Linux host, off most of the time.

### 5.1 The server queues undelivered messages, up to 30 days

WhatsApp Privacy Policy (retrieved 2026-09-26), *Your Messages*. Verbatim:

> We do not retain your messages in the ordinary course of providing our Services to you.
> Instead, your messages are stored on your device and not typically stored on our servers.
> Once your messages are delivered, they are deleted from our servers. The following scenarios
> describe circumstances where we may store your messages in the course of delivering them:
> **Undelivered Messages.** If a message cannot be delivered immediately (for example, if the
> recipient is offline), we keep it in encrypted form on our servers for **up to 30 days** as
> we try to deliver it. If a message is still undelivered after 30 days, we delete it.

**VERIFIED**, verbatim, current policy. The whitepaper states the same delivery model from the
protocol side — "the initiator can immediately start sending messages to the recipient, **even
if the recipient is offline**" — and the engineering blog states that messages are not stored
"after they are delivered", i.e. they *are* stored before delivery. **VERIFIED**.

**The per-device question.** The policy is phrased for "the recipient". In multi-device,
client-fanout makes **each device its own recipient** with its own ciphertext (§3.2), so a
companion being offline while the phone is online should not mark the companion's copy as
delivered. This is **INFERRED** from the architecture, and it is directly supported by
observation 5.2.

### 5.2 Observed: an offline companion is flushed on reconnect

Baileys issue [#2726](https://github.com/WhiskeySockets/Baileys/issues/2726) (third-party,
WhatsApp Web/Desktop client). Verbatim from the report:

> For messages delivered through the **offline queue** after a reconnect ...
> Client was offline for ~8.5h; reconnect at 01:08:20Z.
> ... "handled 85 offline messages/notifications"

**VERIFIED** as a third-party observation. A companion that was offline for hours received the
messages sent in that window, including 1:1 messages, when it reconnected.

Baileys issue [#2810](https://github.com/WhiskeySockets/Baileys/issues/2810) (third-party),
verbatim:

> Messages are not lost server-side: the same message ids keep being redelivered on every new
> connection (40 of 49 distinct ids crossed a container replacement) ... So the server clearly
> still has the batch — it just never sends the offline-complete node.

**VERIFIED** as a third-party observation. The server retains the offline batch and redelivers
it on each reconnect until the client acks it.

**So the crux answer is: messages sent while the companion is off are delivered on reconnect,
not dropped — within the retention window.** **VERIFIED** for the platform's queueing and
retention; **VERIFIED** as observed for a Web/Desktop companion; **INFERRED** for the Android
companion, which uses the same fan-out and queue but whose client I did not measure.

### 5.3 The hard bound: 30 days, twice over

Two independent 30-day rules compound for a device that is off for a long time:

1. **Undelivered messages are deleted after 30 days** (§5.1). Anything sent more than 30 days
   before the companion next connects is gone. **VERIFIED**.
2. **The link itself lapses after 30 days of inactivity** — see §7.1. **VERIFIED** verbatim.

**INFERRED**: a companion off for more than 30 days both loses the queued messages and is
disconnected, so the archive has a permanent hole and then stops receiving entirely. A
companion off for *days* (the "overnight" case in the ticket) is inside both windows and is
safe. The practical rule is therefore: **reconnect well inside 30 days, ideally daily.**

### 5.4 Inside the window, offline delivery can still fail silently

The server delivering the ciphertext is not the same as the client decrypting it. Three
observed failure classes:

- **A message can be lost on the offline flush when decryption fails.** whatsmeow issue
  [#1093](https://github.com/tulir/whatsmeow/issues/1093) reports offline group messages
  arriving undecryptable ("received message with old counter"); the maintainer's reply, verbatim:
  > Old counter means that same message was already decrypted previously, which is why it's
  > ignored. If you interrupt the program right after decryption it can lead to the message
  > being lost because it was decrypted, but not yet dispatched to handlers. ... In practice
  > hitting that should be very rare **unless the program is running on a mobile phone (mobile
  > OSes kill apps all the time)**.

  **VERIFIED** as a third-party report and maintainer reply. The reporter's own framing — "it is
  now not an offline message - it is delivered, so it won't be reemitted next time" — is the
  important part: once the server considers it delivered, a client-side decryption loss is
  **permanent and invisible**. The maintainer's caveat lands directly on this project: the
  capture device is an Android app, and Android kills background apps.

- **Group messages can be permanently undecryptable after an identity change that happens
  while the companion is offline.** Baileys issue
  [#2704](https://github.com/WhiskeySockets/Baileys/issues/2704) documents group messages stuck
  at "Waiting for this message" after a member changes phones, because sender-key distribution
  is not refreshed; it notes the identity-change notification is *skipped when replayed
  offline* ("if (isOfflineNotification) -> return { action: 'skipped_offline' }") and that the
  server-side participant-hash correction is a stub in that client. **VERIFIED** as a
  third-party report. The whitepaper's *Sender Side Backfill* is the official healing path, and
  it is explicitly time-limited: "this backfill mechanism is only allowed **within a short
  duration** after the initial message sending." **VERIFIED** verbatim.

- **The offline-batch handshake can stall and silently ingest nothing.** Baileys #2810
  reports a connection that looked healthy (connection open, no error) yet delivered
  **zero** messages for 8h46m because the server stopped sending the closing offline-complete
  node. **VERIFIED** as a third-party report on an unofficial client; whether the official
  Android client can enter the same state is **UNVERIFIED**. It matters because it is exactly
  the shape of failure this project must detect: a healthy-looking archiver that is silently
  receiving nothing.

### 5.5 The stale-device-list window

Whitepaper, *Sender Side Backfill*, verbatim:

> Any device which is not listed at the sending time will not be able to receive the encrypted
> message.

If a sender's cached device list is stale and omits the companion (most likely right after a
fresh link, before senders refresh), the message is not encrypted for it. The server compares
hashes and tells the sender to resend — but only within the short backfill window quoted above.
**VERIFIED** for the mechanism; the practical frequency is **UNVERIFIED**. This is a
link-time-adjacent gap, not a steady-state one.

**Answer to the ticket's offline bullet.** Messages sent while the companion is off are queued
and delivered on reconnect, up to 30 days, and observed to work for a Web/Desktop companion.
They can still be lost to decryption failures, and after 30 days both the messages and the link
are gone. **VERIFIED / INFERRED** as tagged.

---

## 6. Media

- **The message arrives; the blob is separate.** In the whitepaper, media is transmitted and
  fetched through the same pairwise-session mechanism as messages; the *message* carries the
  media reference and caption. **VERIFIED** for the architecture (whitepaper, *Transmitting
  Media and Other Attachments*).
- **The export carries only media physically on the device.** The export article offers
  "Without media" / "Include media"; #10 established that an export can carry a downloaded
  attachment with its caption. **VERIFIED**.
- **A companion can hold and export media.** On this machine,
  '~/Downloads/WhatsApp Chat with 0 Soong Guan Hui.zip' contains
  'IMG-20260926-WA0000.jpg' with a caption, and 'WhatsApp Chat with Ellia.zip' contains three
  PDFs, each with a '(file attached)' line and a caption continuation line. **VERIFIED** (read
  directly). So media does reach the companion and does reach the export.
- **Auto-download is a client setting, and not everything auto-downloads.** Android exposes
  *Settings > Storage and data > Media auto-download*; the default is per media type and per
  network. The exact defaults on a companion-linked Android install, and whether a companion's
  auto-download differs from a primary's, are **UNVERIFIED** — I found no official statement.
- **Undownloaded media may expire server-side.** Secondary sources claim WhatsApp keeps
  undownloaded media for a limited period (numbers from ~14 to 30 days appear), but I found **no
  primary source** stating an expiry for ordinary (non-forwarded) media. **UNVERIFIED.** If it
  is real, a device that is off for weeks and does not auto-download could find media already
  gone on reconnect — which would be invisible in the transcript, because the line is there and
  only the blob is missing.
- **View-once.** Historically view-once media could not be opened on a linked device at all;
  an Android beta (2.25.3.7, February 2025) added it to companion devices (third-party report,
  Android Authority / WABetaInfo). Whether view-once media can ever reach an *export*, and
  whether it is present locally long enough to be exported, is **UNVERIFIED**.

**Answer to the ticket's media bullet.** Attachments do not arrive as full blobs unless
auto-downloaded or opened; the export carries only local media. The pipeline must either enable
auto-download broadly or accept transcript-only captures for non-downloaded media.
**VERIFIED** for the export's local-only scope; **UNVERIFIED** for the auto-download defaults
and any server-side media expiry.

---

## 7. Link expiry — the silent death

### 7.1 Two documented inactivity rules

Help Center [1317564962315842](https://faq.whatsapp.com/1317564962315842), *How to link a
device*, verbatim:

> You'll need to log in to WhatsApp on your primary phone **every 14 days** to keep linked
> devices connected to your WhatsApp account.

Help Center [378279804439436](https://faq.whatsapp.com/378279804439436), *About linked
devices*, verbatim:

> We'll also automatically disconnect linked devices **after 30 days of inactivity**.

Help Center [1046791737425017](https://faq.whatsapp.com/1046791737425017), *About linking
WhatsApp to a second phone*, verbatim:

> Your companion phones will be logged out if you don't use WhatsApp on your primary phone for
> over 14 days.

**VERIFIED**, verbatim. Two distinct rules:

- **14 days**: driven by the **primary phone** being unused. For this project the owner uses
  their phone daily, so this should not bite — but it is a real dependency on the owner's
  behaviour, and it is about the phone, not the companion.
- **30 days**: "linked devices after 30 days of inactivity." **Whose** inactivity is not stated
  precisely. The natural reading, in a paragraph about reviewing your linked-device list, is the
  **linked device's** inactivity. **INFERRED**. If that reading is right, an emulator off for
  more than 30 days is disconnected and the archiver dies silently.

### 7.2 Who can remove a companion

WhatsApp Security Whitepaper, *Companion Device Removal*, verbatim:

> Companion devices can log themselves out from a WhatsApp account, may be logged out by the
> user's primary device, or may be logged out by the WhatsApp server.

**VERIFIED**, verbatim. So server-side removal is an expected path, consistent with the 30-day
disconnect.

Account-level events also invalidate companions: the whitepaper says that linking a Cloud API
companion "generates a new random Identity Key thereby invalidating all existing companions",
and that during backfill "if a recipient registers on a new phone, all its companion devices
will be excluded from the resending list". **VERIFIED** for those statements; **INFERRED** that
the same invalidation applies to a normal primary re-registration or number change. This matters
because the owner re-registering, changing number, or restoring the phone to a new device can
unlink the companion without any error the archiver notices.

### 7.3 App update and reboot

No official source found says that updating WhatsApp, or rebooting the device, unlinks a
companion. The absence is **VERIFIED**; that it therefore *does not* happen is **INFERRED**
(and is consistent with normal multi-device operation). An app **reinstall**, or clearing app
data, is a different matter — it removes the credentials and would require re-linking — but
that is not an update. **INFERRED**, not tested.

**Answer to the ticket's link-expiry bullet.** A companion link can lapse from the primary
being unused for 14 days, from 30 days of (presumably the device's) inactivity, from the primary
logging it out, from server-side removal, and from account re-registration/number change. App
updates and reboots are not documented triggers. An unattended archiver must therefore monitor
that it is still linked and re-link when it is not — the export will not tell it. **VERIFIED**
for the rules; **INFERRED** for their application to this device.

---

## 8. Non-message events

What actually reaches the export transcript, from the project's own format work (#3, #10) and
the artifacts on this machine:

| Event | Reaches companion? | Appears in export? | Tag |
| --- | --- | --- | --- |
| Deletes | yes | yes — "This message was deleted" / "You deleted this message" | **VERIFIED** (format corpus; Help Center says deletion "will sync across all **online** devices") |
| Edits | yes | yes — a trailing "<This message was edited>" marker (seen in 'WhatsApp Chat with Ellia.zip' on this machine) | **VERIFIED** (artifact) |
| Missed calls | yes | yes — "Missed voice call" / "Missed group voice call" | **VERIFIED** (format corpus) |
| Reactions | yes (as reactions) | **no** — not in the transcript | **VERIFIED** (third-party export-tool limitation + absence in the 50-export corpus) |
| Quoted replies / message relations | yes | **no** — relation is not in the transcript | **VERIFIED** (third-party export-tool limitation) |
| Group metadata (members, admins) | yes | **no** | **VERIFIED** (third-party export-tool limitation) |
| Polls | presumably yes | **UNVERIFIED** — no example found | **UNVERIFIED** |
| View-once media | now yes, on newer companion builds | **UNVERIFIED** — no example found; ephemeral by design | **UNVERIFIED** |
| Call media/content | no (only signalling) | no | **VERIFIED** (whitepaper: SRTP secrets are per-device and in-memory) |

Two consequences worth stating:

- **The archive is a partial social record even when delivery is perfect.** Reactions and quoted
  replies are not in the export at all, so they are lost regardless of delivery.
- **Deletes are the reverse risk.** A delete "will sync across all **online** devices". A device
  that is offline when the delete is issued may not apply it, and may later export the original
  text — so the same message can be present in one capture and absent in the next. That is a
  content difference between captures, not a delivery gap, and it is invisible as such.
  **VERIFIED** for the quote; **INFERRED** for the consequence.

---

## 9. Detection — can a capture tell it missed something?

- **From the export: no.** The transcript has no completeness metadata, no counts, no gap
  marker, no window. This follows from the export format work and from the export article
  documenting none. **VERIFIED** by absence.
- **From the UI: only a per-message placeholder.** "Waiting for this message. Check your
  phone." is shown on a linked device "in place of a message you sent or received" when it
  cannot be decrypted. **VERIFIED** verbatim (Help Center). Whether such a row reaches an export
  as a placeholder or is omitted is **UNVERIFIED** — I have no artifact.
- **The platform knows, but the export does not carry it out.** #14 established that the
  protocol has per-chat completeness state (endOfHistoryTransferType, completeAccessGranted,
  FULL vs RECENT sync types) and that the official export carries none of it. Those signals are
  about **history**, not about live-message gaps, and reaching them requires an unofficial
  client, which is out of bounds. **VERIFIED** (from #14).
- **Operationally, the honest checks are external**: is the app still linked (does it still say
  "This is a linked device"), is it actually connecting, and does the transcript look plausible
  against the phone UI. None of these proves no message was missed. **INFERRED** as the state of
  the art.

**Answer to the ticket's detection bullet.** Nothing in the official capture path lets a
capture detect that it missed messages. The only per-message signal is a UI placeholder whose
export behaviour is unverified. A gap can be found by comparing against the phone, or not at
all. **VERIFIED** by absence.

---

## 10. What this means for the map (#1)

Not a decision — the capture-source decision belongs to
[#11](https://github.com/SoongGuanLeong/footprint-messaging/issues/11). The evidence bounds it:

- The owner's ruling — history irrelevant, forward-only — lands on the **better** of the two
  mechanisms. New-message fan-out is pushed, phone-independent, and server-queued; the
  link-time history copy is the partial one the project has discarded.
- But "forward-only" does **not** mean "complete". The forward archive is only as good as the
  companion's uptime: it needs to reconnect well inside 30 days, and even then it can lose
  messages to offline-flush decryption failures and will not carry reactions, quotes, or
  undownloaded media.
- **No official surface reports new-message completeness**, exactly as #14 found for history.
  So a companion-only capture produces an archive whose gaps are undetectable from the
  artifact.
- The operational requirements this implies (not decisions): keep the companion linked and
  reconnecting on a schedule shorter than 30 days; consider enabling media auto-download;
  monitor link state outside the export; and treat a silently stalled connection as a real
  failure mode.

---

## 11. What remains unknown

1. **Whether a completeness guarantee exists for new messages.** None is published; none is
   denied. The mechanism implies it in steady state; nothing states it. **UNVERIFIED.**
2. **The actual behaviour of an Android app in companion mode**, as opposed to Web/Desktop,
   for offline flush, media, and long-idle reconnection. Every offline observation here is from
   an unofficial Web/Desktop client. **UNVERIFIED**, and the central gap.
3. **The exact meaning of the 30-day linked-device inactivity rule** — the linked device's
   inactivity or the account's — and whether an idle-but-reconnecting emulator is safe.
   **UNVERIFIED / INFERRED.**
4. **Whether ordinary (non-forwarded) media expires server-side** before it is downloaded, and
   after how long. No primary source found. **UNVERIFIED.**
5. **The auto-download defaults on a companion-linked Android install**, and whether an
   unattended device receives full media without being opened. **UNVERIFIED.**
6. **How often offline-flush decryption loss actually happens on an official Android client**,
   and whether Android's background-app killing makes it common (the maintainer's warning
   suggests it is the main risk). **UNVERIFIED.**
7. **Whether the official Android client can enter the "healthy but ingesting nothing" state**
   observed in Baileys #2810. **UNVERIFIED.**
8. **Whether reactions, polls, view-once, and quoted replies can be captured at all through
   official surfaces**, or are permanently outside the export format. **UNVERIFIED** for polls
   and view-once; reactions and quotes are **VERIFIED absent** from the format.
9. **Whether the link survives app updates and device reboots** in practice. No primary source;
   **UNVERIFIED.**
10. **What a fresh, controlled test on this emulator would show** — a message sent while it is
    offline, then reconnect, then export, compared against the phone. This is the one experiment
    that would convert most of the above from INFERRED to VERIFIED. It was out of scope here
    (reading ticket; do not boot the emulator).

---

## Sources

All web sources retrieved 2026-09-26 unless dated otherwise.

**Official (WhatsApp / Meta)**
- *About linked devices* — <https://faq.whatsapp.com/378279804439436>
- *About message history on linked devices* — <https://faq.whatsapp.com/653480766448040>
- *How to link a device* — <https://faq.whatsapp.com/1317564962315842>
- *About linking WhatsApp to a second phone* — <https://faq.whatsapp.com/1046791737425017>
- *Seeing "Waiting for this message. Check your phone."* — <https://faq.whatsapp.com/835452491239734>
- *How to export your chat history* — <https://faq.whatsapp.com/1180414079177245>
- *How WhatsApp enables multi-device capability*, Meta engineering blog, 2021-07-14 — <https://engineering.fb.com/2021/07/14/security/whatsapp-multi-device/>
- *WhatsApp Encryption Overview* (Security Whitepaper), v7, updated 2026-02-25 — <https://www.whatsapp.com/security/WhatsApp-Security-Whitepaper.pdf>
- *WhatsApp Privacy Policy* — <https://www.whatsapp.com/legal/privacy-policy>

**Protocol schemas (corroboration, from #14)**
- whatsmeow WAWebProtobufsHistorySync.proto — <https://github.com/tulir/whatsmeow/blob/main/proto/waHistorySync/WAWebProtobufsHistorySync.proto>
- Baileys WAProto.proto — <https://github.com/WhiskeySockets/Baileys/blob/master/WAProto/WAProto.proto>

**Third-party ecosystem observation (method out of bounds; observations cited as third-party)**
- whatsmeow issue #1093, *Decryption of offline messages problem*, and maintainer reply — <https://github.com/tulir/whatsmeow/issues/1093>
- Baileys issue #2726, offline-flush timestamp corruption, with the offline-queue observation — <https://github.com/WhiskeySockets/Baileys/issues/2726>
- Baileys issue #2810, offline-complete node never arrives; server retains the batch — <https://github.com/WhiskeySockets/Baileys/issues/2810>
- Baileys issue #2704, group sender-key / identity-change stuck at "Waiting for this message" — <https://github.com/WhiskeySockets/Baileys/issues/2704>
- Baileys issue #1965, session logged out after days while phone shows connected — <https://github.com/WhiskeySockets/Baileys/issues/1965>
- chat-export (PyPI), documented export limitations: reactions, message relations, group metadata not included — <https://pypi.org/project/chat-export/0.9.5/>
- Android Authority / WABetaInfo, view-once on companion devices, 2025-02-03 — <https://www.androidauthority.com/whatsapp-view-once-media-linked-devices-3522527/>

**Project record and artifacts**
- Ticket #10 findings — docs/research/linked-device-history.md (branch research/linked-device-history)
- Ticket #14 findings — docs/research/companion-completeness.md (branch research/companion-completeness)
- Ticket #3 export-format findings — docs/research/whatsapp-export-format.md (branch research/whatsapp-export-format)
- Artifacts on this machine: '~/Downloads/WhatsApp Chat with 0 Soong Guan Hui.zip';
  '~/.local/share/Trash/files/WhatsApp Chat with Ellia.zip' (read directly, read-only)

Nothing was modified or deleted on any device or in any artifact. The emulator was not booted or
driven; no device setting was changed; no message was sent. No unofficial client was run or
installed; its published source, issues and documentation were read only.
