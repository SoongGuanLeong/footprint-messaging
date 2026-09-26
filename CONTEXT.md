# Footprint Messaging

Acquiring a person's own messaging history from the apps that hold it, and preserving it as local evidence. This context covers acquisition only. Understanding the messages afterwards is a separate context that does not exist yet.

## Language

### Sources and devices

**Primary device**:
The phone the person actually uses day to day, holding the account's full history.
_Avoid_: Real phone, main phone, original device

**Linked companion device**:
A second device linked to the same account, which receives new traffic from the moment of linking and is not the account's holder of record.
_Avoid_: Secondary device, emulator device, slave device

**Capture source**:
The device an export is actually taken from. A capture source is not fixed: it may be the primary device, a linked companion device, or both at different times.
_Avoid_: Source device, origin

### Acquiring

**Export**:
The artefact the messaging app itself produces when a person asks it to hand over one chat. It is the app's own output, not something we extract, scrape, or reconstruct.
_Avoid_: Backup, dump, download

**Capture**:
One export of one named chat, taken and brought to the host, together with everything needed to know where it came from. A capture is the unit of work; an export is only the file inside it.
_Avoid_: Import, sync, ingestion

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
Whatever lets the store tell two captures of the same conversation apart from captures of different ones, given that the chat name cannot. Deliberately not the same concept as chat name.
_Avoid_: Chat id, thread key

### Preserving

**Raw evidence**:
The captured artefact exactly as received, never modified, never rewritten, never deleted. Everything the project derives lives somewhere else and can be rebuilt from this.
_Avoid_: Raw data, source of truth, original file

**Provenance**:
The recorded facts about where a capture came from and when, kept beside the raw evidence so a capture can be explained years later without guessing.
_Avoid_: Metadata, audit log

**Store**:
The local, append-only home of raw evidence and its provenance. It is the user's archive, and it grows; nothing in it is ever thrown away by the project.
_Avoid_: Database, repository, vault
