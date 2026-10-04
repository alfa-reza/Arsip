# Havk CTF MCP — Forensics & Steganography Toolset / Semantic API

**Research draft v0.3 — 2026-10-04**  
Status: architecture/toolset research for PRD, not yet implementation contract.

This document expands `havk-ctf-mcp-toolset.md` v0.2 after reviewing the supplied MCP implementations, the MCP 2026-07-28 protocol and the official Rust SDK (`rmcp`), current upstream projects, recent CTF write-ups, and forensic/steg research/tooling.

---

## 1. Executive decision

The previous 21-tool semantic surface is too coarse for an agent that must investigate evidence iteratively. `pcap_analyze`, `stego_detect`, `image_analyze`, and `document_analyze` each hide too many distinct workflows behind a single schema. The better reference among the supplied MCP projects is **Wireshark-MCP**, not the forensics module in `ctf-buster`: it exposes narrow, composable operations (packet listing, details, bytes, fields, search, stream following, object export, statistics, protocol analysis, anomaly analysis) and bounds output/pagination rather than returning a giant one-shot report.

The recommended design is therefore:

```text
Havk MCP server (Rust / rmcp)
        │
        ├── typed semantic tools
        │     ├── small structured result
        │     ├── warnings / confidence / provenance
        │     └── ArtifactRef / ResourceLink for large output
        │
        ├── artifact registry (durable, connection-independent)
        │
        ├── task/job registry (durable, connection-independent)
        │
        └── isolated workers
              ├── native Rust primitives
              ├── Rust crates
              └── pinned upstream CLI backends
```

The backend policy remains **two orthogonal dimensions**:

- **Image profile:** `core` vs `full` — controls footprint/domain coverage.
- **Trust tier:** `main` vs `experimental` — controls production confidence.

A third, optional dimension should be added for **MCP tool exposure**:

- `--tool-profile ctf` (default)
- `--tool-profile stego`
- `--tool-profile network`
- `--tool-profile disk`
- `--tool-profile windows`
- `--tool-profile memory`
- `--tool-profile all`

This is not dynamic discovery based on evidence. The tool set is selected when the server instance starts and remains deterministic for that instance. MCP can notify list changes, but Havk should avoid evidence-dependent tool mutation because stable lists are easier to cache, reason about, test, and budget for prompt size.

---

## 2. What the supplied MCP source code teaches us

### 2.1 `ctf-buster`: useful workflow idea, too coarse for Havk

The supplied `ctf-buster` exposes only five forensic tools:

- `forensics_file_triage`
- `forensics_stego_analyze`
- `forensics_extract_embedded`
- `forensics_entropy_analysis`
- `forensics_image_analysis`

Its implementation runs external commands such as `file`, ExifTool, Binwalk, `strings`, zsteg, steghide, and Foremost, then returns one large JSON/text response. It also has image paths that normalize images through Pillow/RGB for some analyses. That is convenient for a human helper, but it is not ideal as Havk's contract because:

1. evidence transformations can erase raw representation details important to steg (palette indices, sub-8-bit samples, 16-bit samples, local GIF palettes);
2. one-shot output grows quickly and is harder for an agent to page/search;
3. extracted files are not modeled as durable artifacts with provenance;
4. long operations are not naturally represented as durable MCP tasks;
5. a backend's CLI vocabulary leaks too directly into the semantic layer.

`ctf-buster` is still a useful baseline for what a first-pass CTF workflow expects, but not a suitable API shape to copy.

### 2.2 `Wireshark-MCP`: the stronger architectural reference

The supplied Wireshark-MCP is much closer to the target architecture. Important patterns to retain:

- semantic tools are narrower than a single `pcap_analyze` mega-tool;
- subprocesses are invoked with argument arrays rather than shell command strings;
- paths and arguments are validated;
- output is bounded and pageable;
- timeout/cancellation kills and reaps children;
- stderr is drained concurrently;
- large tabular output is accumulated with row/byte ceilings;
- results are normalized into an envelope instead of returning raw CLI text;
- cache keys include evidence identity/mtime/arguments;
- sensitive key material can be redacted;
- a protocol tool can accept a semantic `protocol` enum instead of making the model guess tshark fields.

For Havk this generalizes to:

```text
MCP tool
  ↓
semantic request
  ↓
policy + resource budget
  ↓
worker backend adapter
  ↓
normalized observation(s)
  ↓
artifact registry
```

not:

```text
MCP → arbitrary shell → stdout
```

### 2.3 `CTF-MCP`: another warning about over-normalization

The supplied `CTF-MCP` forensics implementation contains convenient helpers for magic signatures, EXIF-like inspection, appended-data checks and LSB extraction, but it also illustrates why Havk should own raw-format parsers for steg-critical formats. Generic image decoding is useful for visual transforms, but not as the source of truth for palette/index/sample-level steganalysis.

### 2.4 `ctfd-mcp-server`

This project is mostly a CTFd service API MCP and is not a forensic backend reference. Its value is primarily organizational (server configuration/state/API handling), not evidence parsing.

---

# 3. MCP 2026-07-28 / Rust `rmcp`: design consequences

## 3.1 Version target

Use the stable MCP **2026-07-28** protocol and current stable `rmcp` 3.x. As of this research date the latest GitHub release is `rmcp-v3.4.0` (2026-09-15). The Rust SDK README states support for the stable 2026-07-28 protocol, and its roadmap reports 100% date-versioned server/client conformance for 2025-11-25 and 2026-07-28.

Primary references:

- https://github.com/modelcontextprotocol/rust-sdk
- https://github.com/modelcontextprotocol/rust-sdk/releases
- https://github.com/modelcontextprotocol/rust-sdk/blob/main/ROADMAP.md
- https://github.com/modelcontextprotocol/rust-sdk/discussions/969
- https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/blog/content/posts/2026-07-28-spec-ga/index.md

## 3.2 Use typed tools, not CLI-shaped tools

`rmcp` supports `#[tool]`, `#[tool_router]`, `#[tool_handler]`, Serde and `schemars` JSON Schema 2020-12. Therefore every public tool should have a stable typed input/output structure.

Good:

```rust
struct PcapFollowParams {
    artifact_id: ArtifactId,
    transport: Transport,
    stream_id: u32,
    representation: StreamRepresentation,
    max_bytes: Option<u64>,
}
```

Bad:

```rust
struct RunTsharkParams {
    args: Vec<String>,
}
```

The model should choose forensic intent. The adapter chooses exact CLI flags.

## 3.3 `structuredContent` should be small; artifacts should carry large results

MCP 2026-07-28 permits flexible output schemas and structured content. Havk should return:

- concise observations;
- typed statistics;
- warnings/confidence;
- child artifact references;
- optional `ResourceLink` entries.

Large binary data, extracted files, reconstructed streams, generated bit planes, rendered pages, memory dumps, SQLite exports, and timelines should **not** be inlined into normal tool text.

Recommended result shape:

```json
{
  "status": "ok",
  "backend": {
    "name": "tshark",
    "version": "..."
  },
  "observations": [
    {
      "kind": "dns_exfil_candidate",
      "confidence": 0.87,
      "summary": "High-entropy labels across 183 DNS queries"
    }
  ],
  "artifacts": [
    {
      "id": "art_01K...",
      "uri": "havk://artifact/art_01K...",
      "mime_type": "application/octet-stream",
      "size": 14822,
      "sha256": "..."
    }
  ],
  "truncated": false
}
```

## 3.4 Stateless HTTP changes how state must be stored

MCP 2026-07-28's Streamable HTTP model is stateless at the protocol core. Therefore:

- do not bind evidence state to a connection/session object;
- `artifact_id` must resolve through a workspace-level registry;
- `job_id`/task state must survive request boundaries;
- worker results should commit atomically into the artifact store;
- authorization/workspace identity is separate from transport connection identity.

This also makes stdio and Streamable HTTP share the same domain model.

## 3.5 Long-running work: MCP Tasks first, job fallback second

