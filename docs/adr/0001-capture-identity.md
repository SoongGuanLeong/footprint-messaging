---
status: accepted
---

# Capture identity is the artifact's content digest; the store computes no chat identity

A capture's identity is its whole-archive `content_sha256` — entry names plus the hash of each entry's bytes — so two captures are the same capture exactly when their bytes agree, and the acquisition command never parses the transcript to judge novelty. The store therefore computes no chat identity: the archive carries no stable chat identifier, and the display title is neither unique nor stable, so grouping captures into conversations is a derived view over the sidecars, owned by the parent project. An exact duplicate is skipped and reported, never written a second time.

## Considered options

- **A normalized, message-level digest** as the identity key, so a re-stamped system notice or an attachment arriving later would not read as change. Rejected: normalization means parsing the transcript, and parsing is out of scope for acquisition — the destination stops at the preserved artifact. It would also make identity a judgment about messages rather than a fact about the held file.
- **An in-store `chat_id`**, from the WhatsApp-written title or from the per-chat LID visible in the share intent. Rejected: no stable identifier is in the archive at all, the title was already ruled out as an identity, and the LID is a device-level fact that is not part of the artifact. Either key silently merges or splits conversations.
- **A `previous_capture` link and a relative `coverage` value** ("nothing was lost since the previous capture"). Rejected: it needs to know which capture is the previous one for *this* chat — the chat identity declined above — and it is a claim about the chat, not the artifact, which nothing in an export can support.

## Consequences

- A re-export whose only difference is a re-stamped system notice is a **new capture**. Accepted: it is a real difference in the artifact. If real re-exports show it happening routinely, the fix is a derived-layer key, never a parser in the command.
- Because an export may be capped to the most recent N messages, a later capture is **not necessarily a superset** of an earlier one; append-only storage keeps both, and neither replaces the other.
- The duplicate check is global across the store, not per-chat. Safe because identical bytes are the same artifact; a false merge would require two chats to share byte-identical transcripts, which would also mean sharing entry names, i.e. the chat title.
- `coverage` keeps `"unverifiable"` as its only legal value; no capture asserts anything relative to another.
