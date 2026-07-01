# Minime Asset Processing Specification V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Storage
**Parent:** `platform/storage/storage.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines what happens to a binary asset between the moment it is received as an upload and the moment its canonical stored form exists in Object Storage.

Asset Processing is responsible for:

* validating that an upload is actually the kind of file it claims to be
* rejecting unsafe or oversized input before it is decoded
* transcoding accepted input into the canonical stored format defined in `storage.architecture.specification.v1.md` — "Canonical Image Format"
* discarding the original upload once transcoding succeeds

It is not responsible for:

* deciding which domains may upload assets, or their business-level size/dimension policies (owned by the calling domain, per `storage.architecture.specification.v1.md` — "Upload Constraints")
* storing or delivering the processed asset (`asset.lifecycle.specification.v1.md`, `asset.delivery.specification.v1.md`)
* choosing a specific image-processing library (an implementation detail)

---

# 2. Scope

V1 Asset Processing applies only to image uploads: avatar images and Image Block uploads. QR Code assets are system-generated SVG, never user-uploaded, and are not processed by this specification — see `storage.architecture.specification.v1.md` — "Canonical QR Format."

---

# 3. Input Validation

Every upload passes through, in order, before any decoding is attempted:

1. **Declared Content-Type check.** The MIME type declared by the client must be one of the accepted input types for the calling domain (see `07-validation-rules.md` — "File Upload Validation"): `image/jpeg`, `image/png`, `image/webp`, `image/heic`. Any other declared type is rejected immediately.
2. **Magic-byte content sniffing.** The first bytes of the file are inspected to confirm the file's actual binary signature matches the declared Content-Type. A file whose content does not match its declared type is rejected — this is not optional and is not satisfied by trusting the client-supplied Content-Type header or file extension alone.
3. **Size limit.** The file must not exceed the calling domain's configured maximum file size (`07-validation-rules.md`). Oversized uploads are rejected before decoding.
4. **Decompression bomb guard.** Before full decoding, the declared/probed pixel dimensions of the image are checked against a maximum decoded-pixel-count ceiling (implementation-configured). An image that would decode to a pixel count beyond this ceiling is rejected without being fully decoded into memory — this prevents a small compressed file from being used to exhaust server memory.

A file that fails any of these four checks is rejected with a validation error; it is never partially processed, never temporarily stored, and never passed to the transcoding step below.

---

# 4. Transcoding

An input that passes Section 3 is transcoded to the canonical stored format:

* **Output format:** WebP, per `storage.architecture.specification.v1.md` — "Canonical Image Format." This applies uniformly to every accepted input type (JPEG, PNG, HEIC, WebP-in/WebP-out).
* **Animated input:** If the accepted input format is capable of animation (this does not include V1's accepted input set today, since `image/gif` is rejected at Section 3 and none of `image/jpeg`, `image/png`, `image/webp`, `image/heic` are treated as animated inputs in V1) — Minime V1 stores only the first frame as a static WebP image if an animated variant of an otherwise-accepted format is ever encountered. Animated output is never produced. This rule exists so that a future input-format addition does not silently reintroduce animated storage without an explicit decision.
* **Dimension normalization:** the calling domain's configured maximum dimensions (`07-validation-rules.md`) are applied by downscaling (never upscaling) the image proportionally; images already within bounds are not resized.
* **Metadata stripping:** EXIF and other embedded metadata (GPS location, device information, etc.) is stripped during transcoding. Minime does not persist upload metadata beyond what is explicitly modeled on `ImageAsset` (`03-canonical-data-model.md`).

---

# 5. Original Discard

Once transcoding succeeds and the WebP output is durably stored via the Storage Platform, the original uploaded bytes are discarded. They are never separately persisted, never retained for reprocessing, and never recoverable after this point — consistent with `storage.architecture.specification.v1.md` — "Canonical Image Format": "Original uploaded files are temporary processing inputs."

If transcoding fails after Section 3 validation passed (a corrupt-but-plausible file, an unsupported HEIC variant, etc.), the original is discarded and the upload is rejected with a processing error. No partial or fallback asset is stored.

---

# 6. Failure Handling

Processing failures at any stage (Section 3 or Section 4) result in the upload being rejected. The calling domain's existing entity (`ProfileContent.avatar_storage_key`, or the referencing Block) is never updated to point at a failed or partial asset. This is the same "upload → validate → DB swap" ordering defined in `implementation/11-platform-services.md` — "Asset Lifecycle Rules": nothing about a failed processing attempt is ever visible to any domain entity.

---

# 7. Final Canon

Asset Processing is a pure function from (validated input bytes) to (canonical WebP output bytes), with no side effects on Product Data. It has no persistence of its own — persistence begins only after Section 4 succeeds, at which point control passes to Storage's upload/delivery responsibilities defined elsewhere in this Platform Service.
