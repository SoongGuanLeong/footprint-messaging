# Has anyone verified that a companion-linked device's history is complete?

Research notes for wayfinder ticket
[#14](https://github.com/SoongGuanLeong/footprint-messaging/issues/14), child of map
[#1](https://github.com/SoongGuanLeong/footprint-messaging/issues/1), and the input to
*Decide the capture source* ([#11](https://github.com/SoongGuanLeong/footprint-messaging/issues/11)).
Findings only, no parser, no code.

Every claim is tagged:

- **VERIFIED**: read directly from a primary source I quote, or from a protocol file I quote.
- **INFERRED**: consistent with the evidence but not proven.
- **UNVERIFIED**: asserted by a third party, or observed once without reproduction. Not safe
  to design against on its own.

Ticket [#10](https://github.com/SoongGuanLeong/footprint-messaging/issues/10) already settled
that a companion-linked device **does** hold and export real history. This ticket asks the
harder question: whether anyone has established that what it holds is **whole**. It is not
the same question as "does export work", and this file keeps them apart.

---

## Headline

**No one has verified completeness, and the only person found who looked for it found the
companion was missing history.**

- Partiality is not a hypothesis. It is stated by WhatsApp, stated by WhatsApp's own
  engineering blog, encoded in the sync protocol, and **directly observed once** by a person
  who compared a linked device against the primary Android phone and found the linked device
  held *fewer* messages ([whatsapp-web.js #201890](https://github.com/wwebjs/whatsapp-web.js/issues/201890)).
  **VERIFIED** (quotes in section 5.1).
- Nobody — not the unofficial-client ecosystem, not a forensic vendor, not a user — has
  published a check of **an official *Export chat* taken from a companion device against the
  primary phone**. The prior-art tools are live mirrors built by people who accept partiality;
  they measure whether capture *works*, not whether history is *whole*. **VERIFIED** by
  absence across the sources in sections 3 and 5.
- Therefore "the wider ecosystem reports that it works" is true and irrelevant. Export working
  was settled by #10. The ecosystem's reports about *how much* arrives are consistent and
  unflattering: a recent, partial, best-effort window, with numbers that range from three
  months to a year depending on the client. **VERIFIED** for the reports (section 5.2); the exact
  window for this project's device is **UNVERIFIED**.

**Confidence.** High that partiality is real and inherent. High that no companion-export
completeness check exists publicly. Medium on any specific number of days or messages, because
the ecosystem's numbers come from emulated **Web/Desktop** clients, not from an Android app
running in companion mode, and WhatsApp documents that the two differ.

---

## 1. Method, trust levels, and the bias this question is exposed to

Three classes of evidence, in order of trust:

1. **Primary sources** — WhatsApp Help Center articles and WhatsApp's own engineering blog,
   retrieved 2026-09-26, quoted verbatim in section 2.
2. **Protocol definitions** — the WhatsApp protobuf schemas as carried in two independent
   open-source clients, tulir/whatsmeow and WhiskeySockets/Baileys, quoted verbatim in
   section 4. These are the platform's own wire format, recovered; they are not documentation.
3. **Ecosystem observation** — issue threads, tool documentation and code from the unofficial
   multi-device ecosystem, in sections 3 and 5. These are claims by third parties. Their *method*
   is out of bounds for this project (unofficial client); their *observations* are evidence
   about how the platform behaves, and are cited as third-party.

### The bias, stated plainly

The question "does anyone report a companion export being complete?" is structurally
unanswerable by opinion, and the ticket is right to distrust it:

- A person only notices missing history if they **compare the companion against the primary
  phone**. Almost nobody does that, because the companion looks self-consistent. A report of
  "works for me" is therefore a report of **non-detection**, not of completeness.
- The people who *do* notice are the ones who post. So complaints are biased *toward*
  detection, and the absence of complaints is biased *toward* non-detection. Neither direction
  is evidence of completeness.
- The prior-art tools were built by people who only needed **forward capture**. They have no
  reason to measure completeness, and their "coverage" features mean "do I have an anchor to
  ask for more", not "am I whole" (section 3.1). Their silence about completeness is not a clean
  bill of health.
- Vendor and tutorial pages claim "full chat history" with no method at all (section 3.1). Those
  claims fail the comparison test by construction.

What survives this filter is small and is stated in section 5: **one** first-hand comparison, plus
several protocol-level observations that partiality is designed in.

### Scope caveat that applies to almost all ecosystem evidence

whatsmeow, Baileys and whatsapp-web.js all link as **WhatsApp Web / Desktop** clients. This
project links an **Android app in companion mode**. WhatsApp's own article says *"WhatsApp
Desktop syncs more message history than WhatsApp Web"* (section 2.1), so the sync window is
client-type dependent by WhatsApp's own account. Ecosystem numbers are therefore a **bound on
the platform**, not a measurement of this project's device. **VERIFIED** for the client-type
distinction; **UNVERIFIED** for the Android-companion window.

---

## 2. The full set of official statements

### 2.1 History is pushed at link time, and it is partial by design

[faq.whatsapp.com/653480766448040](https://faq.whatsapp.com/653480766448040), *About message
history on linked devices*. Retrieved 2026-09-26. Verbatim:

> Right after you link a device, your primary phone sends an end-to-end encrypted copy of your
> most recent message history to your newly linked device, where it's stored locally. It can
> take a few minutes for your message history to appear on linked devices depending on the
> number of messages in your chats.

> **Note**:
>
> - Not all messages and chats are synced to linked devices from your phone. WhatsApp Desktop
>   syncs more message history than WhatsApp Web. To see or search your full history, check
>   your phone.
> - When you delete a message for yourself or for everyone in a chat, the deletion will sync
>   across all online devices where you are logged into WhatsApp.

**VERIFIED**, verbatim. Three facts: history **is** pushed; the push is **partial by design**;
the **phone remains authoritative** ("To see or search your full history, check your phone").
The same article documents a known bug:

> There is a known issue for some linked devices not displaying up to one year of chat history.
> We're working to fix this as soon as possible. In the meantime, you can still see your chat
> history on your primary device.

**VERIFIED**, verbatim. Note the framing: it is about *displaying* up to a year, it is "for some
linked devices", and it is a bug under fix — not a sync window, and not a completeness
guarantee either way.

### 2.2 The full set of official articles found

All retrieved 2026-09-26, all carrying the same "known issue" banner and the same partial-sync
note where relevant:

| Article | ID | What it adds |
| --- | --- | --- |
| *About message history on linked devices* | [653480766448040](https://faq.whatsapp.com/653480766448040) | The link-time push and the "not all messages and chats are synced" note |
| *About linked devices* | [378279804439436](https://faq.whatsapp.com/378279804439436) | Supported linked devices include **Companion Phones**; 30-day inactivity unlink; the known-issue banner |
| *How to link a device* | [1317564962315842](https://faq.whatsapp.com/1317564962315842) | Link flow; primary phone must log in every 14 days; the known-issue banner |
| *How to export your chat history* | [1180414079177245](https://faq.whatsapp.com/1180414079177245) | No message-count limit; no completeness metadata; "can't be re-imported" |
| *Seeing "Waiting for this message. Check your phone."* | [835452491239734](https://faq.whatsapp.com/835452491239734) | A per-message placeholder on linked devices — a message row whose body is not local |

**VERIFIED** (each read directly). The last two are carried over from #10 and re-confirmed
live; the first three were added here. **No official article found says a companion device's
history is complete, and none declares how far back the push reaches.** **VERIFIED** by
absence across every linked-device article retrieved.

### 2.3 WhatsApp's own engineering blog: "a bundle of the messages from recent chats"

[engineering.fb.com, 2021-07-14, *How WhatsApp enables multi-device capability*](https://engineering.fb.com/2021/07/14/security/whatsapp-multi-device/).
Retrieved 2026-09-26. Verbatim:

> For message history: When a companion device is linked, the primary device encrypts a bundle
> of the messages from recent chats and transfers them to the newly linked device. The key to
> this encrypted message history blob is delivered to the newly linked device via an
> end-to-end encrypted message. After the companion device downloads, decrypts, unpacks, and
> stores the messages securely, the keys are deleted. From that point forward, the companion
> device accesses the message history from its own local database.

**VERIFIED**, verbatim. This is the strongest official statement of *shape*: a **bundle**, of
**recent chats**, transferred **once** at link time, then read from a **local database**. It
matches the Help Center's "most recent message history" and makes "recent, partial, one-shot"
an official design description rather than a bug. It does **not** give a window or a
completeness guarantee. **VERIFIED** for the quote; **UNVERIFIED** for any window.

---

## 3. Prior art: what was built, and what it hit

Nobody found has built *this* pipeline — official *Export chat*, taken from an Android
companion device. The nearest prior art splits into three kinds, none of which answers the
completeness question.

### 3.1 Companion-device archivers (live mirrors)

| Project | Links as | What it says about history completeness |
| --- | --- | --- |
| [SeekoeiD/whatsapp-archive](https://github.com/SeekoeiD/whatsapp-archive) (Baileys + Desktop media harvest) | companion (Baileys) | **Caveats (honest):** *"Linked devices only receive history the phone pushes on link (roughly the recent chats, months not years). Media in that initial history often can't be downloaded any more (media_status=failed); anything live is fine."* |
| [0xSm0ky/whatsapp-archiver](https://github.com/0xSm0ky/whatsapp-archiver) (Baileys) | companion | Captures forward (including deleted/edited/view-once). Makes **no** claim about historical completeness; README does not discuss it. |
| [wacli](https://wacli.sh/) (whatsmeow) | companion | Calls history **"Best-effort"**; ships history coverage, history fill --dry-run, history backfill. Its "coverage" means "has at least one local message, so backfill has an anchor" — **not** "is complete". |
| [adelaidasofia/whatsapp-vault-sync](https://github.com/adelaidasofia/whatsapp-vault-sync) | companion | Claims *"your entire WhatsApp history in Obsidian markdown"* with an example of 2,107 messages spanning 2019 to 2026. Reports **no** comparison against the phone and **no** method. **UNVERIFIED** — this is the "works for me" failure mode in the wild. |

**VERIFIED** for the quoted text (read from each project's own docs). The pattern is
consistent: the people who build companion archivers **assume partiality and capture forward**.
None of them tests completeness, and the one that claims it shows no method.

### 3.2 Bridges

mautrix/whatsapp (the Matrix bridge, built on whatsmeow) treats the link-time history
transfer as a **one-time** event configured before first login. **UNVERIFIED** — stated in
bridge documentation found via search, not read line-by-line here; included only to note that a
long-lived production bridge also treats history as a link-time event rather than a queryable
store.

### 3.3 Forensic workflows — the closest method to this project

An [Elite Digital Forensics 2026 guide](https://elitedigitalforensics.com/whatsapp-forensics/)
describes the same shape as this project, for consented civil matters:

> In consented civil matters, some workflows pair a forensic desktop as a companion and export
> chats through the app once history sync completes. This captures chats received during
> pairing; historical scope depends on what the primary device syncs down.

**UNVERIFIED** — vendor practitioner guide, no measurement, no comparison. But it is notable
that the forensic framing is *"historical scope depends on what the primary device syncs
down"* — i.e. a professional extraction guide declines to claim completeness for a companion
extraction. **VERIFIED** that the guide says this; **UNVERIFIED** as a finding about the
platform.

### 3.4 A contrast worth one line: Signal solved this

[Signal's 2025 "A Synchronized Start for Linked Devices"](https://signal.org/blog/a-synchronized-start-for-linked-devices/)
ships an explicit, user-visible choice to transfer full history at link time. **VERIFIED**
that Signal documents this. It is not WhatsApp evidence, and is included only to show that
"linked device = recent window only" is a **WhatsApp design choice**, not a law of messaging.
**INFERRED** as a contrast.

---

## 4. The protocol has a completeness signal — this is the detection answer

The question "can anything tell whether a chat's history is complete?" has a surprising answer:
**at the protocol level, yes** — and this is better evidence than anything the user-facing docs
provide. The signals live in the WhatsApp protobuf schemas carried by both whatsmeow and
Baileys.

### 4.1 Per-chat completeness state

From whatsmeow's
[WAWebProtobufsHistorySync.proto](https://github.com/tulir/whatsmeow/blob/main/proto/waHistorySync/WAWebProtobufsHistorySync.proto),
the Conversation message:

    optional bool endOfHistoryTransfer = 8;
    optional EndOfHistoryTransferType endOfHistoryTransferType = 11;
    ...
    enum EndOfHistoryTransferType {
        COMPLETE_BUT_MORE_MESSAGES_REMAIN_ON_PRIMARY = 0;
        COMPLETE_AND_NO_MORE_MESSAGE_REMAIN_ON_PRIMARY = 1;
        COMPLETE_ON_DEMAND_SYNC_BUT_MORE_MSG_REMAIN_ON_PRIMARY = 2;
        COMPLETE_ON_DEMAND_SYNC_WITH_MORE_MSG_ON_PRIMARY_BUT_NO_ACCESS = 3;
    }

**VERIFIED**, verbatim from whatsmeow. Baileys' own
[WAProto.proto](https://github.com/WhiskeySockets/Baileys/blob/master/WAProto/WAProto.proto)
carries the same fields and the first three enum values (it lacks value 3). **VERIFIED**.
Two independent client codebases agreeing on this shape is strong evidence it is real wire
format, not a client invention.

This is a **per-chat completeness declaration**: the sender tells the companion whether the
transfer is complete *and whether more remains on the primary*. It is exactly the signal the
map wants, and it is not in the export file.

### 4.2 Sync-phase signals

- HistorySync.HistorySyncType (waE2E): INITIAL_BOOTSTRAP = 0, INITIAL_STATUS_V3 = 1,
  **FULL = 2**, **RECENT = 3**, PUSH_NAME = 4, NON_BLOCKING_DATA = 5, ON_DEMAND = 6,
  NO_HISTORY = 7, MESSAGE_ACCESS_STATUS = 8. **VERIFIED**.
- HistorySync carries chunkOrder and progress (0 to 100). **VERIFIED**.
- HistorySyncNotification carries syncType, chunkOrder, progress, oldestMsgInChunkTimestampSec,
  and peerDataRequestSessionID. **VERIFIED**.
- HistorySyncMessageAccessStatus { optional bool completeAccessGranted = 1; } — a field
  literally named **completeAccessGranted**. **VERIFIED**.
- HistorySyncConfig: fullSyncDaysLimit, fullSyncSizeMbLimit, recentSyncDaysLimit,
  storageQuotaMb, onDemandReady. **VERIFIED**. The existence of a *configurable day limit* is
  itself confirmation that a limit exists and is negotiated.

The existence of **FULL** and **RECENT** as distinct sync types means the platform
distinguishes a full sync from a recent-only sync, and a companion can in principle tell which
it received. **VERIFIED** for the enum; **UNVERIFIED** whether FULL is ever delivered to a
given companion in practice (see section 5.3, where one trace shows FULL six times and another
client only ever gets a recent window).

### 4.3 On-demand backfill exists, per chat

HistorySyncOnDemandRequest (chatJID, oldestMsgID, onDemandMsgCount, oldestMsgTimestampMS, ...)
and FullHistorySyncOnDemandRequest / FullHistorySyncOnDemandConfig (historyFromTimestamp,
historyDurationDays) exist in the schema, with FullHistorySyncOnDemandResponseCode including
**DECLINED_SHARING_HISTORY**. **VERIFIED**. So the platform supports asking the phone for more
history, and the phone can **decline**. No user-facing official article was found documenting
this. **VERIFIED** by absence in section 2.2.

### 4.4 The ecosystem's own completion handling — and its unreliability

Baileys exposes, in
[src/Types/Events.ts](https://github.com/WhiskeySockets/Baileys/blob/master/src/Types/Events.ts):

    'messaging-history.status': {
        syncType: proto.HistorySync.HistorySyncType
        status: 'complete' | 'paused'
        /**
         * progress === 100 was received from the server.
         * when false, completion was inferred via timeout (no more chunks arriving).
         */
        explicit: boolean
    }

**VERIFIED**, verbatim. Two things follow. First, the protocol does emit a completion state.
Second, **the ecosystem has to fall back to inferring completion from a timeout when the
explicit signal is absent** — the comment says so in as many words.

And the older flag is broken. Baileys
[#2005](https://github.com/WhiskeySockets/Baileys/issues/2005), open:

> the [isLatest] value is calculated based on the creds.processedHistoryMessages being empty.
> However, when the history message will be processed, the key and timestamp is added to this
> array, and never removed. This means that from the second history event onwards, isLatest
> will always be false. So, the isLatest flag right now means "is first".

**VERIFIED** as the report's text; the code path is quoted in the issue. A completeness flag
that means "is first" is not a completeness flag.

### 4.5 The Web client actually uses the signal — so the platform treats partiality as real

The clearest evidence that endOfHistoryTransferType is load-bearing comes from
whatsapp-web.js, which reads it to decide whether to ask the phone for more. From
[src/Client.js](https://github.com/wwebjs/whatsapp-web.js/blob/main/src/Client.js):

    async syncHistory(chatId) {
        return await this.pupPage.evaluate(async (chatId) => {
            const chat = /* WAWebCollections.Chat.get(...) */;
            if (chat?.endOfHistoryTransferType === 0) {
                await window
                    .require('WAWebSendNonMessageDataRequest')
                    .sendPeerDataOperationRequest(3, { chatId: chat.id });
                return true;
            }
            return false;
        }, chatId);
    }

**VERIFIED**, verbatim. The check === 0 is COMPLETE_BUT_MORE_MESSAGES_REMAIN_ON_PRIMARY: when
the companion knows more remains, it fires an on-demand request. The Web UI itself offers
"sync older messages" for some states and tells the user to use the phone for others
(section 5.1).

---

## 5. Completeness reports, specifically

This is the section the ticket exists for. It is short, and its shortness is the finding.

### 5.1 The one first-hand comparison found

[whatsapp-web.js #201890](https://github.com/wwebjs/whatsapp-web.js/issues/201890),
*"syncHistory() returns false for an incomplete chat when..."*, opened 2026-08-07 by
studiochapunov. Verbatim from the issue body:

> Chat.syncHistory() returns false even though WhatsApp Web reports that the chat history
> transfer has not ended and **the linked device contains fewer messages than the primary
> Android phone.**

The reproduction records the internal state and the method:

> {
>   "endOfHistoryTransfer": false,
>   "endOfHistoryTransferType": null,
>   "materializedMessages": 1
> }
>
> The phone contains lots of messages in the same chat (more than fifteen are directly
> visible), but fetchMessages({ limit: 3 }) returns only one E2E notification.

And from the follow-up comments:

> In a separate chat, the linked Web client still exposed only a partial history window, but
> the messages already materialized in that window remained fully usable. A document sent
> approximately two months earlier was found through the recovered history and downloadMedia()
> retrieved it successfully... This suggests that partial history should not be treated as an
> all-or-nothing failure.

**VERIFIED** as the reporter's account and method (they inspected the chat model fields and
compared the message count against the phone UI). **UNVERIFIED** as a reproducible measurement
— one account, one session, no numbers for the phone side beyond "more than fifteen".

This is the single most on-point artifact found, and it is worth stating what it does and does
not establish:

- **Does establish:** a companion linked device can hold **fewer** messages than the primary
  phone for the same chat, observed by a person who looked; the platform models this
  per-chat; and the "history sync complete" state the client reports is **not** the same as
  "history complete". The issue says the session *"reports ... the initial history sync
  complete"* while the chat still held a partial window.
- **Does not establish:** how much is missing in general, whether it is ever complete, or
  anything about the **official Export chat** (this is the Web client's in-memory store, not an
  export).

The reporter also surfaced a detail that widens the enum beyond the protobuf:

> the **chat model** enum is distinct from the protobuf enum and currently has six values:
> 0 is COMPLETE_BUT_MORE_MESSAGES_REMAIN_ON_PRIMARY, 2 is INCOMPLETE, 4 is
> COMPLETE_ON_DEMAND_SYNC_BUT_MORE_MSG_REMAIN_ON_PRIMARY, and 5 is
> COMPLETE_ON_DEMAND_SYNC_WITH_MORE_MSG_ON_PRIMARY_BUT_NO_ACCESS. The Web UI sends
> HISTORY_SYNC_ON_DEMAND for model state 0 and, behind feature gates, state 5; state 4 tells
> the user to use the phone, while null falls back to a non-actionable informational message.

**UNVERIFIED** (one reporter's inspection of extracted Web modules), but it is consistent with
the protobuf enum in section 4.1 and adds an explicit **INCOMPLETE** state and a **phone-only**
state. If accurate, the platform has a richer completeness vocabulary than the protobuf alone
shows.

### 5.2 The ecosystem's consistent numbers — none of them a completeness check

These are reports of *how much arrives*, not comparisons against a phone. They are
third-party observations, cited as such.

| Source | Claim | Tag |
| --- | --- | --- |
| whatsmeow [#17](https://github.com/tulir/whatsmeow/issues/17), tulir (maintainer) | *"You get all messages from the past 3 months in history sync events soon after logging in. Other than that, there's no way to get history."* | UNVERIFIED (third-party, but maintainer) |
| whatsmeow [#594](https://github.com/tulir/whatsmeow/issues/594), tulir | *"non-full syncs are 3 months worth of history"*; *"whatsmeow has no control over how it's implemented by whatsapp"* | UNVERIFIED |
| whatsmeow [#1045](https://github.com/tulir/whatsmeow/issues/1045) | *"History sync gets stuck after fixed number of messages"*; a reporter adds *"only a couple months or so of messages per contact"*; Beeper reported affected | UNVERIFIED (closed; tulir doubted it was whatsmeow) |
| [wamcp HISTORY_SYNC_RESEARCH](https://github.com/mmw1984/wamcp/blob/main/docs/HISTORY_SYNC_RESEARCH.md), 2025-12-24 | Goal 600+ days; *"Reality: Cannot reliably get 600+ days — limited by WhatsApp's architecture."* Table: initial full sync **~90 days to 1 year**; on-demand **50 messages at a time**; *"Known bug: Some devices stuck at 90 days"*; re-link required to change config | UNVERIFIED (third-party measurement, undated method) |
| [Baileys history-sync docs](https://baileys.wiki/advanced/history-sync) | *"By default, Baileys connects with a Chrome browser profile, which limits how much history WhatsApp returns on the initial sync."* Full history needs a macOS desktop preset + syncFullHistory | VERIFIED (documented behaviour of a maintained client) |
| [SeekoeiD/whatsapp-archive](https://github.com/SeekoeiD/whatsapp-archive) | *"roughly the recent chats, months not years"* | UNVERIFIED (operator caveat) |

**VERIFIED** that each source says this; **UNVERIFIED** as platform fact for this project's
device. The reports do not agree on a number — three months, 90 days to a year, "months not
years" — which is itself informative: **the window is not fixed and not knowable from
outside.** **INFERRED**.

### 5.3 The counter-signal: FULL syncs do occur, and history can arrive very late

Two traces cut against a simple "you only ever get three months" story, and both are worth
recording because they show the behaviour is inconsistent:

- Baileys [#2452](https://github.com/WhiskeySockets/Baileys/issues/2452) is a companion-mode
  trace in which the client received **FULL six times**, RECENT three times, INITIAL_BOOTSTRAP
  once, etc. — while its **on-demand** requests were accepted by the phone and then never
  answered. **VERIFIED** as the reporter's trace. So FULL is not merely theoretical, and
  on-demand backfill is not reliable.
- Baileys [#2218](https://github.com/WhiskeySockets/Baileys/issues/2218) reports a message
  originally dated **2024-08-31** inserted into a database on **2026-02-13** with source
  history_sync — i.e. old history can materialise **long** after linking, and
  messaging-history.set is a *rehydration* event, not only an at-link one. **UNVERIFIED**
  (third-party production report).

The practical consequence: **you cannot conclude "complete" from elapsed time, and you cannot
conclude "partial" from a short export.** Both are **INFERRED**.

### 5.4 What "works for me" actually tells us

Searching the user-facing web for companion completeness returns mostly tutorials and vendor
pages asserting that history "syncs", plus one recurring real symptom: users seeing a
**partial window** on Web/Desktop and being told to unlink/re-link
([e.g. macReports, 2021](https://macreports.com/whatsapp-web-missing-messages-how-to-fix/)).
**UNVERIFIED** (secondary pages). No user-facing source was found in which someone counted a
companion export against the phone. **VERIFIED** by absence.

This is exactly the bias in section 1: the tutorials are non-detection reports; the
unlink/re-link advice is a detection report that never quantifies anything. Neither is a
completeness check.

---

## 6. Detection: can a capture tell whether it is complete?

- **File-level: no.** The official *Export chat* transcript carries no completeness metadata —
  no window, no counts, no per-chat state. This follows from the export format (#10's format
  work) and from the export article documenting none. **VERIFIED** by absence.
- **Protocol-level: yes, and richer than expected.** Per-chat endOfHistoryTransfer /
  endOfHistoryTransferType (including an explicit "complete but more remains on primary" and,
  per section 5.1, an INCOMPLETE state), HistorySyncMessageAccessStatus.completeAccessGranted,
  the FULL vs RECENT sync types, progress/chunkOrder, and an on-demand request path the Web UI
  itself acts on. **VERIFIED** from two independent client schemas and from the Web client's
  own code.
- **But those signals are not in the export, and reaching them requires an unofficial client**
  — which is out of bounds for this project. **VERIFIED**. So: the platform *knows* per chat
  whether it is complete, and **the official Export chat does not carry that knowledge out**.
- **And the signals are unreliable even where available.** isLatest is broken (Baileys #2005);
  endOfHistoryTransferType can be null when history is in fact missing (whatsapp-web.js
  #201890); completion is sometimes inferred from a timeout (Baileys messaging-history.status);
  on-demand requests can go unanswered (Baileys #2452); and the phone can decline
  (DECLINED_SHARING_HISTORY). **VERIFIED** for each.

**Answer:** a capture taken through the official export **cannot** detect its own partiality
from the artifact. A *protocol-level* completeness signal exists but is outside the official
surfaces and is not trustworthy on its own. **VERIFIED / INFERRED**.

---

## 7. What remains unknown

1. **Whether any companion store is ever complete for a chat.** No source claims it; one source
   observed the opposite. Not proven impossible, simply never shown. **UNVERIFIED.**
2. **The actual sync window for an Android app in companion mode.** Every number in section 5.2
   comes from Web/Desktop emulation, and WhatsApp says Desktop is not Web. This project's device
   type is unmeasured. **UNVERIFIED**, and directly relevant to #11.
3. **Whether the official Export chat omits messages the UI shows, and how often.** #10 left
   the empty-export condition unexplained; nothing here closes it. **UNVERIFIED.**
4. **Whether endOfHistoryTransferType is populated and trustworthy for this project's device**,
   given it can be null and the enum differs between the protobuf and the Web chat model.
   **UNVERIFIED.**
5. **Whether on-demand backfill can ever make a chat complete, and whether it can be driven
   through official surfaces only.** The protocol supports it; no official article documents
   it; the unofficial route is out of bounds. **UNVERIFIED.**
6. **Whether re-linking yields a larger or different window**, and whether it is safe. #10 left
   this out of bounds; it is still untested. **UNVERIFIED.**
7. **Whether a completeness check can be constructed from official surfaces alone** — e.g. by
   comparing export length against the UI, or by using the on-demand UI affordance if it exists
   on Android. This is the constructive question #11 may need, and it is not answered here.
   **UNVERIFIED.**

---

## 8. What this means for *Decide the capture source* (#11)

Not a decision — #11 owns that. But the evidence bounds it:

- The premise "a companion export is real" holds (#10). The premise "it is whole" has **no
  supporting evidence and one direct counter-observation**.
- Partiality is **designed in** (Help Center + engineering blog), **encoded** in the protocol,
  and **observable** (whatsapp-web.js #201890). It is not a transient bug to wait out.
- **No official surface reports completeness**, so a companion-only capture (Option C) produces
  an archive whose completeness is unknowable from the artifact. Whether that is acceptable is
  the decision, and the ticket's own framing already anticipates it.
- Backfill from the primary phone (Options A/B) is the only path that addresses *pre-link*
  history, and #10 left the physical-handset adb pull untested.

---

## Sources

All web sources retrieved 2026-09-26 unless dated otherwise.

**Official (WhatsApp / Meta)**
- *About message history on linked devices* — <https://faq.whatsapp.com/653480766448040>
- *About linked devices* — <https://faq.whatsapp.com/378279804439436>
- *How to link a device* — <https://faq.whatsapp.com/1317564962315842>
- *How to export your chat history* — <https://faq.whatsapp.com/1180414079177245>
- *Seeing "Waiting for this message. Check your phone."* — <https://faq.whatsapp.com/835452491239734>
- *How WhatsApp enables multi-device capability* — <https://engineering.fb.com/2021/07/14/security/whatsapp-multi-device/>

**Protocol schemas**
- whatsmeow WAWebProtobufsHistorySync.proto — <https://github.com/tulir/whatsmeow/blob/main/proto/waHistorySync/WAWebProtobufsHistorySync.proto>
- whatsmeow WAWebProtobufsE2E.proto — <https://github.com/tulir/whatsmeow/blob/main/proto/waE2E/WAWebProtobufsE2E.proto>
- Baileys WAProto.proto — <https://github.com/WhiskeySockets/Baileys/blob/master/WAProto/WAProto.proto>

**Ecosystem observation and prior art**
- whatsmeow issues [#17](https://github.com/tulir/whatsmeow/issues/17), [#302](https://github.com/tulir/whatsmeow/issues/302), [#594](https://github.com/tulir/whatsmeow/issues/594), [#654](https://github.com/tulir/whatsmeow/issues/654), [#1045](https://github.com/tulir/whatsmeow/issues/1045)
- Baileys issues [#2005](https://github.com/WhiskeySockets/Baileys/issues/2005), [#2218](https://github.com/WhiskeySockets/Baileys/issues/2218), [#2452](https://github.com/WhiskeySockets/Baileys/issues/2452)
- Baileys docs and source — <https://baileys.wiki/advanced/history-sync>, src/Types/Events.ts, src/Defaults/index.ts, src/Utils/history.ts
- whatsapp-web.js issue [#201890](https://github.com/wwebjs/whatsapp-web.js/issues/201890) and src/Client.js syncHistory()
- wamcp *History Sync Research* — <https://github.com/mmw1984/wamcp/blob/main/docs/HISTORY_SYNC_RESEARCH.md>
- wacli history docs — <https://wacli.sh/history.html>
- SeekoeiD/whatsapp-archive — <https://github.com/SeekoeiD/whatsapp-archive>
- 0xSm0ky/whatsapp-archiver — <https://github.com/0xSm0ky/whatsapp-archiver>
- adelaidasofia/whatsapp-vault-sync — <https://github.com/adelaidasofia/whatsapp-vault-sync>
- Elite Digital Forensics, *WhatsApp Forensics (2026)* — <https://elitedigitalforensics.com/whatsapp-forensics/>
- Signal, *A Synchronized Start for Linked Devices* — <https://signal.org/blog/a-synchronized-start-for-linked-devices/>

**Project record**
- Ticket #10 findings — docs/research/linked-device-history.md on branch research/linked-device-history
- CONTEXT.md glossary (linked companion device, capture source, export)

Nothing was modified or deleted outside this file. The emulator was not booted or driven; no
device setting was changed; no message was sent. No unofficial client was run or installed;
its published source, issues and documentation were read only.