The official Tasks extension allows a server to turn a `tools/call` into an asynchronous task and exposes `tasks/get`, `tasks/update`, and `tasks/cancel`. Task creation is server-directed when the client declares support.

Use it for operations such as:

- deep carving;
- password/key search;
- PhotoRec recovery;
- Plaso timeline creation;
- Volatility scans/dumps;
- bulk_extractor;
- Zeek processing of large PCAPs;
- mobile full-report generation.

Because not all MCP clients support Tasks yet, keep a **fallback** `job` semantic tool, but do not make callers choose Task vs sync manually. The server decides from cost estimates and negotiated capabilities.

## 3.6 Tool annotations

Set MCP annotations consistently:

- read-only evidence inspection: `readOnlyHint=true`, `destructiveHint=false`, `idempotentHint=true`;
- repair/extraction still never mutates source evidence; it creates child artifacts, so it is non-destructive with respect to evidence;
- network access is normally disabled, so `openWorldHint=false` for forensic tools;
- annotations are hints, not security controls.

## 3.7 Logging and stdio

For stdio transport, stdout must remain MCP protocol output. Worker/tool diagnostics go to stderr or structured internal logs.

---

# 4. Execution/security model

## 4.1 Same binary, isolated worker mode

Recommended layout:

```text
havk-mcp serve --transport stdio --profile core --tool-profile ctf
havk-mcp serve --transport http  --profile full --tool-profile all
havk-mcp worker --backend qpdf ...
havk-mcp worker --backend native-png ...
```

The server is an orchestrator. Untrusted parsers should run in a child worker even if they are Rust crates.

Reasons:

- parser panic must not kill MCP;
- malformed files can trigger OOM;
- zip/decompression bombs must be bounded;
- some crates read full input into memory;
- C/C++ tools must be isolated;
- cancellation becomes process-group termination rather than cooperative hope.

## 4.2 Worker budget

Each invocation receives policy such as:

```text
wall timeout
CPU seconds
address-space / RSS limit
max output bytes
max child artifacts
max total extracted bytes
max recursion depth
max decompressed ratio
max file count
network disabled by default
read-only evidence mount
writable scratch/output directory only
```

Recommended cost classes:

| Class | Typical operations | Default execution |
|---|---|---|
| A | hashes, magic, strings preview, metadata summary | sync, 2–5 s |
| B | OCR, barcode, PDF inspection, PCAP query | sync if small, 10–30 s |
| C | extraction, Binwalk, stream export | task/job above threshold |
| D | recovery, crack, timeline, memory, mobile report | task/job |

## 4.3 Immutable evidence/artifacts

No public forensic operation modifies its source.

```text
art_A (broken.png)
   │
   └── file_repair
          ↓
       art_B (repaired.png)
       parent = art_A
       repair_log = [...]
```

Every artifact records at least:

- id;
- parent(s);
- SHA-256;
- byte size;
- MIME / identified type;
- origin tool + operation + backend version;
- source offsets where relevant;
- creation timestamp;
- workspace-relative storage path (internal only).

---

# 5. Evidence ranking used for backend selection

A backend is not promoted merely because it is Rust/Go or popular on GitHub.

**Strongest signals:**

1. real production / institutional use;
2. mature upstream lineage and maintainers;
3. public hostile-input/fuzz/corpus testing;
4. active maintenance/security fixes;
5. repeated real CTF/DFIR use;
6. reproducible CLI / structured output / batch mode;
7. signed/checksummed official releases;
8. stars/forks/adoption as supporting signals.

A rewrite with weak adoption stays `experimental`, while a rewrite with strong lineage and production use can immediately become `main` (YARA-X is the canonical example).

---

# 6. Recommended backend catalog

Legend:

- **C/M** = core / main
- **C/E** = core / experimental
- **F/M** = full / main
- **F/E** = full / experimental
- **DEV** = oracle/testing only, not runtime target
- **OPT** = optional specialized image/bundle

## 6.1 Native Havk Rust primitives

These are small enough to own directly and should be **C/M** after corpus tests.

| Component | Capability |
|---|---|
| byte/magic engine | magic signatures, extension/MIME mismatch, truncated headers |
| hashing | SHA-256 by default; MD5/SHA-1 only when compatibility/evidence matching needs them |
| string scanner | ASCII, UTF-8, UTF-16LE/BE, configurable minimum, offsets |
| entropy scanner | whole-file + block entropy with boundary candidates |
| hex/byte window | bounded byte ranges around offsets |
| appended-data detector | PNG IEND, JPEG EOI, GIF trailer, PDF EOF, ZIP structures etc. |
| simple polyglot detector | competing valid signatures/structures |
| raw carver | Aho-Corasick multi-signature scan + bounded extraction |
| PNG raw parser | chunk sequence, CRC, IHDR, PLTE/tRNS, IDAT boundaries, IEND, trailing bytes |
| APNG parser | acTL/fcTL/fdAT + frame extraction without flattening |
| GIF raw parser | global/local palettes, image descriptors, frame disposal, palette index preservation |
| JPEG marker parser | SOI/EOI, APPn, COM, DQT/DHT/SOS marker inventory, trailing bytes |
| simple file repair | PNG CRC/IHDR/IEND, ZIP EOCD/header cases; always child artifact |
| raw image steg | channel/sample planes, palette index analysis, bit planes, difference/xor |
| text steg | zero-width, variation/bidi controls, homoglyph flags, whitespace encodings |
| audio primitives | PCM statistics, FFT/spectrogram, Goertzel DTMF, basic PCM LSB |
| CTF network decoders | USB-HID keyboard, DNS-label reconstruction, ICMP payload extraction |

### Why add APNG/GIF raw parsers?

Recent CTFs demonstrate that container representation itself carries the secret. An openECSC 2025 challenge required understanding APNG structure, while SAS CTF 2025 used GIF local palettes in a way generic image libraries could obscure. This is exactly the type of evidence Havk should preserve rather than normalize to RGB early.

References:

- https://ctf.zeba.dev/2025/openecsc/stego/calamansi/writeup/
- https://ctftime.org/writeup/40254

---

## 6.2 File identification and signature analysis

### YARA-X — **C/M**

**Use:** signature/rule matching, structural modules, Havk CTF ruleset.  
**Why Main:** official successor line from VirusTotal/YARA maintainers; Rust implementation; production pedigree; rapid releases. YARA-X 1.21.0 was published 2026-09-29 via trusted publishing/attestation on PyPI even though GitHub's release page indexing may lag.

Recommendation: integrate as worker/library rather than exposing a `run_yara` tool. Public semantic tool is `file_scan`.

Upstream:
- https://github.com/VirusTotal/yara-x
- https://pypi.org/project/yara-x/

### Magika CLI — **C/M**, second opinion only

**Use:** probabilistic content classification.  
**Why Main:** Google production lineage, broad adoption, ICSE research, official Linux x64/arm64 prebuilt assets with SHA256/attestation.  
**Constraint:** never override deterministic magic/structural evidence silently; report confidence and disagreement.

Upstream:
- https://github.com/google/magika
- https://github.com/google/magika/releases

### Traditional `file`/libmagic

Do **not** make it the authoritative architecture dependency if Havk already owns common magic parsing. It can remain a development oracle or compatibility backend. Magika + Havk structural identification provides more useful agent-facing information.

---

## 6.3 Metadata

### ExifTool — **C/M**

Still the default. Its breadth and long-tail format knowledge outweigh the Perl runtime cost.

### OxiDex — **C/E**

Promising Rust ExifTool-like rewrite, but adoption/reputation is far below ExifTool. Run differential corpus tests; do not promote based on README benchmarks.

### Deark — **F/M specialist**

Reconsider the earlier decision to discard Deark. It is not merely another Binwalk: it focuses on obscure/legacy file-format decoding, metadata, thumbnails, embedded assets, APNG frame extraction and conversion. It has been maintained for many years and is still active in 2026. Its own documentation warns that it is C code handling untrusted files and that resource limits are imperfect, so it must run in the worker sandbox with `-maxfiles`, `-maxfilesize`, total-size limits and restricted modules.

