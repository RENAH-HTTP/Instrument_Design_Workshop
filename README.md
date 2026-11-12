# Instrument Design Workshop

Build your own sampled instrument for **Decent Sampler**, a free plugin that plays `.dspreset` files.

**Official documentation:** https://www.decentsamples.com/docs/format-documentation.html
**Download Decent Sampler:** https://www.decentsamples.com/product/decent-sampler-plugin/

---

## Getting started

1. Install Decent Sampler (as a standalone app or as a plugin in your DAW).
2. In Decent Sampler, choose **File → Load Preset…** and open a `.dspreset` file from this folder.
3. Play notes on your MIDI keyboard, or use the on-screen keyboard.
4. Open the same `.dspreset` file in a text editor and change something.
5. Save it, then reload it in Decent Sampler to hear the change.

> A `.dspreset` file is plain XML text. It doesn't store the sounds themselves. It only tells Decent Sampler which audio files to play and how to play them.

## Folder layout

```
Instrument_Design_Workshop/
├── My First Instrument - Easy.dspreset     ← Level 1
├── My First Instrument - Medium.dspreset   ← Level 2
├── My First Instrument - Hard.dspreset     ← Level 3
├── Example Code/          ← 15 small presets, one idea each
├── Instrument Examples/   ← finished instruments to explore
├── Samples/               ← the audio recordings (.flac)
└── IR/                    ← impulse responses for reverb
```

## The three levels

| Level | File | What you learn |
|---|---|---|
| 1. Easy | `My First Instrument - Easy.dspreset` | Sample mapping, the volume control and a simple envelope |
| 2. Medium | `My First Instrument - Medium.dspreset` | Round robins and the full ADSR envelope |
| 3. Hard | `My First Instrument - Hard.dspreset` | Mono legato and glide, an effects chain, an LFO and a full interface |

Work through them in order. Each level adds new ideas to the one before.

---

## How a `.dspreset` file is built

```xml
<DecentSampler minVersion="1.27.0">
  <groups>      ... which samples play on which keys ...  </groups>
  <effects>     ... filter, chorus, delay, reverb ...     </effects>
  <modulators>  ... LFOs that move things over time ...   </modulators>
  <ui>          ... the knobs and text you see ...        </ui>
</DecentSampler>
```

Only `<groups>` is required. Everything else is optional.

---

## 1. Samples and mapping

```xml
<sample rootNote="60" loNote="55" hiNote="66" path="Samples/DH WARP18/DH WARP18-C3-V127-O9JD.flac" />
```

| Attribute | Meaning |
|---|---|
| `path` | Where the audio file is, **relative to the `.dspreset` file**. Use forward slashes `/`. |
| `rootNote` | The note the sample was recorded at. At this note, it plays at its original pitch. |
| `loNote` / `hiNote` | The range of keys (the **zone**) that play this sample. Outside the root note, the sample is pitched up or down. |

MIDI note numbers: **60 = middle C (C3 in Decent Sampler)**. Each step of 12 is one octave: 48 = C2, 72 = C4.

**Tips**
- Zones should not leave gaps. If one zone ends at 54, the next should start at 55.
- Set the lowest zone's `loNote` to `0` and the highest zone's `hiNote` to `127`, so every key makes a sound.
- Pitching a sample too far from its root sounds unnatural. That is why we record several notes.

## 2. Groups

A `<group>` holds samples that share settings, such as velocity range, envelope and round robin.

```xml
<group loVel="1" hiVel="127" ampVelTrack="0.5"> ... </group>
```

| Attribute | Meaning |
|---|---|
| `loVel` / `hiVel` | The range of key velocities (how hard a key is played, 1–127) this group responds to |
| `ampVelTrack` | How much velocity changes the volume. `0` means every note is equally loud; `1` means the volume follows velocity fully. |

## 3. Round robin (Medium, Hard)

When you play the same note twice, a real instrument never sounds exactly the same. Round robin cycles through different recordings of the same note, so repeated notes don't sound like a "machine gun".

