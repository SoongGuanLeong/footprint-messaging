# Footprint Messaging

Acquiring a person's own messaging history from the apps that hold it, and preserving it as local evidence. This context covers acquisition only. Understanding the messages afterwards is a separate context that does not exist yet.

## Language

### Sources and devices

**Primary device**:
The phone the person actually uses day to day, holding the account's full history.
_Avoid_: Real phone, main phone, original device

**Linked companion device**:
A second device linked to the same account. It is not the account's holder of record, but it is not a pure forwarder either: at link time the primary device pushes it a partial copy of the most recent history, and it receives new traffic from then on.
_Avoid_: Secondary device, emulator device, slave device

**Capture source**:
The device an export is actually taken from. It is not necessarily the device that holds the account's history, and for this project it is not.
_Avoid_: Source device, origin

### Acquiring

**Export**:
The artefact the messaging app itself produces when a person asks it to hand over one chat. It is the app's own output, not something we extract, scrape, or reconstruct.
_Avoid_: Backup, dump, download

**Capture**:
One export of one named chat, taken and brought to the host, together with everything needed to know where it came from. A capture is the unit of work; an export is only the file inside it.
_Avoid_: Import, sync, ingestion

**Capture identity**:
A capture's identity is its content digest. Two captures are the same capture exactly when their bytes agree, so novelty is decided by one comparison and never by reading messages.
_Avoid_: Capture id, version key, fingerprint

**Chat name**:
The title a person reads in the app's chat list, and the only handle a user or tool can act on to select a chat. It is also what the app puts in the export filename. It is chosen by the people in the chat, is not unique, and can change at any time.
_Avoid_: Chat title, thread name, conversation id

**Stated chat name**:
The name the user gives on the capture command to say which chat was exported. It is a claim, not a fact: it is checked against the chat name WhatsApp wrote, and when the two disagree the chat name wins.
_Avoid_: Target chat, chat argument, requested chat

**Sender label**:
The name written beside each message inside a transcript. It is the sender's app profile name, which is a different thing from the chat name: a chat carries both at once, and the two routinely disagree.
_Avoid_: Author, participant, sender name

**Chat identity**:
Whatever lets two captures be recognised as the same conversation. Deliberately not the same concept as chat name, and deliberately not computed by the store: the archive carries no stable identifier, so this is a derived-layer concern built from the recorded names.
_Avoid_: Chat id, thread key

### Preserving

**Raw evidence**:
The captured artefact exactly as received, never modified, never rewritten, never deleted. Everything the project derives lives somewhere else and can be rebuilt from this.
_Avoid_: Raw data, source of truth, original file

**Content digest**:
A fingerprint of an export's logical contents — its entries and their bytes — independent of the container's own metadata, so re-exporting the same chat yields the same digest.
_Avoid_: Content hash, checksum, file hash

**Artifact hash**:
A fingerprint of the export file exactly as received, container metadata included. It proves the held file is unchanged, where the content digest proves what it contains.
_Avoid_: Zip hash, container hash

**Duplicate**:
A capture whose content digest already exists in the store. It is not written; the store holds evidence about the chat, not a log of runs.
_Avoid_: Repeated capture, re-run

**Provenance**:
The recorded facts about where a capture came from and when, kept beside the raw evidence so a capture can be explained years later without guessing.
_Avoid_: Metadata, audit log

**Capture time**:
When the export was brought onto the host and preserved. The store orders captures by it.
_Avoid_: Import time, download time

**Export time**:
When the messaging app wrote the export, as recorded in the export itself. It is device-local wall-clock with no reliable offset, so it is a record of when, not an instant.
_Avoid_: Snapshot time, created time

**Coverage**:
What a capture can vouch for about the chat it came from, as distinct from what the capture contains. A capture is evidence of its own contents and never a claim about the chat, so coverage is never asserted.
_Avoid_: Completeness, fullness

**Store**:
The local, append-only home of raw evidence and its provenance. It is the user's archive, and it grows; nothing in it is ever thrown away by the project.
_Avoid_: Database, repository, vault
