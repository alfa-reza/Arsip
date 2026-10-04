
# Havk CTF MCP — Forensics, Steganography & Network Forensics

**Focused research draft v0.4 — 2026-10-04**  
**Scope:** digital forensics, steganography, PCAP/network forensics, and file/document/disk/memory artifacts that directly support those categories.  
**Out of scope for this document:** general pentesting, recon, web exploitation, reverse engineering, pwn, cloud security, generic OSINT, and general-purpose password cracking.

This document replaces the broader v0.2/v0.3 direction with a deliberately narrower forensic scope. The goal is not to make Havk another Kali-style catalog of executable names. The goal is a **typed semantic MCP layer** that an agent can use iteratively while the Docker image hides a larger, replaceable backend toolchain.

---

## 0. Executive decisions

1. **Do not collapse network forensics into `tshark`.** Treat Wireshark CLI as a suite: `tshark`, `capinfos`, `captype`, `editcap`, `mergecap`, `reordercap`, `text2pcap`, `dumpcap`, and optionally `sharkd`. Each has a distinct forensic role.
2. **Do not expose backend executables as MCP tools.** `pcap_merge` is a semantic tool; `mergecap` is an implementation detail. `pdf_objects` is semantic; `mutool show` or `qpdf --json` are backend choices.
3. **`mutool` is technically excellent but should not be the only default PDF backend.** MuPDF's CLI covers low-level object inspection, repair, rendering/text extraction, resource extraction, audit and scripting, but MuPDF is AGPL/commercial. Default Havk should retain an independent, machine-structured PDF path such as qpdf + pdfcpu + security-specific scanners. `mutool` is recommended as a supported optional/copyleft backend unless Havk intentionally adopts an AGPL-compatible distribution model.
4. **HexStrike-AI is useful as a breadth/orchestration reference, not as an executor design reference.** Its source demonstrates how much security tooling an MCP can expose, but its generic `additional_args` style and shell-based execution path are exactly what Havk should avoid.
5. **Docker is the supported runtime contract.** Evidence is mounted read-only; artifacts are written to a separate volume. Live packet capture is disabled by default and only enabled with an explicit server flag plus narrowly scoped container capabilities.
6. **Keep the two existing dimensions:**
   - image profile: `core` / `full`;
   - backend trust tier: `main` / `experimental`.
7. **Add static MCP tool profiles:** `forensics`, `stego`, `network`, `all`. The server chooses the list at startup. Do not add/remove tools because a particular artifact was discovered.
8. **Use MCP Tasks for long operations when negotiated, with a `job` fallback.** The server decides sync vs task based on input size/cost; the model should not need to know implementation thresholds.
9. **All evidence is immutable.** Repair, decryption, merge, reorder, extraction, reconstruction and conversion always produce child artifacts with provenance.
10. **Prefer upstream release artifacts and upstream source builds over distro packages.** Docker's base distribution is not the version authority for forensic tools.

---

# 1. Scope and non-goals

## 1.1 Included

### Steganography

- PNG/BMP raw LSB and channel/bit-plane analysis;
- indexed-color/palette techniques;
- GIF/APNG/WebP frame/container techniques;
- JPEG stego families such as steghide, OutGuess, JSteg;
- trailing/appended data and polyglots;
- metadata hiding;
- text/unicode/whitespace stego;
- audio waveform/spectrogram/DTMF/PCM-LSB/modem/SSTV;
- QR/barcode recovery where directly relevant to stego/forensics;
- classical statistical steganalysis, clearly marked probabilistic;
- optional ML steganalysis.

### Digital forensics

- file identification and metadata;
- archives and encrypted archive recovery techniques common in CTF;
- carving and embedded object recovery;
- PDF and Office forensics;
- SQLite and deleted-record/WAL analysis;
- disk images, EWF/E01, filesystem metadata, deleted files and raw recovery;
- memory images;
- core Windows artifacts: EVTX, Registry, MFT, Prefetch, LNK, PST where appropriate;
- forensic timelines and bulk feature extraction in `full`.

### Network/PCAP forensics

- capture-file metadata and validation;
- repair, merge, reorder, slicing, deduplication and format conversion;
- text/hex dump conversion to PCAP;
- packet listing/detail/field extraction/search;
- stream following and object export;
- conversations/endpoints/protocol statistics;
- CTF-specific USB HID, DNS exfiltration, ICMP covert data and payload reconstruction;
- Zeek session/event logs in `full`;
- optional live capture using `dumpcap`, off by default.

## 1.2 Explicitly out of scope

This document does **not** design tools for:

- Nmap/masscan/recon;
- web scanning/exploitation;
- CVE exploitation;
- generic red-team automation;
- reverse engineering/malware decompilation;
- binary exploitation;
- cloud/Kubernetes security;
- social OSINT;
- arbitrary shell execution;
- unrestricted John/hashcat orchestration.

HexStrike includes these domains, but Havk should not import its scope wholesale.

---

# 2. MCP reference projects reviewed

Every MCP reference discussed in this research is listed here with its repository URL.

| Project | Repository | What Havk should learn | What Havk should not copy |
|---|---|---|---|
| **Wireshark-MCP** | https://github.com/bx33661/Wireshark-MCP | granular packet tools, bounded output, packet paging/context, field extraction, search, follow-stream, stats, protocol abstractions, write-root security | backend-specific proliferation without Havk profiles; any assumptions that only PCAP files matter |
| **HexStrike-AI** | https://github.com/0x4m4/hexstrike-ai | large tool registry, health/process management, Docker/orchestration ideas, proof that MCP can coordinate many security tools | generic `additional_args`, shell command strings, huge always-on tool list, arbitrary command endpoints |
| **ctf-buster** | https://github.com/agentfanclub/ctf-buster | simple CTF workflow baseline; triage → steg → extraction → entropy → image analysis | only five coarse forensic tools; large one-shot responses; image normalization as evidence source |
| **CTF-MCP** | https://github.com/Coff0xc/CTF-MCP | native/simple forensic helpers and CTF-oriented ergonomics | treating normalized image representations as authoritative raw steg evidence |
| **ctfd-mcp-server** | https://github.com/MrJamescot/ctfd-mcp-server | MCP service organization/configuration examples | not a forensic backend reference; mainly CTFd platform API |
| **steganography-mcp** | https://github.com/badchars/steganography-mcp | broad steg technique catalog: LSB, RS, SPA, JPEG families, BPCS, GIF/video, network covert channels, MP3, text | 128 always-visible tools; young project; algorithms must be independently corpus-tested |
| **Mulder** | https://github.com/calebevans/mulder | typed forensic interfaces, no-shell philosophy, audit/provenance, broad DFIR backend mapping | enterprise incident-report workflow is broader/heavier than Havk CTF MCP |
| **SIFT-MCP / Valhuntir** | https://github.com/AppliedIR/sift-mcp | separate forensic endpoints/profiles, evidence-oriented tool boundaries, audit/human review | entire case/RAG/OpenCTI platform is outside current scope |
| **volatility-mcp** | https://github.com/Gaffx/volatility-mcp | semantic wrapping of Volatility plugins | thin REST/MCP wrapping is not enough for Havk resource limits/provenance |

## 2.1 HexStrike-AI source audit

The supplied HexStrike snapshot is valuable because it exposes a much larger security-tool surface than the earlier CTF MCP examples. Its CTF/forensic inventory names Volatility 3, Foremost, PhotoRec, TestDisk, Sleuth Kit, StegSolve, zsteg, OutGuess, Scalpel, bulk_extractor, ExifTool and Binwalk.

However, its source reinforces Havk's need for a stricter design:

