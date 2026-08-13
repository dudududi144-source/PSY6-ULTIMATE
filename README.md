# PSY6 ULTIMATE — Standalone Psytrance Performance Instrument

> One file. Zero server. Infinite groove.

PSY6 ULTIMATE is the **unified endpoint** of the PSY device family:
- **Pooled Engine** from PSY5 (zero GC dropouts)
- **Groovebox UI** from PSY3 (knobs, XY pad, sequencer, arranger)
- **Factory Presets** from PSY5 (Psytrance / Techno / Trance / Progressive)
- **Creative Brain** from PSY4 (CandidateGenerator + Grammar System)
- **Master FX Chain** (Filter + Delay + Reverb + Drive)

## Quick Start

Just open `index.html` in any modern browser. Or serve locally:

    npx serve .

Or use the live version:

    https://dudududi144-source.github.io/PSY6-ULTIMATE/

## Brain Modes

| Mode | Behavior |
|------|----------|
| **GENERATIVE** | CandidateGenerator creates 5 candidates/bar, picks best |
| **MANUAL** | Only plays what you program in the sequencer |
| **ADAPTIVE** | Learns from what plays + what you perform, generates from learned grammars |

### Grammar System (ADAPTIVE mode)

The instrument builds statistical models in real-time:

- **BassGrammar**: 12x12 interval transition matrix — learns which bass notes follow which
- **MelodicGrammar**: interval histogram — learns melodic contour tendencies
- **RhythmGrammar**: kick onset pattern — learns rhythmic placement

**How to teach it:** Play the performance pads or MIDI keyboard while in ADAPTIVE mode. The system learns your melodic choices and applies them to generation. Watch the "Grammar confidence" percentage climb in the Brain panel.

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| SPACE | Play / Stop |
| V | New variation (reseed) |
| W | Generate chord progression |
| D | Generate drum pattern |
| H | Generate melody |
| Z | Generate arpeggio |
| R | Record + Export WAV (4 bars) |
| S | Sound design randomizer |
| A | Cycle arpeggiator mode |
| 1-8 | Jump to section |

## Control Knobs

BPM · SWING · FILTER · RESO · DELAY · REVERB · DRIVE · VOL

## MIDI Support

Connect any MIDI controller:
- **Notes** -> Performance pads + teaches melodic grammar
- **CC 20-27** -> Knobs (auto-learn on first touch)
- **CC 28** -> SPACE macro (reverb + delay + filter)
- **CC 29** -> ENERGY macro (drive + volume + filter)
- **CC 30** -> TENSION macro (filter + reso + swing)

### MIDI Out
Toggle with **📤 MIDI OUT** button:
- Sends LEAD notes to external synth
- Sends MIDI Clock (0xF8) for tempo sync
- First available MIDI output port is used
- **MIDI Clock** (0xF8) -> External tempo sync (toggle with CLOCK button)
- **MIDI Start/Stop** (0xFA/0xFC) -> Transport control when clock enabled

## Architecture

    UI Layer        Knobs, XY Pad, Seq, Arranger, Pads
    Brain Layer     CandidateGenerator + Grammar System (3 grammars)
    Persistence     localStorage (patterns, grammars, settings, auto-save)
    Scheduler       Swing, Section Arranger, Step Scheduler
    Engine Layer    PooledEngine (20 synth + 24 drum voices)
    FX Chain        Drive -> Filter -> [Delay, Reverb] -> Comp
    Presets         Factory Library (4 genres)

## Version History

| Version | Changes |
|---------|---------|
| 1.0 | Initial unified instrument |
| 1.1 | MIDI guard + WAV export + GitHub Pages |
| 1.2 | Master FX chain + swing fix + bass state tracking |
| 1.3 | Grammar System + ADAPTIVE brain mode + grammar learning from performance |
| 1.4 | State Persistence (localStorage) + RhythmGrammar integration + Save/Load project |
| 1.5 | Multiple named projects + Style Builder (sound design randomizer) |
| 1.6 | Live Recording (MediaRecorder) + Preset Export/Import |
| 1.7 | MIDI Clock sync (external tempo + transport control) |
| 1.8 | Macro Controls (SPACE/ENERGY/TENSION) + Performance Mode |
| 1.9 | Grammar Decay (adaptive forgetting) + Grammar Visualization |
| 2.0 | Chord Progression Engine (7 progressions, per-bar changes, bass anchoring) |
| 2.1 | Chord-aware Arpeggiator (5 modes: UP/DOWN/UP-DOWN/RANDOM/CHORD) |
| 2.2 | MIDI Out (notes + clock to external gear) + Track Solo |
| 2.3 | Per-track FX Sends (delay/reverb per track) + Song Export/Import |
| 2.4 | Tap Tempo + Section Loop + Sequencer Undo (T/L/U shortcuts) |
| 2.5 | Bass Pattern Modes (4 modes) + Fill Generator + Energy Automation |
| 2.6 | Step Velocity Editing (right-click) + Section Patterns + Pad Octave Shift |
| 2.7 | Humanization (probability + micro-timing) + MIDI Learn Mode + Preset Morpher |
| 2.8 | Pattern Nudge + Track Volume controls + Preset Favorites |
| 2.9 | Help Panel (shortcut reference) + Pattern Duplicate |
| 3.0 | Preset Search + Global Quantize + BPM Automation per section |
| 3.1 | Pattern Rotate (double-click M) + Randomize All Presets + Section Length Control |
| 3.2 | Pattern Variation Generator (E) + MIDI Program Change + Scale Lock (K) |
| 3.3 | Pattern Copy/Paste (I/O) + Preset Chain (P) + Step Repeat |
| 3.4 | Pattern Swap (J) + MIDI Note Action Mapping (C0-G0) + Auto-Morph (M) |
| 3.5 | Pattern Merge (;) + Preset History + Step Probability (shift+right-click) |
| 3.6 | Pattern Invert (0) + Preset Clone (C) + Track FX Modes (N/D/W/P per track) |
| 3.7 | Pattern Stretch/Compress (7/8) + Track Delay Offset + Preset Blend (9) |
| 3.8 | BPM Ramp (5/6) + Mute Groups (DRUMS/SYNTHS) + Preset Evolve (4) |
| 3.9 | Track Pan (per-track stereo) + Section Transition FX (riser/impact) + Preset Snapshot |
| 4.0 | Section Repeat (Home) + Track Compressor (End) + Preset Random Walk (PageUp) |
| 4.1 | Help Panel updated with all 46 shortcuts + mouse controls + MIDI map |
| 4.2 | Bug fixes: keydown guard, disconnect safety, state resets, overflow protection, div-by-zero, infinite loop |
| 4.3 | Deep bug fixes: pattern fields, empty scale/progression guards, NaN handling, null checks, empty buffer handling |
| 4.4 | MIDI hot-plug detection, AudioContext resume, envelope overlap prevention (synth + drum) |
| 4.5 | Song export deep copy, genre fallback, UI null checks (5 functions), BPM bounds (MIDI clock + ramp) |
| 4.6 | safeParse wrapper, canvas fallback text, removed all console.log (security + a11y + quality) |

## License

MIT

---

*Built from the ashes of psy3, psy4, psy5, and psy-foundation.*
*One file to rule them all.*
