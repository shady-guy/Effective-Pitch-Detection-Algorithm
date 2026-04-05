# Robust Pitch Detection in Noisy Environments

## Overview
This project implements a robust pitch detection algorithm designed to accurately track the fundamental frequency of human speech and singing, even in environments with heavy background noise. It is based on a 2008 academic approach that combines digital signal processing techniques to outperform standard, baseline pitch trackers.

## Problem Statement
Pitch detection is the process of identifying the "musical note" or frequency of a sound. While traditional algorithms (like raw Autocorrelation) work well in quiet environments, they fall short in the noisy real world. 

When exposed to constant background noise (like a running air conditioner, computer fan, or static), basic algorithms get "confused." They often track the frequency of the noise instead of the voice, resulting in jagged, jittery pitch lines, or they suffer from "Octave Errors"—mathematical glitches where the algorithm guesses a pitch that is exactly double or half the actual pitch. This project implements a specialized algorithm to solve this exact problem.

## Methodology
To ensure the algorithm only tracks the human voice, it processes the audio through four distinct phases. Here is how it works:

1. **Noise Erasing (Improved Spectral Subtraction):** 
  The algorithm requires a short buffer (0.1 seconds) of silence before the speaker begins talking. It uses this buffer to learn the exact acoustic profile of the room's background noise. Once the speaker starts, it mathematically subtracts that specific noise profile from the entire recording. The "improved" part of the name comes from one of the parameters used (beta = 5) which results in aggressive subtraction.
2. **Throat Filter (Linear Predictive Coding - LPC):**
   Human speech is a combination of vocal cord vibration (the pitch) and the shape of the mouth (the vowel). The algorithm uses an LPC filter to effectively "ignore" the shape of the mouth, leaving behind only the raw, pure pulses of the vocal cords.
3. **Finding Patterns (Cepstrum & Autocorrelation):**
   Once the raw vocal cord pulses are isolated, the algorithm compares the audio wave to itself to see how long it takes for the wave to repeat. The time it takes to repeat determines the exact pitch in Hertz (Hz).
4. **The Smooth Operator (Post-Processing):**
   Finally, the algorithm checks its own work. If it spots a sudden, anomalous jump in pitch, it recognizes it as a mathematical glitch and smooths it out using the surrounding, correct notes. Also, if any half frequency or double frequency errors are detected, they get corrected.

## Testing and Comparison
To prove the effectiveness of this method, its results were tested against a Baseline algorithm (Standard Autocorrelation with an energy gate).

* **The Input:** A recording of a human male speaking in a noisy environment.
* **The Goal:** See which algorithm could successfully track the rising scale without getting distracted by the static.

## Explanation of Results

The advanced Paper Method vastly outperformed the Baseline algorithm across all metrics.

| Metric | Paper Method (Advanced) | Baseline Method (Control) |
| :--- | :--- | :--- |
| **Mean Pitch** | $184.2\text{ Hz}$ | $167.6\text{ Hz}$ |
| **Standard Deviation** | $27.5$ | $74.1$ |
| **Voiced Frames Detected** | $508 / 509$ | $489 / 509$ |
| **Half Errors Errors (<120 Hz)** | $15$ | $118$ |
| **Double Errors (>300 Hz)** | $0$ | $41$ |

### What these numbers mean:
1. **Stability (Standard Deviation):** The Baseline algorithm jumped wildly between the vocal pitch and the noise, resulting in a massive standard deviation of $74.1$. The Paper method ignored the noise relatively well, providing a smooth, accurate pitch line with a deviation of only $27.5$ (a $63\%$ % improvement).
2. **Elimination of Octave Errors:** The most glaring failure of the Baseline method was that it was tricked by high-pitched static and low pitched hums, guessing a pitch above 300 Hz a total of 41 times and a pitch of below 120 Hz a total of 118 times. Because the Paper method uses LPC filtering and smart post-processing, it reduced the "Double Errors" to 0 ($100\%$ % improvement) and "Half Errors" to 15 ($87\%$ % improvement)
3. **Reliability:** In heavy noise, the Baseline completely lost the voice for 20 frames, classifying them as unvoiced. The Paper method successfully tracked the voice for 508 out of 509 frames.
4. **Mean:** The mean pitch is roughly equal as expected. But lower in the Baseline method, possibly due to significant "Half Errors"

**Conclusion:** While computationally heavier, the combination of Spectral Subtraction and LPC inverse filtering is superior for accurate pitch detection in general noisy environments. 
