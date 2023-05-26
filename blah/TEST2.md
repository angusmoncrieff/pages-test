## Soundworld builder

A soundworld creator can submit a new soundworld comprising:

* Instruments:
    * `.fxp` VST preset files
    * sample folders
* Midi files
    * for rhythm of many elements (e.g. drum elements, or keys playing chords)
    * arpeggiator patterns (todo...)
* `.txt` metadata files
    * `_soundworld.txt` - metadata for whole SW - contains e.g. tempo range, chord progressions
    * `_element.txt` - metadata for each element - e.g. element role (bass/chords/drums/...), pitch range
    * `MyInst1.txt` - metadata for each instrument (both VST and samples) - e.g. chromatic or unpitched, tags, etc.
    * `_midi.txt` - metadata for midi folder - e.g. duration of midi clips

...In directory structure organised primarily by Element, like this:

```
my-new-soundworld
├── BassBed
│   ├── instruments
│   │   ├── LongBass.fxp
│   │   └── LongBass.txt
│   └── _element.txt
├── BassGroove
│   ├── instruments
│   │   ├── LowQuack.fxp
│   │   └── LowQuack.txt
│   ├── midi
│   │   ├── _midi.txt
│   │   ├── bass_00.mid
│   │   ├── bass_01.mid
│   │   └── bass_02.mid
│   └── _element.txt
├── Keys
│   ├── instruments
│   │   ├── AmbientOoh.fxp
│   │   ├── AmbientOoh.txt
│   │   └── ShinyLovely.fxp
│   ├── midi
│   │   ├── _midi.txt
│   │   ├── chords_00.mid
│   │   └── chords_01.mid
│   └── _element.txt
├── Clap
│   ├── instruments
│   │   ├── Clap1
│   │   │   ├── Clap 01.wav
│   │   │   ├── Clap 02.wav
│   │   │   ├── Clap 03.wav
│   │   │   └── Clap 04.wav
│   │   └── Clap1.txt
│   └── midi
│       ├── _midi.txt
│       └── clap_00.mid
├── Lead
│   ├── instruments
│   │   ├── HappyFizz.fxp
│   │   └── HappyFizz.txt
│   ├── midi
│   │   ├── lead_00.mid
│   │   └── lead_01.mid
│   └── _element.txt
└── _soundworld.txt
```

## `_soundworld.txt`

A text file to describe/define the soundworld. 

NB. generally speaking, the parser tries to 'fill in the blanks' in a smart way, so you do not necessarily need to specify everything.

Example content:

```
version: 1.1
date: 17/05/2023
creator: John Doe

name: Classic House
alt names: House, Classic Dance
tempo range: 115-130

chord progressions:
- C: C | % | Bb | F- Bb |
- C-: C- | Bbmaj7 | C- | Bbmaj7 
- C-: C- | Bbmaj7 | Abmaj7 | / / Bbmaj7 / |

elements:
- BassGroove (50), BassBed (50)
- OpenHat (50), muted (50)
```

### Settings list - soundworld

#### `tempo range` 

* e.g. `99-104`
* in bpm

#### `chord progressions`

* a bunch of chord progressions - each run will choose one of them
* each run will also transpose to a random key, so just write each in a key you find convenient (and certainly no need to 'duplicate' the same progression in different keys)
* first part says the key - e.g. `C:` means the progression is in C major; `C-:` means C minor
* `|` is a bar line - two chords in a bar get half the bar each

Possible chord symbols: 
* major triad: `maj` or just the root, e.g. `Bbmaj` or just `Bb`
* minor triad: `min` or `m` or `-`, e.g. `Fmin`, `Fm`, `F-`
* `maj7` or `^7`, `6`, `maj9`, `maj7#11`, `maj9#11`, `maj13`, `69`, `add9`, `maj7#5`, `maj7sus2` 
* `min7` or `m7` or `-7`, `m9`, `-9`, `m11`, `-11`, `min(maj7)` or `-^7`, `min6`, `min69`, `min(b6)`, `min#5`
* `7`, `7b9`, `7b9#11`, `9`, `7#9`, `7b9b13`, `7#11`, `9#11`, `13`, `13b9`, `13#11`, `7+`, `7altered`, `7sus4`, `9sus4`, `13sus4`, `7b9sus`
* `dim` or `o`, `dim7` or `o7`
* `aug` or `+`
* `min7b5` or `-7b5` 

