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

## Your own instrument

`My First Instrument.dspreset` is an empty template. Add your own samples, effects and sliders to it, using the three levels below as a guide.

## The three levels

| Level | File | What you learn |
|---|---|---|
| 1. Easy | `My First Instrument - Easy.dspreset` | Sample mapping, volume, attack and release |
| 2. Medium | `My First Instrument - Medium.dspreset` | Round robins, the full ADSR envelope, a filter and reverb |
| 3. Hard | `My First Instrument - Hard.dspreset` | More zones (10 notes × 3 takes), Mono legato and glide, an effects chain, an LFO and a full interface |

Work through them in order. Each level adds new ideas to the one before.

---

## How a `.dspreset` file is built

```xml
<DecentSampler minVersion="1.27.0">
  <groups>      ... which samples play on which keys ...  </groups>
  <effects>     ... filter, chorus, delay, reverb ...     </effects>
  <modulators>  ... LFOs that move things over time ...   </modulators>
  <ui>          ... the sliders and text you see        </ui>
</DecentSampler>
```

Only `<groups>` is required. Everything else is optional.

---

## 1. Samples and mapping

```xml
<sample rootNote="62" loNote="53" hiNote="62" path="Samples/DH WARP3/DH WARP3_062_RR1.flac" />
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

Give each take **its own group**, and put the settings they share on `<groups>`:

```xml
<groups seqMode="round_robin" seqLength="3" attack="0.01" release="0.6">
  <group seqPosition="1"> ...all RR1 samples... </group>   <!-- 1st hit -->
  <group seqPosition="2"> ...all RR2 samples... </group>   <!-- 2nd hit -->
  <group seqPosition="3"> ...all RR3 samples... </group>   <!-- 3rd hit, then back to 1 -->
</groups>
```

- `seqLength` is the number of takes. Each group's `seqPosition` says which take it is.
- Every take group should have the **same zones**, using the same `rootNote`, `loNote` and `hiNote` values.
- Settings on `<groups>`, such as the envelope, apply to every group inside it. A group can still override one, e.g. `<group seqPosition="2" tuning="0.03">`.

### Other ways to use groups
| Use | How |
|---|---|
| Velocity layers | `<group loVel="1" hiVel="80">` for soft recordings, `<group loVel="81" hiVel="127">` for hard ones |
| Layering two sounds | Two groups without `seqPosition` play **together**. Use this on purpose, e.g. plucks plus a pad. |
| Release noises | `<group trigger="release">` plays when you let go of a key |
| Mix or mute a layer | Knob or button bound to `type="amp" level="group" position="N"`, with parameter `AMP_VOLUME` or `ENABLED` |

> **Common mistake:** two full groups with no `seqPosition` don't alternate. They both play at once, which makes the sound twice as loud and phasey.

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

The three "My First Instrument" presets use a plain white window at Decent Sampler's default size, **812 × 375** pixels, with one vertical slider per setting.

```xml
<ui width="812" height="375" bgColor="FFFFFFFF">
  <tab name="main">
    <label x="24" y="16" width="600" height="30" text="My First Instrument - Hard" textColor="FF000000" textSize="18" />
    <control ...> <binding ... /> </control>
    <label ... text="Cutoff" textColor="FF000000" />
  </tab>
</ui>
```

### Positions
`x` and `y` are measured in pixels from the **top-left corner**. `width` and `height` set the size of the element. The on-screen keyboard covers the bottom of the window, so keep everything above about **y = 230**.

### Colours
Colours are written as `AARRGGBB` hex codes. The first two characters are the opacity: `FF` is solid and `00` is invisible.

| Code | Colour |
|---|---|
| `FFFFFFFF` | white (the background) |
| `FF000000` | black (the text) |
| `FF333333` | dark grey (the filled part of a slider) |
| `FFDDDDDD` | light grey (the empty part of a slider) |

### Sliders: how a slider controls the sound

A slider does **nothing** on its own. It needs a `<binding>` inside it to tell it what to control. The name underneath it is a separate `<label>`, so you can always choose its colour:

```xml
<control x="..." y="72" width="20" height="120" style="linear_bar_vertical"
         parameterName="Cutoff" showLabel="false" type="float"
         minValue="200" maxValue="20000" value="15000"
         trackForegroundColor="FF333333" trackBackgroundColor="FFDDDDDD">
  <binding type="effect" level="instrument" position="0" parameter="FX_FILTER_FREQUENCY" />
</control>
<label x="..." y="196" width="50" height="16" text="Cutoff" textColor="FF000000" hAlign="centred" />
```

| Attribute | Meaning |
|---|---|
| `style` | `linear_bar_vertical` makes a vertical slider. Other styles: `linear_vertical`, `linear_horizontal`, `rotary_vertical_drag` (a knob). |
| `parameterName` | The name your DAW shows for automation |
| `minValue` / `maxValue` / `value` | The slider's range and starting value |
| `trackForegroundColor` / `trackBackgroundColor` | The colours of the filled and empty parts |

**One binding per group:** a setting that lives on a group, such as Attack or Release, needs one `<binding>` for **each** group. With 3 round-robin groups, use `position="0"`, `"1"` and `"2"`. If you only bind position 0, the slider changes just the first take.

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

> **Common mistake:** if you add or move an effect, the position numbers change. Update the `position` in your slider bindings to match.

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
| No sound, or "file not found" | Check the `path`. It is relative to the `.dspreset` file. In the main folder, use `Samples/...`. Inside `Example Code/` or `Instrument Examples/`, use `../Samples/...`. Always use forward slashes `/`. |
| Some keys are silent | There is a gap between zones. Check the `loNote` and `hiNote` values. |
| A slider does nothing | It has no `<binding>`, or its `position` points at the wrong effect or group. |
| A slider only affects some notes | Add a binding for every group (`position` 0, 1, 2 …). |
| The preset won't load at all | There is an XML typo, such as a missing `"`, a missing `/>` or an unclosed tag. |
