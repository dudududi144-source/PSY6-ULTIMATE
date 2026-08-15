# PSY6 ULTIMATE — Full Architecture Specification (v6.4)

> One file. Zero server. Infinite groove.
> 136KB of pure Web Audio API. No dependencies. No build step.

## 1. Design Philosophy

PSY6 ULTIMATE follows the non-negotiable rules of the PSY family:

1. **One source of truth** per piece of musical state
2. **Transport is not renderer**, renderer is not UI
3. **No device policy** — this IS the device, built from foundation primitives
4. **Every claim has evidence** — benchmarks included
5. **Zero GC dropouts** — all voices pre-allocated
6. **Learn, then generate** — grammars built from observation
7. **Local, deterministic, independent, musical**

## 2. Layer Architecture

The instrument is organized in 8 layers, top to bottom:

| Layer | Components |
|-------|-----------|
| **UI Layer** | Knobs, XY Pad, Seq (M/S/Vol/FX/Pan), Arranger, Pads, Visualizers, Help, **Track Colors**, **Probability Viz** |
| **Brain Layer** | CandidateGen, Grammars, ADAPTIVE |
| **Composition** | Chord Progressions, Arpeggiator, Bass Modes, Pattern Operations |
| **Scheduler** | Swing, Humanize, Step Repeat/Prob, Track Delay Offset, Section Arranger |
| **Engine Layer** | PooledEngine (20 synth + 24 drum), AudioBus, Compressors, Panners |
| **FX Layer** | Drive, Filter, Delay, Reverb, Per-track sends/modes |
| **Persistence** | localStorage (projects, patterns, grammars, snapshots, favorites, **pattern banks A-D**) |
| **I/O Layer** | MIDI In/Out, Clock, WAV, Live Rec |

## 3. PooledEngine — Zero GC Architecture

### Problem
Creating Web Audio nodes per-note causes GC pauses leading to audio dropouts.

### Solution
Pre-allocate all voices at init. Reuse via round-robin.

- SYNTH_VOICES = 20 (melodic: bass, lead, arp, pad)
- DRUM_VOICES = 24 (percussive: kick, snare, hat, perc, fx)

Each voice:
- Created ONCE at AudioContext init
- All nodes pre-connected (osc -> filter -> vca -> bus)
- noteOn() only updates parameters (freq, gain, envelope)
- panic() cancels scheduled values and zeroes gain

### Voice Lifecycle
- INIT: create voice -> connect to bus -> idle
- NOTE: nextVoice() -> set params -> schedule envelope -> active
- END: envelope completes -> voice returns to pool -> idle
- PANIC: cancelScheduledValues -> gain=0 -> idle

### Per-Track Control
Each of the 6 tracks (KICK, BASS, PERC, LEAD, ARP, PAD) has:
- **Mute/Solo** (M/S buttons)
- **Volume** (click to cycle 0.3/0.5/0.8/1.0)
- **FX Mode** (Normal/Dry/Wet/Pump)
- **Pan** (StereoPannerNode: C/L50/R50/L100/R100)
- **Delay/Reverb sends** (per-track gain nodes)
- **Delay offset** (+/-30ms micro-timing)
- **Compressor** (DynamicsCompressorNode, togglable)

## 4. Brain Layer — Generative Composition

### Grammar System (3 statistical models)
- **BassGrammar**: 12x12 interval transition matrix, learned from observed bass lines, used by ADAPTIVE mode
- **MelodicGrammar**: Interval histogram (-12 to +12), contour direction tracking, learned from user performance (pads + MIDI)
- **RhythmGrammar**: 16-step kick onset probability, learned from observed kick patterns

### Grammar Decay
- Every 50 entries: multiply all weights by 0.95
- Allows adaptation to new styles without catastrophic forgetting
- Visualized in real-time (grammar canvas)

### CandidateGenerator (GENERATIVE mode)
- Per-bar: generates 5 candidates with different characteristics
- Candidate dimensions: density, registerShift, syncopationBias, contourDirection, repetitionFactor
- Scoring (6 axes): harmonicFit, bassComplement, continuity, novelty, styleFit, energyFit
- Best candidate = highest totalScore

### Brain Modes
- **GENERATIVE**: CandidateGen creates 5 candidates/bar, picks best
- **MANUAL**: Only plays sequencer patterns
- **ADAPTIVE**: Generates from learned grammars (bass + melodic)