- MCP wrappers repeatedly accept free-form `additional_args` strings;
- forensic wrappers are backend-shaped (`volatility3_analyze`, `foremost_carving`, `steghide_analysis`, `exiftool_extract`) rather than stable semantic contracts;
- the server contains shared command-execution paths using `subprocess.Popen(..., shell=True)`;
- upstream has open 2026 issues/PRs specifically discussing command-injection risks around `additional_args`, target fields, and generic command execution;
- upstream also has a real usability issue caused by exposing more than 128 MCP tools at once.

**Havk conclusion:** use HexStrike as a **coverage checklist and orchestration warning**, not as an executor template.

Relevant upstream links:

- Repository: https://github.com/0x4m4/hexstrike-ai
- Issue: command injection via `additional_args`: https://github.com/0x4m4/hexstrike-ai/issues/265
- PR attempting to fix it: https://github.com/0x4m4/hexstrike-ai/pull/266
- Issue on excessive MCP tool count: https://github.com/0x4m4/hexstrike-ai/issues/119
- Security/RCE discussions: https://github.com/0x4m4/hexstrike-ai/issues/204
- Ecosystem article: https://cybersecuritynews.com/hackers-leverage-hexstrike-ai-tool/

The article is useful only as evidence of adoption/interest; technical design decisions should come from source and primary documentation.

---

# 3. MCP 2026-07-28 and Rust `rmcp`: architecture rules

Primary sources:

- Official Rust SDK: https://github.com/modelcontextprotocol/rust-sdk
- MCP specification repository: https://github.com/modelcontextprotocol/modelcontextprotocol
- MCP specification site: https://modelcontextprotocol.io/
- rmcp 3.x migration discussion: https://github.com/modelcontextprotocol/rust-sdk/discussions/969
- rmcp roadmap/conformance: https://github.com/modelcontextprotocol/rust-sdk/blob/main/ROADMAP.md

## 3.1 Protocol target

Target **MCP 2026-07-28** and a stable `rmcp` 3.x release pinned in Cargo.lock. At research time, `rmcp-v3.4.0` was the latest indexed stable release (2026-09-15). The SDK documents compatibility with the stable 2026-07-28 protocol and 100% conformance for the date-versioned server/client conformance suite.

## 3.2 Typed tools

Use Rust structs + Serde + `schemars`, with `#[tool]`, `#[tool_router]`, and `#[tool_handler]`.

Good public contract:

```rust
#[derive(Deserialize, JsonSchema)]
struct PcapMergeParams {
    artifacts: Vec<ArtifactId>,
    order: MergeOrder,
    output_format: CaptureFormat,
    idb_mode: Option<IdbMergeMode>,
}
```

Bad public contract:

```rust
struct RunTool {
    command: String,
    args: String,
}
```

Backends may use exact CLI flags internally, but the model selects forensic intent.

## 3.3 Statelessness and durable handles

MCP 2026-07-28 is stateless at the protocol level. Related operations must therefore carry explicit handles:

```text
artifact_id
job_id / task_id
workspace_id (transport/auth context, not inferred from prior call)
```

Do not assume the same HTTP connection or even the same handler instance will process successive calls.

## 3.4 Results: structured observations + ArtifactRefs

Normal tool responses should be small:

```json
{
  "status": "ok",
  "observations": [
    {
      "kind": "out_of_order_timestamps",
      "severity": "info",
      "count": 134
    }
  ],
  "artifacts": [
    {
      "id": "art_01K...",
      "uri": "havk://artifact/art_01K...",
      "mime_type": "application/vnd.tcpdump.pcapng",
      "sha256": "...",
      "size": 1842812
    }
  ],
  "truncated": false
}
```

Use MCP `structuredContent` for normalized data and MCP Resources/ResourceLinks for large or binary outputs.

Never inline:

- entire recovered files;
- huge packet tables;
- rendered PDF pages;
- all bit planes;
- thousands of carved objects;
- memory dumps;
- full Zeek logs;
- full Volatility JSON.

## 3.5 Tasks

Operations expected to run long should become MCP Tasks when the client negotiates the Tasks extension. Otherwise Havk returns an internal `job_id` through a static `job` tool.

Typical task candidates:

- deep carving;
- Binwalk/unblob recursion;
- archive attacks;
- OCR on many pages;
- PCAP object export on large captures;
- Zeek processing;
- PhotoRec;
- Plaso;
- Volatility scans/dumps;
- bulk_extractor.

## 3.6 Static tool profiles

2026-07-28 supports list-change notification and caching, but Havk should still expose a deterministic tool list for the lifetime of a server instance.

Recommended startup profiles:

```text
--tool-profile forensics
--tool-profile stego
--tool-profile network
--tool-profile all
```

Do **not** enable `memory_*` because a `.raw` file appeared. Start a server profile that includes those tools.

## 3.7 Stdio rules

For stdio transport:

- stdout belongs to MCP protocol messages;
- logs go to stderr;
- external subprocess stdout/stderr must be captured by workers, never inherited blindly;
- a backend producing unbounded text must be truncated/paged before reaching MCP.

---

# 4. Docker-first execution model

## 4.1 Images

Recommended images:

```text
ghcr.io/<org>/havk-mcp:core
ghcr.io/<org>/havk-mcp:full
ghcr.io/<org>/havk-mcp:stego-ml   # optional later
```

`core` contains high-frequency CTF forensics/stego/network tools. `full` adds memory, heavier disk/timeline recovery, Windows artifact enrichment and Zeek/unblob-class backends.

## 4.2 Filesystem model

Suggested container layout:

```text
/evidence   read-only input mount
/artifacts  writable immutable artifact store
/scratch    ephemeral worker scratch
/config     tools.lock.toml + rules + optional profiles
```

External API should prefer `artifact_id`, not host paths.

If an initial user file is mounted under `/evidence`, the ingestion operation registers it and returns an `artifact_id`.

## 4.3 Worker isolation

Even Rust parsers should normally execute in `havk worker` when processing attacker-controlled evidence.

Per-worker budgets:

- wall timeout;
- CPU limit;
- RSS/address-space limit;
- maximum stdout/stderr bytes;
- maximum artifact count;
- maximum single artifact size;
- maximum aggregate extracted size;
- maximum decompression ratio;
- maximum recursion depth;
- no network by default;
- no shell;
- no inherited credentials;
- evidence path read-only.

## 4.4 Live capture is a separate privilege mode

`dumpcap` belongs in the Wireshark toolchain, but offline forensics should not grant packet-capture privileges by default.

Normal container:

```text
no NET_RAW
no NET_ADMIN
no host networking requirement
pcap_capture tool hidden/disabled
```

Explicit live mode can be run with narrowly scoped capabilities, for example conceptually:

```text
--cap-add=NET_RAW
--cap-add=NET_ADMIN
--tool-profile network
--enable-live-capture
```

The exact network namespace/host-network configuration is a deployment decision. Havk should never require `--privileged` merely to analyze saved PCAP files.

---

# 5. Acquisition and version locking

## 5.1 Policy

Priority order:

1. official upstream prebuilt binary + checksum/signature/attestation;
2. official upstream release/tag/source tarball built in a Docker builder stage;
3. language package only when that is the upstream distribution channel, pinned by hash/version;
4. distro package only for build dependencies/bootstrap, not as the forensic-tool version authority.

Never download `/releases/latest` inside Docker build.

Instead:

```text
updater resolves latest upstream
→ verifies provenance/checksum
→ builds candidate
→ doctor probes
→ golden corpus
→ footprint measurement
→ PR updates tools.lock.toml
```

## 5.2 Example lock

```toml
[wireshark]
version = "4.6.9"
source = "https://www.wireshark.org/download/src/all-versions/wireshark-4.6.9.tar.xz"
kind = "source"
sha256 = "<pinned-by-updater>"
components = [
  "tshark",
  "capinfos",
  "captype",
  "editcap",
  "mergecap",
  "reordercap",
  "text2pcap",
  "dumpcap",
  "sharkd"
]

[qpdf]
version = "12.4.2"
kind = "official-release-binary"
sha256 = "<arch-specific>"

[pdfcpu]
version = "0.16.0"
kind = "official-release-binary"
sha256 = "<arch-specific>"
```

