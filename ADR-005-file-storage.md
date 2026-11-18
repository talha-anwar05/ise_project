# ADR-005: File Intake and Binary Storage Boundary

**Status:** Accepted
**Date:** 2026-11-14
**Deciders:** Project team (all members)
**Related:** ADR-001 (Files module), ADR-002 (metadata and versions in the relational database, audit), ADR-003 (guard for downloads)

## Context

- **ASR-7 (BR-26, BR-09):**
  - Deliverables and Action-item evidence accept untrusted uploads. The server must check each file's extension, actual content type and size (EC-16, AC-10d).
  - URLs must use HTTPS. An HTTP URL is rejected, but a valid HTTPS URL that cannot be reached is still accepted, with a warning (EC-17, AC-10e).
  - Allowed file types and the maximum size are still open (OQ-07).
  - Malware scanning is deferred (D-04), but it must be possible to add it later without redesign.
- **ASR-4 (BR-15, BR-25):** Deliverable versions and evidence are never overwritten, and each upload is audited.
- **ASR-3 / NFR-03:** files are readable only by authorized users of the owning Group. Former members and other Groups are denied.
- **ASR-8:** large binary data must not slow down the database that serves the Dashboard.
- **C-03:** no storage provider is fixed. The development and production environments may differ.

## Alternatives considered

1. **Server-side intake check + private storage behind a Storage port + downloads only through the authorized API.**
2. **Clients upload straight to public storage and share public links.**
3. **Files stored inside the relational database.**
4. **Short-lived signed upload and download links to private storage**, with checks after upload and a holding area for unchecked files.

## Decision

1. **Server-side intake check (Files module)**
   - Every upload is checked on the server before it is accepted:
     - its **extension** against a configurable allowlist;
     - its **actual content type**, detected from the file's contents, which must match the extension;
     - its **size** against a configurable maximum.
   - Allowed extensions and the maximum size come from configuration (OQ-07).
   - URLs are checked by **format only**: an `https://` scheme and a well-formed address. The server does **not** fetch the URL before accepting it. When a URL cannot be reached, the user is warned but the URL is still accepted (BR-09).
   - Checks on the client are allowed for user experience only and are never trusted.
2. **Pre-commit scan hook**
   - The intake process has a **reserved step** after validation and before the version is committed. In this release it does nothing.
   - A malware scanner (D-04) can later be plugged in there. It could reject the file or mark it "pending scan" without changing other modules.
3. **Storage port**
   - Binary content is stored through a **Storage port** with two adapters:
     - **local filesystem** for development;
     - **private object storage** for production.
   - Storage is **never public**.
   - Each stored file gets a **unique, never-reused key per version**. Files are never overwritten in place.
4. **Metadata in the database**
   - File details live in the relational database (ADR-002): owning record, Group, version number, original name, detected type, size, storage key, uploader and timestamp.
   - A new upload creates a **new version linked to the previous one** (BR-15).
   - The upload is audited through the shared audit hook.
5. **Write order**
   - The file is written to storage first, then the version record is committed.
   - If the database commit fails, the stored file is left orphaned and removed by a cleanup job.
   - No storage call happens while a database lock is held.
6. **Downloads through the API**
   - Files are downloaded only through an API endpoint. That endpoint applies the **same guard policy as the owning record** (ADR-003), then streams the content from the Storage port.
   - Clients never receive a direct storage URL.

## Rationale

- **Server-side checks** are required because a client can lie about extension or type (EC-16). Configuration-driven limits let OQ-07 be resolved without code changes.
- **Proxying downloads through the guard** makes file access follow exactly the same rules as other records. NFR-03's "100% denied" therefore covers files too, including Former members. Signed links can be forwarded while they are valid, and public links fail NFR-03 outright.
- **A Storage port with local and object-storage adapters** keeps development simple and leaves the production provider open (C-03).
- **Keeping binaries out of the database** stops versioned files from growing the database and slowing Dashboard queries and backups (ASR-8). Storing files in the database remains the fallback if OQ-07 ends with very small limits.
- **The reserved scan step** satisfies D-04's "addable later without redesign", at almost no cost now.

## Trade-offs / consequences

**Positive**
- Untrusted content is checked once, centrally. File access is as secure as record access.
- Versions are immutable and audited. The database stays lean.
- The storage provider and a future scanner can be added or swapped behind stable interfaces.

**Negative / risks**
- **Two stores can disagree.** A crash between the storage write and the database commit leaves orphaned files. A periodic cleanup job is needed, and it must never delete a file that a committed version references.
- Proxied downloads use server bandwidth and connections. This is acceptable at FYP scale. If load grows, a later ADR could adopt short-lived signed links issued after the guard check.
- Content-type detection depends on a library. Its accuracy must be tested with files that have misleading extensions (AC-10d).
- Backups now cover two stores, the database and file storage, and must be kept consistent. The backup strategy itself is deferred (D-03).
- Until D-04 is implemented, accepted files have not been scanned for malware. This must be stated as a known limitation.
