[Version0.1](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%200.1.html)
[Version1.0](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.0.html)
[Version1.1](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.1.html)
[Version1.2](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.2.html)
[Version1.3](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.3.html)
[Version1.4](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.4.html)
[Version1.5](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.5.html)
[Version1.6](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.6.html)
[Version1.7](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.7.html)
[Version1.8](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%201.8.html)
[Version2.0](https://reperium.github.io/Morse-Code-Straight-Key-Simulator/Previous%20Versions/Version%202.0.html)

[FigmaInterface/Prototype](https://www.figma.com/design/6LZveNoewl4CwfvIkai4Yy/Morse-Code?node-id=0-1&t=7jPWGRvZy7lyZdF2-1)
⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯
# Morse Code Straight Key Simulator

A browser-based Morse code simulator replicating a mechanical straight key. Self-contained single HTML file — no build step, no dependencies, no server. Open the file and it works offline.

## Core Function

The simulator takes keyboard, mouse, or touch input and converts key-down/key-up durations into Morse code symbols. A key press shorter than two units produces a dot (`·`). A press of two units or longer produces a dash (`—`). The threshold is `unitDuration × 2`, where `unitDuration` is derived from the WPM setting and the selected word standard (PARIS = 1200/WPM ms per unit, CODEX = 1000/WPM ms per unit).

Decoded symbols accumulate into the current letter buffer. When the key is released for three units, a letter boundary is detected. At seven units, a word boundary is inserted. The decoded text appears in the Decoding box; the raw Morse appears in the Morse Code box.

## Input

- **Morse key**: Spacebar, mouse-click on the key bar, or touch on the key bar. All three call the same `handleKeyDown` / `handleKeyUp` pipeline.
- **Delete Letter**: Backspace. Removes the last symbol from the current letter, or commits the letter and removes the last decoded character.
- **Delete Message**: Shift+Backspace. Wipes the entire Morse sequence and decoded text.
- **Broadcast**: Enter. Plays the full Morse sequence as scheduled audio tones with a green highlight tracking each symbol. Supports pause/resume and live-restart when waveform, frequency, volume, or speed change mid-playback.
- **Arrow keys**: Navigate between dictionary language cards (left/right) and scroll within a card (up/down).

Key function assignments are configurable via the Keybinds panel. Any key can be mapped to any of the four functions, with conflict detection preventing duplicate assignments.

## Audio

A single `AudioContext` generates tones via `OscillatorNode` + `GainNode` pairs. Four waveforms are available: sine, square, triangle, sawtooth. Each tone has an attack ramp (0–10ms) and decay ramp (0–10ms) to prevent clicks. A `BiquadFilterNode` lowpass filter is applied to the sawtooth waveform to soften its harmonics.

The `_scheduleTone` helper builds one oscillator per Morse symbol. `buildMorseBroadcastEvents` precomputes the full event timeline (tone events + gap events) so pause/resume and live-restart can re-schedule from any position without re-parsing.

## Timing Standards

Two WPM standards are supported:

- **PARIS**: 50 units per word. `unitMs = 1200 / WPM`. The word "PARIS" is the standard reference — it contains exactly 50 units including inter-character and inter-word gaps.
- **CODEX**: 60 units per word. `unitMs = 1000 / WPM`. Used by some services; produces slightly faster timing at the same WPM number.

The `wordStandard` variable drives every timing computation through `computeUnitMs()` and `stdFactorFor()`. Changing the standard mid-playback triggers `recomputeElapsed()` which converts the elapsed wall-time to the new unit basis so playback continues from the same Morse position without skipping or replaying.

## Visualizer

The straight-key bar is divided into four segments (1 dot unit each). The space-key bar has eight segments (covering 7 word-gap units). Each segment fills proportionally as the key is held — `updateKeySegments` runs on `requestAnimationFrame`, computing `elapsedUnits = (now - keyDownTime) / unitDuration` and setting each segment's width to `clamp(elapsedUnits - index, 0, 1) × 100%`.

Segment fill colors are red, orange, yellow, green — so the user sees a rainbow fill as they hold the key longer. A dot fills one segment; a dash fills three. The space bar fills eight segments over a seven-unit word gap.

## Dictionary

Ten language dictionaries are included: Gerke, International, Spanish, German, Cyrillic, Japanese (Kana), Greek, Hebrew, Arabic, Korean (Jamos). Each dictionary contains letters, numbers, punctuation, prosigns, and Q-codes. Japanese includes dakuten, handakuten, and yōon digraphs. Korean includes lead, vowel, and trail jamos plus compound variants.

Cards render virtually — only the active card has full DOM; adjacent cards are lightweight placeholders that render on scroll. This keeps the DOM small despite 10 dictionaries × 50+ entries each.

Each entry shows the character, its Morse code, and timing columns (units, milliseconds, frames) computed from the current WPM. An error column displays the user's average deviation percentage for that character based on recorded practice attempts.

## Error Tracking

Every key-down/key-up cycle is recorded. The `recordDictSymbolAttempt` function classifies each symbol (dot or dash) by comparing the key-down duration to the 2-unit threshold, then computes the error percentage relative to the ideal duration. When a letter boundary is detected, `finalizeDictCharAttempt` aggregates the per-symbol data into a per-character attempt record stored in `dictErrorDetailHistory[langCode][char]`.

The error detail popover shows each attempt with per-element breakdown (dot, dash, intra-character gap, preceding letter/word gap). Each element shows its actual duration, ideal duration, and signed error percentage. Attempts are sortable by newest-first or oldest-first.

A colored indicator box appears next to each dictionary character. Its color reflects the absolute deviation magnitude (green = low, yellow = medium, red = high). A ranking number inside the box shows the character's rank by deviation — 1 is the highest-deviation character, N is the lowest. This lets the user see which characters need the most practice.

## Input Metrics

Five sub-panels track timing statistics:

- **Key Pressed Duration**: The last key-down duration in units, ms, and frames, compared against the ideal (1 unit for dot, 3 for dash).
- **Space Duration**: The last inter-symbol gap, classified as unit (1u), letter (3u), or word (7u) space.
- **Input Quantity**: Count and percentage of each symbol type keyed.
- **Average Durations**: Running averages for dot, dash, unit space, letter space, and word space.
- **Fist Accuracy**: Ratio of each average to the dot average, divided by the ideal ratio. 100% = perfect. Below 100% = too short. Above 100% = too long.

## Notes Section

Each dictionary card has a Notes section with two fields: a text input and a Morse input. The text field accepts plain text and converts it to Morse in real-time. The Morse field accepts Morse code (dots, dashes, slashes for word breaks) and converts it to text. Both fields support prosigns via the combining overline character (U+0305).

Notes are persisted per-language in localStorage. The Morse field supports audio playback with green highlighting tracking each symbol, matching the Broadcast playback behavior.

## Theme

Three-way theme toggle: auto (follows OS), light, dark. An inline `<head>` script resolves the theme before first paint to prevent a flash of the wrong theme. The theme toggle cycles through auto → light → dark → auto. In auto mode, a `matchMedia` listener re-applies the theme when the OS preference changes.

## Persistence

All state is saved to localStorage under `Morse_sim_state_v1`: broadcast history (capped at 200 entries), per-character error history (capped at 100 attempts per character), per-character error detail history (capped at 50 entries), custom dictionary rows, UI settings (WPM, frequency, volume, waveform, case, text style, error symbol, keybinds), and per-language notes. An Export/Import Save File feature serializes the entire state as a base64 string for backup or transfer.

## Localization

The interface is translated into 16 languages: English, Spanish, French, German, Japanese, Chinese, Arabic, Russian, Portuguese, Italian, Korean, Hebrew, Greek, Hindi, Dutch, Swedish. A language modal appears on load. Q-code and prosign descriptions are also translated. Missing translations fall back to English.

## PWA

A service worker is registered via a Blob URL for offline caching. The app is installable on Chrome/Edge via a Web App Manifest (also a Blob URL). Apple-specific meta tags enable Add to Home Screen on iOS.