---

# 6. Shared forensic primitives

These operations support stego, network and disk/document forensics and therefore belong in the focused project.

## 6.1 Native Havk — Core/Main

| Primitive | Reason to own it |
|---|---|
| SHA-256 + optional MD5/SHA-1 | deterministic, trivial, used everywhere |
| ASCII/UTF-8/UTF-16 strings with byte offsets | avoids fragile CLI parsing and supports paging |
| bounded hex/byte ranges | key primitive for forensic reasoning |
| entropy by range/window | useful for packed/encrypted/embedded regions |
| common magic/signature engine | deterministic first opinion |
| extension/MIME disagreement | CTF high-value anomaly |
| appended/trailing data | PNG/JPEG/GIF/PDF/etc. |
| simple polyglot indicators | multiple valid structures/signatures |
| raw signature carver | bounded, provenance-aware extraction |
| PNG/APNG/GIF/JPEG structural parsers | preserves raw representation for steg |
| simple repair primitives | child-artifact only |

## 6.2 Main external/common backends

| Backend | Tier | Role |
|---|---|---|
| **YARA-X** | core/main | rules/signature engine |
| **Magika CLI** | core/main | probabilistic second opinion for file type |
| **ExifTool** | core/main | broad metadata knowledge |
| **Binwalk v3** | core/main | embedded signature scan/extract |
| **7zz** | core/main | archives + many image/container formats |
| **unblob** | full/main | deep recursive extraction fallback |
| **Deark** | full/main | obscure/legacy formats, thumbnails/embedded assets |

Experimental replacements such as OxiDex should be evaluated with differential corpora rather than promoted for being Rust.

---

# 7. Steganography toolset

## 7.1 Native raw image analysis — Core/Main

Havk must preserve raw storage representation before any RGB normalization.

### PNG

- chunk inventory and order;
- IHDR fields;
- palette (`PLTE`) and transparency (`tRNS`);
- raw sample bit depth, including sub-8-bit and 16-bit;
- CRC verification;
- IDAT boundaries;
- post-IEND bytes;
- APNG `acTL`, `fcTL`, `fdAT`;
- frame extraction.

### GIF

- global color table;
- local color table per frame;
- palette indices, not merely rendered RGB;
- frame disposal methods;
- comments/application extensions;
- per-frame extraction and diff.

### JPEG

- marker sequence;
- APPn/COM;
- DQT/DHT/SOS inventory;
- EOI and trailing bytes;
- metadata/appended payload candidates;
- route to specialized JPEG stego backends.

## 7.2 LSB / bit-plane

### `zsteg` — Core/Main transitional

Keep as an oracle and fallback during v1. It has real CTF adoption and a mature search vocabulary for PNG/BMP.

Do not treat all `zsteg -a` detections as facts. Normalize and score results.

### Havk LSB engine — Core/Experimental → Core/Main

Required dimensions:

- bits/sample: 1–4 initially;
- channel selections/permutations;
- LSB/MSB;
- XY/YX traversal;
- row reversal;
- byte/bit packing variants;
- offset/skip;
- raw palette-index mode;
- extraction score: printable ratio, entropy, known magic, YARA-X hit.

Promotion requires differential corpus parity against zsteg plus synthetic ground truth.

## 7.3 Visual steg operations — Core/Main

Native semantic capabilities replacing StegSolve-like GUI workflows:

- channel isolation;
- channel arithmetic;
- per-bit plane generation;
- XOR/diff between images/frames;
- alpha visualization;
- histogram;
- palette usage map;
- frame extraction;
- frame difference;
- coordinate-based crop/enlarge for OCR/barcode.

## 7.4 JPEG stego families

| Tool | Tier | Why keep it |
|---|---|---|
| **StegSeek** | core/main | fast steghide-compatible extraction/cracking |
| **steghide** | core/main compatibility | exact legacy/info/extract behavior |
| **OutGuess** | core/main | distinct JPEG DCT family; modern 2026 CTF evidence shows it solving cases where steghide/JSteg do not |
| **jsteg** | core/main specialist | cheap coverage of another JPEG DCT embedding family |
| **OpenStego** | full/main | additional established data-hiding/watermark formats; Java footprint too heavy for minimal core |

A 2026 SCTF challenge explicitly used an OutGuess payload after steghide and jsteg attempts failed, which is a strong reason not to postpone it.

Reference write-up: https://github.com/hax1ng/SCTF-2026-writeups/blob/main/misc/SYC4113/README.md

## 7.5 Statistical steganalysis

Native experimental methods:

- chi-square;
- RS analysis;
- Sample Pair Analysis.

Never return:

```json
{"stego": true}
```

Return applicability and probabilistic evidence:

```json
{
  "method": "rs",
  "score": 0.71,
  "confidence": "medium",
  "applicability": "lsb-replacement-like embedding",
  "limitations": [
    "probabilistic",
    "may false-positive on natural image statistics",
    "not a generic detector for adaptive steganography"
  ]
}
```

### Aletheia — Optional `stego-ml`

Repository: https://github.com/daniellerch/aletheia

Use as a separate optional image/profile for advanced statistical/ML steganalysis. Do not inflate normal `core`/`full` with its ML stack.

## 7.6 Text steg — Core/Main native

Detect/extract:

- zero-width spaces/joiners;
- bidi controls;
- variation selectors;
- NBSP and unusual whitespace;
- tabs/spaces binary encodings;
- Unicode normalization deltas;
- mixed-script/homoglyph anomalies.

`stegsnow` can remain a DEV oracle, not a runtime dependency.

## 7.7 Audio steg — Core/Main + Full extensions

### Core

- FFmpeg/ffprobe for decode/demux;
- native PCM sample inventory;
- FFT/spectrogram;
- DTMF via Goertzel;
- channel difference;
- PCM LSB extraction;
- silence/tone segmentation.

### Full

- `multimon-ng` for radio/modem protocols;
- `minimodem` for FSK-style modem signals;
- SSTV decoder (experimental until corpus quality is sufficient).

## 7.8 QR/barcode/OCR

| Backend | Tier |
|---|---|
| zxing-cpp | core/main |
| rxing | core/experimental replacement |
| Tesseract | core/main |
| ocrs | core/experimental replacement |

---

# 8. Network / PCAP forensics — revised design

The old `pcap_analyze` + `pcap_extract` split is insufficient. Wireshark's own distribution contains separate tools because these are separate forensic operations.

At research time, the current stable Wireshark release is **4.6.9 (2026-09-23)**. Build the upstream source once in the Docker builder and copy only the required CLI binaries/libraries into runtime.

Official documentation index:

- Wireshark CLI guide: https://www.wireshark.org/docs/wsug_html_chunked/
- TShark: https://www.wireshark.org/docs/man-pages/tshark.html
- capinfos: https://www.wireshark.org/docs/man-pages/capinfos.html
- captype: https://www.wireshark.org/docs/man-pages/captype.html
- editcap: https://www.wireshark.org/docs/man-pages/editcap.html
- mergecap: https://www.wireshark.org/docs/man-pages/mergecap.html
- reordercap: https://www.wireshark.org/docs/man-pages/reordercap.html
- text2pcap: https://www.wireshark.org/docs/man-pages/text2pcap.html
- dumpcap: https://www.wireshark.org/docs/man-pages/dumpcap.html
- sharkd: https://www.wireshark.org/docs/man-pages/sharkd.html

## 8.1 Wireshark CLI suite classification