```xml
<group seqMode="round_robin" seqLength="3">
  <sample ... seqPosition="1" />   <!-- 1st hit -->
  <sample ... seqPosition="2" />   <!-- 2nd hit -->
  <sample ... seqPosition="3" />   <!-- 3rd hit, then back to 1 -->
</group>
```

`seqLength` is the number of takes. Each sample's `seqPosition` says which take it is.

## 4. Amp envelope: ADSR

The envelope shapes the volume of each note over time.

| Stage | Attribute | What it does |
|---|---|---|
| **A**ttack | `attack` | Seconds for the note to fade in |
| **D**ecay | `decay` | Seconds to fall from full volume to the sustain level |
| **S**ustain | `sustain` | Volume while the key is held (0–1) |
| **R**elease | `release` | Seconds to fade out after the key is let go |

Try these: a slow attack (1–2 s) gives a pad sound, and a long release gives a ringing tail.

## 5. Monophonic and legato (Hard)

```xml
<groups polyphony="1" glideTime="0.08" glideMode="legato" returnToHeldNote="true">
```

| Attribute | Meaning |
|---|---|
| `polyphony="1"` | Only one note sounds at a time, like a voice or a synth lead |
| `glideTime` | Seconds it takes to slide from one note to the next |
| `glideMode="legato"` | Glide only when notes overlap, meaning you hold one key while pressing the next |
| `returnToHeldNote` | When you let go of the top note, it goes back to the note you're still holding |

## 6. Effects (Hard)

Effects run in order from top to bottom. Each effect has a **position number**, counting from 0:

```xml
<effects>
  <effect type="lowpass" frequency="15000" resonance="0.7" />      <!-- position 0 -->
  <effect type="chorus" mix="0.18" modDepth="0.2" modRate="0.2" /> <!-- position 1 -->
  <effect type="delay" delayTimeFormat="musical_time" delayTime="4.0"
          feedback="0.3" wetLevel="0.15" />                        <!-- position 2 -->
  <effect type="convolution" irFile="IR/Hall.flac" mix="0.2" />    <!-- position 3 -->
  <effect type="gain" level="-3" />                                <!-- position 4 -->
</effects>
```

| Effect | What it does | Key settings |
|---|---|---|
| `lowpass` | Removes high frequencies so the sound is darker and softer | `frequency` (Hz), `resonance` (a peak at the cutoff) |
| `chorus` | Adds a thicker, shimmering sound | `mix`, `modDepth`, `modRate` |
| `delay` | Echoes | `delayTime`, `feedback` (how many repeats), `wetLevel` |
| `convolution` | Reverb built from a recording of a real space | `irFile` (a file from `IR/`), `mix` |
| `gain` | Volume change in dB | `level` |

Try a different reverb: change `IR/Hall.flac` to `IR/Plate.flac`, `IR/Spring.flac` or `IR/Lush.flac`.

## 7. Modulators: LFO (Hard)

An LFO (low-frequency oscillator) is a slow wave that moves a setting up and down on its own. Here it moves the pitch, which creates **vibrato**.

```xml
<lfo shape="sine" frequency="5" modAmount="0.2" scope="voice">
  <binding type="general" level="group" position="0" parameter="GROUP_TUNING"
           modBehavior="add" translation="linear"
           translationOutputMin="0" translationOutputMax="0.5" />
</lfo>
```

- `frequency`: how fast it moves (5 = five times per second)
- `modAmount`: how deep the effect is (0–1)
- `shape`: the wave shape: `sine`, `triangle`, `square` or `saw`
- The `<binding>` decides **what** it moves. Here that is the tuning, up to 0.5 semitones.

---

## 8. The user interface

```xml
<ui width="812" height="375" bgColor="FF14161F">
  <tab name="main">
    <rectangle x="16" y="68" width="480" height="142" fillColor="FF1E2230" />
    <label x="28" y="74" width="300" height="18" text="AMP ENVELOPE" textColor="FF4FD1C5" />
    <labeled-knob ...> <binding ... /> </labeled-knob>
  </tab>
</ui>
```