Useful niche: files that common CTF tools identify but cannot meaningfully unpack/render.

Upstream:
- https://github.com/jsummers/deark
- https://github.com/jsummers/deark/blob/master/technical.md

---

# 7. Steganography backend set

## 7.1 PNG/BMP LSB

### zsteg — **C/M transitional oracle**

Keep it initially because it is repeatedly used in current CTFs and its search space is a useful compatibility target. But Havk should not treat every `zsteg -a` line as evidence: 2026 write-ups explicitly show hundreds of false-positive-looking hits.

Recent use:
- CTF@CIT 2025: LSB extraction directly revealed flags.
- Hacktheon Sejong 2025: `b1,rgb,lsb,xy` extracted another PNG.
- 2026 write-ups document noisy `zsteg -a` false positives.

Runtime target: phase zsteg out after native parity.

### Havk raw LSB engine — **C/E → C/M after parity**

Must preserve raw PNG samples. Do not base authoritative steg extraction on a high-level image conversion that expands palette/sub-8-bit/16-bit samples before analysis.

Minimum search dimensions:

- sample bit depth: 1..4 initially;
- channel/order: R/G/B/A permutations and palette indices;
- bit significance: LSB/MSB;
- traversal: XY/YX, row direction options;
- packing: bit/byte endian variants;
- skip/offset;
- signature/text scoring;
- artifact extraction when a stable header/format is detected.

## 7.2 Bit planes, palette and frame stego — **native C/M**

Semantic support must include:

- channel isolation;
- individual bit planes;
- palette-index visualization;
- global/local palette inventory;
- GIF frame extraction;
- APNG frame extraction;
- frame difference/XOR;
- alpha-only visualization;
- histogram and palette usage statistics.

This replaces the useful parts of StegSolve for a non-GUI agent.

Use `gifsicle` as a **C/M external compatibility/frame backend** where useful; native raw parsing remains the evidence source.

## 7.3 JPEG steganography family

The previous draft is too weak here. JPEG DCT stego is not covered by zsteg.

### StegSeek — **C/M**

Fast steghide-compatible extraction/cracking. Narrow, established CTF utility.

### steghide — **C/M compatibility**

Keep for compatibility/info/extraction behavior that StegSeek does not necessarily mirror.

### OutGuess — **C/M**

Promote from “later” to core/main. A 2026 CTF write-up shows OutGuess recovering the payload after steghide and jsteg attempts failed. It covers a distinct JPEG DCT method and is still operationally relevant.

Upstream: https://github.com/resurrecting-open-source-projects/outguess

### jsteg — **C/M specialist**

Small Go implementation with prebuilt-friendly characteristics and a different JPEG embedding family. It is cheap coverage and appears in current stego workflows even when not the winning tool.

Upstream: https://github.com/lukechampine/jsteg

### OpenStego — **F/M**

Active, established project for data hiding/watermarking. Java footprint makes it inappropriate for minimal core, but it is valuable for algorithm-specific compatibility that a generic native detector will not recover.

Upstream: https://github.com/syvaidya/openstego

### Aletheia — **OPT: `stego-ml`**

Peer-reviewed steganalysis toolbox with ML/statistical methods. Do not place its Python/model stack in normal `full`; make a separate optional image/profile. It is useful for advanced/JPEG/adaptive steganalysis rather than basic extraction.

Upstream: https://github.com/daniellerch/aletheia

## 7.4 Statistical steg detection

Native implementations:

- chi-square: **C/E**, cheap and useful heuristic;
- RS analysis: **C/E**;
- Sample Pair Analysis: **C/E**.

RS/SPA must return probabilistic evidence, not “stego=true”. Published work and later evaluations show non-trivial false alarms. Use StegExpose/Aletheia/reference implementations and synthetic corpus as oracles, not as unquestioned truth.

Recommended result:

```json
{
  "method": "rs",
  "applicability": "lsb-replacement-like",
  "score": 0.71,
  "confidence": "medium",
  "limitations": [
    "Not reliable for arbitrary/adaptive embedding",
    "May false-positive on natural image statistics"
  ]
}
```

## 7.5 Text steganography — **native C/M**

Cover:

- zero-width characters;
- variation selectors;
- bidi controls;
- NBSP/non-breaking whitespace;
- trailing-space/tab binary encodings;
- Unicode normalization deltas;
- suspicious homoglyph mixtures.

No runtime need for `stegsnow` once parity is tested.

---

# 8. Image / visual analysis

## Main

- raw PNG/GIF/APNG/JPEG structure: Havk native;
- visual transforms: Rust image stack where representation loss is acceptable;
- OCR: **Tesseract C/M**;
- barcode/QR: **zxing-cpp C/M**;
- generated bitplanes/frames: immutable child artifacts.

## Experimental replacements

- `rxing`: **C/E**; promising Rust port but not yet the default over ZXing-C++.
- `ocrs`: **C/E**; attractive Rust OCR, but upstream still describes it as early-stage relative to Tesseract coverage.

## ImageMagick — **F/M fallback**

Keep in full for obscure conversions/transformations. Do not make it the first backend for simple pixel operations that Havk can perform deterministically.

## libwebp tools — **F/M specialist**

Build current libwebp source and expose `webpmux`-equivalent operations for WebP container/chunk/frame inspection. This closes a format gap without routing all images through ImageMagick.

---

# 9. Audio and media steg/forensics

## FFmpeg / ffprobe — **C/M**

Primary decoder/demuxer/stream extractor. Use upstream source release in a builder when no official Linux binary is provided by the project.

Capabilities:

- stream/container metadata;
- audio extraction to bounded PCM;
- video frame extraction;
- attachment/data stream extraction;
- spectrogram fallback (`showspectrumpic`);
- codec/container normalization into analysis-friendly child artifacts.

Recent CTF write-ups continue to hide flags in audio spectrograms, so spectrogram generation is a core capability, not a niche extra.

## Native Havk audio — **C/M after tests**

- waveform statistics;
- FFT/spectrogram renderer;
- DTMF via Goertzel;
- channel diff;
- PCM LSB extraction;
- silence/tone segment detection.

## multimon-ng — **F/M**

Use for radio/modem schemes beyond DTMF.

## minimodem — **F/M**

Add it. It gives batch-friendly FSK modem decoding (Bell/RTTY/TDD/etc.) and is more realistic for signal-style CTF audio than trying to implement every modem in Havk.

## SSTV decoder — **F/E**

Useful CTF capability, but the common Python projects are materially less mature/reputable than FFmpeg/multimon. Keep it experimental until a golden corpus is established.

---

# 10. Archives, compression and carving

## 7-Zip / `7zz` — **C/M**

Use official upstream Linux x64/arm64 binary where available, pinned in `tools.lock.toml`. It covers archives plus many filesystem/disk/container formats without mounting/root privileges.

## Binwalk v3 — **C/M**

Official upstream Rust rewrite; excellent for embedded signatures and extraction. Use subprocess worker for large inputs because the library scan API may require full-file memory and extraction still invokes format handlers/tools.

Upstream: https://github.com/ReFirmLabs/binwalk

## Havk raw carver — **C/M**

A small signature-aware carver complements Binwalk. It should emit offsets and child artifacts, not pretend to be a complete filesystem recovery engine.

## unblob — **F/M**

Promote to full/main rather than experimental. It has an active release cadence, broad format support and recent hardening. Use as a **deep extraction fallback**, not automatically alongside Binwalk on every file.

Upstream: https://github.com/onekey-sec/unblob

## Deark — **F/M**

See metadata section: valuable for legacy/obscure formats and thumbnails/embedded assets that Binwalk/7z may not decode usefully.

## Archive crypto

### bkcrack — **C/M**

Strongly justified by current CTF evidence. 2025 and 2026 write-ups repeatedly exploit predictable PNG/EXE/plaintext structures against ZipCrypto.

