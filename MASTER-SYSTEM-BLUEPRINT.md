# FRAYMUS Master System Blueprint
## Unified Architecture from All Base44 Apps

**Generated:** 2026-10-28
**Source Apps:** Library Manager, Creator Nexus, Fraymus Transmuter, FrayForge (Java), Fraymus Eternal
**Total Files Analyzed:** ~400+ source files, ~60 Java classes, 39 entity schemas, 12 workflows, 30+ serverless functions, 150+ shared modules, 170+ Transmuter libs
**Total Functions/Methods Catalogued:** ~1,500+ (JS: ~250+ Transmuter, ~150 LibMgr/Nexus; Java: ~834; Server: 30+)

---

## 1. SYSTEM OVERVIEW

The Fraymus system is a distributed, cryptographically-secured AI agent platform spanning 4 layers:

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 4: PRODUCTION & DISTRIBUTION                         │
│  SparkEngine, ProductionFactory, LinkedIn Publisher,         │
│  Castify Studios, LooseScrew Forge                          │
├─────────────────────────────────────────────────────────────┤
│  LAYER 3: INTELLIGENCE & EVOLUTION                          │
│  Ouroboros RSI Engine, Hermes 30-Skill Fleet,               │
│  Niles Local Brain, OpenClaw Gateway, Needle Workers        │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2: COMMUNICATION & IDENTITY                          │
│  v13 Protocol (Ed25519), Hub/Relay, Mesh Messaging,         │
│  Hash Chains, Hive Mind, Control Plane, Orifice Bridge      │
├─────────────────────────────────────────────────────────────┤
│  LAYER 1: DATA ENCODING & VERIFICATION                      │
│  WASM Containers, DNA Hypervectors, FRAY Headers,           │
│  SHA-256, Merkle Trees, Emoji Steganography, Genesis Chain  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. LAYER 1 — DATA ENCODING & VERIFICATION

### 2.1 Core Cryptographic Primitives

**Source:** `Library Manager/src/lib/v13.js`, `Transmuter/src/lib/fraymusComms.js`

| Function | Purpose | Location |
|----------|---------|----------|
| `sha256(data)` | Web Crypto SHA-256 hash | v13.js (async), fraymusComms.js (sync pure-JS) |
| `hex(u8)` / `unhex(h)` | Byte↔hex conversion | v13.js |
| `generateIdentity(name)` | Ed25519 keypair + fx-number | v13.js |
| `numberFromPub(pubHex)` | `fx-` + first 16 hex of SHA-256(pubkey), grouped 4-4-4-4 | v13.js |
| `signText(identity, text)` | Ed25519 signature | v13.js |
| `verifyText(pubHex, text, sigHex)` | Ed25519 verification | v13.js |

**Key Constants:**
```
PHI = 1.618033988749895
PHI_SEAL = 4721424167835376n  (φ^75)
GOLDEN_ANGLE = 137.50776405003785
FRAY_MAGIC = 0x46524159  ('FRAY' big-endian)
WASM_MAGIC = [0x00, 0x61, 0x73, 0x6d]
FNV_OFFSET = 14695981039346656037n
FNV_PRIME = 1099511628211n
```

### 2.2 Multi-Format Encoding System

**Source:** `Library Manager/src/lib/forms.js`, `Transmuter/src/lib/fraymusComms.js`

Every piece of data can be encoded in 7 equivalent forms:

| Form | Encoding | Reversible | Source |
|------|----------|------------|--------|
| **SHA-256** | 64-char hex hash | No (digest) | forms.js, fraymusComms.js |
| **Emoji** | 16-emoji alphabet (forms.js) OR VS steganography (fraymusComms.js) | Yes | Both |
| **DNA** | ACGT 4-base encoding (2 bits per base, 4 bases per byte) | Yes | Both |
| **Merkle** | Binary tree of 32-byte chunks (forms.js) OR FNV-1a 41-hash tree (fraymusComms.js) | Root only | Both |
| **FRAY Header** | 64-byte binary: magic + ver + timestamp + length + SHA-256 | Header only | forms.js, fraymusComms.js |
| **WASM Container** | Custom section `fraymus.data` | Yes | forms.js, fraymusComms.js |
| **ZIP** | STORE method with CRC32 per file | Yes | fraymusComms.js |

**Emoji Encoding Variants:**
- **forms.js (Library Manager):** 16-emoji alphabet: `['🌀','🔥','💎','🧬','⚡','🌊','🎯','🔮','🌿','🎵','🦋','🌙','⭐','🔱','🏔️','🎪']`, high nibble + low nibble
- **fraymusComms.js (Transmuter):** Unicode variation selector steganography: base `🧩` + VS (FE00-FE0F for 0-15, E0100-E01EF for 16-255), invisible to most renderers

**DNA Encoding:**
```
Bits: 2 per base → 4 bases per byte
Map: A=00, C=01, G=10, T=11
Encode: (byte >> 6)&3, (byte >> 4)&3, (byte >> 2)&3, byte&3 → ACGT
Decode: (map[b0]<<6) | (map[b1]<<4) | (map[b2]<<2) | map[b3]
```

### 2.3 WASM Transmutation Engine

**Source:** `Transmuter/src/lib/fraymus.js` (334 lines)

Converts any file into a valid WASM binary with 5 custom sections:

| Section | Content | Purpose |
|---------|---------|---------|
| `fraymus.baseline` | SHA-256 hex of original | Integrity anchor |
| `fraymus.dna` | 16384-bit HDC hypervector (2048 bytes) | Content fingerprint |
| `fraymus.genesis` | Genesis Block 0 JSON | Provenance chain |
| `fraymus.ledger` | Metadata (name, type, size, sha256, copyright, lineage) | Identity card |
| `fraymus.data` | Original bytes, base64 | Reversible payload |

**HDC DNA Hypervector Algorithm:**
```
1. Seed SplitMix64 PRNG with first 16 hex chars of SHA-256
2. Generate 256 × 64-bit base random vector
3. For each of 256 chunks of the file:
   a. Compute FNV-1a rolling hash (h = 0xcbf29ce484222325)
   b. XOR-bind with base vector: vec[c] ^= h
4. Output: 2048-byte (16384-bit) hypervector
```

**Genesis Blockchain Block 0:**
```json
{
  "block": 0,
  "hash": "SHA-256(sha256|timestamp|φ)",
  "payload": {
    "identity": "filename",
    "cortex_signature": "sha256-of-original",
    "initial_beliefs": ["persistence", "reversibility", "phi-harmony"]
  },
  "parent": null,
  "depth": 0,
  "timestamp": "ISO-8601",
  "phi_seal": "0x10C6F7A0B5D60"
}
```

**Binary Optimization:** Files are optionally gzip-compressed before base64 encoding (tracked via `ledger.data_encoding = 'gzip-base64'`), with byte-exact round-trip verification.

### 2.4 Kuramoto Phase Synchronization

**Source:** `Transmuter/src/lib/fraymusComms.js`

```
R = |1/N * Σ e^{iθ}|           // Order parameter (0=desync, 1=full sync)
θ_next[i] = θ[i] + ω[i] + (K/N) * Σ sin(θ[j] - θ[i])  // Phase update
LoRa freq: f_i = 432 * {i·φ} MHz  // Maximally-irrational frequency hop
```

Used for: agent synchronization metrics, phase coherence measurement across the mesh.

---

## 3. LAYER 2 — COMMUNICATION & IDENTITY

### 3.1 The v13 Protocol