| Utility | Tier | MCP role | Why separate |
|---|---|---|---|
| **tshark** | core/main | packet dissection, filter, fields, streams, exports, taps/stats | protocol engine |
| **capinfos** | core/main | `pcap_info` | fast capture metadata/stats without packet-table dumping |
| **captype** | core/main internal | type detection inside `pcap_info`/`pcap_validate` | too narrow for its own public tool |
| **editcap** | core/main | `pcap_transform`, `pcap_secrets` | slice/split/dedup/chop/time-shift/encapsulation/secrets |
| **mergecap** | core/main | `pcap_merge` | chronological/appended multi-capture merge |
| **reordercap** | core/main | `pcap_reorder` | timestamp-order repair without conflating it with general editcap transforms |
| **text2pcap** | core/main | `pcap_from_text` | converts hex/text records to analyzable capture; can synthesize headers |
| **dumpcap** | core/main binary, runtime-disabled | `pcap_capture` | live capture, ring buffer, autostop; explicit privilege mode only |
| **sharkd** | core/experimental internal backend | alternate implementation for repeated packet/frame/filter/follow queries | JSON-RPC, same dissector engine; official warning says never expose to untrusted users |
| **rawshark** | not public by default | possible low-level specialist | low value vs tshark/sharkd for current Havk API |

## 8.2 Why these are not redundant

### capinfos

This is a first-pass evidence orientation tool: capture format, encapsulation, packet counts, file/data size, duration, timestamps, packet rate, etc. Recent PCAP forensic write-ups still begin with it because this information often exposes capture anomalies before packet analysis.

Semantic result should include normalized fields, not raw stdout.

### editcap

`editcap` should back a typed transformation API, not a free-form flag string. Useful forensic operations include:

- convert capture format;
- packet-number slicing;
- time-range slicing;
- split by packet count/time interval;
- remove duplicates;
- timestamp shift/strict adjustment;
- snaplen/chop selected bytes;
- encapsulation conversion;
- capture-secret extraction/injection.

All outputs are child artifacts.

### mergecap

Multi-file captures occur in CTFs and real acquisitions. `mergecap` combines multiple captures chronologically by default or in input order when requested.

Recent CTF example: a 2025 247CTF network challenge starts by merging three PCAPs before MPTCP analysis.

### reordercap

Timestamp disorder can prevent correct reconstruction. A documented CTF example used `reordercap` to restore packet ordering before analyzing an ICMP-tunnel capture.

Write-up: https://ctftime.org/writeup/29374

### text2pcap

Useful when the challenge gives raw hex dumps/application payload rather than a PCAP. It supports timestamp/direction parsing, regex extraction, and synthetic Ethernet/IP/TCP/UDP/SCTP headers.

This is a distinct semantic operation: **construct an evidence artifact that Wireshark dissectors can understand**.

### dumpcap

`dumpcap` is for capture, not analysis. It has ring buffers, packet/file/time autostop and capture filters. Wireshark intentionally keeps capture functionality in a smaller component.

Havk should ship it but hide `pcap_capture` unless explicit live mode is enabled.

### sharkd

This deserves experimental evaluation. It exposes a JSON-RPC API with methods such as:

- `load`;
- `analyse`;
- `check`;
- `complete`;
- `frames`;
- `frame`;
- `follow`;
- `download`;
- `iograph`;
- `intervals`;
- `tap`.

That shape is naturally compatible with repeated agent queries and avoids repeatedly starting tshark. However, Wireshark's official manual warns that unfiltered sharkd access may lead to information disclosure or arbitrary command execution. Therefore:

```text
MCP client
   ↓
Havk typed semantic layer
   ↓
isolated worker
   ↓
sharkd console JSON-RPC over local pipe
```

Never:

```text
MCP client → open sharkd TCP socket
```

Promote only after corpus/performance/security testing proves it materially improves repeated-query workflows.

## 8.3 pcapfix

Repository: https://github.com/Rup0rt/pcapfix

`pcapfix` 1.1.7 is old (2021 release) but narrowly scoped, known, and still useful for corrupted pcap/pcapng headers/blocks. It supports pcap and pcapng, deep scan and soft packet detection.

Classification:

```text
core/main specialist
```

Reason: no clearly superior, mature modern replacement was found for this exact repair role. Age alone does not disqualify a stable, narrow file-repair utility. Every repair output is a new child artifact; the original is never overwritten.

## 8.4 Zeek — Full/Main

Repository: https://github.com/zeek/zeek

Zeek complements rather than replaces tshark:

```text
tshark/Wireshark suite → packet/frame/field/stream level
Zeek                    → connection/session/application event logs
```

Use it for larger PCAPs where the model benefits from compact structured connection/DNS/HTTP/TLS/file-event datasets.

Do not automatically run Zeek on every small CTF capture.

## 8.5 Other network candidates

| Tool | Decision | Reason |
|---|---|---|
| tcpdump | optional/live fallback | highly reputable, but redundant for saved-PCAP analysis; dumpcap preferred privileged capture component |
| Suricata | optional separate detection bundle | strong IDS but rule ecosystem/overlap is outside minimal CTF forensic core |
| tcpflow | reject default | stream reconstruction already covered by tshark/sharkd; project lineage less active |
| NetworkMiner | reject default | GUI/.NET workflow poorly matched to headless MCP |
| Arkime | reject current scope | server/indexing platform too heavy for CTF artifact workflow |
| ngrep | reject public tool | simple payload search is covered by `pcap_search` |
| rawshark | internal/watch | no compelling semantic gap yet |

---

# 9. Network semantic MCP API

The following replaces the old two-tool PCAP API.

Legend:

- **S**: normally synchronous;
- **T**: may become MCP Task / fallback job;
- every file-changing operation creates a child artifact.

| MCP tool | Intent | Main backend | Exec |
|---|---|---|---|
| `pcap_info` | capture format, encapsulation, interfaces, packet count, sizes, duration, timestamps, rates, hashes | capinfos + captype | S |
| `pcap_validate` | readability, structural anomalies, interface blocks, timestamp sanity, parser diagnostics | capinfos + tshark + native | S |
| `pcap_repair` | repair damaged pcap/pcapng to child artifact | pcapfix | T |
| `pcap_transform` | convert/slice/split/dedup/timeshift/snaplen/chop/encapsulation | editcap | S/T |
| `pcap_secrets` | inspect/extract/inject supported capture secrets with sensitive-result handling | editcap | S |
| `pcap_merge` | merge capture artifacts chronologically or append-order | mergecap | T |
| `pcap_reorder` | sort frames by timestamp | reordercap | T |
| `pcap_from_text` | parse hex/text dump, timestamp/direction, synthesize L2/L3/L4 headers | text2pcap | S/T |
| `pcap_packets` | pageable packet list, semantic columns, display filter | tshark; exp sharkd | S |
| `pcap_packet` | detailed protocol tree + bounded raw bytes + neighboring context | tshark; exp sharkd | S |
| `pcap_filter` | validate display/capture filter, protocol/field lookup/completion | tshark glossary; exp sharkd `check`/`complete` | S |
| `pcap_fields` | typed extraction of selected known fields with row/byte ceiling | tshark | S/T |
| `pcap_search` | text/hex/field/pattern search over packets/streams | tshark + native | S/T |
| `pcap_follow` | TCP/UDP/HTTP/etc. reassembled stream preview or artifact | tshark; exp sharkd | S/T |
| `pcap_export` | exported protocol objects/files | tshark | T |
| `pcap_stats` | endpoints, conversations, protocol hierarchy, I/O, expert info, response-time stats | tshark taps; exp sharkd tap/iograph | S/T |
| `pcap_protocol` | semantic analyzer for DNS/HTTP/TLS/SMB/FTP/SMTP/DHCP/QUIC/MQTT/etc. | tshark | S/T |
| `pcap_ctf_decode` | USB HID, DNS exfil, ICMP payload, basic covert-channel reconstruction | tshark fields + native Rust | S/T |
| `network_logs` | generate/query Zeek connection/application logs | Zeek | T |
| `pcap_capture` | live capture with filter, snaplen, autostop, ring buffer | dumpcap | T; disabled by default |

## 9.1 Why `pcap_protocol` remains one tool