(..That's not an exhaustive list.)

#### `elements`

This is for elements which you want to switch off or on randomly on different runs - no need to list all the 'regular' elements here. For each line, one item will be chosen on each run. Each element name must match one of the folder names of the elements in your SW. The numbers are weights - they don't have to add to 100.

Examples:

```
elements:
- BassGroove (50), BassBed (50)                     # 1. 
- OpenHat (25), muted (75)                          # 2. 
- GuitarStrums (20), PickedGuitar (10), muted (10)  # 3. 
- ChordGroove, Arpeggiator                          # 4. 
```
1. each run plays EITHER BassGroove OR BassBed, 50/50 chance
2. OpenHat plays in a quarter of runs (else silent)
3. GuitarStrums (50% of runs) OR PickedGuitar (25% of runs) OR nothing (25%)
4. No weight specified -> weight 1 (so ChordGroove/Arpeggiator are equally likely)

## `_element.txt`

A text file to describe/define the element in the soundworld. 

NB. generally speaking, the parser tries to 'fill in the blanks' in a smart way, so you do not necessarily need to specify everything.

### Lead element example:
``` 
role: lead
pitch range: C4-F#6           # somewhere in this range
interval range: 12-12         # but not more than an octave
scale type: minor pentatonic
chord lookahead: yes
```

Lead elements will generate a melodic figure based on one of the provided rhythms (i.e. midi files).  If a scale type is provided then it will use that - but pitches will be 'corrected' if necessary to fit the chord symbol at any point.

### Chords element:
``` 
role: chords
pitch range: F3-Ab4 
chord lookahead: yes
```

Chords elements will use one of the provided rhythms - or if no rhythm is provided then each chord voicing will be played for the duration of each chord symbol.

Use `chord lookahead` if you want chord 'hits' just before a chord change to anticipate the new chord.

### Bass element:
``` 
role: bass
pitch range: E1-E3
interval range: 12-16
```

Bass elements will play root pitches (or specified bass pitch in the case of slash chords like G7/D) on one of the provided rhythms - or if you want long notes then just don't provide any rhythms (i.e. midi files). 


### Drum piece element:
``` 
type: drum piece
```
(...In fact, if the element is named e.g. Kick, Clap, Snare, Hat, Hihat, Ride, etc. then the parser will infer that this is a drum piece element - so often you won't need to have a .txt file for these atm.) 

Drum piece elements assume the provided instrument(s) are 'sample menus' with a different sample on each pitch. The element will pick a random sample and play that according to one of the provided rhythms.

### Settings list - elements

#### `type`

* some 'specific' behaviour (for future expansion...)
* e.g. `drum piece` or `arpeggiator`
* otherwise don't specify

#### `role`

* the broad **musical function** of the element 
* `lead` or `chords` or `bass` or `drums` or `percussion`
* if this value is absent then we'll attempt to infer the role from the name of the element (e.g. if called `BassGroove` then it'll be considered a `bass` role)

Role `chords` means that the function of the element is to provide harmony. So for instance an arpeggiator element would typically have role `chords`.  

#### `pitch range`

* e.g. like `C4-F#5` or `60-80` (i.e. midi pitch numbers)
* the lowest and highest pitches that the element will output
* will always be concert (i.e. untransposed) pitches; the pitches you expect to ***hear***
* can combine with interval range...

#### `interval range`

* e.g. like `7-12` meaning between a perfect 5th and an octave
* atm can only express in number-of-semitones
* for a non-variable interval range, just do e.g. `12-12`
* can combine with pitch range...

#### `scale type`

* for lead elements
* NB. generated pitches are corrected for the current chord symbol, so the output may not always be exactly the scale type you specify
* possible values: `major`, `natural minor`, `harmonic minor`, `melodic minor`, `chromatic`, `whole tone`, `major pentatonic`, `minor pentatonic`, `blues (minor)`, `blues (major)`, `diminished (half-whole)`, `diminished (whole-half)`, `ionian`, `dorian`, `phrygian`, `lydian`, `mixolydian`, `aeolian`, `locrian`, `locrian natural 6`, `ionian augmented`, `dorian #11`, `phrygian dominant`, `lydian #2`, `ultra locrian`, `dorian b2`, `lydian augmented`, `lydian dominant`, `mixolydian b6`, `locrian natural 2`, `altered`, `super locrian`, `mixolydian b9 b13`, `mixolydian b13`, `overtone scale`, `lydian b7` 

#### `chord lookahead` 

* `yes` or `no`
* Yes means that notes played at the very end of a bar will anticipate the chord symbol that is about to start, rather than the one that is about to end.

#### Pitch range + interval range

For variation you might want a different range on each run, so that sometimes (for example) your lead is say, in a middle octave, and sometimes it's in a very high octave. (But without having an extremely wide range on any run.)

You can do this by specifying an 'overall' pitch range and an interval range like this:
```
pitch range: C4-G7
interval range: 12-15
```
...Which means choose a random range of between 12 and 15 semitones somewhere between C4 and G7.

## Instrument `.txt` files

A text file to describe/define each instrument. Generally speaking instruments should still work without a .txt file. 

The name of the txt file should correspond with the name of the fxp file, so e.g. `MetallicPluck.fxp` and `MetallicPluck.txt` (and 'characterful' names like that are encouraged, rather than 'SynthBass1.fxp`). 

### VST instrument:

```
vst: massive
tags: smooth, organic, dark
nature: synth
pitched: chromatic
octave: -1
attack: short
sustain: long
```

### Settings list - instrument

#### `vst`

* the VST instrument 
* default is `Massive` - i.e. Native Instruments' Massive synth (which is also the only synth available atm)  

#### `tags`

* comma-separated list of descriptive terms so that Polymuse can make smarter decisions about how/when the instrument is used

#### `nature`

* `synth` or `acoustic` 
* (though 'acoustic' is not very relevant atm, until we can use VST sample instruments...)

#### `pitched`

* `chromatic` or `unpitched`
* `unpitched` means no perceivable 'main' pitch (such as a percussive sound, or noise)

#### `octave`

* an integer like `-1` or `1` (default 0)
* the 'sounding octave'
* e.g. if when you press key C4 the **main** pitch you perceive is C3 then this value should be -1

If (for example) you have a bass instrument set up with -12 on the primary oscillator(s) and you _don't_ set this value to -1, then that bass instrument will probably sound way too low. (If you DO want to hear a really low bass then you should specify an appropriate `pitch range` in `_element.txt`). 

#### `attack`

* `short` or `medium` or `long` - todo: we should pin down precisely what these mean!
* how quickly the 'full' sound comes after a note is triggered

#### `sustain`

* `short` or `medium` or `long` - todo: we should pin down precisely what these mean!
* how quickly the note dies away (even though the note is still on)
* e.g. pads are `long`


## `_midi.txt`

Atm all midi files must be the same length, and they must be a whole number of bars. It's not always possible to infer the duration of the DAW clip that the midi file was created from, so please specify in `_midi.txt`:

```
num bars: 4
```