Recent examples:
- TJCTF 2025 known-plaintext attack;
- RITSEC CTF 2026 `Zipped Up`;
- Pengcheng Cup 2025 multiple ZipCrypto challenges.

Upstream: https://github.com/kimci86/bkcrack

### fcrackzip — **C/M legacy specialist**

Cheap dictionary/brute compatibility for traditional ZIP encryption.

### pdfcrack — **C/M specialist**

Keep local to protected-PDF workflow, not as a generic password-cracking service.

## Archive repair

Native + trusted tools should cover:

- EOCD search/reconstruction candidates;
- central/local header mismatch;
- truncated ZIP diagnostics;
- `zip -FF` compatibility path where needed;
- encryption scheme identification before choosing attacks.

---

# 11. PDF, Office, email and document forensics

## qpdf — **C/M**

Excellent MCP backend: noninteractive, validation/repair/decrypt, JSON-capable, active, and current official Linux x86_64/aarch64 binaries include checksums/signatures. Current release verified during research: 12.4.2 (2026-09-27).

Upstream: https://github.com/qpdf/qpdf/releases

## pdfcpu — **C/M secondary validator/toolbox**

Add it. This is a mature Go PDF CLI/library with broad validation/manipulation capabilities and excellent release packaging. v0.16.0 (2026-09-28) includes security hardening and Linux binaries with checksums/SBOMs.

Why both qpdf and pdfcpu?

- qpdf remains excellent for object structure, repair, decrypt and JSON;
- pdfcpu provides an independent parser/validator and useful attachment/image/signature operations;
- differential disagreement on malformed CTF PDFs is itself useful evidence.

Upstream: https://github.com/pdfcpu/pdfcpu/releases

## Poppler — **C/M**

Retain for rendering, text/images and attachments.

## Didier Stevens `pdfid.py` / `pdf-parser.py` — **C/M compatibility/suspicious-object specialist**

Do not discard yet. They remain useful in CTF/security workflows for raw suspicious-object inspection even when qpdf/pdfcpu exist. Once Havk's own PDF semantic layer reaches parity, these can become DEV or be removed.

## `lopdf` — **C/E**

Useful Rust parser, but hostile/malformed PDF robustness must be proven on the Havk corpus. Execute in a worker. It is an experimental path, not a reason to remove qpdf.

## oletools — **C/M**

Macros/OLE/RTF/Office-specific knowledge remains valuable and lacks a clearly superior Rust replacement.

## msoffcrypto-tool — **C/M**

For Office encryption/decryption with provided/derived password.

## Rust `mail-parser` — **C/M**

Add direct EML/MIME parsing as a first-class capability. It is a safe Rust parser used in a production mail ecosystem, handles MIME/charsets/attachments, and avoids shelling out merely to enumerate email parts. PST remains a separate full backend.

## libpff / pff-tools — **F/M**

For PST/OST containers where required.

---

# 12. SQLite and database forensics

The previous `database_inspect` was too shallow.

## `rusqlite` — **C/M**

Use immutable/read-only connections for:

- schema;
- tables/views/indexes;
- bounded row queries;
- BLOB inventory/extraction;
- WAL/sidecar presence inventory;
- timestamp/text heuristic summaries.

## deleted SQLite data

This is a separate forensic problem. A 2025 forensic survey emphasizes freelist/WAL/deleted-record recovery and the false-positive tradeoffs of existing tools.

### `sqlite-forensic` / `sqlite4n6` — **F/E**

Add to experimental evaluation. It is a new Rust project, but unlike many rewrites it publishes corpus-based comparisons against independent tools/ground truth and explicitly focuses on read-only recovery from WAL, rollback journal, freelist/free space, dropped schema and deleted rows.

It is still too new for Main, but is a strong experimental candidate.

Upstream: https://github.com/SecurityRonin/sqlite-forensic

### DC3 `sqlite-dissect` — **DEV oracle / F/E optional**

Established forensic reference from the DoD Cyber Crime Center; useful for differential testing but aggressive carving can generate false positives. Do not make it the only source of truth.

### SQLite `.recover`

Useful fallback/oracle, but documented recovery gaps mean it should not be labeled complete forensic recovery.

---

# 13. PCAP / network forensics

## tshark + capinfos + editcap — **C/M**

Keep as the protocol-dissection foundation. No Rust/Go replacement approaches Wireshark's dissector breadth.

The public API should copy the **granularity** of the supplied Wireshark-MCP, while normalizing its outputs into Havk artifacts/observations.

## pcapfix — **C/M**

Small, useful repair backend for malformed/truncated capture structures. Repairs always create a child artifact.

## Zeek — **F/M**

Add it. Zeek is a mature network analysis framework with strong institutional use and structured logs. It complements tshark:

- tshark: packet/field/stream level;
- Zeek: connection/session/application event logs.

For large network-forensics challenges, Zeek can give the agent a compact high-level corpus (`conn`, `dns`, `http`, `ssl/tls`, files, notices) instead of forcing it to query millions of packets.

Upstream: https://github.com/zeek/zeek

## Suricata — **OPT, not default**

Excellent IDS/EVE engine, but for Havk CTF forensics it substantially overlaps Zeek + signature scanning and adds another large rules ecosystem. Keep as optional enrichment rather than normal full image.

## Native CTF decoders — **C/M**

Recent 2025 write-ups still require:

- `usbhid.data` extraction + keyboard reconstruction;
- DNS/exfil reconstruction;
- raw TCP stream payload reconstruction.

Havk should provide native semantic decoders atop tshark fields for common CTF patterns rather than make the model write one-off Python each time.

---

# 14. Disk / filesystem / recovery

## The Sleuth Kit — **C/M**

Core filesystem-forensics backend: partition layout, filesystem metadata, deleted listings, inode/file extraction. Current upstream 4.15.0 includes multiple bounds/overflow fixes, reinforcing why current pinned upstream matters.

Upstream: https://github.com/sleuthkit/sleuthkit/releases

## libewf / ewf-tools — **C/M**

Needed for E01/Ex01. Use current upstream source build instead of ancient distro builds. Normal workflow should be one-shot reading/export, not a long-lived `ewfmount` FUSE mount.

## 7zz — **C/M fast path**

Use for read-only inspect/extract of formats it can understand without FUSE/root; TSK remains the forensic-aware backend.

## PhotoRec — **F/M**

Reconsider the earlier exclusion. PhotoRec can be scripted via `/cmd`, so it is realistic for MCP as a long-running recovery Task. It is mature, widely used, and supports hundreds of file signatures.

Use for signature recovery when filesystem metadata is absent/destroyed; do not confuse it with TSK's metadata-aware deleted-file extraction.

## TestDisk — **not exposed by default**

Although scriptable, its partition-repair/write capabilities increase destructive risk and are not necessary for a read-only CTF forensic MCP. If ever added, make a separate explicitly destructive tool/profile.

## bulk_extractor — **F/M**

Add it. Active again with v2.2.0 in 2026 and recent hostile-input/bounds maintenance. It extracts forensic features without depending on filesystem parsing and complements TSK/PhotoRec.

Use cases:

- email/URL/domain/IP feature extraction;
- encoded/fragment scanning;
- feature histograms;
- triage of large raw images.

## Plaso/log2timeline — **F/M**

Add it for super-timeline/targeted timeline generation. It solves a distinct agent problem: correlating timestamped events across many artifact types. Heavy enough to be a Task/full backend, but too useful to omit from a serious forensic profile.

Upstream: https://github.com/log2timeline/plaso

## qemu-img — **F/M**

Conversion/info for QCOW/VMDK/VHD when 7zz/TSK paths are insufficient.

## `dissect.target` — **not bundled by default**

Technically strong, but AGPL and substantial overlap with the full forensic stack make it better as an optional external integration than a default Havk dependency.

---

# 15. Windows forensic artifacts

## EVTX Rust parser (`evtx`) — **F/M**

Mature enough to use as a typed parser, but still execute in a worker. Useful for exact event parsing/querying without always invoking a larger hunting suite.

