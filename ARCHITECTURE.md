# PSY6 ULTIMATE — Full Architecture Specification

## 1. Design Philosophy

PSY6 ULTIMATE follows the non-negotiable rules of the PSY family:

1. One source of truth per piece of musical state
2. Transport is not renderer, renderer is not UI
3. No device policy — this IS the device, built from foundation primitives
4. Every claim has evidence — benchmarks included

## 2. Layer Architecture

The instrument is organized in 5 layers, top to bottom:

- **UI Layer**: Knobs, XY Pad, Step Sequencer, Arranger View, Performance Pads
- **Brain Layer**: ContinuousMusicalState, CandidateGenerator, GrammarBuilder, StyleSelector
- **Scheduler Layer**: TempoClock (BPM + Swing), SectionArranger, StepScheduler
- **Engine Layer**: PooledEngine, SynthVoice pool, DrumVoice pool, AudioBus
- **Preset Layer**: Factory Library, Genre Tags, Style Builder

## 3. PooledEngine — Zero GC Architecture

### Problem
Creating Web Audio nodes per-note causes GC pauses leading to audio dropouts.

### Solution
Pre-allocate all voices at init. Reuse via round-robin.

SYNTH_VOICES = 20 (melodic: bass, lead, arp, pad)
DRUM_VOICES = 24 (percussive: kick, snare, hat, perc, fx)

Each voice:
- Created ONCE at AudioContext init
- All nodes pre-connected (osc -> filter -> vca -> bus)
- noteOn() only updates parameters (freq, gain, envelope)
- panic() cancels scheduled values and zeroes gain

### Voice Lifecycle
INIT: create voice -> connect to bus -> idle
NOTE: nextVoice() -> set params -> schedule envelope -> active
END: envelope completes -> voice returns to pool -> idle
PANIC: cancelScheduledValues -> gain=0 -> idle

## 4. Brain Layer — Generative Composition

### ContinuousMusicalState
Persists across phrase boundaries. Never resets.

State contains:
- leadLastMidi (last note played by lead)
- bassLastMidi (last note played by bass)
- contourMomentum (direction of melodic movement)
- registerCenter (average pitch)
- energy (0-1, from section + user input)

### CandidateGenerator
Per-bar, generates 5 candidates with different characteristics:

Candidate dimensions:
1. density (notes per bar)
2. registerShift (octave displacement)
3. syncopationBias (off-grid probability)
4. contourDirection (up/down/flat)
5. repetitionFactor (how much to repeat previous motif)

Scoring (6 axes):
- harmonicFit (does it fit current chord/scale?)
- bassComplement (register separation from bass)
- continuity (distance from previous phrase)
- novelty (information gain)
- styleFit (matches selected genre grammar)
- energyFit (matches current energy level)

Best candidate = highest totalScore

## 5. Scheduler — Timing Architecture

TempoClock:
  - bpm: 60-200
  - swing: 0-100%
  - timeSource: AudioContext.currentTime (NOT Date.now)

StepScheduler:
  - lookahead: 25ms
  - interval: setInterval(scheduler, 25)

SectionArranger:
  sections: [INTRO, BUILD, DROP, BREAK, RISER, DROP2, LOOP]
  Each section: { bars, energy, density, activeTracks }
  Auto-advance: when bar count reached -> next section

## 6. Audio Signal Flow

SynthVoice:
  osc1 -> filter -> vca -> trackBus -> master
  osc2 -> filter

DrumVoice:
  noise -> bandpass -> gain -> out -> master
  osc -> gain -> out

Master chain:
  master -> compressor -> limiter -> analyser -> destination

## 7. Preset System

Genres: TECHNO, PSYTRANCE, TRANCE, PROGRESSIVE
Categories: drum, bass, lead, pad, pluck, arp, fx

## 8. MIDI Integration

Web MIDI API: navigator.requestMIDIAccess()
- Note On (0x90): trigger pad / sequencer note
- Note Off (0x80): release
- CC (0xB0): map to knob (auto-learn)

## 9. Performance Budget

Target: 60fps UI, 0 audio dropouts at 145 BPM
Memory: 20 SynthVoice + 24 DrumVoice (pre-allocated), 0 runtime allocations
CPU: Scheduler < 1ms, Voice trigger < 0.1ms, UI render < 8ms

## 10. Deployment

Option A: Open index.html directly in browser
Option B: npx serve . (local dev server)
Option C: Deploy to Cloudflare Pages / Netlify / GitHub Pages

No build step. No dependencies. No server required.

---
Architecture version: 1.0
Status: IMPLEMENTED
