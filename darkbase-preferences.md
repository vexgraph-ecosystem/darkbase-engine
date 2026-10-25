# darkbase — Repo-Local Living Preferences
> Repo-local preferences governed by the Living Documentation Law.
> Universal constitution: `gist.md` is the compressed law list; `PREFERENCES.md` is its rationale.

## 0. Constitution Link (supreme)
- [gist.md](https://gist.github.com/vex-graph/4132a6c45cb6d3797c3e8eff2e94035a#file-gist-md) — the compressed law list, mirrored in this workspace at `../../../metadata/gists/preferences/gist.md`. It is the only normative law text; `../../../metadata/gists/preferences/PREFERENCES.md` carries the rationale and is never ingested whole. Numbers are slice addresses inside one file, never law identity: cite by Title.
- All universal laws bind this repository. Read one on demand from the workspace root: `./tools/prefs law "<title>"`, `./tools/prefs section "<section>"`, `./tools/prefs repo <name>`.
- This document codifies **exclusive** preferences for `darkbase` (R3 Database Driver). Database semantics and persistence remain R3; allocation/storage and native span search belong to Relational Engine R2, with Vexspoke supplying CPU computation/behavior. Either R2 public contract may be borrowed. This blueprint has no implemented engine integration; migration and durability require owner proof.
- darkbase is a **native vex store, not a SQL engine**. The schema is C reflection; the native query contract is structured, and any text/SQL parser is an optional `cli/` convenience or a foreign driver's private translation.

## 1. Repo-Local Law Index (Binding Matrix)

Universal laws inherit from the compressed constitution (`../../../metadata/gists/preferences/gist.md`); the rows below are repo-specific additions.

| Law Title | Scope | Enforcement |
| :--- | :--- | :--- |
| **Entity Model Law** | R3 Database Driver | Mandatory for `darkbase` |
| **Reactive Program Law** | R3 Database Driver | Mandatory for `darkbase` |
| **Store Versus Executable Separation Law** | R3 Database Driver | Mandatory for `darkbase` |
| **Byte-Native Persistence Law** | R3 Database Driver | Mandatory for `darkbase` |
| **Zero-Alloc Relational Table Scan Law** | R3 Database Driver | Mandatory for `darkbase` |
| **B-Tree Page Consistency Law** | R3 Database Driver | Mandatory for `darkbase` |
| **Row Borrow and Owned Copy Law** | R3 Database row ownership | Mandatory for `darkbase` |
| **Durable Page Store Boundary Law** | Single-file page-level Store | Embedded redo ordering, reopen, refusal and reuse owner proof |

## 2. Exclusive Repo-Local Laws (FULL PROSE RESTATEMENT)

### Entity Model Law
**Definition.** darkbase defines no schema types; it consumes R2 reflection: `struct` = `Struct` (a table: ordered `Field` list, `size` = entity row stride), `class` = `Class` (`Struct` plus constructor and `Method`s), `field` = `Field`, `function` = `Method`; `Field` carries behavior (name + read/set + target) and physical layout (typeId, offset, size, flags); plain values are named typed bindings in the same scope — one shared C-types-as-schema vocabulary serving reflection, persistence and export without drift. **Rule.** 1. **Reflection IS the schema** — no ad-hoc schema language, no darkbase-owned descriptor duplicating `Field`. 2. **Field owns the physical layout** — persistence/export read offset, size, typeId from the `Field` (or `Struct` row size), never a hand-maintained table; Vexspoke's reflection `Field` is the single source of truth. 3. **One binding namespace** — Structs, Methods and plain values resolve through the same scope and naming grammar.

### Row Borrow and Owned Copy Law
**Definition.** `Database_insert` stores a borrowed row pointer; the caller keeps row and schema alive for the Database lifetime. `Database_insertOwned` copies one exact `Struct.size` byte row into engine-owned memory. One ownership mode per entity — borrowed and owned never mix; loaded rows use owned mode, released with the Database — the zero-copy path with a clear release owner. **Rule.** 1. Reject a copy-in whose byte length is not exactly the registered schema stride; count and ownership mode unchanged. 2. Reject mode mixing on one entity; owned insertion copies bytes, never retains the caller's address. 3. Free each owned row exactly once with its Database; never free borrowed rows; schemas stay borrowed, outlive the Database. 4. Snapshot compatibility loading validates the full input and stages all owned rows before publishing any entity — all-or-nothing, limited to that legacy loader; the M2 flat dump is not the specified persistence design (pages plus WAL).

### Reactive Program Law
**Definition.** The reactive layer is the SQL analog of triggers, views and stored procedures — first-class named programs bound to store events (`beforeInsert`, `afterInsert`, `beforeUpdate`, `afterDelete`, `onOpen`, `onCommit`, `onRollback`); persistence hooks ship with the data and export with it. **Rule.** 1. **Programs fire on the cold admission path only**, never per element on a hot read (Cold-Strict, Hot-Minimal Validation Law; Cold-Only Reflection Law). 2. **Programs are data plus behavior** — exported as a descriptor; code rebinds by name on load, never by a stored address. 3. **Views are derived Structs** — rows computed by a program against the authoritative store, not a second synchronized copy.

### Store Versus Executable Separation Law
**Definition.** The R3 repository is the `Database` interface and native vex store (future name `darkbase-db`); the R5 executable `darkbase` is an interactive host that opens a `.vexdb`. Conflating them couples the store to a UI and blocks other consumers. **Rule.** 1. **The store never includes R5** — hosts borrow an opaque `Database*` plus callbacks (Vertical Integration Law). 2. **The executable is a separate application** owning no store internals. 3. **Naming is deliberate:** store = `darkbase-db` (future), executable = `darkbase`.

### Byte-Native Persistence Law
**Definition.** Values are raw byte spans (`uint8_t[]`), written in memory order and read back the same with **no byte-order machinery** — no endianness detection, byte swapping or text encoding; `toString` is a cold converter for humans, never storage. All supported hosts are little-endian (Apple Silicon arm64 floor; Windows x86_64), so memory order beats a swap layer. **Rule.** 1. **Bytes, not text, for storage** — the schema (`Field.typeId` + offset/size) interprets durable spans; no string encoding in a file. 2. **No endianness code** — no swap helpers or byte-order fields; a `// INTENTIONAL(vex)` note states this at the format definition. 3. **`toString` is a cold converter** — debugging only (the toString Law). 4. **Pointer-free rows** — persisted row bytes carry no raw pointers; references and functions rebind by name/index on load. 5. **Canonical variable** — `{ uint8_t name[24], void* pointer }` (the engine's `VariableSlot` shape): names 24 bytes, the `void*` value a pointer to its bytes.

### Zero-Alloc Relational Table Scan Law
**Definition.** Table scans, range queries and projections run directly across memory-mapped relational row slices via strided record offsets — zero heap allocation during query execution; strided memory-mapped scanning reaches near-memory-bus throughput. **Rule.** 1. **Strided Projections** — query iterators yield pointers to existing record spans. 2. **Zero Row Allocs** — never allocate temporary row wrappers in scan loops.

### B-Tree Page Consistency Law
**Definition.** B-tree splits, leaf balance and index updates run strictly within fixed-size page blocks from dedicated database memory pools — predictable index geometry bounds traversal depth and prevents fragmentation. **Rule.** 1. **Page Bounded** — fixed-size power-of-two blocks. 2. **Atomic Splits** — page balance updates commit transactionally before updating parent pointers.

## 3. Repo-Local Extensions (managed, per the Conflict Triage Law)

### Durable Page Store Boundary Law

`store/store.{h,c}` owns a page-level durable owner over R2's real Rust growable mapping, Page and Wal. One physical file: immutable bootstrap page, caller-sized embedded redo region, data pages — no sidecar or C mapping clone. Full detached sealed page images enter redo; the commit marker syncs before mapped data changes, and data sync precedes success. Reopen admits only after replay and free-chain validation; no checkpoint baseline other than zero. Uncertain IO blocks admission until close/reopen; a failed call may be recovered as committed. Separate primitive: not a durable `Database`/Headless integration, schema/catalog or ordered row index. Release/reuse on logical page ids; no stale row-handle generation promise. Embedded history retained: journal recycling/checkpoint compaction unimplemented; writes refuse/count when its explicit capacity or the data reservation is exhausted; `Store_sync` never truncates or retires this history.

**Bridge** (`Database_saveStore`/`Database_loadStore`): dedicated reflected catalog encoding + raw owned-row spans split into page chains, published through one catalog root-page replacement — not the legacy M2 snapshot parser. Loading requires the same schemas registered first and stages all owned rows before publication. Whole-image persistence, not incremental row addressing or a multi-page transaction. Old catalog/data pages and redo records are retained; capacity refusal before root publication preserves the previously selected image; uncertain root commit requires close/reopen. No checkpointing, schema migration, structured indexes, or thread-safe Store operations.

- `Database_queryEqual` — first structured query: resolve one reflected Field cold, validate its nonzero span against the entity row stride, compare exact bytes across in-memory rows. Cursor owns only its temporary row-pointer list; matching row values remain borrowed, cursor freed before Database. Not numeric comparison, projection, ordering, persistent catalog lookup, or an index; results remain caller-serialized/owner-affine.
- `Database_queryInt32Range` — interpret one exact-width field span as native `int32_t`, rows within inclusive signed bounds; no type coercion — use only where the field's row bytes carry that representation. Reversed bounds and invalid spans reject loudly, Database state unchanged. Uses the declared int32 index when one covers the field, row-list scan otherwise — same matches either way.
- `Database_indexBuild` — in-memory int32 index for one field; admits only a Field carrying `REFLECT_FIELD_INDEXED` whose span is exactly `int32_t` (the schema's flag is the single declaration point). Records: a key-sorted array of borrowed row pointers; rows are append-only, so an insert only marks the records stale and the next query rebuilds them — admission stays allocation-free, free of partial state. While an index exists, the indexed field of a bound row must not be mutated in place; the Database does not detect that.
- `Database_saveStore` persists each declared index as `{int32_t key; uint32_t rowIndex}` records after the entity's row span, inside the same catalog payload and so the same single root-page publication: rows and index cannot tear apart. `Database_loadStore` re-validates every record before publishing — key ordering, `rowIndex` range, each key against the bytes of the row it names — and refuses the whole image when the registered schema no longer declares that exact indexed span. Addresses are never persisted; records re-point at the loaded rows. An entity whose image carries no index keeps its local declaration and rebuilds on the next query. Still a whole-image rewrite, not incremental index pages: no page-level index, no ordered scan of other field types, no multi-field index.
- `Database_fieldSpan` resolves one field NAME to its in-row byte span (`{code, offset, size}`), letting a consumer without a C struct — headless service, tool, schema-driven reader — read a field without walking the reflection tree and folding names itself. Name folded (case irrelevant); null, unknown or ambiguous names reject explicitly, and a declared span that would leave the row is refused rather than handed to a reader. Read-only: allocates nothing, disturbs no index. Named-field resolution, not projection: no field list, no computed values, no row shaping. catalog index after validating the root, page chain, checksum and record spans. The value contains name, row stride and row count, borrowing no Store memory. Discovery only; loading data still requires the caller to register matching reflected schemas.
- **Caller contract:** one serialization/exclusion domain, stable trusted paths and all external alias/truncation exclusion are caller-owned; no OS lock or hostile-writer sandbox inferred. Geometry, owned pointers, page count, free head, sequence and failure state mutate only through construction, admission and durable operations; validated getters, not lifetime-forging setters, expose them — a scoped managed accessor exception under the Conflict Triage Law and Single Class Per File Law (Java Law). A non-reentrant observation callback may observe commit boundaries, but production correctness never depends on it. Initial creation is not crash-atomic and may retain a rejected artifact for recovery.
- **Proof:** `tests/darkbase-engine/store/run.py` (workspace root command) — mirrored Store/Page/Wal owners, fresh native C dependencies, real Rust archive, optimized assertions, C ASan/UBSan. Power loss, Rust sanitizer instrumentation, allocator failure, full OS-error and other-platform/runtime-floor proof remain explicit gaps. The global graph's post-migration ownership-ratchet failure is separate and not silently bypassed.

;;INTENTION("R3 Database Driver: native vex entity store over the shared reflection vocabulary (Struct=entity, Field=entity field with physical layout, Method=function) with reactive store-event programs; zero-alloc table scans; strict page consistency; structured native query, not a SQL engine.")

---

## 4. Readiness Cross-Reference (Living Documentation Law)

- Feature readiness matrix: [darkbase](https://gist.github.com/vex-graph/6943f92acb931b25dad1073c46da6ce7#file-darkbase-md).
- Open blockers and deferred decisions: [ecosystem blockers Gist](https://gist.github.com/vex-graph/e921fa188eebbd0c68c4e59646109887).