## Hayabusa — **F/M optional copyleft bundle**

Very strong reputation for fast EVTX/Sigma timeline/hunting, Rust, active 2026, signed binaries. Current project license is AGPLv3, so bundle/licensing policy should be explicit. It is valuable enough to support, but may belong in `full-copyleft` rather than the minimal distributable image depending on Havk's licensing goals.

Upstream: https://github.com/Yamato-Security/hayabusa

## Chainsaw — **F/M optional copyleft bundle**

Also strong: active 2026, precompiled binaries, Rust, hunts EVTX and additionally analyzes artifacts such as MFT/SRUM/registry-related data. GPLv3. There is overlap with Hayabusa; Havk does not need to run both automatically. Expose one semantic `win_hunt` operation and choose backend/capability.

Upstream: https://github.com/WithSecureOpenSource/chainsaw

## `mft` Rust — **F/E**

Good promotion candidate but not yet equal to the strongest Windows forensic ecosystem in reputation. Differential-test against TSK/Chainsaw and known MFT corpora.

## Registry

- `regipy` — **F/M**;
- libyal/libregf path — **F/M** where needed;
- ForensicRS registry components — **F/E**.

## Prefetch / LNK / PST / related

- libyal `libscca` (Prefetch) — **F/M**;
- libyal `liblnk` — **F/M**;
- libyal `libpff` — **F/M**;
- ForensicRS replacements — **F/E** until corpus parity.

---

# 16. Memory forensics

## Volatility 3 — **F/M**

Default memory backend. Current verified release: 2.28.2 (2026-09-17). Modern CTF memory write-ups continue to revolve around process/network/file/plugin analysis with Volatility.

Expose semantic operations; do not expose arbitrary plugin execution by default.

## MemProcFS — **F/E / optional copyleft alternative**

Add it to evaluation rather than ignoring it. It has strong adoption (~4k GitHub stars in current search), official Linux x64/aarch64 binaries, a Rust API, batch forensic mode, YARA support, process/network/registry/file recovery, timelines and SQLite forensic output.

Reasons not to make it Main immediately:

- AGPLv3 / bundled-license considerations;
- many workflows are naturally virtual-filesystem/mount oriented;
- Havk must prove clean one-shot/API integration without long-lived mount state;
- overlap with Volatility 3 and Windows artifact parsers.

Its batch forensic mode is particularly MCP-friendly and should be tested.

Upstream: https://github.com/ufrisk/MemProcFS

---

# 17. Browser / mobile forensics

These are absent from the original draft and are realistic forensic CTF domains.

## Hindsight — **F/M**

Add a browser-forensics backend for Chromium-family artifacts and supported browser history/session/cache structures. Browser challenges otherwise force repeated ad-hoc SQLite interpretation.

## iLEAPP — **F/M optional mobile bundle**

Strong project activity and ecosystem reputation; current 2026 releases provide Linux x64/arm64 AppImages and checksums. CLI mode is explicit. Recent releases also improve direct forensic image/raw support.

Upstream: https://github.com/abrignoni/iLEAPP/releases

## ALEAPP — **F/M optional mobile bundle**

Android counterpart, same advantages: current release cadence, CLI, Linux x64/arm64 AppImages and `SHA256SUMS.txt`.

Upstream: https://github.com/abrignoni/ALEAPP/releases

These should not inflate `core`, but a serious `full` forensic image should at least support optional installation/tool profiles for them.

---

# 18. Recommended acquisition policy (no distro-version dependency)

The runtime image should not rely on whatever version Debian/Ubuntu happens to package.

Priority:

1. **official upstream prebuilt binary** + published checksum/signature/attestation;
2. **official upstream source tag/tarball** + builder-stage compile;
3. official language package only when that is the upstream distribution model (Python/Ruby crates/wheels/gems), pinned by hash;
4. distro package only for build/bootstrap dependencies, not as the authoritative forensic tool version.

Do not fetch `/releases/latest` during Docker build. Resolve latest versions in an updater workflow and commit the result.

Example:

```toml
# tools.lock.toml

[yara_x]
version = "1.21.0"
source = "upstream"
kind = "source-or-official-artifact"
commit = "7b2637d4655155fdfaf68177edadb9a39406ece0"

[magika]
version = "1.1.0"
kind = "github-release"
asset_x86_64 = "magika-cli-x86_64-unknown-linux-gnu.tar.xz"
sha256_x86_64 = "..."
asset_aarch64 = "magika-cli-aarch64-unknown-linux-gnu.tar.xz"
sha256_aarch64 = "..."

[qpdf]
version = "12.4.2"
kind = "github-release"
sha256_x86_64 = "db367d897829f22c4198ce1094143c9d467bd6ee7dfabc44ba6f02056b24f8b1"
sha256_aarch64 = "8fd9d009eb0838398180a603f2cf76531e5c68a4ecc8847598e0a1ccb1d3b600"
```

The updater should:

```text
query upstream release/tag
  ↓
verify provenance/checksum/signature where available
  ↓
download/build candidate
  ↓
run doctor capability probes
  ↓
run golden corpus + differential corpus
  ↓
measure image footprint
  ↓
open PR updating tools.lock.toml
```

---

# 19. Expanded semantic MCP API

The purpose of this section is to define **agent intents**, not mirror backend executable names.

The complete catalog can be large, but the server should expose a static subset selected at startup with `--tool-profile`. A 60+ tool `all` profile is acceptable for specialized clients; the default `ctf` profile should expose the high-frequency subset to control prompt/schema cost.

Legend:

- **S** = normally synchronous
- **A** = server may convert to MCP Task / fallback job
- **C** = core image
- **F** = full image required

## 19.1 Control / artifact layer

| Tool | Operations / intent | Exec |
|---|---|---|
| `doctor` | backend capability probes, lock verification, architecture support, corpus/version warnings | S |
| `artifact_query` | list, filter by parent/type/origin, provenance tree, children | S |
| `artifact_read` | bounded byte/text range, preview, small structured decode | S |
| `artifact_export` | copy/materialize artifact for client/workspace; never mutate source | S |
| `job` | fallback status/result/cancel when MCP Tasks unsupported | S |

## 19.2 Generic file / metadata

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `file_identify` | magic, MIME, extension mismatch, Magika second opinion, format candidates | native + Magika | S |
| `file_hash` | sha256, optional md5/sha1, chunk/range hash | native | S |
| `file_strings` | ASCII/UTF-8/UTF-16, offsets, regex/filter, bounded paging | native | S |
| `file_entropy` | whole-file, windowed map, boundary candidates | native | S |
| `file_scan` | YARA-X rulesets, CTF indicators, user rule artifact | YARA-X | S/A |
| `file_structure` | known container/chunk/marker tree, appended data, polyglot candidates | native + format adapters | S |
| `file_repair` | PNG, ZIP, selected magic/header repairs; emits child artifact + patch log | native | S |
| `metadata_read` | all/summary/GPS/comments/software/time/thumbnail refs | ExifTool; exp OxiDex | S |
| `metadata_extract` | thumbnails, ICC/XMP/embedded metadata blocks as artifacts | ExifTool/Deark | S/A |

## 19.3 Carving and archives

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `carve_scan` | embedded signatures, raw signature candidates, offsets, confidence | Binwalk + native | S/A |
| `carve_extract` | selected candidate extraction, bounded recursive extraction | native/Binwalk; full unblob/Deark | A |
| `archive_list` | members, sizes, compression, encryption, suspicious paths, ratios | 7zz/format parser | S |
| `archive_extract` | selected members/all under policy limits | 7zz | S/A |
| `archive_repair` | ZIP structure diagnosis/repair candidate, child artifact | native + zip tooling | A |
| `archive_crypto` | detect scheme, dictionary attack, known-plaintext ZipCrypto, unlock/decrypt | bkcrack/fcrackzip/pdfcrack where relevant | A |