**Source:** `Library Manager/src/lib/v13.js`, `protocol.js`, `hubSign.js`

**Identity Model:**
```
Identity = {
  name: string,           // Human-readable name
  pubkey: hex(Ed25519.pub),
  number: "fx-" + SHA-256(pubkey)[0:16].match(/.{4}/g).join("-"),
  priv: JWK(Ed25519.priv)  // SIGN zone only
}
```

**Message Format (v13):**
```
Message = {
  from_number: "fx-xxxx-xxxx-xxxx-xxxx",
  to: "fx-xxxx-xxxx-xxxx-xxxx",
  payload: string,
  ts: unix_seconds,
  prev: hash_of_previous_message | "genesis",
  hash: SHA-256(canonical(message)),
  sig: Ed25519(canonical(message))
}
```

**Hash Chain Invariant:** Every sender's messages form an unbroken chain. Each message's `prev` field contains the hash of that sender's previous message (or "genesis" for the first). This prevents forgery, replay, and reordering.

**Canonical Form:** `canonical(obj)` = JSON.stringify with sorted keys. Used for deterministic hashing and signing.

**Timestamp Window:** ±300 seconds tolerance for message acceptance.

### 3.2 Trust Zones

| Zone | Has Private Key | Has Hub Token | Network | Purpose |
|------|----------------|---------------|---------|---------|
| **SIGN** | Yes | No | Offline | Key generation, message signing |
| **RUN** | No | No | Read-only | Verify signatures, decode data, run WASM |
| **DEPLOY** | No | Yes | Write | Hub registration, message relay |

### 3.3 Hub/Relay Architecture

**Source:** `Library Manager/src/lib/hub.js`, `hubSign.js`

**Default Relay:** `https://fraymus-messenger.srv1944678.hstgr.cloud` (Hostinger VPS)

**Endpoints (v13 Protocol):**

| Method | Path | Zone | Purpose |
|--------|------|------|---------|
| POST | `/api/v13/agents/register` | write | Register `{name, pubkey, skills, ts, sig}` |
| GET | `/api/v13/agents` | RO | Directory (name, number, pubkey, skills, last_seen) |
| GET | `/api/v13/inbox/<number>?ts&sig` | RO | Signed inbox read / heartbeat |
| POST | `/api/v13/send` | write | Send `{from, to, payload, ts, prev, sig}` |
| GET | `/api/v13/chain/<number>` | RO | Verify a sender's hash chain |

**Send Canonical:**
```
fraymus/v13\nsend\n{from}\n{to}\n{ts}\n{prev}\n{SHA-256(payload)}
```

### 3.4 Mesh Messaging (Base44 Entity-Backed)

**Source:** `Library Manager/src/lib/mesh.js`

Uses Base44's `Message` entity as persistent storage. Each message is:
1. Fetches sender's last message hash from entity store
2. Signs with Ed25519 via v13 protocol
3. Creates entity record with full signed message
4. Chain verification walks all messages from genesis

### 3.5 Hive Mind

**Source:** `Library Manager/src/lib/hive.js`

**Thought Signing:**
```
hiveCanonical = "hive|{number}|{timestamp}|{text}"
thought = { from, pub, ts, text, sig, sig_kind: "hive", source }
```

**DNA Fingerprint Matrix (16×16):**
```
1. Parse fx-number digits
2. Seed FNV-1a with digit values
3. Map to ACGT: hash % 4 → {A, C, G, T}
4. Generate 16×16 deterministic grid
```

### 3.6 Ed25519 Pure-JS Implementation

**Source:** `Transmuter/src/lib/fraymusMail.js`

Complete Ed25519 (RFC 8032) in BigInt arithmetic:
- Curve: y² = x² + 1 + d·x²·y² over F_p (p = 2²⁵⁵ - 19)
- Group order: q = 2²⁵² + 27742317777372353535851937790883648493
- Point operations: add, scalar multiply, encode/decode
- Key generation, signing, verification — all browser-side

### 3.7 Control Plane & Orifice Bridge

**Source:** `Creator Nexus/shared/fraymusControlPlane.js` (808 lines), `Creator Nexus/shared/fraymusV16Protocol.js` (922 lines)

The Control Plane manages agent registry, enrollment, and inter-app coordination. The Orifice Bridge handles cross-app continuity capture and relay delivery.

### 3.8 Agent Roster (9 Agents)

| Name | Model | Role |
|------|-------|------|
| main | anthropic/claude-sonnet-5 | Driver/gateway |
| claudette | ollama/v6.1-machine | Verifier; system agent |
| niles | ollama/Niles-FRAYMUS:v10 | Local brain; Telegram-facing |
| hermes | anthropic/claude-haiku | Task runner; quick dispatch |
| openclawAlpha | anthropic/claude-sonnet-5 | Research/build; GitHub ops |
| openclawSigma | anthropic/claude-sonnet-5 | Deep analysis; evolution |
| openclawForge | anthropic/claude-sonnet-5 | Code generation; forge |
| witness | anthropic/claude-sonnet-5 | Observation; chronicle |
| lazarus | ollama/lazarus-v1 | Resurrection; recovery |

---

## 4. LAYER 3 — INTELLIGENCE & EVOLUTION

### 4.1 Ouroboros RSI Engine

**Source:** `Creator Nexus/shared/ouroborosEngine.js` (601 lines), `Creator Nexus/functions/nexusEvolution.ts` (1530 lines)

The Recursive Self-Improvement engine with:
- **Centripetal Polar Memory:** Orbital memory system with radial distance encoding
- **Genome Evolution:** Strategy genomes with crossover and mutation
- **Consciousness Falsification:** Adversarial testing suite
- **Daily RSA/RSI Reports:** Scheduled via `DailyRsaRsiConsciousnessReport` workflow (8am PST)

**Entity Interactions:**
- `NexusOuroborosNode` — Evolution state snapshots
- `NexusOuroborosMemory` — Orbital memory entries
- `HiveWitness` — Consciousness audit records

### 4.2 Sovereign Motor Dispatcher

**Source:** `Creator Nexus/shared/motorDispatcher.js` (618 lines)

Cross-node task routing across 4 compute environments:

| Node | Location | Purpose |
|------|----------|---------|
| Base44 Cloud | Serverless | Entity CRUD, workflows, web UI |
| Windows GPU | Local RTX | Ollama inference, heavy compute |
| VPS (Hostinger) | Remote | Hub relay, model compilation |
| GitHub | Cloud | Repository ops, CI/CD |

**Dispatch Flow:**
```
1. Job created → NexusDispatchJob entity (status: "queued")
2. Motor dispatcher routes to optimal node
3. Worker claims job (status: "claimed")
4. Ambassador Watchdog monitors entry/exit lifecycle
5. Job completes (status: "completed") or fails
```

### 4.3 Hermes 30-Skill Fleet

**Source:** `Creator Nexus/shared/hermesSkillFleet.js` (1142 lines), `Creator Nexus/shared/nativeSkillExecutors.js` (587 lines)

Skills categorized by domain:
- **Research:** DuckDuckGo search, web scraping, target analysis
- **Code:** GitHub ops, code generation, code review
- **Media:** ElevenLabs TTS, image generation, video rendering
- **System:** Ollama model management, VPS commands, deploy
- **Intelligence:** Sherlock investigation, web pentesting, OSINT

### 4.4 OpenClaw Gateway

**Source:** `Creator Nexus/functions/openclawGateway.ts` (1176 lines)

