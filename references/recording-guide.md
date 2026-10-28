# Recording guide for noise-nuisance samples

Give the user the parts that apply. The goal is a recording whose spectrum reflects the
noise, not the phone's speech processing.

## Device and app

- **Android: ASR Voice Recorder (by NLL).** In its recording settings, set the audio
  source to **Unprocessed** where the phone offers it (Android's raw input, no noise
  suppression, AGC or speech EQ). Fall back to *Voice recognition* if Unprocessed is
  missing, as it is usually less processed than the default *Mic*. Set the format to
  **WAV** (or FLAC), **48 kHz**, and turn off any skip-silence, gain or noise-reduction
  options.
- **iPhone:** Voice Memos applies processing and records AAC. Use a recorder that
  writes uncompressed WAV and offers a raw or measurement mode, and turn off
  voice-isolation or enhancement options.
- **Dedicated recorder (Zoom, Tascam and similar):** WAV, 48 kHz or more, low-cut filter
  OFF, limiter/AGC OFF, fixed manual gain.
- **Measurement mic (e.g. USB measurement mic with a calibration file):** best option
  for low-frequency work; note the calibration file in `devices.md`.

Phone microphones roll off steeply below roughly 50-100 Hz, and many apps also high-pass
filter. A phone recording can **under-represent rumble and drone**. Treat a weak
low-frequency result from a phone as "not shown", not as "absent".

## Protocol

1. **Turn off your own noise sources** you can: fridge, AC, fans, dehumidifier, computer.
   Note any that stay on.
2. **Fixed position.** Put the device down on a soft surface (a folded towel), not in
   the hand. Hand handling adds thumps. Note the room and position.
3. **No talking, no movement** during the take. Speak a note before or after, not
   during, or keep notes in `sample.md`.
4. **Length:** at least 60 s; 2-5 min for cycling noise (pumps, compressors) so several
   cycles are captured.
5. **Background reference.** If the nuisance stops at times, record the same spot, same
   settings, when it is absent. This is the single most useful extra: the analysis then
   shows exactly which bands the nuisance adds.
6. **Fixed gain.** Don't change gain between nuisance and background takes; otherwise
   the comparison is meaningless.
7. **Doors and windows:** record their state. Louder with windows closed points to
   structure-borne low frequency; louder with them open points to airborne.
8. **Multiple rooms** help locate the path the noise takes in.

## What not to do

- No phone calls, video-call apps or "voice" modes: they are built to delete steady noise.
- No MP3 or low-bitrate AAC if avoidable.
- Don't normalise, denoise or edit the file before analysis. Keep the original.