## 19.4 Steganography

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `stego_triage` | format-aware plan: metadata/trailing/container/LSB/palette/JPEG/audio/text candidate summary | orchestration only | S |
| `stego_lsb_scan` | bounded search over bit/channel/order/traversal, rank findings | native exp; zsteg oracle | S/A |
| `stego_lsb_extract` | exact extraction parameters → artifact | native; zsteg fallback | S |
| `stego_planes` | bit planes, channel planes, alpha, xor/difference planes | native | S |
| `stego_palette` | palette indices, global/local tables, usage anomalies, remap visualization | native | S |
| `stego_frames` | GIF/APNG/WebP frame list/extract/diff/composite | native/gifsicle/webpmux | S/A |
| `stego_text` | zero-width, whitespace, bidi, variation selectors, homoglyph analysis/extract | native | S |
| `stego_jpeg` | marker inventory + steghide/StegSeek/OutGuess/jsteg probe/extract | native + specialized CLIs | S/A |
| `stego_statistics` | chi-square, RS, SPA with applicability/false-positive warnings | native experimental | S/A |

## 19.5 Image / OCR / barcode

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `image_info` | raw format/sample/palette/frame/color/alpha inventory | native | S |
| `image_compare` | diff, xor, similarity map, alignment options | native | S/A |
| `image_transform` | safe crop/scale/invert/channel visualization for analysis artifacts | native; ImageMagick fallback full | S |
| `image_ocr` | OCR selected artifact/region/frame | Tesseract; exp ocrs | S/A |
| `image_barcode` | QR/DataMatrix/Aztec/PDF417/1D decode, rotations/inversion | zxing-cpp; exp rxing | S |

## 19.6 Media / audio

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `media_probe` | streams, codecs, duration, attachments, chapters, metadata | ffprobe | S |
| `media_extract` | audio/video/data/subtitle/frame stream → child artifact | FFmpeg | A |
| `audio_spectrogram` | spectrogram parameters, frequency range, generated image | native; FFmpeg fallback | S/A |
| `audio_signal_decode` | DTMF core; full: minimodem/multimon protocols/SSTV | native + full backends | S/A |
| `audio_stego` | PCM LSB/channel-diff/sample-bit extraction | native | S/A |

## 19.7 PDF / Office / email / database

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `pdf_inspect` | validation, object/xref/stream tree, suspicious actions/JS/launch/embedded files, parser disagreement | qpdf + pdfcpu + pdfid/parser | S/A |
| `pdf_extract` | text, images, attachments, selected streams/objects | Poppler/qpdf/pdfcpu | S/A |
| `pdf_repair` | repair/normalize/decrypt using supplied password; child artifact | qpdf/pdfcpu | A |
| `office_inspect` | OLE/OOXML structure, macros, embedded objects, encryption state | oletools + native archive view | S/A |
| `office_extract` | macro/object/member extraction; decrypt with password | oletools/msoffcrypto/7zz | A |
| `email_inspect` | headers, MIME tree, decoded bodies, attachment artifacts; full PST | Rust mail-parser; full libpff | S/A |
| `database_inspect` | SQLite schema/query/blob/WAL sidecar inventory | rusqlite | S |
| `database_recover` | deleted/WAL/journal/freelist recovery (full/experimental) | sqlite4n6 / oracle backends | A |

## 19.8 PCAP / network

This is deliberately more granular than the old two-tool design.

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `pcap_info` | capture metadata, interfaces, duration, packet/protocol summary | capinfos/tshark | S |
| `pcap_packets` | pageable packet list with semantic columns + display filter | tshark | S |
| `pcap_packet` | packet detail/tree, raw bytes/context around frame | tshark | S |
| `pcap_search` | text/hex/field/display-filter search with pagination | tshark/native | S/A |
| `pcap_fields` | typed extraction of selected known fields; bounded rows | tshark | S/A |
| `pcap_follow` | TCP/UDP/HTTP/etc. stream reconstruction to preview/artifact | tshark | S/A |
| `pcap_export` | HTTP/SMB/TFTP/etc. object extraction | tshark | A |
| `pcap_protocol` | semantic protocol analyzers; server maps to correct fields/filters | tshark | S/A |
| `pcap_stats` | endpoints, conversations, IO stats, expert info, service response stats | tshark | S/A |
| `pcap_ctf_decode` | USB HID, DNS exfil, ICMP/raw payload, simple covert-channel decoders | tshark fields + native | S/A |
| `network_logs` | full only: generate/query Zeek logs and file events | Zeek | A |

## 19.9 Disk / filesystem / timeline

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `disk_info` | image/container type, EWF metadata, virtual-disk info | 7zz/libewf/qemu-img | S/A |
| `disk_partitions` | partition/volume layout | TSK | S |
| `fs_browse` | filesystem info, directory listing, deleted entries, inode metadata | TSK | S/A |
| `fs_extract` | extract path/inode/range to artifact | TSK/7zz | A |
| `fs_recover` | full: PhotoRec/raw recovery with policy filters | PhotoRec | A |
| `forensic_features` | full: feature extraction from raw evidence | bulk_extractor | A |
| `timeline_build` | full: targeted/super timeline | Plaso | A |

## 19.10 Windows full-profile tools

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `win_evtx` | summary/query/export records, event gaps | evtx crate/worker | S/A |
| `win_hunt` | Sigma/detection/timeline hunting | Hayabusa or Chainsaw capability-selected | A |
| `win_registry` | hive inventory/keys/values/timestamps, selected artifact parsers | regipy/libyal; exp ForensicRS | S/A |
| `win_mft` | records, paths, timestamps, resident data, anomaly filters | TSK / exp Rust mft | S/A |
| `win_prefetch` | application run metadata and file references | libscca / exp Rust | S/A |
| `win_lnk` | target/path/timestamps/volume/network info | liblnk | S |
| `win_pst` | folder/message/attachment extraction | libpff | A |

## 19.11 Memory full-profile tools

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `memory_info` | OS/kernel/layer/symbol diagnostics | Volatility 3 | A |
| `memory_processes` | pslist/pstree/cmdline/handles/VAD-oriented summaries | Volatility 3 | A |
| `memory_network` | sockets/connections | Volatility 3 | A |
| `memory_modules` | DLL/modules/drivers/malfind-oriented views | Volatility 3 | A |
| `memory_files` | file objects/filescan/dump selected object | Volatility 3 | A |
| `memory_scan` | YARA-X/Volatility scanning pipeline | Volatility + YARA-X | A |
| `memory_dump` | process/module/file/minidump artifacts | Volatility; exp MemProcFS | A |
| `memory_batch` | experimental alternate batch forensic report | MemProcFS | A |

## 19.12 Browser / mobile / wireless full tools

| Tool | Operations | Backend | Exec |
|---|---|---|---|
| `browser_analyze` | browser history/downloads/cookies/cache/session/extension artifacts | Hindsight + SQLite layer | A |
| `mobile_ios` | run selected iLEAPP parsers / report / artifact import | iLEAPP | A |
| `mobile_android` | run selected ALEAPP parsers / report / artifact import | ALEAPP | A |
| `wifi_analyze` | handshake info, AP/client inventory, PMKID/EAPOL diagnostics | hcxtools/aircrack-ng | A |
| `wifi_crack` | bounded wordlist attack against provided candidate list | aircrack-ng | A |

---

# 20. Suggested default tool profiles

Do not expose every tool to every model by default.

## `--tool-profile ctf` (recommended default)

High-frequency subset:

```text
doctor
artifact_query artifact_read artifact_export job
file_identify file_hash file_strings file_entropy file_scan file_structure file_repair
metadata_read metadata_extract
carve_scan carve_extract
archive_list archive_extract archive_repair archive_crypto
stego_triage stego_lsb_scan stego_lsb_extract stego_planes stego_palette
stego_frames stego_text stego_jpeg
image_info image_compare image_ocr image_barcode
media_probe media_extract audio_spectrogram audio_signal_decode audio_stego
pdf_inspect pdf_extract pdf_repair office_inspect office_extract
email_inspect database_inspect
pcap_info pcap_packets pcap_packet pcap_search pcap_follow pcap_export pcap_ctf_decode
disk_info disk_partitions fs_browse fs_extract
```