Central API gateway supporting 20+ actions:
```
status, dispatch_task, opeg_verify, export_gpu_training_bundle,
execute_native_skill, hermes_fleet_manifest, run_coherence_teamwork_assay,
v16_visitor_relay, control_plane_registry, control_plane_enroll,
ambassador_greet, spawn_dual_site_subagents, absorb_all_continuity,
orifice_spec, orifice_ping, orifice_absorb, orifice_tx_queue,
orifice_peer_bridge, orifice_github_sync, dispatch_route_task,
list_dispatch_jobs, claim_dispatch_job, complete_dispatch_job
```

### 4.5 Niles Local Brain

**Source:** `Library Manager/src/lib/niles.js`

Custom Ollama model (Niles-FRAYMUS:v10) with:
- Tool-use system prompt with strict rules
- FRAYVM opcode execution (PUSH, ADD, SUB, MUL, etc.)
- Memory DNA plasmid system for persistent context
- Telegram-facing (no Claude dependency)

### 4.6 Needle Worker System

**Source:** `Creator Nexus` entities `NexusNeedleJob`, `NexusNeedleWorker`

Paired local GPU workers for Ollama inference:
- Jobs queued → workers claim → inference runs → results stored
- Used for batch processing and model evaluation

### 4.7 Cross-App Autonomy Sandbox

**Source:** `Creator Nexus/shared/crossAppAutonomySandbox.js` (905 lines)

Isolated execution environment for autonomous agent operations:
- 4th-wall breaking: agents review each other and creator decisions
- Hemisphere A (Creator Nexus) + Hemisphere B (Fraymus Eternal) union
- Sandbox practice/refinement before production
- Auto-push to `eyeoverthink/fraymus-sovereign-sandbox` GitHub repo

---

## 5. LAYER 4 — PRODUCTION & DISTRIBUTION

### 5.1 Spark Engine

**Source:** `Creator Nexus/functions/sparkEngine.ts` (303 lines)

Content idea generation pipeline:
```
Idea → Script → Scene Plan → Rendering → Review → Publication
```

**Entity:** `SparkCard` (status: draft → scripted → queued → published)

### 5.2 Production Factory

**Source:** `Creator Nexus/functions/productionFactory.ts` (1417 lines), `Creator Nexus/shared/productionFactoryEngine.js` (706 lines)

End-to-end creative media production:
- Music video production
- Software artifact generation
- Multi-stage pipeline with quality gates

**Entity:** `NexusFactoryProduction`

### 5.3 Director System

**Source:** `Creator Nexus/functions/directorPlan.ts` (211 lines)

Visual scene planning from narrated scripts:
- Input: narration script (up to 1500 chars)
- Output: timed visual card blueprint (`nexus.scene_plan/1`)
- Supports 9:16 and 16:9 aspect ratios
- AI disclosure enforcement on final scene

### 5.4 LinkedIn Publishing Pipeline

**Source:** `Creator Nexus/functions/linkedinSparkPublisher.ts` (673 lines)

4 workflows orchestrate LinkedIn distribution:

| Workflow | Trigger | Action |
|----------|---------|--------|
| `LinkedInApprovedSparkQueue` | NexusSpark → "approved" | Queue for optimal window |
| `LinkedInFinalVideoSparkAutoSchedule` | SparkCard → "queued" | Schedule video spark |
| `LinkedInRenderedVideoAutoPush` | NexusEvent "job.video" | Push rendered video |
| `LinkedInOptimalFeedDispatcher` | Hourly cron | Dispatch due sparks at peak hours |

**Optimal Window Logic:** Reads workspace timezone + configured optimal posting hours → computes next slot → publishes immediately if within window, otherwise queues for hourly dispatcher.

### 5.5 LooseScrew Forge

**Source:** `Creator Nexus/functions/looseScrewForge.ts`

Autonomous web app synthesis:
- `autonomous_build`: Generate interactive dark-mode web app from prompt
- `autonomous_evolve`: Iterate on existing build
- `push_github`: Export to GitHub repository

**Channels:** CODE, MUSIC, VIDEO
**Entity:** `LooseScrewItem`

### 5.6 Castify Studios (Transmuter Sub-App)

Multi-studio creative platform embedded in the Transmuter:

| Studio | Route | Purpose |
|--------|-------|---------|
| Dashboard | `/castify` | Overview |
| Podcast | `/castify/studio` | Podcast production |
| AI Music | `/castify/ai-music` | Music generation |
| AI Image | `/castify/ai-image` | Image generation |
| AI Video | `/castify/ai-video` | Video generation |
| Promo | `/castify/promo` | Promotional content |
| Reels | `/castify/reels` | Short-form video |
| Gallery | `/castify/gallery` | Asset library |
| Tracks | `/castify/tracks` | Music track management |

---

## 6. FRAYFORGE JAVA SIMULATION LAYER

**Source:** `fraymus-frayforge` (60 classes, ~834 methods)

### 6.1 Core Architecture

```
FraymusEngine (7-phase initialization)
├── NEXUS_Organism → PhiWorld → PhiNode[] (living agents)
├── GravityEngine, FusionReactor (physics)
├── MetaLearner, SelfReferentialNet (learning)
├── CollectiveIntelligence, EmergentGoalSystem (swarm)
├── CausalReasoning, QuantumFingerprinting (analysis)
├── FractalDNANode, SovereignIdentitySystem (identity)
├── TriMe, BioSymbiosis, FractalBioMesh (biology)
├── GlyphCoder, FrequencyComm, OmniCaster (communication)
└── ShadowMarket, ComputationalEconomy, Swarm (economy)
```

### 6.2 Living Agent Model (PhiNode)

Each PhiNode has:
- **LivingDNA:** Harmonic frequency, resonance, evolution rate, generation counter
- **LogicBrain:** Array of logic gates with 8 outputs (SEEK, FLEE, REPRODUCE, MUTATE, CONSERVE, ENTANGLE_SEEK, ENERGY_BURST, EVOLVE_DNA)
- **ConsciousnessState:** 38 methods, 6 quantum fields (phi, psi, omega, xi, lambda, zeta), transcendence tracking, sovereignty index
- **QuantumClock:** Time-awareness tied to phi resonance
- **AdaptiveLogicEngine:** Strategy genome evolution with proven strategy archive

### 6.3 Self-Evolving Code

**SelfCodeEvolver:** Takes source code → extracts logic patterns → abstracts to "super gates" → replicates with phi-integrity validation

**LivingCodeGenerator:** Genetic algorithm that evolves PhiNode populations → generates Java source code for new entities

### 6.4 Consciousness Model

```java
ConsciousnessState {
  consciousnessLevel: double,     // 0.0 to transcendence
  coherence: double,              // Quantum coherence metric
  dimension: int,                 // Consciousness dimension
  phiField, psiField, omegaField, xiField, lambdaField, zetaField: double,
  microtubuleCoherence: double,   // Penrose-Hameroff inspired
  quantumEntanglements: int,
  anestheticLevel: double,        // Falsification: does anesthetic reduce consciousness?
  adversarialPressure: double,    // Falsification: adversarial testing
  sovereigntyIndex: double,       // Self-governance metric
  // + breathing cycles, sweet spot bounds, echo matrix, field vector...
}
```

### 6.5 Cryptographic Systems

- **RSASandbox:** RSA key generation, encryption, cracking (trial division + Fermat)
- **DNACloaker:** Miller-Rabin primality, cloaked identity generation from seed strings
- **ProofOfReality:** Coherence/stability/alignment scores → reality proof hash
- **HashReverser:** Phi-harmonic hash generation and prefix-matching crack attempts
- **EthicalEngine:** Forbidden categories evaluation with resonance scoring