Do not create `dns_analyze`, `http_analyze`, `tls_analyze`, `mqtt_analyze`, etc. for every dissector. That would recreate HexStrike's schema-count problem.

Use a typed protocol enum and backend mapping:

```json
{
  "artifact_id": "art_capture",
  "protocol": "dns",
  "operation": "summary",
  "filter": null,
  "limit": 500
}
```

The server knows the correct tshark fields/taps for each supported semantic protocol mode.

## 9.2 CTF-specific network decoders

`pcap_ctf_decode` should have explicit modes rather than arbitrary scripts:

```text
usb_keyboard
usb_mouse
usb_storage_summary
dns_labels
dns_txt
icmp_payload
tcp_payload_concat
http_body_candidates
timing_channel_basic
ip_id_channel_basic
```

Some modes can be experimental at first. The point is to stop forcing the agent to write ad-hoc Python every time a common CTF representation reappears.

---

# 10. PDF/document forensics — mutool reassessment

Gemini's suggestion to use only `mutool` is technically understandable because `mutool` is a real Swiss-army knife. It is **not** the best default architecture for Havk unless licensing and parser diversity are intentionally traded away.

## 10.1 What mutool actually gives us

Current MuPDF docs expose, among others:

- `mutool show`: print object dictionaries and decoded/raw streams, trailer/xref, object paths; can force repair;
- `mutool clean`: rewrite/repair, garbage collect, decompress/recompress streams, decrypt, pretty-print;
- `mutool draw`: render pages, extract text/structured text including JSON/XML, OCR support, memory limits/low-memory mode;
- `mutool extract`: extract images and fonts;
- `mutool info`: page objects/resources information;
- `mutool pages`: page boxes/rotation/user units;
- `mutool grep`: content search;
- `mutool audit`: object/operator/storage usage report;
- `mutool run`: JavaScript access to low-level PDF APIs, including embedded files.

This is extremely attractive for a forensic MCP.

## 10.2 Why not `mutool` only

### 1. License

MuPDF open-source releases are **AGPL**; Artifex offers a commercial alternative. Bundling it in a Docker-delivered Havk product requires an explicit licensing decision and compliance review.

This document is not legal advice, but the license is strong enough that it should affect default-backend architecture.

### 2. Independent parser disagreement is useful forensic evidence

Malformed/adversarial PDFs regularly trigger different parser behavior. A forensic system benefits from:

```text
qpdf validation/object model
pdfcpu independent validator
mutool optional parser/renderer
Didier security-oriented raw inspection
```

Disagreement can itself be surfaced as an observation.

### 3. qpdf has an MCP-friendly JSON representation

qpdf JSON v2 provides a stable object/document representation designed for machine processing. That makes normalization safer than parsing arbitrary human CLI text.

### 4. mutool extraction is not one universal attachment command

`mutool extract` directly focuses on images/fonts. Embedded/associated files can be accessed through MuPDF's document APIs / `mutool run`, which is powerful but means a “one binary solves everything” claim needs a custom scripting layer anyway.

## 10.3 Recommended PDF backend tiers

| Backend | Tier | Role |
|---|---|---|
| **qpdf** | core/main | structure, JSON object model, validation, repair, decrypt, normalization |
| **pdfcpu** | core/main | independent Go parser/validator, attachments/images/forms/signatures, additional corruption detection |
| **pdfid.py** | core/main security specialist | quick suspicious-key triage |
| **pdf-parser.py** | core/main compatibility/specialist | object/stream inspection familiar in CTF/security workflows |
| **mutool** | optional `pdf-agpl` or full/main if licensing accepted | independent parser, render/text/OCR, object view, repair, resources, audit, JS API |
| **Poppler CLI** | optional core/full depending final license/footprint decision | `pdftotext`, `pdfimages`, `pdfdetach` compatibility |
| **lopdf** | core/experimental | Rust parser; worker only until hostile-PDF corpus proves robustness |

`pdfcpu` is especially interesting for Havk because current releases provide Linux binaries/checksums/SBOMs, stateless configuration support, cancellation-aware APIs, resource limits and recent malformed-PDF security hardening.

## 10.4 Modern CTF evidence for mutool

Mutool is absolutely worth supporting. Recent CTF PDF write-ups use it for tasks such as:

- extracting embedded/custom fonts from redacted PDFs;
- extracting a hidden page image from a PDF;
- decompressing PDF streams (`mutool clean -d`) before searching hidden content.

Examples:

- Securinets 2026 PDF custom-font challenge: mutool extraction used to recover the hidden font.
- Nullcon CTF 2026: `mutool extract` exposed an image underneath apparent redaction.

That justifies support; it does **not** justify eliminating qpdf/pdfcpu.

---

# 11. PDF semantic MCP API

Replace one `document_analyze` mega-tool with narrower operations.

| MCP tool | Intent | Main backend | Optional/experimental | Exec |
|---|---|---|---|---|
| `pdf_triage` | version, pages, encryption, metadata, suspicious names/actions, validator disagreement | qpdf + pdfcpu + pdfid + ExifTool | mutool info/pages | S |
| `pdf_objects` | trailer/xref/object tree, selected object, raw/decoded stream | qpdf JSON + pdf-parser | mutool show, lopdf | S/T |
| `pdf_search` | names/keys/text/decoded streams/YARA-X patterns | pdf-parser + qpdf-normalized streams + native/YARA-X | mutool grep/show | S/T |
| `pdf_extract` | images, attachments, fonts, selected streams/objects | qpdf/pdfcpu + specialist extractors | mutool extract/run, Poppler | T |
| `pdf_render` | selected pages/regions to image; OCR-ready output | optional mutool or chosen renderer | Poppler/Tesseract | T |
| `pdf_repair` | repair/normalize/decompress to child artifact | qpdf + pdfcpu | mutool clean | T |
| `pdf_decrypt` | decrypt known password to child artifact | qpdf/pdfcpu | mutool clean | T |
| `pdf_audit` | object/storage/operator/resource anomalies, cross-parser disagreement | qpdf + pdfcpu + native metrics | mutool audit | S/T |

`pdf_render` should not be mandatory in minimal `core` if the selected renderer's license/footprint conflicts with Havk goals; it can be installed as a profile component.

---

# 12. Office and generic document forensics

Still within scope because Office/PDF are recurring forensic evidence formats.

## Core/Main

- `oletools`: VBA/macros/OLE/RTF objects;
- `msoffcrypto-tool`: Office encryption/decryption when password is available;
- 7zz/native ZIP view for OOXML container structure;
- ExifTool for metadata.

## Semantic API

| Tool | Intent |
|---|---|
| `office_triage` | format/container, encryption state, metadata, macro presence, embedded object inventory |
| `office_macros` | list/extract/analyze VBA streams and suspicious constructs |
| `office_extract` | embedded objects/media/package members to artifacts |
| `office_decrypt` | known-password decryption to child artifact |

Do not add generic document editing/conversion unrelated to forensics.

---

# 13. Archive, carving and recovery

## Core/Main

- 7zz;
- Binwalk v3;
- native Havk signature carver;
- bkcrack;
- fcrackzip;
- pdfcrack where PDF password workflow needs it.

## Full/Main

- unblob;
- Deark;
- PhotoRec;
- bulk_extractor.

## 13.1 bkcrack stays Core/Main

Recent 2025–2026 CTFs repeatedly use known-plaintext ZipCrypto attacks. The important abstraction is not “brute force ZIP password” but:

```text
identify ZipCrypto
→ inspect candidate known plaintext
→ recover internal keys
→ decrypt selected/all members
```

Examples:

- RITSEC CTF 2026 “Zipped Up”;
- SK-CERT CyberGame 2026 “Zippy zip”.

## 13.2 Semantic API

