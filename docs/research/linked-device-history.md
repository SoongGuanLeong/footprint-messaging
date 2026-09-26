# Can a linked companion device export real message history?

Research notes for wayfinder ticket
[#10](https://github.com/SoongGuanLeong/footprint-messaging/issues/10), child of map
[#1](https://github.com/SoongGuanLeong/footprint-messaging/issues/1). Findings only, no
parser, no code. Every claim is tagged:

- **VERIFIED**: I read it directly from an artifact on this machine, or it is stated
  verbatim in a primary source I quote.
- **INFERRED**: consistent with the evidence, but I did not prove it.
- **UNVERIFIED**: asserted by a third party, or observed once without reproduction. Not
  safe to design against on its own.

## Answer

**Yes, qualified.** A device linked to the account as a companion device does hold real
message history, and WhatsApp's own *Export chat* does write that history into the
transcript. This is now established two independent ways:

1. **Empirically, from an export already on this machine.** `WhatsApp Chat with 0 Soong
   Guan Hui.zip`, taken from `emulator-5554` while companion-linked, contains
   ordinary messages, a received message roughly three months older than the export, a
   media attachment with a caption, and the chat's encryption notice. **VERIFIED** (content
   read directly; device provenance from the project's own record, see caveat below).
2. **By WhatsApp's own documentation.** The Help Center article *About message history on
   linked devices* states that at link time the primary phone sends "an end-to-end
   encrypted copy of your most recent message history" to the newly linked device, where it
   is stored locally. **VERIFIED** (verbatim quote in section 2).

The earlier finding on ticket
[#3](https://github.com/SoongGuanLeong/footprint-messaging/issues/3), that a companion
device exports **only** system notices, is therefore not merely unreproduced — it is
refuted by a primary source and by an artifact. It describes a real, transient condition
(see section 4), not a property of companion linking.

**The qualification, and it is the load-bearing part.** What a companion device holds is a
**partial, most-recent subset** of history, not the account's full history. WhatsApp says
so outright: "Not all messages and chats are synced to linked devices from your phone." It
separately documents a known issue where a linked device fails to display up to one year
of history. So a companion export is a genuine transcript of *some* history, and nothing
inside the export says whether it is complete for that chat. **VERIFIED** (quoted in
section 2).

## 1. Method, and what I did not do

Three classes of evidence, in order of trust:

1. **Primary documentation** — WhatsApp Help Center articles, retrieved 2026-09-26, quoted
   verbatim in section 2.
2. **Export artifacts on this machine** — zips in `~/Downloads` and
   `~/.local/share/Trash/files`, inventoried in section 3 with hashes. These are
   the real bytes; I read the transcripts directly.
3. **The project's own prior record** — the correction comment on ticket #3 and the comment
   on ticket #10. These are claims by an earlier session, not primary sources. I use them
   only for device provenance, which the artifact itself cannot carry.

**I did not boot or drive the emulator in this pass.** `adb devices` reported no
attached device and no `emulator`/`qemu` process was running. The
ticket instruction is to prefer evidence already on disk and only drive the emulator if
that cannot settle the question; it can, so I did not. Two consequences, stated plainly:

- I could **not** re-check the nine exports the earlier findings left in the device's
  `/sdcard/Download`; the device is off. Only one of the nine is present on the
  host (in the Trash) and matches its recorded hash — see section 3.
- I did **not** run a fresh UI-vs-export discriminating test. The correction on #3 says
  that test was already run and passed; I am relying on that record, which is the weakest
  link in the chain. Section 7 says what a fresh run would add.

## 2. What WhatsApp officially says

### 2.1 History is synced to a linked device at link time, and it is partial

[faq.whatsapp.com/653480766448040](https://faq.whatsapp.com/653480766448040), *About
message history on linked devices*. Retrieved 2026-09-26. Verbatim:

> Right after you link a device, your primary phone sends an end-to-end encrypted copy of
> your most recent message history to your newly linked device, where it's stored locally.
> It can take a few minutes for your message history to appear on linked devices depending
> on the number of messages in your chats.

> **Note**:
>
> - Not all messages and chats are synced to linked devices from your phone. WhatsApp
>   Desktop syncs more message history than WhatsApp Web. To see or search your full
>   history, check your phone.
> - When you delete a message for yourself or for everyone in a chat, the deletion will
>   sync across all online devices where you are logged into WhatsApp.

Three facts fall out, and they answer the ticket's core question directly. **VERIFIED**,
verbatim:

1. History **is** pushed to the linked device. "Most recent message history" is copied and
   stored locally on the linked device — it is not merely fetched on demand.
2. The sync is **partial by design**. "Not all messages and chats are synced."
3. The authoritative copy remains the phone. "To see or search your full history, check
   your phone."

This also disposes of a hypothesis recorded in the ticket: that a companion device
"receives new traffic from the moment of linking" and nothing older. The documentation
says the opposite — history is sent *at* link time. **VERIFIED** from the quote above.
(The `CONTEXT.md` definition of "linked companion device" as receiving only
traffic from the moment of linking is therefore **wrong for history**, and is worth
correcting in the glossary.)

### 2.2 A documented bug: linked devices may fail to display up to a year of history

The same article, and repeated at the top of
[faq.whatsapp.com/1046791737425017](https://faq.whatsapp.com/1046791737425017), *About
linking WhatsApp to a second phone*, and
[faq.whatsapp.com/378279804439436](https://faq.whatsapp.com/378279804439436), *About linked
devices*. Verbatim:

> There is a known issue for some linked devices not displaying up to one year of chat
> history. We're working to fix this as soon as possible. In the meantime, you can still
> see your chat history on your primary device.

**VERIFIED**, verbatim. Note what it does and does not say: it is about *displaying* up to
a year of history, it is "for some linked devices", and it is acknowledged as a bug under
fix. This is the closest official statement to the map's phrase "chat lists and avatars
appearing without message bodies". **The exact framing "chat lists and avatars appearing
without message bodies" is not WhatsApp's wording, and I did not find it in any article I
retrieved. UNVERIFIED as an official statement.**

### 2.3 The related "waiting for this message" placeholder

[faq.whatsapp.com/835452491239734](https://faq.whatsapp.com/835452491239734), *Seeing
"Waiting for this message. Check your phone."*. Verbatim:

> At times, you might see the above message on your linked device in place of a message you
> sent or received. Due to end-to-end encryption, you might need to wait for the message to
> arrive on your linked device. This can happen if you or the person you're chatting with
> recently reinstalled WhatsApp or are on an older version.

**VERIFIED**, verbatim. This is a per-message placeholder on a linked device, not a missing
transcript — a third, distinct failure mode from the empty export and the "known issue"
above. It matters because it means a linked device can show a chat with message rows whose
bodies are not present locally. Whether such a row reaches an export transcript as a
placeholder or is omitted entirely is **UNVERIFIED**; I have no artifact.

### 2.4 No official setting moves history onto a linked device

There is **no** user-facing setting or flow to request, resume, or re-run the history sync
for an already-linked companion device. The documentation describes it as an automatic
event at link time. **VERIFIED** by absence from all linked-device articles retrieved.
**INFERRED:** the only supported way to re-trigger it is to unlink and re-link the device —
which logs the device out and was explicitly out of bounds for this research, so it was not
tested.

The official *Chat Transfer* feature
([faq.whatsapp.com/209942271778103](https://faq.whatsapp.com/209942271778103)) is a
phone-to-phone migration, not a companion sync: it requires "Your new phone must not be
registered on WhatsApp until you start the migration on your old phone." **VERIFIED**,
verbatim. It therefore cannot be used to populate an already-linked companion device.

### 2.5 Correction to the earlier findings' transfer-article quote

The #3 findings file quotes faq.whatsapp.com/209942271778103 as saying, verbatim:

> It's not possible to transfer your chat history on linked devices such as WhatsApp on
> Web, WhatsApp on Mac, or WhatsApp on Windows. To transfer your chats, you'll need to use
> your phone

**As retrieved on 2026-09-26, that sentence is not in the article.** The live article is
now about Android-to-Android *Chat Transfer* and contains no linked-device sentence at all.
**VERIFIED** (live retrieval). There is no Wayback snapshot of that URL
(`archive.org/wayback/available` returned `"archived_snapshots": {}`), so
I could not check whether the sentence was there earlier. **UNVERIFIED:** whether the quote
was ever on that page, or was mis-attributed. Either way it is not a current source and
should not be cited as one. It does not affect the answer, which rests on 2.1.

### 2.6 The export article

[faq.whatsapp.com/1180414079177245](https://faq.whatsapp.com/1180414079177245), *How to
export your chat history*. Retrieved 2026-09-26. Still documents the media choice
("Tap **Without media** or **Include media**"), still contains **no message-count limit**,
and adds "Your chat history can't be re-imported because it's a text file and not a backup
file." **VERIFIED**, verbatim. This confirms the #3 finding that the 10,000 / 40,000
message limits were dropped from the live Help Center.

## 3. Evidence on disk

All transcripts below were read directly from the zip payloads with `python3
zipfile`. Hashes are SHA-256 of the whole zip. Paths are host paths, not device paths.

| artifact | zip entry written (device-local) | entries | transcript content |
| --- | --- | --- | --- |
| `~/Downloads/WhatsApp Chat with 0 Soong Guan Hui.zip` | 2026-09-26 12:38 | `.txt` + `IMG-20260926-WA0000.jpg` | **5 lines: a received message dated 6/29/26, a message dated 9/26/26 11:23, the encryption notice, a media line dated 9/26/26 12:37, and a caption continuation line** |
| `~/.local/share/Trash/files/WhatsApp Chat with 0 Soong Guan Hui.zip` | 2026-09-26 11:24 | `.txt` | 3 lines: the same 6/29/26 message, the same 11:23 message, the encryption notice (no media yet) |
| `~/.local/share/Trash/files/WhatsApp Chat with @SoongGuanLeong.zip` | 2026-09-26 11:19 | `.txt` | 3 lines: a self-chat notice dated 8/5/26, the encryption notice, a message "Test" |
| `~/.local/share/Trash/files/WhatsApp Chat with @SoongGuanLeong (1).zip` | 2026-09-26 11:22 | `.txt` | 4 lines: as above plus "Another test" |
| `~/.local/share/Trash/files/WhatsApp Chat with Ellia.zip` | 2026-09-25 18:53 | `.txt` + 3 PDFs | 22 lines of real messages 19–21/09/2026, three `(file attached)` lines with caption continuation lines, one `<This message was edited>` marker |
| `~/.local/share/Trash/files/WhatsApp Chat with Ellia.2.zip` | 2026-09-25 18:53 | identical to above | byte-identical to `Ellia.zip` (same SHA-256) |
| `~/.local/share/Trash/files/WhatsApp Chat with Ellia (8).zip` | 2026-09-26 10:47 | `.txt` | **2 system lines only, zero messages** |

Hashes:

| artifact | bytes | sha256 (zip) |
| --- | --- | --- |
| `0 Soong Guan Hui.zip` (Downloads) | 21404 | `a7ba3933e587319eec09a3ef29ef01007f690eefdcb008116ef7e36e54a22f5d` |
| `0 Soong Guan Hui.zip` (Trash) | 368 | `072739950338dd30d434c60d4f17178d3e440b960cb1a155a49869f923ee21dd` |
| `@SoongGuanLeong.zip` | 388 | `b17a5409cadd25be2453389cd529f20b465fc194a39b0355a6c0baff397919f1` |
| `@SoongGuanLeong (1).zip` | 401 | `70930bfa6d570ca660c48df2f0c97a5e8e70e2b956f85f8905bdf72be1250649` |
| `Ellia.zip` / `Ellia.2.zip` | 8601334 | `80b64e93182c72a7cc0eec4888065ce1bf0dc750c9c05335c87a43a35095b5e5` |
| `Ellia (8).zip` | 320 | `d00c2a333ea2248191aca0dc71b16dd7829923d2dad6cadee04eea0a9dfc3370` |

### 3.1 The load-bearing artifact

`~/Downloads/WhatsApp Chat with 0 Soong Guan Hui.zip`, transcript verbatim (the
`-` character is the transcript's separator; the space before `AM` is
U+202F):

```
6/29/26, 10:02 AM - Chris Soong: Mom is looking for saga key
9/26/26, 11:23 AM - Chris Soong: Testing
9/26/26, 11:24 AM - Messages and calls are now end-to-end encrypted. Only people in this chat can read, listen to, or share them. Learn more.
9/26/26, 12:37 PM - Chris Soong: IMG-20260926-WA0000.jpg (file attached)
Another test
```

**VERIFIED**, read directly from the zip. This single artifact settles several things at
once:

- A companion-linked device's export contains **ordinary messages**, not only system
  notices.
- It contains a **received message roughly three months older than the export**
  (6/29/26 → 9/26/26). Since the sync sends "most recent message history" at link time, and
  the export is three months later, a message this old must have come across in a sync
  rather than being composed on the device. **INFERRED** that it was backfilled; I do not
  know the exact link date, so I cannot prove 6/29/26 predates it.
- It contains a **media attachment with a caption**: `IMG-20260926-WA0000.jpg (file
  attached)` followed by the bare caption line `Another test`. So a companion
  device can produce a **with-media** export, which the #3 findings could never obtain.
  **VERIFIED**.
- Messages sent **after** linking reach the device and are exported: the 11:23 "Testing"
  message and the 12:37 media message. **VERIFIED**.

**Provenance caveat.** The zip carries no device identity (that is itself a #3 finding), so
the *content* is verified but the *source device* is not, from the file alone. Its
attribution to `emulator-5554` rests on the correction comment on #3, which says
a real export of this chat, on that device, taken 2026-09-26, "contains four real lines
including messages from 2026-06-29". The four timestamped lines here match that description
exactly. I treat the attribution as **VERIFIED-as-recorded**: the project's own record
states it, but I did not independently re-derive it from the device. A future session with
the emulator running can close this in one command (section 7).

### 3.2 The empty artifact, and the earlier nine

`Ellia (8).zip` is the only one of the nine exports listed in the #3 findings
that is still on the host. Its hash `d00c2a333ea22481…` matches the findings
table's row for the en-IN run, and its content — two system lines, no messages — matches
the "empty export" headline. **VERIFIED**. The other eight are not on the host and their
device copies could not be reached (device off).

Note the ordering this creates, which is the crux of section 4:

- `Ellia.zip` (8.6 MB, **real messages** and three PDFs) was written 2026-09-25
  18:53.
- `Ellia (8).zip` (320 bytes, **no messages**) was written 2026-09-26 10:47, the
  next morning.

Same chat, same app build, roughly sixteen hours apart: full one day, empty the next. That
is not "companion devices cannot export messages". It is an unstable local store.

## 4. Reconciliation with the earlier findings

The #3 findings file's headline was that every companion export contains only system
notices. That is refuted by:

- the `0 Soong Guan Hui` artifact above (real messages, media, a three-month-old
  message);
- the `@SoongGuanLeong` artifacts (real messages);
- WhatsApp's own statement that history is synced at link time (section 2.1).

The correction comment on #3 already withdrew the headline; this file confirms the
withdrawal with the bytes and adds the primary source the original pass lacked. The #3
findings' **format** work — line grammar, locale variation, the U+202F detail, the zip
layout, the content-provider export path — is untouched by this and still stands.

Why the earlier pass saw empty exports, and why `Ellia (8).zip` is empty while
`Ellia.zip` is not, is **UNVERIFIED**. The candidates, none proven:

- The documented "known issue for some linked devices not displaying up to one year of chat
  history" (section 2.2) — the store was there but not being read.
- The "Not all messages and chats are synced" caveat (section 2.1) — that chat's messages
  were genuinely never synced.
- A transient app-state problem — the local store not yet populated after a reload, or the
  export reading a store that had not been re-indexed.

The map's own note is the right lesson here and is worth repeating: a transcript containing
only system notices is indistinguishable at a glance from a transcript that failed to
export. **The discriminating test is the UI, not the export.** **VERIFIED** as a method;
the specific cause remains **UNVERIFIED**.

## 5. The ticket's sub-questions, answered

- **What docs say about what a linked device syncs.** History, not just new messages: "your
  primary phone sends an end-to-end encrypted copy of your most recent message history to
  your newly linked device, where it's stored locally." It is partial: "Not all messages
  and chats are synced." **VERIFIED**, section 2.1.
- **What is stated about chats appearing without message bodies.** The closest official
  statements are the "known issue … not displaying up to one year of chat history" (2.2)
  and the per-message "Waiting for this message. Check your phone." placeholder (2.3). The
  exact phrase "chat lists and avatars appearing without message bodies" is not WhatsApp's
  wording. **VERIFIED** for the quotes; **UNVERIFIED** for that framing.
- **Whether any official setting or flow moves history onto a linked device.** No
  user-facing setting exists; it is automatic at link time. Chat Transfer is phone-to-phone
  and requires an unregistered target, so it cannot populate a linked companion.
  **VERIFIED** by absence plus the verbatim requirement; the unlink/re-link workaround is
  **INFERRED** and untested.
- **Whether the emulator chats are empty in the UI or hold messages that fail to export.**
  Neither exclusively: some exports from the companion hold real messages, at least one
  does not, and the UI is where the question has to be settled. **VERIFIED** for the
  artifacts; the UI test itself was run by an earlier session, not re-run here.
- **Whether new messages after linking reach the emulator.** Yes. Messages dated 2026-09-26
  11:23 and 12:37 are in the export. **VERIFIED**, section 3.1.
- **Whether old and new messages arrive identically.** No. History arrives once, at link
  time, as a partial copy; new messages arrive live thereafter. **VERIFIED** from the
  documentation wording ("Right after you link a device…"). Whether a partial backfill can
  later be topped up without re-linking is **UNVERIFIED**.
- **Whether the primary Android phone can export over USB and be pulled with `adb`.**
  Mechanically yes: `adb` is not emulator-specific, USB debugging works on a
  physical handset, and if the export is saved to shared storage such as
  `/sdcard/Download` it can be `adb pull`ed without root. But the export
  still has to be driven through the UI and saved via the share sheet (a #3 finding), so
  `adb` only replaces the *pull*, not the *export*. **UNVERIFIED** — not tested on
  a physical device. This is the backfill route the capture-source decision may want, and it
  needs its own experiment.

## 6. What I could not establish

1. **Why the earlier exports were empty, and why `Ellia (8).zip` is empty.** Three
   candidate causes in section 4; none proven.
2. **Whether `Ellia.zip` (the 8.6 MB with-media export) came from the companion
   device or from the primary phone.** Its locale is en-GB and its entry timestamp is
   2026-09-25 18:53, both consistent with the emulator during the #3 locale work, but the
   artifact carries no device identity and it is not one of the nine exports in the #3
   table. Its provenance is **UNVERIFIED**, and I have not used it as evidence for the
   answer.
3. **The exact link date of the emulator**, and therefore whether the 6/29/26 message is
   strictly pre-link. The inference that it was backfilled is **INFERRED**, not proven.
4. **How far back "most recent message history" reaches.** The docs give no window. The
   known issue mentions one year as a display bound, not a sync bound. **UNVERIFIED**.
5. **Whether a companion's store is ever complete for a chat**, and whether the export could
   silently omit messages. Nothing in the export declares completeness. **UNVERIFIED** and
   important — see section 7.
6. **Whether a fresh companion export reproduces the real-message result on demand**, and
   whether the UI matches the export. Not run in this pass (device off).
7. **Whether `adb` against a physical primary handset works in practice**
   (section 5). Untested.

## 7. What a follow-up pass should do

Not decisions — the capture-source decision belongs to ticket
[#11](https://github.com/SoongGuanLeong/footprint-messaging/issues/11). Just the gaps:

1. **Boot `emulator-5554` and re-run the discriminating test on camera**: open
   one chat in the UI, screenshot it, export it, and compare. This closes the provenance
   caveat in 3.1 and gap 6 in one pass. It is read-only and the earlier session did the same.
2. **Re-check the nine exports in `/sdcard/Download`** against the hashes in the #3
   findings table while the device is up. Only one could be checked here.
3. **Test the physical-handset `adb pull` route** (gap 7), since it is the only
   proposed backfill path that does not depend on the companion store being complete.
4. **Decide whether an export that might be silently partial is acceptable** for the capture
   unit. This is the real consequence of section 2.1 and it is a decision, not a research
   question. Ticket #11 should carry it.

## Sources

Retrieved 2026-09-26.

- *About message history on linked devices* —
  <https://faq.whatsapp.com/653480766448040>
- *About linked devices* — <https://faq.whatsapp.com/378279804439436>
- *About linking WhatsApp to a second phone* —
  <https://faq.whatsapp.com/1046791737425017>
- *Seeing "Waiting for this message. Check your phone."* —
  <https://faq.whatsapp.com/835452491239734>
- *How to export your chat history* — <https://faq.whatsapp.com/1180414079177245>
- *How to transfer your chat history* — <https://faq.whatsapp.com/209942271778103>
- Artifacts: the zips listed in section 3, on this machine, hashed there.
- Project record: correction comment on
  [#3](https://github.com/SoongGuanLeong/footprint-messaging/issues/3#issuecomment-5843372240)
  and comment on
  [#10](https://github.com/SoongGuanLeong/footprint-messaging/issues/10#issuecomment-5843372779).

Nothing was modified or deleted on any device or in any artifact. The emulator was not
started, so no device setting was changed. No message was sent. Raw evidence files were
read only.