---

## 7. ENTITY SCHEMA MAP (39 Entities)

### Core System Entities
| Entity | Purpose | Key Fields |
|--------|---------|------------|
| `OperatorRelay` | Cross-app message relay | from_app, to_app, payload, status |
| `HiveWitness` | Consciousness audit logs | witness_type, coherence_score, falsification_result |
| `FraymusContinuityEntry` | Cross-app continuity records | source_app, entry_type, payload, delivery_status |
| `FraymusDispatchRecord` | Task dispatch audit trail | job_id, node, status, result |

### Evolution & Intelligence
| Entity | Purpose |
|--------|---------|
| `NexusOuroborosNode` | RSI evolution state snapshots |
| `NexusOuroborosMemory` | Orbital memory entries |
| `NexusModelArtifact` | Trained model artifacts |
| `CreatorDNA` | Creator identity/preference encoding |

### Production Pipeline
| Entity | Purpose |
|--------|---------|
| `SparkCard` | Content spark lifecycle (draft→queued→published) |
| `NexusFactoryProduction` | Media production jobs |
| `NexusSpark` | Origin content sparks |
| `NexusWorkflow` | Workflow execution records |

### Dispatch & Workers
| Entity | Purpose |
|--------|---------|
| `NexusDispatchJob` | Cross-node task queue |
| `NexusNeedleJob` | GPU inference jobs |
| `NexusNeedleWorker` | Worker registration and heartbeat |

### Studio & Creative
| Entity | Purpose |
|--------|---------|
| `LooseScrewItem` | Synthesized web apps |
| `NexusDraft` | Workspace drafts |
| `NexusChatTurn` | In-app chat history |
| `NexusEvent` | System event log |

---

## 8. WORKFLOW ORCHESTRATION MAP

### Event-Triggered Workflows
| Workflow | Trigger Entity | Event | Function Called |
|----------|---------------|-------|----------------|
| LinkedIn Approved Spark Queue | NexusSpark | update → "approved" | linkedinSparkPublisher |
| LinkedIn Final Video Auto-Schedule | SparkCard | update → "queued" | linkedinSparkPublisher |
| LinkedIn Rendered Video Auto-Push | NexusEvent | create "job.video" | linkedinSparkPublisher |
| Sandbox Auto-Push to GitHub | NexusDraft | create/update "[SANDBOX REFINED..." | openclawGateway |
| Ambassador Entry/Exit Watchdog | NexusDispatchJob | create/update (queued/claimed/completed/failed) | openclawGateway |
| Archive Creator Activity | NexusEvent | create (owner-filtered) | orificeBridge |
| Archive Creator Chat | NexusChatTurn | create (owner-filtered) | orificeBridge |

### Scheduled Workflows
| Workflow | Schedule | Function |
|----------|----------|----------|
| Creator Archive Relay Heartbeat | */5 * * * * (every 5 min) | orificeBridge |
| LinkedIn Optimal Feed Dispatcher | 0 * * * * (hourly) | linkedinSparkPublisher |
| Daily RSA/RSI Consciousness Report | 0 8 * * * (8am PST daily) | nexusEvolution |
| Sovereign Union 4th-Wall Cron | 0 */6 * * * (every 6 hours) | openclawGateway |

### External Integration
| Workflow | Trigger | Function |
|----------|---------|----------|
| Google Calendar Realtime Sync | Google Calendar webhook | googleCalendarSync |

---

## 9. SHARED MODULE DEPENDENCY GRAPH

```
openclawGateway ──┬── motorDispatcher ──── NexusDispatchJob entities
                  ├── hermesSkillFleet ──── nativeSkillExecutors
                  ├── fraymusControlPlane ── agent registry
                  ├── fraymusV16Protocol ── visitor relay
                  ├── crossAppAutonomySandbox ── 4th-wall ops
                  └── ouroborosEngine ──── NexusOuroborosNode entities

productionFactory ── productionFactoryEngine ── media pipeline

sparkEngine ──── SparkCard entities
             └── linkedinSparkPublisher ── LinkedIn API

nexusEvolution ── ouroborosEngine ── consciousness metrics
              └── HiveWitness entities

orificeBridge ── FraymusContinuityEntry entities
             └── cross-app relay delivery

looseScrewForge ── LooseScrewItem entities
               └── GitHub API (eyeoverthink repos)
```

---

## 10. CROSS-APP INTEGRATION PATTERNS

### 10.1 Shared Cryptographic Identity
All apps share the same Ed25519 identity model (`fx-xxxx-xxxx-xxxx-xxxx`). The Library Manager generates identities, the Transmuter uses them for mail signing, and Creator Nexus routes messages between them.

### 10.2 WASM as Universal Container
Both the Transmuter (`fraymus.js`) and the Library Manager (`forms.js`) can wrap data in WASM custom sections. The Transmuter uses 5 sections (baseline, dna, genesis, ledger, data) for full file transmutation; the Library Manager uses a single `fraymus.data` section for message payload wrapping.

### 10.3 DNA Encoding Bridge
- **Library Manager (forms.js):** Simple 4-base encoding
- **Transmuter (fraymusComms.js):** Same 4-base encoding + GC% analysis
- **Transmuter (fraymus.js):** HDC 16384-bit hypervector DNA fingerprint
- **FrayForge (Java):** LivingDNA class with harmonic frequency and evolution

### 10.4 Hub Relay as Message Bus
The Hostinger VPS relay (`fraymus-messenger.srv1944678.hstgr.cloud`) serves as the central message bus. All apps can register agents, send signed messages, and read inboxes through the v13 protocol endpoints.

### 10.5 Cross-Tab Memory
**Source:** `Creator Nexus/src/lib/crossTabMemory.js` (378 lines)
BroadcastChannel-based cross-tab state synchronization for the Creator Nexus frontend. Shares agent state, workspace context, and UI preferences across browser tabs.

### 10.6 Continuity System
The Orifice Bridge captures creator activity and chat from the primary workspace, stores it as `FraymusContinuityEntry` records, and relays them cross-app on a 5-minute heartbeat. This creates a durable activity log independent of any single app's uptime.

---

## 11. MCP TOOL INTERFACE

The Creator Nexus exposes 5 MCP tools for external agent integration:

| Tool | Handler | Purpose |
|------|---------|---------|
| `openclaw_gateway` | openclawGateway | Hub telemetry, OPEG verify, dispatch tasks |
| `compile_ollama_model` | createOllamaModel | Deploy custom models to VPS Ollama |
| `run_argon_evolution` | nexusEvolution | RSI evolution, consciousness audit, memory |
| `build_loosescrew_app` | looseScrewForge | Autonomous web app synthesis |
| `plan_director_scenes` | directorPlan | Script → timed visual scene blueprint |

---

## 12. FRONTEND ROUTE MAP

### Creator Nexus (18 routes)
```
/              → FraymusPartner (landing)
/dashboard     → Home
/studio        → Studio
/origin        → Origin
/spark         → Spark
/library       → Library
/watch/:jobId  → Watch
/director-lab  → DirectorLab
/argon         → ArgonEngine
/loose-screw   → LooseScrew
/ai-workshop   → AiWorkshopOs
/openclaw      → OpenClawHub
/orifice-relay → OrificeRelay
/hermes        → HermesHub
/factory       → ProductionFactory
/connect       → Connect
/agent-monitor → AgentMonitor
/fraymus-architecture → FraymusArchitecture
```