| Tool | Intent |
|---|---|
| `archive_info` | member list, sizes, methods, encryption, ratio/bomb/path anomalies |
| `archive_extract` | selected/all bounded extraction |
| `archive_repair` | ZIP structure diagnosis/repair candidate |
| `archive_crypto` | scheme detect, dictionary mode, ZipCrypto known-plaintext, decrypt |
| `carve_scan` | signatures/embedded structures with offsets |
| `carve_extract` | selected candidates / bounded recursion |
| `raw_recover` | full-profile PhotoRec-style recovery task |
| `forensic_features` | full-profile bulk_extractor features/histograms |

---

# 14. Disk/filesystem forensics

## Core/Main

### The Sleuth Kit

Repository: https://github.com/sleuthkit/sleuthkit

Semantic roles:

- partitions/volumes;
- filesystem info;
- files/directories;
- deleted entries;
- inode/metadata;
- file extraction.

### libewf / ewf-tools

Use current upstream source build for E01/Ex01 rather than relying on an ancient distro package. Normal Havk workflows should use one-shot read/export APIs/tools; do not require FUSE `ewfmount`.

### 7zz fast path

Use 7zz when it can list/extract a disk/container without root or mount. TSK remains the forensic-aware path.

## Full/Main

- PhotoRec for metadata-independent signature recovery;
- Plaso/log2timeline for timelines;
- bulk_extractor;
- qemu-img for virtual disk conversion/info when necessary.

## Semantic API

| Tool | Intent |
|---|---|
| `disk_info` | disk/container/EWF metadata, geometry/image properties |
| `disk_partitions` | partition/volume layout |
| `fs_info` | filesystem parameters |
| `fs_list` | directory/deleted entry listing with paging |
| `fs_stat` | inode/file metadata |
| `fs_extract` | path/inode extraction to artifact |
| `fs_recover` | PhotoRec/full raw recovery |
| `timeline_build` | Plaso targeted/super timeline |

TestDisk partition-write/repair functionality remains excluded from the normal read-only MCP surface.

---

# 15. Memory forensics

## Volatility 3 — Full/Main

Repository: https://github.com/volatilityfoundation/volatility3

Do not expose arbitrary plugin names as a public free-form field by default. Havk should map stable semantic operations to vetted plugin sets.

## MemProcFS — Full/Experimental/optional

Repository: https://github.com/ufrisk/MemProcFS

Worth evaluating because it offers batch forensic mode, Linux builds, Rust APIs and useful process/network/registry/file/timeline capabilities. Keep experimental because:

- AGPL distribution implications;
- mount/VFS-oriented lifecycle for many workflows;
- overlap with Volatility + Windows artifact stack;
- Havk needs one-shot worker behavior rather than long-lived mount dependence.

## Semantic API

| Tool | Intent |
|---|---|
| `memory_info` | layer/OS/kernel/symbol diagnostics |
| `memory_processes` | process tree/list/cmdline summary |
| `memory_network` | sockets/connections |
| `memory_modules` | DLLs/modules/drivers/malfind-oriented summaries |
| `memory_files` | file-object inventory/scan |
| `memory_scan` | YARA-X / targeted memory scan |
| `memory_dump` | selected process/module/file dump to artifact |

All memory operations are Tasks above trivial thresholds.

---

# 16. Windows artifact forensics

Keep only artifacts that directly support digital forensics; do not expand into full enterprise IR orchestration in this document.

## Main

- Rust `evtx` parser in isolated worker;
- `regipy` and/or proven registry parsers;
- libyal family where useful: `liblnk`, `libscca`, `libpff`;
- TSK for NTFS context.

## Experimental replacements

- Rust `mft` parser;
- ForensicRS components after differential testing.

## Optional hunt bundle

- Hayabusa: https://github.com/Yamato-Security/hayabusa
- Chainsaw: https://github.com/WithSecureOpenSource/chainsaw

Both are useful for event-log hunting/timeline workflows but have copyleft licensing considerations and overlap. Do not expose separate `hayabusa_run` and `chainsaw_run`; provide one semantic `win_hunt` capability and select the configured backend.

## Semantic API

| Tool | Intent |
|---|---|
| `win_evtx` | query/summary/export events |
| `win_hunt` | Sigma/rules/timeline hunt |
| `win_registry` | hive/keys/values/timestamps and common artifact paths |
| `win_mft` | MFT records/paths/times/resident data/anomalies |
| `win_prefetch` | application execution metadata |
| `win_lnk` | target, volume, path, timestamps, network info |
| `win_pst` | message/folder/attachment extraction |

---

# 17. SQLite forensic analysis

SQLite appears in browser/app/mobile/Windows artifacts even when we are not yet building those entire domains, so direct database forensics belongs here.

## Core/Main

`rusqlite` read-only/immutable path:

- schema;
- table/view/index inventory;
- bounded rows;
- BLOB inventory/extraction;
- sidecar (`-wal`, `-shm`, journal) inventory;
- timestamp/text heuristics.

## Full/Experimental

Evaluate `sqlite-forensic` / sqlite4n6:

- https://github.com/SecurityRonin/sqlite-forensic

Reason: focused read-only recovery from WAL/journal/freelist/deleted regions with corpus-oriented validation. Too new for Main.

## Semantic API

```text
database_info
database_query
database_extract
database_recover   # full/experimental
```

---

# 18. Master semantic MCP API

The full catalog is intentionally richer than 21 mega-tools. Profiles keep prompt/schema cost manageable.

## 18.1 Control and artifacts

1. `doctor`
2. `artifact_query`
3. `artifact_read`
4. `artifact_export`
5. `job`

## 18.2 Generic forensic file tools

6. `file_identify`
7. `file_hash`
8. `file_strings`
9. `file_hex`
10. `file_entropy`
11. `file_scan`
12. `file_structure`
13. `file_repair`
14. `metadata_read`
15. `metadata_extract`

## 18.3 Archive/carving

16. `archive_info`
17. `archive_extract`
18. `archive_repair`
19. `archive_crypto`
20. `carve_scan`
21. `carve_extract`
22. `raw_recover` *(full)*
23. `forensic_features` *(full)*

## 18.4 Steganography

24. `stego_triage`
25. `stego_lsb_scan`
26. `stego_lsb_extract`
27. `stego_planes`
28. `stego_palette`
29. `stego_frames`
30. `stego_text`
31. `stego_jpeg`
32. `stego_statistics`
33. `image_compare`
34. `image_ocr`
35. `image_barcode`
36. `media_probe`
37. `media_extract`
38. `audio_spectrogram`
39. `audio_signal_decode`
40. `audio_stego`

## 18.5 PDF/Office

41. `pdf_triage`
42. `pdf_objects`
43. `pdf_search`
44. `pdf_extract`
45. `pdf_render`
46. `pdf_repair`
47. `pdf_decrypt`
48. `pdf_audit`
49. `office_triage`
50. `office_macros`
51. `office_extract`
52. `office_decrypt`

## 18.6 Network/PCAP

53. `pcap_info`
54. `pcap_validate`
55. `pcap_repair`
56. `pcap_transform`
57. `pcap_secrets`
58. `pcap_merge`
59. `pcap_reorder`
60. `pcap_from_text`
61. `pcap_packets`
62. `pcap_packet`
63. `pcap_filter`
64. `pcap_fields`
65. `pcap_search`
66. `pcap_follow`
67. `pcap_export`
68. `pcap_stats`
69. `pcap_protocol`
70. `pcap_ctf_decode`
71. `network_logs` *(full)*
72. `pcap_capture` *(explicit live mode only)*

## 18.7 Disk/filesystem

73. `disk_info`
74. `disk_partitions`
75. `fs_info`
76. `fs_list`
77. `fs_stat`
78. `fs_extract`
79. `fs_recover` *(full)*
80. `timeline_build` *(full)*

## 18.8 Memory

