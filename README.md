# PSY6 ULTIMATE — Standalone Psytrance Performance Instrument

> One file. Zero server. Infinite groove.

PSY6 ULTIMATE is the **unified endpoint** of the PSY device family:
- **Pooled Engine** from PSY5 (zero GC dropouts)
- **Groovebox UI** from PSY3 (knobs, XY pad, sequencer, arranger)
- **Factory Presets** from PSY5 (Psytrance / Techno / Trance / Progressive)
- **Creative Brain** from PSY4 (ContinuousMusicalState + CandidateGenerator)

## Quick Start

Just open `index.html` in any modern browser. Or serve locally:

    npx serve .

## Architecture

    +---------------------------------------------------------+
    |                    PSY6 ULTIMATE                         |
    +---------------------------------------------------------+
    |  UI Layer        |  Knobs, XY Pad, Seq, Arranger        |
    |  Brain Layer     |  ContinuousState, CandidateGen       |
    |  Scheduler       |  Swing, Arranger, Section Flow       |
    |  Engine Layer    |  PooledEngine, VoicePool, AudioBus   |
    |  Presets         |  Factory Library, Style Builder      |
    +---------------------------------------------------------+

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| SPACE | Play / Stop |
| V | New variation (reseed) |
| W | Generate chord progression |
| D | Generate drum pattern |
| H | Generate melody |
| Z | Generate arpeggio |
| R | Record loop |
| 1-8 | Jump to section |
| M | Toggle MIDI learn |

## MIDI Support

Connect any MIDI controller. The instrument auto-maps:
- **Notes** -> Performance pads + sequencer
- **CC** -> Knobs (auto-learn on first touch)

## Proof of Concept

This instrument proves that the entire PSY ecosystem can be unified into a single, deployable artifact:

- 0 GC dropouts (pooled voices)
- 60fps UI (requestAnimationFrame)
- Continuous playback (no gaps between sections)
- Generative composition (candidate scoring)
- MIDI input (Web MIDI API)
- Zero dependencies (pure Web Audio API)

## License

MIT — do whatever you want with it.

---

*Built from the ashes of psy3, psy4, psy5, and psy-foundation.*
*One file to rule them all.*