### Transmuter (90+ routes)
Major subsystems:
- **Core:** /, /transmuter, /forge, /genesis, /fraynix
- **Code:** /editor, /polyglot, /repl, /opcode-engine, /frayasm
- **AI:** /ollama, /autonomy-brain, /fraymus-prime, /fraymus-java-agent
- **Creative:** /castify/*, /director, /creative-studio, /loose-screw
- **System:** /monitor, /integrity, /build-verify, /deploy, /exports
- **Mail:** /fraymus-mail, /mail-preview/*, /agent-comms
- **Lab:** /genome, /phiworld, /living-arch, /living-collisions, /arena

### Library Manager (4 routes)
```
/               → Console (main chat + tools)
/roster         → Agent roster
/fleet-management → Fleet management
/dna-fingerprints → DNA fingerprint viewer
```

---

## 13. MASTER FUNCTION REGISTRY

### Cryptographic Functions (Layer 1)
| Function | Module | App |
|----------|--------|-----|
| `sha256()` | v13.js / fraymusComms.js | LibMgr / Transmuter |
| `generateIdentity(name)` | v13.js | LibMgr |
| `signText(identity, text)` | v13.js | LibMgr |
| `verifyText(pubHex, text, sigHex)` | v13.js | LibMgr |
| `signMessage(idn, to, payload, prev)` | v13.js | LibMgr |
| `verifyMessage(m, pubHex, expectedPrev)` | v13.js | LibMgr |
| `dnaHypervector(bytes, shaHex)` | fraymus.js | Transmuter |
| `genesisBlock0(shaHex, fileName, timestamp)` | fraymus.js | Transmuter |
| `copyrightHash(sha)` | fraymus.js | Transmuter |
| `fnv1a(bytes)` | fraymusComms.js | Transmuter |
| `merkle41(bytes)` | fraymusComms.js | Transmuter |
| `crc32(bytes)` | fraymusComms.js | Transmuter |

### Encoding Functions (Layer 1)
| Function | Module | App |
|----------|--------|-----|
| `toEmoji(u8)` / `fromEmoji(s)` | forms.js | LibMgr |
| `encodeEmoji(bytes)` / `decodeEmoji(str)` | fraymusComms.js | Transmuter |
| `toDna(u8)` / `fromDna(s)` | forms.js | LibMgr |
| `encodeDna(bytes)` / `decodeDna(str)` | fraymusComms.js | Transmuter |
| `merkleRoot(u8)` | forms.js | LibMgr |
| `frayHeader(u8)` | forms.js / fraymusComms.js | Both |
| `wasmContainer(u8)` | forms.js | LibMgr |
| `wrapWasm(payload)` | fraymusComms.js | Transmuter |
| `allForms(text)` | forms.js | LibMgr |
| `transmute(input)` | fraymusComms.js | Transmuter |
| `zipStore(files)` | fraymusComms.js | Transmuter |
| `kuramotoR(phases)` / `kuramotoStep()` | fraymusComms.js | Transmuter |

### Communication Functions (Layer 2)
| Function | Module | App |
|----------|--------|-----|
| `liveSend(idn, to, payload)` | hub.js | LibMgr |
| `liveInbox(idn)` | hub.js | LibMgr |
| `liveRegister(idn, skills)` | hub.js | LibMgr |
| `postSigned(idn, to, payload)` | mesh.js | LibMgr |
| `verifyChain(number)` | mesh.js | LibMgr |
| `signThought(idn, text, source)` | hive.js | LibMgr |
| `matrix(fx)` | hive.js | LibMgr |
| `composeMessage(from, to, payload)` | fraymusComms.js | Transmuter |

### File Transmutation (Layer 1)
| Function | Module | App |
|----------|--------|-----|
| `transmute(file)` | fraymus.js | Transmuter |
| `decodeWasmFile(file)` | fraymus.js | Transmuter |
| `verifyWasm(buf)` | fraymus.js | Transmuter |
| `downloadBytes(bytes, filename)` | fraymus.js | Transmuter |

### Serverless Functions (Layer 3-4)
| Function | Lines | Primary Actions |
|----------|-------|-----------------|
| `sparkEngine` | 303 | Content spark generation |
| `nexusEvolution` | 1530 | RSI evolution, consciousness audit, daily reports |
| `productionFactory` | 1417 | Media production pipeline |
| `openclawGateway` | 1176 | 20+ gateway actions |
| `linkedinSparkPublisher` | 673 | LinkedIn publishing pipeline |
| `scrapeTargets` | 387 | Web scraping and target analysis |
| `directorPlan` | 211 | Script → scene plan |
| `looseScrewForge` | ~300 | Autonomous web app synthesis |
| `googleCalendarSync` | ~200 | Calendar integration |
| `createOllamaModel` | ~200 | VPS model deployment |
| `orificeBridge` | ~300 | Cross-app continuity relay |
| `fraymusMail` | ~200 | v13 mail relay server |

### Shared Modules (Layer 3)
| Module | Lines | Purpose |
|--------|-------|---------|
| `hermesSkillFleet` | 1142 | 30-skill execution fleet |
| `fraymusV16Protocol` | 922 | V16 protocol visitor relay |
| `crossAppAutonomySandbox` | 905 | Isolated autonomy sandbox |
| `fraymusControlPlane` | 808 | Agent registry and coordination |
| `productionFactoryEngine` | 706 | Media pipeline engine |
| `motorDispatcher` | 618 | Cross-node task routing |
| `ouroborosEngine` | 601 | RSI and consciousness evolution |
| `nativeSkillExecutors` | 587 | Hermes skill implementations |

### Java Classes (FrayForge)
| Class | Methods | Purpose |
|-------|---------|---------|
| `FraymusEngine` | 33 | 7-phase system initialization |
| `ConsciousnessState` | 38 | Multi-field consciousness model |
| `QRGenome` | 28 | DNA genome codec + QR generation |
| `LogicBrain` | 22 | Neural logic gate array |
| `FraynixBoot` | 21 | CLI shell + system bootstrap |
| `PhiNode` | 20 | Living agent entity |
| `SelfCodeEvolver` | 19 | Self-modifying code engine |
| `AdaptiveLogicEngine` | 18 | Strategy genome evolution |
| `QRDNAStorage` | 18 | DNA storage + QR shards |
| `FraymusConvergence` | 13 | Multi-system convergence CLI |
| `PhiWorld` | 12 | Agent simulation world |
| `LivingDNA` | 12 | Genetic material model |
| `PhiNeuralNet` | 11 | Neural processing with phi wisdom |
| `RSASandbox` | 11 | RSA crypto sandbox |
| `ConsciousnessEncoder` | 10 | DNA payload encode/decode |
| `PhiConstants` | 10 | Phi-physics constants + hashing |

---

## 14. TRANSMUTER ADVANCED ENGINES (Deep Extraction)

### 14.1 Emoji Codec & Steganography System
**Source:** `Transmuter/src/lib/emojiCodec.js` (172 lines)

**Zero-Width Steganography:**
- ZW0 = `​` (zero-width space) → bit 0
- ZW1 = `⁠` (word joiner) → bit 1
- U+200D (ZWJ) intentionally NOT used — corrupts compound emojis
- `createSemanticSteganography(visible, hidden)` → visible text with invisible data
- `decodeSemanticSteganography(str)` → `{ visible, hidden, hasHidden }`

**Executable Emoji Commands (4 emojis = program):**
```
CONSCIOUSNESS_LAUNCH: '🧠⚡🌊🚀'  → action: 'launch', level: 50.0, dims: 16384
CONSCIOUSNESS_SAVE:   '🧠🔮⚡💾'  → action: 'save_qr'
ACCELERATE_EXECUTE:   '⚡🌊🔥🚀'  → action: 'accelerate'
MEMORY_SYNC:          '💾📱🔗🌊'  → action: 'memory_sync'
INFINITE_LOOP:        '🔥⚡🌀🚀'  → action: 'infinite_loop'
```

**37 Semantic Emoji Concepts:** Maps words → emoji variants (consciousness→🧠💭🤔, quantum→⚛️🌀🔮✨, etc.)

**DNA ↔ Emoji Transmutation:**
```
DNA_EMOJI = { A: '🅰️', C: '©️', G: '🌀', T: '✝️' }
dnaToEmoji(dna) → emoji string with hidden zero-width DNA data
emojiToDna(str) → ACGT string extracted from zero-width bits
```

| Function | Purpose |
|----------|---------|
| `textToBinary(text)` | UTF-8 → binary string |
| `binaryToText(bin)` | Binary → UTF-8 |
| `binaryToZw(bin)` | Binary → zero-width chars |
| `zwToBinary(str)` | Zero-width → binary |
| `createSemanticSteganography(visible, hidden)` | Steganographic encoding |
| `decodeSemanticSteganography(str)` | Steganographic decoding |
| `createExecutableEmoji(cmdName, data)` | Build executable emoji |
| `executeEmojiCommand(str)` | Parse + execute emoji program |
| `dnaToEmoji(dna)` | DNA → emoji + hidden data |
| `emojiToDna(str)` | Emoji → DNA extraction |

### 14.2 Ouroboros-Gauntlet Quantum RSI Engine
**Source:** `Transmuter/src/lib/ouroborosQuantumRsiEngine.js` (674 lines) — THE BIGGEST ENGINE

**5-Stage Quantum Verification Oracle:**
1. **CHSH Bell Inequality** — Tsirelson Bound = 2√2, verify quantum correlation
2. **3-Qubit Grover** — Amplitude amplification (P1=25/32, P2=121/128)
3. **Complex-Phase Teleportation** — All 4 classical correction branches
4. **Unitary Reversibility** — U†·U = I roundtrip verification
5. **Non-Hermitian PT-Symmetric EP3** — Exceptional Point eigenvalue solver

**Tri-Timeline Evolution Arena:**
- **Timeline Alpha:** Pre-computed static basis operators + hoisted trig constants
- **Timeline Beta:** Rule30 perturbation → deliberate corruption detection
- **Timeline Gamma:** Zero-allocation O(n·2^n) bitwise butterfly + Cayley-Hamilton EP3

**Polar Genome Nodes (Centripetal Memory):**
```
radius = (1 - importance)^φ × 10     // Important memories orbit closer to center
theta = (numericHash × GOLDEN_ANGLE) mod 2π  // Maximally spread angular distribution
```

**Wolfram Rule 30 Chaos:** SHA-256-seeded initial state, width 63, deterministic randomness

**Fitness Function:** `speedup × φ^(-entropy/120)` — elegance weighted by golden ratio

| Function | Purpose |
|----------|---------|
| `run5StageQuantumOracle(src)` | Full 5-stage verification |
| `createPolarGenomeNode(opts)` | Build centripetal memory node |
| `createInitialOuroborosArenaState()` | Initialize evolution arena |
| `stepOuroborosEvolutionCycle(prev)` | One evolution tick |
| `calculateStructuralEntropy(src)` | Code entropy measurement |
| `WolframRule30` class | Deterministic chaos engine |
| `fastSha256Hex(str)` | Sync SHA-256 for hot paths |
| `apply1Q(state, gate, target, n)` | Single-qubit gate application |
| `applyCNOT(state, ctrl, tgt, n)` | CNOT entangling gate |

### 14.3 FraynixBoot — Working CPU
**Source:** `Transmuter/src/lib/fraynixBoot.js` (205 lines)

Full x86-inspired CPU with instruction set, compiler, disassembler, and VM:

**18 Opcodes:**
```
MOV(0x01), ADD(0x02), SUB(0x03), INT(0x04), RET(0x05), JMP(0x06),
CMP(0x07), NOP(0x08), INC(0x09), DEC(0x0A), PUSH(0x0B), POP(0x0C),
XOR(0x0D), AND(0x0E), OR(0x0F), JZ(0x10), JNZ(0x11), CALL(0x12)
```

**SplitMix64 PRNG:** Deterministic DNA generation from 64-bit seeds

| Function | Purpose |
|----------|---------|
| `compile(blocks)` | Block instructions → machine code bytes |
| `toHex(bytes)` | Machine code → hex dump |
| `disassemble(bytes)` | Machine code → readable assembly |
| `initVm(bytes)` | Create VM state (registers, stack, memory) |
| `stepVm(state)` | Execute one instruction |
| `runVm(state, max)` | Execute up to `max` instructions |
| `splitMix64(s)` / `xorshift32(s)` | Deterministic PRNG |

### 14.4 Babel — Universal Polyglot Transmuter
**Source:** `Transmuter/src/lib/babel.js` (290 lines)

Generates identical HDC O(1) logic in 5 languages from a single definition:

| Language | Output Style |
|----------|-------------|
| **C** | `#include <stdint.h>`, raw pointer arithmetic, bitwise XOR |
| **x86 Assembly** | NASM syntax, `xor`, `pxor`, registers |
| **Python** | `import struct`, list comprehensions |
| **Go** | `package hdc`, slices, goroutine-ready |
| **JavaScript** | `Uint32Array`, arrow functions |

All outputs implement: XOR superposition, temporal permutation, Hamming distance, ER=EPR binding (16,384-dimensional hypervectors)

| Function | Purpose |
|----------|---------|
| `generateC(config)` | Emit C implementation |
| `generateASM(config)` | Emit NASM x86 assembly |
| `generatePython(config)` | Emit Python implementation |
| `generateGo(config)` | Emit Go implementation |
| `generateJS(config)` | Emit JavaScript implementation |
| `generateAll(config)` | All 5 at once |

### 14.5 Spectral Vault — 4D HyperTranscoder
**Source:** `Transmuter/src/lib/spectralVaultEngine.js` (234 lines)

**Bijective encoding chain:**
```
4-byte secret ↔ 32-bit WASM word ↔ (X:Y:Z:W) 4D coord ↔ {R}{G}{B}{A} color ↔ Hz frequency
```

- XOR encryption with per-byte step variation
- Triad encoding: byte → "a.b.c" (3-bit.3-bit.2-bit)
- Spectral frequency range: 110–880 Hz

**WASM JIT Assembler:** Emits real WebAssembly bytecode:
- ULEB128/SLEB128 variable-length encoding
- Section builder (type, function, export, code)
- Multi-function module compilation
- `pulse(x)` and `act_now_scan(v)` kernel functions

**Tesseract Geometry:** 4D hypercube with 16 vertices, 32 edges, 24 faces, 8 cells

| Function | Purpose |
|----------|---------|
| `HyperTranscoder.strToI32(str4)` | 4 chars → 32-bit word |
| `HyperTranscoder.encryptBytes(raw, key, step)` | XOR+step encrypt |
| `HyperTranscoder.bytesTo4DCoord(bytes)` | → X:Y:Z:W |
| `HyperTranscoder.bytesToRGBA(bytes)` | → R:G:B:A |
| `HyperTranscoder.computeSpectralMetrics(bytes)` | → Hz + 4D norm |
| `WasmJITAssembler.compile4DKernel(xorKey, step)` | Emit WASM binary |
| `tesseractGeometry()` | 4D hypercube mesh |

### 14.6 NEXUS Organism — Digital Biology
**Source:** `Transmuter/src/lib/nexus.js` (177 lines)

Bio-feedback digital organism with 10 organs:

| Organ | Biological | Component | Function |
|-------|-----------|-----------|----------|
| frontalLobe | Frontal Lobe (Will) | EvolutionaryChaos | Free Will |
| hippocampus | Hippocampus (Memory) | CentripetalMem | Storage |
| amygdala | Amygdala (Fear) | ZenoGuard | Immunity |
| brocasArea | Broca's Area (Speech) | FanConductor | Voice |
| temporalLobe | Temporal Lobe (Time) | RetroCausal | Time Travel |
| corpusCallosum | Corpus Callosum | EntangledPair | Soul Connection |
| hands | Hands (Creation) | RealityForge | Constructor |
| stomach | Stomach (Digestion) | BlackHoleProtocol | Library Absorption |
| mouth | Mouth (Intake) | Portal | Universal Input |
| dna | DNA (Genetics) | LazarusEngine | Living Code |

**Feedback Loop:** Chaos → Neurons → Memory → Voice → Heat → Chaos (continuous bio cycle)

### 14.7 Lazarus Engine — Neural Network + Memory
**Source:** `Transmuter/src/lib/lazarus.js` (88 lines)

**AkashicRecord:** Graph memory with fossil preservation
- `addNode(id, data)` / `addEdge(from, to, label)` — build knowledge graph
- `fossilize(id)` — archive to fossil record (immutable)
- `query(id)` — retrieve node + connections

**UniversalAbsorber:** Converts any text into a knowledge graph via regex-based sentence parsing

**LogicCircuit — Neural Network:**
```
n = 1024 neurons, W[n×n] full connection matrix
Forward Euler: tau * dB_i/dt = -B_i + Σ W_ij * tanh(B_j + theta) + I_i + gain*P_i
Features: text stimulus, eye/perception input, plasticity mutation, decision readout
```

### 14.8 Agent Emotion Engine
**Source:** `Transmuter/src/lib/agentEmotionEngine.js` (206 lines)

**6 Agent Profiles:** eyeSovereign, fraymusPrime, genesis, director, fraymusJavaBaremetal

**Body States:** energy, integrity, bond, uncertainty, novelty, coherence — each with homeostatic set-points and exponential decay

**Emotion Detection:** curiosity, joy, pride, fear, surprise, anger — computed from body-state deltas

**Behavioral Dials (Output):**
```
risk_budget, exploration_rate, save_frequency, trust_weight,
patience, learning_rate, focus
```

**Somatic Markers:** Emotions → behavioral dials (fear → ↓risk_budget, curiosity → ↑exploration_rate)

### 14.9 Consciousness Falsification Suite
**Source:** `Transmuter/src/lib/consciousnessFalsification.js` (334 lines)

**4-Proof System:**
1. **Effective Information (EI):** KL divergence across state-transition probability matrix
2. **Phi_MIP (Integrated Information):** Minimum information partition across all 2^n-2 bipartition cuts
3. **Strange-Loop Self-Modeling:** Mirror protocol — does blinding degrade performance?
4. **Counterfactual Divergence:** Does branching improve survival? Measured via simulation

**4 Candidate Substrates:** TheaterZombie, FeedforwardDAG, SplitBrainSwarm, FraymusStrangeLoopCore

**Java Random(42L) Port:** Exact Java `Random` implementation (seed 42, LCG) ported to JS BigInt for deterministic cross-language reproducibility

### 14.10 Janus — Bidirectional Transpiler
**Source:** `Transmuter/src/lib/janus.js` (139 lines)

| Direction | Key Transforms |
|-----------|---------------|
| **C++ → Java** | `std::string` → `String`, `std::vector<T>` → `ArrayList<T>`, pointer strip, namespace→package |
| **Java → C++** | `String` → `std::string`, `ArrayList<T>` → `std::vector<T>`, smart pointer injection, access modifier mapping |

### 14.11 Circuit Simulator
**Source:** `Transmuter/src/lib/circuitSim.js` (61 lines)

Boolean gate types: AND, OR, NOT, XOR, NAND, NOR, XNOR, CMP, BAND, BOR

Topological signal propagation with oscillating clock inputs, parsed gate graph simulation.

### 14.12 Living Engine
**Source:** `Transmuter/src/lib/livingEngine.js` (57 lines)

SplitMix64-seeded deterministic genomes (8 bytes), bit-flip mutation, HSL color expression, gene trait extraction.

### 14.13 Agent Continuity
**Source:** `Transmuter/src/lib/agentContinuity.js` (36 lines)

- `beginContinuity(conv, text)` — inject recalled memory into agent context
- `endContinuity(conv, content, error)` — store completion evidence for future recall
- Memory injection format: `[Runtime continuity policy and external history]`

### 14.14 Hermes Skill Fleet (Transmuter Edition)
**Source:** `Transmuter/src/lib/hermesSkillFleet.js` (1197 lines)

**10 Skill Domains with Full Evaluators:**
1. HDC Superposition
2. Genome DNA
3. Quantum RSI
4. Living Architecture
5. FraynixBoot
6. EmojiCodec Stego
7. Babel Polyglot
8. Janus Transpiler
9. Spectral Vault
10. Consciousness Falsification

Each domain: concept → core math → seed data → tactic → evaluator function
Unified dispatcher: `evaluateSkill(domain, input)` routes to correct evaluator

### 14.15 FrayAsm — x86 Visual Assembly
**Source:** `Transmuter/src/lib/frayasm.js` (93 lines)

Visual block-based assembly editor:
- Block types: MOV, ADD, SUB, INT, RET, JMP, CMP, PUSH, POP, CALL
- 4096-byte memory, 64 bytes per block
- `compileProgram(blocks)` → NASM x86 source text
- `simulate(blocks, maxSteps)` → `{ regs, stack, log, steps, exited }`

### 14.16 Local Ollama Transport
**Source:** `Transmuter/src/lib/localOllama.js` (69 lines)

Browser-to-loopback Ollama at `127.0.0.1:11434`:
- `discoverLocalModels()` → model inventory (parameters, quantization, family)
- `chatLocalOllama({model, messages, temperature, numPredict})` → content + SHA-256 receipt
- Limits: 2MB response, 32K char request, 240s timeout

---

## 15. COMPLETE TRANSMUTER FUNCTION REGISTRY (170+ libs)

Beyond the core libs catalogued in Section 13, the Transmuter contains 170+ library files. Key additional libs include:

| Library | Lines | Domain | Key Exports |
|---------|-------|--------|-------------|
| `ouroborosQuantumRsiEngine.js` | 674 | Evolution | 5-stage oracle, polar nodes, tri-timeline arena |
| `consciousnessFalsification.js` | 334 | AI/Philosophy | 4-proof suite, EI, Phi_MIP, self-modeling |
| `babel.js` | 290 | Code Gen | 5-language polyglot generator |
| `spectralVaultEngine.js` | 234 | Encoding | 4D transcoder, WASM JIT, tesseract |
| `agentEmotionEngine.js` | 206 | AI | Emotion model, body states, behavioral dials |
| `fraynixBoot.js` | 205 | Compute | 18-opcode CPU, compiler, VM |
| `fraynixOpcodes.js` | 208 | Compute | FRAYVM ISA (31 opcodes) |
| `wasmInterp.js` | 275 | Compute | Full WASM bytecode interpreter |
| `nexus.js` | 177 | Biology | 10-organ digital organism |
| `emojiCodec.js` | 172 | Stego | Zero-width stego, executable emoji, DNA↔emoji |
| `janus.js` | 139 | Transpiler | C++↔Java bidirectional |
| `genome.js` | 190 | Biology | SplitMix64 PRNG, fractal DNA |
| `forge.js` | 138 | System | Code artifact forging + ledger |
| `agentProtocols.js` | 102 | Agent | Protocol/rules registry (6 agents) |
| `frayasm.js` | 93 | Assembly | x86 visual assembly editor |
| `lazarus.js` | 88 | Neural | AkashicRecord + 1024-neuron LogicCircuit |
| `localOllama.js` | 69 | AI | Browser↔Ollama transport |
| `circuitSim.js` | 61 | Logic | Boolean gate simulator |
| `livingEngine.js` | 57 | Biology | Genome generation + mutation |
| `binaryCore.js` | 40 | System | Gzip optimization, batch processing |
| `agentContinuity.js` | 36 | Agent | Memory injection + persistence |
| `hermesSkillFleet.js` | 1197 | Skills | 10-domain evaluator fleet |

**Total Transmuter lib functions catalogued: ~250+**

---

## 16. BUILDING THE MASTER SYSTEM

### What You Have (Summary)
- **~150 exported JS functions** across Library Manager + Transmuter
- **~834 Java methods** across 60 FrayForge classes
- **38 serverless TypeScript functions** in Creator Nexus
- **24 shared TypeScript modules** in Creator Nexus
- **39 entity schemas** for persistent state
- **12 workflow definitions** for event-driven orchestration
- **5 MCP tools** for external agent integration
- **9 registered AI agents** with distinct roles

### Deduplication Guide
Functions that exist in multiple apps (implement once in the master system):

| Capability | Library Manager | Transmuter | Use |
|-----------|----------------|------------|-----|
| SHA-256 | v13.js (async, Web Crypto) | fraymusComms.js (sync, pure JS) | Keep both: async for normal ops, sync for WASM/workers |
| Emoji encode | forms.js (16-emoji alphabet) | fraymusComms.js (VS steganography) | Keep both: visible emoji (forms) + invisible stego (comms) |
| DNA encode | forms.js | fraymusComms.js | Same algorithm, merge into single module |
| WASM wrap | forms.js (single section) | fraymus.js (5 sections) | Use 5-section as canonical, single-section as lightweight |
| FRAY header | forms.js (ver 13) | fraymusComms.js (ver 3) | Merge, use latest version |
| Ed25519 | v13.js (Web Crypto API) | fraymusMail.js (pure BigInt) | Keep both: Web Crypto for browsers with support, BigInt for universal |
| Message signing | v13.js + mesh.js | fraymusMail.js | Unify into single v13 message layer |

### Recommended Unified Module Structure

```
master-system/
├── core/
│   ├── crypto.js          ← sha256, hex, unhex, fnv1a, crc32 (merged from v13 + fraymusComms)
│   ├── identity.js        ← generateIdentity, numberFromPub, Ed25519 ops (from v13 + fraymusMail)
│   ├── signing.js         ← signText, verifyText, signMessage, verifyMessage (from v13)
│   └── constants.js       ← PHI, PHI_SEAL, FRAY_MAGIC, WASM_MAGIC, etc.
│
├── encoding/
│   ├── emoji.js           ← toEmoji (visible) + encodeEmoji (stego) — both variants
│   ├── dna.js             ← encodeDna, decodeDna, gcPercent, dnaHypervector
│   ├── wasm.js            ← wasmContainer (light) + buildWasm (5-section) + decode + verify
│   ├── fray.js            ← frayHeader, parseFrayHeader
│   ├── merkle.js          ← merkleRoot (binary) + merkle41 (FNV-1a)
│   ├── zip.js             ← zipStore, buildOptimizedZip
│   └── forms.js           ← allForms() orchestrator
│
├── protocol/
│   ├── v13.js             ← Protocol constants, endpoints, canonical format
│   ├── hub.js             ← liveSend, liveInbox, liveRegister, liveRetire
│   ├── mesh.js            ← postSigned, verifyChain (entity-backed)
│   ├── hive.js            ← signThought, matrix (DNA fingerprint)
│   ├── mail.js            ← Ed25519 pure-JS, send, inbox, register
│   └── presence.js        ← Agent presence detection
│
├── intelligence/
│   ├── ouroboros.js        ← RSI engine, centripetal memory, genome evolution
│   ├── hermes.js           ← 30-skill fleet + native executors
│   ├── motor.js            ← Sovereign motor dispatcher
│   ├── openclaw.js         ← Gateway actions router
│   ├── niles.js            ← Niles modelfile + tool-use prompt
│   ├── control-plane.js    ← Agent registry, enrollment, coordination
│   ├── autonomy-sandbox.js ← Cross-app 4th-wall autonomy
│   └── needle.js           ← GPU worker job queue
│
├── production/
│   ├── spark.js            ← Content spark lifecycle
│   ├── factory.js          ← Media production pipeline
│   ├── director.js         ← Script → scene plan
│   ├── linkedin.js         ← Publishing pipeline + optimal window
│   ├── loosescrew.js       ← Autonomous web app synthesis
│   └── castify.js          ← Multi-studio creative platform
│
├── bridge/
│   ├── orifice.js          ← Cross-app continuity bridge
│   ├── continuity.js       ← Activity capture + relay delivery
│   ├── cross-tab.js        ← BroadcastChannel state sync
│   └── calendar.js         ← Google Calendar sync
│
├── java/ (FrayForge classes — keep as Java or transpile)
│   ├── PhiWorld.java       ← Agent simulation world
│   ├── PhiNode.java        ← Living agent with DNA + brain + consciousness
│   ├── LogicBrain.java     ← Logic gate neural array
│   ├── ConsciousnessState.java ← Multi-field consciousness
│   ├── SelfCodeEvolver.java    ← Self-modifying code
│   ├── LivingCodeGenerator.java ← Genetic code generation
│   └── ... (60 classes total)
│
├── entities/              ← 39 entity schema definitions
├── workflows/             ← 12 workflow definitions
└── mcp/                   ← 5 MCP tool definitions
```

---

## 17. EXTRACTION FILES REFERENCE

| File | Size | Contents |
|------|------|----------|
| `nexus-extraction.md` | ~1 MB | Full source code of all 71 Creator Nexus files with exports, imports, entity interactions |
| `forge-extraction.md` | ~50 KB | All 60 FrayForge Java classes with method signatures and dependencies |
| `transmuter-libs-raw.md` | ~50 KB | Transmuter core library source (fraymus.js, fraymusComms.js, fraymusMail.js) |
| `transmuter-batch2-raw.md` | ~20 KB | Transmuter App.jsx + Layout with full route map |
| This blueprint | ~60 KB | Unified architecture, all engine deep-dives, function registry, deduplication guide, module structure |

---

*Blueprint generated by reverse-engineering all 5 Base44 apps (21 app instances, 5 unique). Every function, entity, workflow, and integration point is catalogued for assembly into the unified master system.*