81. `memory_info` *(full)*
82. `memory_processes` *(full)*
83. `memory_network` *(full)*
84. `memory_modules` *(full)*
85. `memory_files` *(full)*
86. `memory_scan` *(full)*
87. `memory_dump` *(full)*

## 18.9 Windows artifacts

88. `win_evtx` *(full)*
89. `win_hunt` *(full/optional hunt bundle)*
90. `win_registry` *(full)*
91. `win_mft` *(full)*
92. `win_prefetch` *(full)*
93. `win_lnk` *(full)*
94. `win_pst` *(full)*

## 18.10 Database

95. `database_info`
96. `database_query`
97. `database_extract`
98. `database_recover` *(full/experimental)*

The `all` profile therefore approaches ~100 semantic tools, but **normal clients should not receive the `all` profile**. That is the lesson from HexStrike's 150+ tool-count problem.

---

# 19. Recommended MCP tool profiles

## 19.1 `forensics`

A practical default forensic profile should expose roughly 45–60 tools, not all 98.

Include:

```text
control/artifact
file + metadata
archive + carving
PDF/Office
database core
PCAP offline core
disk core
```

Exclude by default:

```text
live capture
memory
Windows specialized parsers
stego statistics/ML
Zeek/timeline/raw recovery heavy paths
```

## 19.2 `stego`

Expose:

```text
control/artifact
file structure/metadata
carve/archive core
all stego/image/audio tools
PDF extraction/render (because PDFs often contain hidden images/fonts/streams)
small subset of PCAP covert-channel tools if network steg is desired
```

## 19.3 `network`

Expose all 20 network tools plus:

```text
artifact/file/hash/strings/entropy
carve_scan/carve_extract
archive_info/archive_extract
YARA-X scan
```

This allows recovered HTTP/SMB/TFTP objects to flow directly into forensic analysis without a second MCP server.

## 19.4 `all`

For dedicated forensic clients/evaluation only. Tool-schema cost must be measured and documented.

---

# 20. Main / Experimental backend matrix

## 20.1 Core/Main

```text
Havk native primitives
YARA-X
Magika
ExifTool
Binwalk v3
7zz
zsteg (transitional)
StegSeek
steghide
OutGuess
jsteg
zxing-cpp
Tesseract
FFmpeg/ffprobe
qpdf
pdfcpu
pdfid.py / pdf-parser.py
oletools
msoffcrypto-tool
bkcrack
fcrackzip
pdfcrack
Wireshark CLI suite:
  tshark
  capinfos
  captype
  editcap
  mergecap
  reordercap
  text2pcap
  dumpcap (disabled runtime capability)
pcapfix
The Sleuth Kit
libewf
rusqlite
```

## 20.2 Core/Experimental

```text
Havk native generalized LSB scanner (until parity)
chi-square / RS / SPA
sharkd internal backend
OxiDex
rxing
ocrs
lopdf
```

## 20.3 Full/Main

```text
unblob
Deark
PhotoRec
bulk_extractor
Plaso
Zeek
qemu-img
Volatility 3
regipy
libyal LNK/Prefetch/PST family
ImageMagick fallback
multimon-ng
minimodem
OpenStego
```

## 20.4 Full/Experimental / optional

```text
Rust mft parser
ForensicRS
sqlite-forensic/sqlite4n6
MemProcFS
SSTV decoder
Hayabusa/Chainsaw optional copyleft hunt bundle
mutool optional AGPL PDF backend
```

## 20.5 Separate optional

```text
stego-ml:
  Aletheia
```

---

# 21. Backends deliberately not default

| Tool | Decision | Reason |
|---|---|---|
| YARA classic | replace for new Havk rules | YARA-X successor path |
| Binwalk v2 | reject | v3 upstream Rust rewrite |
| Foremost | not default | native carver + PhotoRec/unblob cover better-defined roles |
| Scalpel | not default | same overlap |
| StegSolve | no runtime dependency | GUI functions become typed bit-plane/palette/frame tools |
| StegCracker | reject | StegSeek |
| stegsnow | no runtime dependency | native whitespace/unicode steg |
| pngcheck | DEV oracle | native PNG parser/repair |
| ZBar | reject default | zxing-cpp main, rxing experimental |
| SoX | no core requirement | FFmpeg + native DSP cover the selected scope |
| Suricata | optional only | IDS/rules ecosystem overlaps Zeek/tshark for CTF forensic scope |
| tcpflow | reject default | stream reconstruction already available |
| Arkime | reject current scope | heavy server/indexing platform |
| TestDisk write/repair operations | reject normal public API | destructive partition operations conflict with evidence immutability |
| ewfmount/xmount | reject normal path | long-lived mount state/FUSE unnecessary |
| arbitrary `vol.py <plugin>` | reject public API | semantic allowlisted memory operations only |
| generic shell / `additional_args` | reject | injection risk and unstable contract |

---

# 22. Doctor and capability probing

`doctor` should verify behavior, not just executable presence.

Example capability probes:

## Wireshark suite

```text
versions all match expected lock
capinfos parses fixture
tshark JSON/fields fixture
editcap transforms without mutation
mergecap combines two fixture captures
reordercap corrects known disorder
text2pcap round-trips known hex fixture
dumpcap binary exists but reports live capability disabled unless server mode allows it
sharkd experimental JSON-RPC handshake if installed
```

## PDF

```text
qpdf JSON parses fixture
qpdf validator handles malformed fixture within limits
pdfcpu strict + relaxed validation fixture
pdfid suspicious-key fixture
pdf-parser selected-object fixture
mutool optional backend only if profile/license-enabled
```

## Stego

```text
zsteg known PNG fixture
StegSeek known steghide fixture
OutGuess known JPEG fixture
JSteg known JPEG fixture
native LSB parity fixture
GIF local palette fixture
APNG frame fixture
```

---

# 23. Golden corpus

Tests should assert forensic capability, not exact stdout.

```text
tests/corpus/
├── file/
│   ├── extension-mismatch/
│   ├── appended/
│   ├── polyglot/
│   └── damaged/
├── stego/
│   ├── png-lsb/
│   ├── png-palette/
│   ├── gif-local-palette/
│   ├── apng/
│   ├── jpeg-steghide/
│   ├── jpeg-outguess/
│   ├── jpeg-jsteg/
│   ├── text/
│   └── audio/
├── archive/
│   ├── zipcrypto-known-plaintext/
│   ├── encrypted/
│   └── truncated/
├── pdf/
│   ├── object-stream/
│   ├── embedded-file/
│   ├── javascript-openaction/
│   ├── hidden-image/
│   ├── custom-font/
│   ├── xref-damaged/
│   └── encrypted/
├── pcap/
│   ├── metadata/
│   ├── out-of-order/
│   ├── damaged-pcap/
│   ├── damaged-pcapng/
│   ├── merge/
│   ├── text2pcap/
│   ├── http-export/
│   ├── smb-export/
│   ├── usb-hid/
│   ├── dns-exfil/
│   └── icmp-covert/
├── disk/
│   ├── ext4-deleted/
│   ├── ntfs-deleted/
│   ├── e01/
│   └── raw-recovery/
├── memory/
│   └── small-published-images/
├── windows/
│   ├── evtx/
│   ├── registry/
│   ├── mft/
│   ├── prefetch/
│   └── lnk/
└── sqlite/
    ├── wal/
    ├── journal/
    └── deleted/
```

## 23.1 Differential tests

Use older/external tools as oracles even when they are not shipped:

```text
native LSB ↔ zsteg
native PNG parser ↔ pngcheck
OxiDex ↔ ExifTool
rxing ↔ zxing-cpp
ocrs ↔ Tesseract
lopdf ↔ qpdf/pdfcpu/mutool
sqlite4n6 ↔ sqlite-dissect / known-ground-truth corpus
sharkd ↔ tshark result equivalence
```

---

# 24. Recommended first implementation order

## Phase A — MCP and artifact foundation