### Positions
`x` and `y` are measured in pixels from the **top-left corner**. `width` and `height` set the size of the element.

### Colours
Colours are written as `AARRGGBB` hex codes:

| Code | Colour |
|---|---|
| `FFFFFFFF` | white |
| `FF14161F` | very dark blue (the background) |
| `FF4FD1C5` | teal |
| `80FFFFFF` | white at 50% opacity (`80` is the alpha) |

The first two characters are the opacity: `FF` is solid and `00` is invisible.

### Knobs: how a knob controls the sound

A knob does **nothing** on its own. It needs a `<binding>` inside it to tell it what to control:

```xml
<labeled-knob x="562" y="96" width="80" height="106" label="Cutoff"
              type="float" minValue="200" maxValue="20000" value="15000"
              textColor="FFE8ECF5" trackForegroundColor="FFF6AD55">
  <binding type="effect" level="instrument" position="0" parameter="FX_FILTER_FREQUENCY" />
</labeled-knob>
```

| Attribute | Meaning |
|---|---|
| `label` | The text shown under the knob |
| `textColor` | Colour of that text. **Always set it**, or it can be invisible on a dark background. |
| `minValue` / `maxValue` / `value` | The knob's range and starting value |
| `trackForegroundColor` | Colour of the knob's arc |

**Binding cheat sheet**

| What to control | `type` | `level` | `position` | `parameter` |
|---|---|---|---|---|
| Main volume | `amp` | `instrument` | `0` | `AMP_VOLUME` |
| Attack / Decay / Sustain / Release | `amp` | `group` | group number | `ENV_ATTACK` / `ENV_DECAY` / `ENV_SUSTAIN` / `ENV_RELEASE` |
| Filter cutoff / resonance | `effect` | `instrument` | effect position | `FX_FILTER_FREQUENCY` / `FX_FILTER_RESONANCE` |
| Chorus or reverb mix | `effect` | `instrument` | effect position | `FX_MIX` |
| Chorus speed / depth | `effect` | `instrument` | effect position | `FX_MOD_RATE` / `FX_MOD_DEPTH` |
| Delay level / feedback | `effect` | `instrument` | effect position | `FX_WET_LEVEL` / `FX_FEEDBACK` |
| LFO depth | `modulator` | `instrument` | modulator position | `MOD_AMOUNT` |

> **Common mistake:** if you add or move an effect, the position numbers change. Update the `position` in your knob bindings to match.

---

## Example Code

Each file in `Example Code/` shows one idea on its own:

| # | File | Topic |
|---|---|---|
| 01 | Static_Recording | One sample across the whole keyboard |
| 02 | Sample_Mapping | Several samples in zones |
| 03 | Dynamic_Recording | Velocity layers |
| 04 | Round_Robin | Alternating takes |
| 05 | Amplitude_Envelope | ADSR |
| 06 | User_Interface | Knobs and labels |
| 07 | Monophonic_Glide | One note at a time, with glide |
| 08 | True_Sampled_Legato | Recorded note transitions |
| 09 | Velocity_Sensitivity | `ampVelTrack` |
| 10 | Tuning_and_Pan | Tuning and stereo position |
| 11 | Filter_Effect | Low-pass filter |
| 12 | Convolution_Reverb | Reverb using impulse responses |
| 13 | LFO_Modulation | Vibrato and tremolo |
| 14 | Oscillator_Synthesis | Built-in oscillators instead of samples |
| 15 | Effects_Chain | Several effects in a row |

## Troubleshooting

| Problem | Fix |
|---|---|
| No sound, or "file not found" | Check the `path`. It is relative to the `.dspreset` file, so use `Samples/...`, not `\Samples\...` or `..\Samples\...`. |
| Some keys are silent | There is a gap between zones. Check the `loNote` and `hiNote` values. |
| A knob does nothing | It has no `<binding>`, or its `position` points at the wrong effect. |
| You can't read the knob text | Add `textColor="FFFFFFFF"` to the knob. |
| The preset won't load at all | There is an XML typo, such as a missing `"`, a missing `/>` or an unclosed tag. |
