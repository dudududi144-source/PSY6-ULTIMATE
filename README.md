# PSY6 ULTIMATE — Standalone Psytrance Performance Instrument

> One file. Zero server. Infinite groove.

PSY6 ULTIMATE is the **unified endpoint** of the PSY device family:
- **Pooled Engine** from PSY5 (zero GC dropouts)
- **Groovebox UI** from PSY3 (knobs, XY pad, sequencer, arranger)
- **Factory Presets** from PSY5 (Psytrance / Techno / Trance / Progressive)
- **Creative Brain** from PSY4 (ContinuousMusicalState + CandidateGenerator)
- **Master FX Chain** (Filter + Delay + Reverb + Drive)

## Quick Start

Just open `index.html` in any modern browser. Or serve locally:

    npx serve .

Or use the live version:

    https://dudududi144-source.github.io/PSY6-ULTIMATE/

## Architecture

    +---------------------------------------------------------+
    |                    PSY6 ULTIMATE                         |
    +---------------------------------------------------------+
    |  UI Layer        |  Knobs, XY Pad, Seq, Arranger        |
    |  Brain Layer     |  ContinuousState, CandidateGen       |
    |  Scheduler       |  Swing, Arranger, Section Flow       |
    |  Engine Layer    |  PooledEngine, VoicePool, AudioBus   |
    |  FX Chain        |  Filter, Delay, Reverb, Drive        |
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
| R | Record + Export WAV (4 bars) |
| 1-8 | Jump to section |
| M | Toggle MIDI learn |

## Control Knobs

| Knob | Function |
|------|----------|
| BPM | Tempo (60-200) |
| SWING | Offbeat delay (0-100%) |
| FILTER | Master filter cutoff (100Hz-18kHz) |
| RESO | Master filter resonance (0-30) |
| DELAY | Delay send level |
| REVERB | Reverb send level |
| DRIVE | Waveshaper saturation |
| VOL | Master volume |

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
- WAV export (OfflineAudioContext)

## Version History

| Version | Changes |
|---------|---------|
| 1.0 | Initial unified instrument |
| 1.1 | MIDI guard + WAV export |
| 1.2 | Master FX chain + swing fix + bass state tracking |

## License

MIT — do whatever you want with it.

---

*Built from the ashes of psy3, psy4, psy5, and psy-foundation.*
*One file to rule them all.*