## 5. Composition Layer — Harmony & Melody

### Chord Progression Engine
7 progressions (scale degrees, 0-indexed):
- i-VI-III-VII -> Epic/Trance
- i-iv-v-i -> Minor classic
- I-IV-V-I -> Major classic
- i-VII-VI-V -> Andalusian (Phrygian)
- i-III-VII-VI -> Melodic
- i-v-iv-v -> Psytrance hypnotic
- I-V-vi-IV -> Pop/Prog

Chord behavior:
- Chord advances per bar
- PAD plays chord triad (root + 3rd + 5th)
- BASS anchors to chord root
- ARP plays chord-aware arpeggios

### Arpeggiator (5 modes)
- UP: root->3rd->5th->oct
- DOWN: oct->5th->3rd->root
- UP-DOWN: ping-pong (psytrance hypnotic)
- RANDOM: random order from chord
- CHORD: chord tones on strong beats

### Bass Pattern Modes
- ROLLING: K-B-B-B (psytrance classic, step%4!=0)
- OFFBEAT: Trance offbeat (step%4==2)
- PUMPING: 8th notes (step%2==0, skip kick steps)
- HALFTIME: Heavy (step 0 + 8, sparse)

### Pattern Operations (12 types)
- Nudge: shift all tracks left/right
- Rotate: rotate single track
- Copy/Paste: clipboard between tracks
- Swap: exchange two tracks
- Merge: OR-logic combine
- Invert: flip on/off
- Stretch: time-stretch (2x)
- Compress: time-compress (0.5x)
- Duplicate: LEAD to ARP
- Variation: mutate 30% of steps
- Quantize: snap velocities to 0.25
- Repeat: 1x/2x/3x/4x step subdivision

## 6. Scheduler Layer — Timing & Groove

### TempoClock
- bpm: 60-200 (tap tempo supported)
- swing: 0-100% (delays offbeat steps)
- timeSource: AudioContext.currentTime (NOT Date.now)
- BPM automation: per-section offsets (INTRO:-5, BREAK:-10, RISER:+5)
- BPM ramp: gradual +/-10 over 4 bars
- **Metronome (v5.2): Practice click with visual feedback (F5)**

### Humanization
- Probability: per-step play chance (100/75/50/25%)
- Micro-timing: +/-15ms jitter (scaled by humanize amount)
- Velocity: +/-20% variation on PERC
- KICK immune: keeps groove solid

### Section Arranger
- sections: [INTRO, BUILD, DROP, BREAK, RISER, DROP2, LOOP]
- Each section: bars (2-32), energy, density, patterns
- Features:
  - Auto-advance on bar count
  - Section loop (stay on current)
  - Section repeat (1x/2x/4x)
  - Pattern auto-save/load per section
  - Energy automation per section
  - Transition FX (riser/impact)
  - Auto-fill on transitions

## 7. FX Layer — Master Chain & Per-Track

### Master Chain
- voices -> master -> DRIVE -> FILTER -> comp -> analyser -> destination
- Parallel: DELAY (feedback) -> comp
- Parallel: REVERB (convolver) -> comp

FX components:
- **DRIVE**: WaveShaper (tanh curve, 2x oversample)
- **FILTER**: BiquadFilter lowpass (XY pad controls)
- **DELAY**: DelayNode + feedback (dotted 8th, tempo-synced)
- **REVERB**: ConvolverNode (generated impulse, 2.5s, decay 3.0)
- **COMP**: DynamicsCompressor (threshold -8, ratio 6)
- **Safety limiter**: prevents clipping

### Per-Track FX
- Delay send: per-track gain node -> master delay
- Reverb send: per-track gain node -> master reverb
- FX modes: NORMAL / DRY (0 sends) / WET (high sends) / PUMP
- Transition FX: Riser (noise sweep) / Impact (sub hit + noise burst)

## 8. Persistence Layer — State Management

### Storage Structure (localStorage)
- psy6-ultimate-state: last opened project (auto-save)
- psy6-ultimate-state:<name>: named projects (registry)
- psy6-ultimate-state:registry: list of all project names

### What Gets Saved
- patterns (all 6 tracks)
- grammars (bass + melodic + rhythm)
- settings (bpm, swing, genre, scale, brainMode, arpMode, bassMode)
- knobValues (8 knobs)
- sectionPatterns (per-section)
- musicalState (lead/bass last midi, energy)
- seed + variation