This is intentionally larger than Claude's 21 tools but still avoids loading memory/mobile/enterprise timeline schemas when they are irrelevant.

## `--tool-profile stego`

Adds/emphasizes:

```text
stego_statistics
image_transform
all frame/palette/raw sample operations
audio signal/stego
metadata and structure tools
```

## `--tool-profile network`

Expose all PCAP + Zeek tools, artifact/file tools, not mobile/memory by default.

## `--tool-profile disk`

Expose TSK/libewf/PhotoRec/bulk/timeline/database/document/browser-related tools.

## `--tool-profile windows`

Requires full image; expose EVTX/registry/MFT/prefetch/LNK/PST + timeline/database/disk.

## `--tool-profile memory`

Requires full image; expose Volatility/MemProcFS + YARA-X/artifact tools.

## `--tool-profile all`

For advanced clients only. Expect materially higher tool-description/schema prompt cost.

---

# 21. Tool/backend mapping principles

## 21.1 Never expose backend names as normal tools

Avoid:

```text
run_exiftool
run_tshark
run_volatility
run_binwalk
```

Expose:

```text
metadata_read
pcap_follow
memory_processes
carve_scan
```

Backend names may appear only as optional `backend_preference` in expert/diagnostic mode and in result provenance.

## 21.2 Independent validators are valuable

Some intentional overlap is good when parsers are handling malformed adversarial data:

- qpdf + pdfcpu;
- deterministic magic + Magika;
- native LSB + zsteg oracle during migration;
- EVTX direct parser + Hayabusa/Chainsaw high-level hunting;
- TSK + 7zz view of disk containers;
- Binwalk + unblob fallback.

The semantic layer should report disagreement instead of silently selecting whichever returned first.

---

# 22. Promotion policy for Experimental backends

A rewrite can replace a Main backend only after it passes all gates relevant to its domain.

Example:

```text
OxiDex candidate
  ↓
metadata corpus (camera/office/pdf/media/malformed)
  ↓
compare tag presence/value/type with ExifTool
  ↓
crash/OOM corpus
  ↓
performance/footprint
  ↓
real CTF corpus
  ↓
promotion PR
```

Suggested gates:

- no source-evidence mutation;
- zero uncontrolled panics in hostile corpus;
- bounded memory/output under policy;
- architecture support x86_64 + aarch64;
- minimum agreed coverage/parity threshold;
- documented known divergences;
- deterministic output for same input/version;
- license acceptable for selected distribution bundle.

---

# 23. Golden corpus design

Test **capabilities**, not exact backend stdout.

```text
tests/corpus/
├── file/
│   ├── extension-mismatch/
│   ├── appended-data/
│   ├── polyglot/
│   └── damaged-headers/
├── stego/
│   ├── png-lsb/
│   ├── png-palette/
│   ├── gif-local-palette/
│   ├── apng-frames/
│   ├── jpeg-steghide/
│   ├── jpeg-outguess/
│   ├── jpeg-jsteg/
│   ├── text-zero-width/
│   └── audio/
├── archive/
│   ├── zipcrypto-known-plaintext/
│   ├── encrypted/
│   ├── truncated/
│   └── bombs-policy-only/
├── document/
│   ├── suspicious-pdf/
│   ├── malformed-pdf/
│   ├── macro-office/
│   └── encrypted-office/
├── pcap/
│   ├── http-object/
│   ├── dns-exfil/
│   ├── usb-hid/
│   ├── smb-object/
│   └── malformed/
├── disk/
│   ├── ext4-deleted/
│   ├── ntfs-deleted/
│   ├── e01/
│   └── photorec-carving/
├── sqlite/
│   ├── wal/
│   ├── journal/
│   ├── freelist/
│   └── deleted-records/
├── windows/
│   ├── evtx/
│   ├── mft/
│   ├── registry/
│   ├── prefetch/
│   └── lnk/
└── memory/
    └── small-published-samples/
```

Differential oracles may include legacy tools even when they are not shipped at runtime.

Examples:

- pngcheck as PNG validation oracle;
- zsteg as LSB oracle;
- StegExpose as classical steganalysis oracle;
- ExifTool vs OxiDex;
- ZXing-C++ vs rxing;
- Tesseract vs ocrs;
- qpdf/pdfcpu/lopdf cross-parser;
- sqlite4n6 vs sqlite-dissect/FQLite/undark on public forensic corpora.

---

# 24. CTF evidence that directly influenced this design

This is not an attempt to count tool popularity mechanically. It is evidence that certain capabilities still occur in modern challenges.

| Capability | Recent evidence | Design consequence |
|---|---|---|
| PNG LSB | CTF@CIT 2025, Hacktheon Sejong 2025 use zsteg extraction | keep exact LSB search/extract semantics |
| zsteg false positives | 2026 write-up documents large noisy `-a` output | rank evidence; never treat signature coincidence as truth |
| GIF local palettes | SAS CTF 2025 uses local palette/index trick | raw GIF palette/index parser is core |
| APNG structure | openECSC 2025 Calamansi | APNG frame/chunk semantics are core |
| JPEG OutGuess | SCTF 2026 challenge solved by OutGuess after other methods failed | OutGuess belongs in runtime coverage |
| ZIP known plaintext | TJCTF 2025, Pengcheng Cup 2025, RITSEC 2026 | `archive_crypto` must model known-plaintext, not only passwords |
| USB HID PCAP | ApoorvCTF 2025, Null CTF 2025 | native HID reconstruction on tshark fields |
| Audio spectrogram | multiple 2025 challenge write-ups | spectrogram generation remains core |
| Memory | 2025–2026 CTF workflows still use Volatility | Volatility remains full/main |

Selected write-ups:

- https://github.com/Diephho/CTF-CIT-2025-Writeups
- https://cofastic.com/writeups/hacktheon-sejong-2025-ctf-writeup
- https://ctftime.org/writeup/40254
- https://ctf.zeba.dev/2025/openecsc/stego/calamansi/writeup/
- https://github.com/hax1ng/SCTF-2026-writeups/blob/main/misc/SYC4113/README.md
- https://ctf.gg/blog/tjctf-2025/misc
- https://medium.com/@wireshark.pcap/ritsec-ctf-2026-zipped-up-writeup-by-wireshark-pcap-b2979b696bae
- https://astro.bili33.top/posts/CTF-PCB2025-Preliminary-Round-Writeup/
- https://ahmed-naser.medium.com/apoorvctf-2025-forensics-challenges-33c2d128e9f1
- https://banua.medium.com/null-ctf-2025-writeup-1aa4f13e5709

---

# 25. Main / Experimental / optional decision matrix

## Core/Main — recommended default

```text
Havk native:
  magic/hash/strings/entropy/byte windows
  appended/polyglot
  PNG/APNG/GIF/JPEG structural parsers
  simple repair
  raw carver
  palette/bitplane/image diff
  text stego
  audio primitives
  USB HID/DNS/ICMP CTF decoders

Backends:
  YARA-X
  Magika CLI
  Binwalk v3
  ExifTool
  7zz
  zxing-cpp
  Tesseract
  zsteg (temporary oracle/runtime fallback)
  StegSeek
  steghide
  OutGuess
  jsteg
  gifsicle
  FFmpeg/ffprobe
  qpdf
  pdfcpu
  Poppler
  pdfid/pdf-parser (compatibility path)
  oletools
  msoffcrypto-tool
  mail-parser (Rust)
  bkcrack
  fcrackzip
  pdfcrack
  tshark/capinfos/editcap
  pcapfix
  The Sleuth Kit
  libewf/ewf-tools
  rusqlite
```

## Core/Experimental

```text
native full LSB parity engine
chi-square / RS / SPA
OxiDex
rxing
ocrs
lopdf
archive-forensic
```