1. `rmcp` 3.x server for stdio + Streamable HTTP;
2. artifact registry;
3. immutable child-artifact writes;
4. worker execution budgets;
5. Tasks + `job` fallback;
6. tools.lock + doctor framework.

## Phase B — high-value Core/Main

1. generic file primitives;
2. ExifTool/YARA-X/Magika;
3. 7zz/Binwalk;
4. raw PNG/GIF/APNG/JPEG parsers;
5. zsteg/StegSeek/OutGuess/jsteg;
6. qpdf/pdfcpu/Didier PDF layer;
7. complete Wireshark CLI suite semantics;
8. TSK/libewf.

## Phase C — CTF-native logic

1. native LSB parity;
2. USB HID;
3. DNS/ICMP covert reconstruction;
4. audio spectrogram/DTMF/PCM LSB;
5. native repair primitives;
6. archive known-plaintext workflow.

## Phase D — Full

1. Volatility 3;
2. Zeek;
3. PhotoRec/bulk_extractor/Plaso;
4. Windows artifacts;
5. unblob/Deark;
6. experimental sharkd;
7. optional mutool/AGPL profile if licensing decision permits it.

---

# 25. Key research conclusions

## Network

The important correction is that **TShark is the dissector, not the whole forensic toolchain**. Capinfos, editcap, mergecap, reordercap, text2pcap, dumpcap and potentially sharkd deserve explicit backend roles because they solve different forensic transformations and acquisition problems.

## PDF

The important correction is the opposite of Gemini's “mutool only” simplification: **mutool deserves first-class support, but parser/tool diversity is valuable and MuPDF's AGPL/commercial license matters for a Docker-distributed MCP**. qpdf + pdfcpu should remain the machine-structured permissive default; Didier scripts remain useful security specialists; mutool is an excellent optional independent engine.

## Steganography

The important correction is to keep raw representation as evidence. PNG/GIF/APNG/JPEG structures, palette indices, bit depths and frame structures cannot be safely reduced to a generic RGB image before analysis.

## MCP design

The important correction from HexStrike is that breadth should live **behind profiles and semantic contracts**, not in a 150-tool flat list and not behind `additional_args` strings.

---

# 26. Primary references

## MCP / Rust

- MCP specification: https://modelcontextprotocol.io/
- MCP source/spec repository: https://github.com/modelcontextprotocol/modelcontextprotocol
- Official Rust SDK (`rmcp`): https://github.com/modelcontextprotocol/rust-sdk
- rmcp migration guide: https://github.com/modelcontextprotocol/rust-sdk/discussions/969
- rmcp roadmap: https://github.com/modelcontextprotocol/rust-sdk/blob/main/ROADMAP.md

## MCP implementations reviewed

- HexStrike-AI: https://github.com/0x4m4/hexstrike-ai
- Wireshark-MCP: https://github.com/bx33661/Wireshark-MCP
- ctf-buster: https://github.com/agentfanclub/ctf-buster
- CTF-MCP: https://github.com/Coff0xc/CTF-MCP
- ctfd-mcp-server: https://github.com/MrJamescot/ctfd-mcp-server
- steganography-mcp: https://github.com/badchars/steganography-mcp
- Mulder: https://github.com/calebevans/mulder
- SIFT-MCP / Valhuntir: https://github.com/AppliedIR/sift-mcp
- volatility-mcp: https://github.com/Gaffx/volatility-mcp

## Network

- Wireshark: https://www.wireshark.org/
- Wireshark command-line docs: https://www.wireshark.org/docs/wsug_html_chunked/
- tshark: https://www.wireshark.org/docs/man-pages/tshark.html
- capinfos: https://www.wireshark.org/docs/man-pages/capinfos.html
- editcap: https://www.wireshark.org/docs/man-pages/editcap.html
- mergecap: https://www.wireshark.org/docs/man-pages/mergecap.html
- reordercap: https://www.wireshark.org/docs/man-pages/reordercap.html
- text2pcap: https://www.wireshark.org/docs/man-pages/text2pcap.html
- dumpcap: https://www.wireshark.org/docs/man-pages/dumpcap.html
- sharkd: https://www.wireshark.org/docs/man-pages/sharkd.html
- pcapfix: https://github.com/Rup0rt/pcapfix
- Zeek: https://github.com/zeek/zeek

## PDF/document

- MuPDF / mutool docs: https://mupdf.readthedocs.io/en/latest/tools/mutool.html
- MuPDF licensing/releases: https://mupdf.com/releases
- qpdf: https://github.com/qpdf/qpdf
- qpdf JSON docs: https://qpdf.readthedocs.io/en/latest/json.html
- pdfcpu: https://github.com/pdfcpu/pdfcpu
- Didier Stevens Suite: https://github.com/DidierStevens/DidierStevensSuite
- oletools: https://github.com/decalage2/oletools
- msoffcrypto-tool: https://github.com/nolze/msoffcrypto-tool

## Steganography

- zsteg: https://github.com/zed-0xff/zsteg
- StegSeek: https://github.com/RickdeJager/stegseek
- steghide: https://github.com/StegHigh/steghide
- OutGuess maintained source: https://github.com/resurrecting-open-source-projects/outguess
- JSteg: https://github.com/lukechampine/jsteg
- OpenStego: https://github.com/syvaidya/openstego
- Aletheia: https://github.com/daniellerch/aletheia

## Forensic foundation

- YARA-X: https://github.com/VirusTotal/yara-x
- Magika: https://github.com/google/magika
- Binwalk: https://github.com/ReFirmLabs/binwalk
- ExifTool: https://github.com/exiftool/exiftool
- 7-Zip: https://www.7-zip.org/
- unblob: https://github.com/onekey-sec/unblob
- Deark: https://github.com/jsummers/deark
- The Sleuth Kit: https://github.com/sleuthkit/sleuthkit
- TestDisk/PhotoRec: https://www.cgsecurity.org/wiki/TestDisk_Download
- bulk_extractor: https://github.com/simsong/bulk_extractor
- Plaso: https://github.com/log2timeline/plaso
- Volatility 3: https://github.com/volatilityfoundation/volatility3
- MemProcFS: https://github.com/ufrisk/MemProcFS
- bkcrack: https://github.com/kimci86/bkcrack
- sqlite-forensic: https://github.com/SecurityRonin/sqlite-forensic

## Selected CTF evidence

- SCTF 2026 OutGuess multi-stage steg: https://github.com/hax1ng/SCTF-2026-writeups/blob/main/misc/SYC4113/README.md
- IJCTF `reordercap` example: https://ctftime.org/writeup/29374
- RITSEC 2026 bkcrack: https://medium.com/@wireshark.pcap/ritsec-ctf-2026-zipped-up-writeup-by-wireshark-pcap-b2979b696bae
- Nullcon 2026 PDF image extraction with mutool: https://www.fu11shoot.com/en/writeups/nullcon-2026/writeuprdctd3/

---

# 27. Final proposed architecture

```text
                 MCP client
                     │
              MCP 2026-07-28
                     │
          ┌──────────▼──────────┐
          │ Havk MCP / rmcp 3.x│
          │ typed tool profiles│
          └──────────┬──────────┘
                     │
         ┌───────────┼─────────────┐
         │           │             │
  Artifact store   Task/job     Policy/router
  + provenance     registry     + budgets
         │           │             │
         └───────────┼─────────────┘
                     │
              isolated worker
                     │
       ┌─────────────┼──────────────┐
       │             │              │
  Native Rust   Rust crates    Upstream CLI
       │             │              │
 raw formats      YARA-X       Wireshark suite
 steg/CTF logic   rusqlite      ExifTool / 7zz
 decoding                       qpdf/pdfcpu
                                TSK/libewf
                                Volatility/etc.
```

The design objective is not the smallest backend count. It is the smallest **stable semantic surface per profile** that still lets an agent investigate evidence iteratively without escaping to arbitrary shell commands.