### Preset Management
- Favorites: Set of preset IDs (right-click to toggle)
- History: Last 10 selected presets
- Snapshots: Save/restore all 4 presets
- Export/Import: JSON files
- Search: Real-time filter by name/ID

### Pattern Banks (v5.0)
- 4 banks: A, B, C, D
- Each bank stores complete patterns (all 6 tracks)
- Auto-save current bank before switching
- Deep copy on save/load (no shared references)
- Undo support on bank load
- Persisted in project save/load
- Keyboard: F1-F4 to switch banks
- LCD shows current bank

### Metronome (v5.2)
- Practice click with adjustable tempo (follows currentBpm)
- Downbeat: 1200Hz, Beat: 800Hz
- Visual feedback: LCD text-shadow flash
- Keyboard: F5 to toggle
- Stops automatically on engine stop

### Track Colors (v5.2)
- 6 distinct colors for visual distinction
- KICK: red, BASS: orange, PERC: yellow
- LEAD: green, ARP: blue, PAD: purple
- Applied to track labels and active steps

### Round 13 Fixes (v5.4)
- Help panel: Added Shift+Space, F1-F4, Shift+F1-F4, F5 to Transport section
- Version display: Added v5.3 to header
- Meta description: Added for SEO

## 9. I/O Layer — MIDI & Audio Export

### MIDI In
- Notes C0-G0: Action triggers (fill, variation, reseed, chord, etc.)
- Notes C1+: Performance pads + teaches grammar
- CC 20-27: 8 Knobs
- CC 28-30: Macros (SPACE/ENERGY/TENSION)
- PC: Preset switching (cycles categories)
- Clock (0xF8): Tempo sync (24 ppq)
- Start/Stop: Transport control (0xFA/0xFC/0xFB)

### MIDI Out
- LEAD notes: External synth (Note On/Off)
- Clock: External sequencer (0xF8 per step)

### Audio Export
- WAV Export: 4 bars via OfflineAudioContext render
- **HQ WAV Export (v5.0): 44.1kHz CD-quality render via dedicated OfflineAudioContext**
- Live Rec: MediaRecorder API (WebM/Opus), full performance

### Sound Quality Enhancements (v6.4)
- **Explicit Limiter (v6.4): Brickwall limiting at -1 dB**
- **Saturator (v6.4): Warmth enhancement with soft saturation**
- **Improved Analyser (v6.4): fftSize 2048 for better resolution**
- Master chain: gain -> compressor -> limiter -> analyser -> destination
- Parallel: drive, saturator, delay, reverb, masterFilter

### MIDI File Export (v5.5)
- Standard MIDI File (SMF) format 0
- 480 ticks per quarter note
- Tempo meta event (current BPM)
- 4 bars exported
- Channels: KICK/PERC=9, BASS=0, LEAD=1, ARP=2, PAD=3
- Delta-time encoded Note On/Off events

### MIDI File Import (v5.9)
- Standard MIDI File (SMF) parsing
- MThd header + MTrk track parsing
- Delta-time decoding (variable length)
- Note On/Off event extraction
- Maps first 16 notes to LEAD pattern
- Converts MIDI note to scale degree
- Undo support (pushUndo)
- **ROUND-TRIP COMPLETE: Export → Import → Export**

### Song Arrangement (v6.0)
- Arrange sections into a complete song
- Add current section to arrangement (+ ADD)
- Remove section from arrangement (✕)
- Clear entire arrangement (✕ CLEAR)
- Play arrangement from start (▶ PLAY)
- Stop arrangement (■ STOP)
- Repeat count per section (1x/2x/3x/4x)
- Auto-advance to next section on completion
- Toast notifications for all actions
- **Arrangement Persistence (v6.1): Saved in project save/load**
- **Arrangement Export/Import (v6.1): JSON file export/import**

### Round 21 Fixes (v6.2)
- songArrangement added to serializeState (persistence fix)
- Arrangement persistence fully working

### Arrangement Reorder + Duplicate (v6.3)
- **Reorder (v6.3): Move sections up/down with ↑ ↓ buttons**
- **Duplicate (v6.3): Duplicate entire arrangement with 📋 DUPLICATE button**

### Step Probability Visualization (v5.6)
- Steps with probability < 1.0 shown with reduced opacity
- opacity = 0.4 + prob * 0.6
- Visual distinction for humanized patterns
- Helps identify ghost notes and probabilistic steps