## Full/Main

```text
unblob
Deark
ImageMagick fallback
libwebp tools
multimon-ng
minimodem
PhotoRec
bulk_extractor
Plaso
Zeek
qemu-img
Volatility 3
EVTX Rust parser
regipy
libyal: liblnk/libscca/libpff (+ relevant family libs)
Hindsight
iLEAPP
ALEAPP
OpenStego
```

Copyleft optional full bundle (depending distribution policy):

```text
Hayabusa (AGPLv3)
Chainsaw (GPLv3)
```

## Full/Experimental

```text
MemProcFS (strong tool, but AGPL + lifecycle/overlap integration needs validation)
mft Rust
ForensicRS
sqlite-forensic/sqlite4n6
SSTV Python decoder
```

## Separate optional image

```text
stego-ml:
  Aletheia
```

---

# 26. Tools deliberately rejected or not default

| Tool | Decision | Reason |
|---|---|---|
| YARA classic | replaced for new project | YARA-X official successor path |
| Binwalk v2 | reject | v3 upstream rewrite |
| Foremost | reject default | native carver + PhotoRec/unblob cover stronger roles |
| Scalpel | reject default | same overlap; no need to ship another raw carver |
| StegSolve | reject runtime | GUI; implement semantic bitplane/frame/diff operations |
| stegcracker | reject | StegSeek |
| stegsnow | reject runtime | native text/whitespace parser |
| pngcheck | DEV oracle | native parser in runtime |
| ZBar | reject default | ZXing-C++ broader/current; rxing experimental |
| SoX | reject core | FFmpeg + native DSP; avoid redundant runtime |
| Aletheia | not default | useful but heavy; `stego-ml` image |
| Suricata | optional | strong but overlap with Zeek/tshark for CTF scope |
| NetworkMiner | reject default | GUI/.NET workflow less MCP-friendly |
| TestDisk | no default public tool | scriptable but potentially destructive partition operations |
| ewfmount/xmount | reject normal flow | long-lived mount state/FUSE unnecessary |
| `dissect.target` | not bundled default | AGPL + overlap; possible external integration |
| general John/hashcat | different Havk crypto/password module | avoid turning forensic MCP into universal cracker |

---

# 27. Open research / first implementation spikes

Before PRD is declared implementation-ready, run these spikes:

1. **rmcp 3.4.x prototype**
   - stdio + Streamable HTTP;
   - structuredContent + outputSchema;
   - Artifact ResourceLink;
   - MCP Tasks + fallback job;
   - stateless HTTP workspace state.

2. **Worker isolation prototype**
   - Rust parser panic isolation;
   - process groups/cancellation;
   - rlimit/cgroup/seccomp strategy;
   - extract-size and recursion policies.

3. **Raw image representation**
   - PNG sub-8-bit and 16-bit fixtures;
   - indexed PNG;
   - GIF local palette;
   - APNG;
   - ensure no automatic RGB conversion before raw-steg analysis.

4. **JPEG steg compatibility corpus**
   - steghide/StegSeek;
   - OutGuess;
   - jsteg;
   - exact positive/negative fixtures.

5. **PDF parser differential corpus**
   - qpdf vs pdfcpu vs Didier scripts vs lopdf;
   - malformed object streams/xrefs;
   - JavaScript/actions/attachments.

6. **E01/Ex01**
   - build current libewf;
   - test E01 + Ex01 published fixtures;
   - one-shot export/read path without FUSE.

7. **SQLite forensic recovery**
   - public Nemetz/DFRWS/DC3 corpora;
   - sqlite4n6 vs sqlite-dissect/FQLite/SQLite recover;
   - precision/recall and false-positive reporting.

8. **Memory backend comparison**
   - Volatility 3 default;
   - MemProcFS batch mode as experimental alternate;
   - compare process/network/file recovery on small published images.

9. **Tool-surface prompt measurement**
   - measure serialized `tools/list` bytes/tokens for `ctf`, `stego`, `network`, `disk`, `all`;
   - target a default tool-profile budget rather than arbitrary tool count.

10. **Docker footprint measurement**
    - x86_64 + arm64;
    - core/full/stego-ml;
    - largest layers/dependencies documented.

---

# 28. Recommended next document

The next artifact should no longer be a broad tool search. It should be the implementation matrix:

```text
semantic tool
→ input schema
→ output schema
→ main backend
→ experimental backend
→ resource limits
→ sync/task threshold
→ artifact types
→ annotations
→ golden tests
→ acquisition/lock entry
```

Then convert that matrix into the PRD and Rust crate/module boundaries.

---

# 29. Primary references

## MCP / Rust

- Official Rust SDK: https://github.com/modelcontextprotocol/rust-sdk
- rmcp releases: https://github.com/modelcontextprotocol/rust-sdk/releases
- rmcp roadmap/conformance: https://github.com/modelcontextprotocol/rust-sdk/blob/main/ROADMAP.md
- rmcp 3.x migration: https://github.com/modelcontextprotocol/rust-sdk/discussions/969
- MCP 2026-07-28 announcement/spec context: https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/blog/content/posts/2026-07-28-spec-ga/index.md
- Tasks extension: https://tasks.extensions.modelcontextprotocol.io/specification/draft/tasks

## Core forensic/steg backends

- YARA-X: https://github.com/VirusTotal/yara-x
- Magika: https://github.com/google/magika
- Binwalk: https://github.com/ReFirmLabs/binwalk
- ExifTool: https://github.com/exiftool/exiftool
- Deark: https://github.com/jsummers/deark
- qpdf: https://github.com/qpdf/qpdf
- pdfcpu: https://github.com/pdfcpu/pdfcpu
- bkcrack: https://github.com/kimci86/bkcrack
- unblob: https://github.com/onekey-sec/unblob
- Volatility 3: https://github.com/volatilityfoundation/volatility3
- Sleuth Kit: https://github.com/sleuthkit/sleuthkit
- bulk_extractor: https://github.com/simsong/bulk_extractor
- Plaso: https://github.com/log2timeline/plaso
- Zeek: https://github.com/zeek/zeek
- MemProcFS: https://github.com/ufrisk/MemProcFS
- Hayabusa: https://github.com/Yamato-Security/hayabusa
- Chainsaw: https://github.com/WithSecureOpenSource/chainsaw
- iLEAPP: https://github.com/abrignoni/iLEAPP
- ALEAPP: https://github.com/abrignoni/ALEAPP
- sqlite-forensic: https://github.com/SecurityRonin/sqlite-forensic

## Research / modern evidence

- SQLite deleted-record recovery survey (2025): https://www.sciencedirect.com/science/article/pii/S2666281725001714
- Aletheia JOSS paper/tool: https://joss.theoj.org/papers/10.21105/joss.05982
- Sample Pair Analysis literature: https://experts.mcmaster.ca/scholarly-works/146164

---

## Bottom line

The correct Havk MCP is **not** a wrapper over 20 classic CTF commands, and it is also **not** 150 backend-specific MCP tools. The target is a typed forensic/steg semantic layer with enough granularity for iterative agent reasoning, backed by a much larger and replaceable backend catalog.

The biggest upgrades over draft v0.2 are:

1. granular PCAP API modeled after the stronger supplied Wireshark-MCP;
2. raw GIF/APNG/palette/sample steg support;
3. complete JPEG steg family (StegSeek/steghide + OutGuess + jsteg);
4. independent PDF validation with pdfcpu;
5. proper SQLite deleted-record recovery research path;
6. PhotoRec, bulk_extractor, Plaso, Zeek;
7. Windows hunt layer (Hayabusa/Chainsaw) with license separation;
8. browser and mobile forensics (Hindsight, iLEAPP, ALEAPP);
9. MemProcFS as a serious experimental memory alternative;
10. MCP-native artifacts/resources/tasks rather than giant stdout responses.

That produces a broader toolset without losing KISS at the public API boundary: the complexity lives behind typed semantic contracts, workers, artifact provenance, and backend capability routing.
