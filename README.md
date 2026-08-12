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

## License

MIT

---

*Built from the ashes of psy3, psy4, psy5, and psy-foundation.*
*One file to rule them all.*