### Step Repeat Visualization (v5.7)
- Steps with repeat > 1 shown with orange border
- Tooltip displays repeat count ("Repeat x2", "Repeat x3", etc.)
- Helps identify complex rhythmic patterns
- Visual distinction for multi-hit steps

### Preset Favorites Visualization (v5.7)
- Favorite presets shown with gold border and glow
- Star prefix (★) for favorites
- Visual hierarchy in preset list
- Quick identification of preferred sounds

### Round 17 Fixes (v5.8)
- Favorites added to help panel (Mouse Controls section)
- Right-click chip: Toggle preset favorite (gold border)

## 10. Preset System

### Genres & Categories
- Genres: TECHNO, PSYTRANCE, TRANCE, PROGRESSIVE
- Categories: drum, bass, lead, pad, pluck, arp, fx

### Preset Operations
- Randomizer: Random musical-constrained preset (per category)
- Randomize All: All categories at once
- Clone: Duplicate with subtle variations
- Morph: Blend two presets (50/50)
- Blend: Weighted blend of current + random
- Evolve: Mutate every 4 bars
- Random Walk: Adjacent preset every 2 bars
- Chain: Cycle presets per bar
- **Morphing Chain (v5.6): Auto-morph through categories every 4 bars**

## 11. Performance Budget

Target: 60fps UI, 0 audio dropouts at 145 BPM

Memory:
- 20 SynthVoice + 24 DrumVoice (pre-allocated)
- 0 runtime allocations in audio path
- Total heap: < 50MB

CPU:
- Scheduler: < 1ms per tick
- Voice trigger: < 0.1ms per note
- UI render: < 8ms per frame (60fps)

Latency:
- Audio output: < 10ms (AudioContext)
- MIDI input: < 5ms (Web MIDI)
- UI response: < 16ms (rAF)

## 12. Keyboard Shortcuts (46 total)

### Transport
SPACE=Play/Stop, Shift+Space=Global mute, T=Tap tempo, L=Section loop, Home=Section repeat, F1-F4=Pattern banks A-D, Shift+F1-F4=Clear banks, F5=Metronome

### Generation
V=Variation, S=Randomizer, W=Chords, D/H/Z=Drums/Melody/Arp, E=All variations, F=Fill

### Patterns
Q=Duplicate, I/O=Copy/Paste, J=Swap, ;=Merge, 0=Invert, 7/8=Compress/Stretch, </>=Nudge, U=Undo, X=Quantize

### Sound
A=Arp mode, B=Bass mode, C=Clone, 9=Blend, N=Randomize all, 4=Evolve, P=Chain, PageUp=Random walk, Del/Ins=Snapshot

### Timing
5/6=BPM ramp, [/]=Section length, Y=BPM automation, G=Humanize

### Performance
K=Scale lock, +/-=Octave, M=Auto-morph, Esc=Transition FX, End=Compressor, ?=Help, 1-3=Sections

## 13. Deployment

- Option A: Open index.html directly in browser
- Option B: npx serve . (local dev server)
- Option C: GitHub Pages (auto-deploy via CI)

Live: https://dudududi144-source.github.io/PSY6-ULTIMATE/

No build step. No dependencies. No server required.

## 14. Version History Summary

- v1.x: Foundation (engine, FX, grammar, learning, persistence)
- v2.x: Musicianship (chords, arp, MIDI, patterns, macros)
- v3.x: Control (40+ shortcuts, per-track, transitions, pan)
- v4.x: Professional (repeat, compressor, random walk) + Bug Fixes (59) + Refactoring + Accessibility
- v5.x: **MAJOR**: Pattern Banks (A/B/C/D) + HQ WAV Export (44.1kHz) + Bank Persistence + Metronome + Track Colors + Round 12-13 Fixes + Polish + **MIDI File Export (SMF)** + **Probability Viz + Morphing Chain** + **Repeat Viz + Favorites Viz** + **Round 17 Fixes** + **MIDI File Import (round-trip)**
- v6.x: **MAJOR**: **Song Arrangement** (arrange sections into complete songs with repeat) + **Arrangement Persistence + Export/Import** + **Round 21 Fixes** + **Arrangement Reorder + Duplicate** + **Sound Quality Enhancements**

---

Architecture version: 6.4
Status: IMPLEMENTED
Total code: 169.3 KB (single file)
Total shortcuts: 53
Total features: 100+