# Design and Simulation of an EEG Alpha Wave Extraction Circuit with Notch Filtering

##  Team: Ohm Shanti Ohm
**Competition:** Analogverse Competition  


---

##  Project Overview
Electroencephalography (EEG) records non-invasive microvolt-level electrical potential fluctuations ($10–100\ \mu\text{V}$) originating from postsynaptic dendritic currents in the cerebral cortex. Within the standard neural spectral bands, **Alpha waves (8–12 Hz)** serve as prominent biomarkers tightly coupled with states of conscious relaxation, attentional modulation, and cognitive workload distribution. 

Capturing these fragile biomarkers directly from the scalp presents an extreme signal integrity challenge. The physiological microvolts are routinely buried under layers of severe, destructive interference:
* **Electrode Polarization:** Half-cell DC offset potentials ($\sim 300\text{ mV}$) generated at the skin-electrolyte interface can instantly saturate high-gain amplifiers.
* **Mains Interference:** 50 Hz power line hum couples electrostatically into the human body, introducing noise several orders of magnitude larger than the target neural signals.
* **Environmental & Biomechanical Artifacts:** Ultra-low frequency baseline wander ($< 1\text{ Hz}$) caused by subject respiration or electrode movement, alongside high-frequency electromyographic (EMG) muscle noise.

To overcome these constraints, this project details the design, optimization, and validation of a high-precision, multi-stage **Analog Front-End (AFE)** developed in **LTspice**. The finalized architecture provides robust differential amplification and ultra-narrow isolation of the 8–12 Hz alpha window—achieving an exceptional out-of-band attenuation profile exceeding **-40 dB** alongside aggressive, narrow-band rejection of the 50 Hz power line hum without distorting nearby brainwave components.

---

##  System Architecture
The processing pipeline implements a strictly structured modular chain to extract the brainwaves linearly without saturating individual gain blocks.

<img width="861" height="488" alt="image" src="https://github.com/user-attachments/assets/9f826a78-42c0-4f56-85a2-69cb2e202086" />
 

---

##  Hardware Stage Specifications

### 1. Instrumentation Amplifier (In-Amp) Stage
* **Function:** Extracts microvolt differential inputs from raw electrodes, isolates weak biological signals, and offers huge input impedance to prevent signal degradation.
* **Rejection Mechanism:** Includes an active **Driven Right Leg (DRL) circuit** to actively sense and counteract the common-mode potential of the human body.
* **Gain Equation:** $$\text{Gain } (G) = 1 + \frac{2R}{R_G}$$  
  *(Where $R_G$ sets the primary stage gain to $21\times$, magnifying microvolts safely without hitting voltage rail limits).*

<img width="935" height="699" alt="image" src="https://github.com/user-attachments/assets/25217dc9-3923-4560-a1d9-76baf7e61b6f" />

### 2. Bandpass Filter Design & Topology Optimization
Isolation of the 8–12 Hz alpha range requires an optimized cascading layout of a High-Pass Filter (HPF) and a Low-Pass Filter (LPF). 

#### Topology Analysis: Cascaded 2nd-Order vs. Higher-Order Single Stages
To maintain phase stability, reduce tuning complexity, and mitigate extreme sensitivity to component tolerances (where a 1% capacitor variation can alter corner frequencies by up to 15%), a **Modular Cascaded 2nd-Order block topology** was selected.

| Feature Parameter | 1st + 3rd Order Cascade | Cascaded 2nd + 2nd Order (Chosen) |
| :--- | :---: | :---: |
| **Passband Flatness** | Moderate | **Excellent** |
| **Stopband Attenuation** | Good | **Excellent** |
| **Circuit Stability** | Moderate | **High** |
| **Component Sensitivity** | High | **Lower** |
| **Ease of Tuning** | Difficult | **Easy** |

#### A. High-Pass Filter (HPF)
* **Configuration:** 4th-Order Cascaded Sallen-Key HPF (Two 2nd-order stages).
* **Specification:** Target cutoff at $f_{HP} \approx 8\text{ Hz}$ with a steep **80 dB/decade roll-off**.
* **Purpose:** Blocks baseline wander and isolates electrode half-cell DC offsets up to 300mV.

#### B. Low-Pass Filter (LPF)
* **Configuration:** 4th-Order Cascaded active Butterworth LPF.
* **Specification:** Cutoff corner calibrated at $12\text{ Hz}$.
* **Why Butterworth?** Chosen over Chebyshev to secure a maximally flat passband response with absolutely **zero gain peaking**, ensuring alpha waves aren't artificially distorted before software digitization.

<img width="883" height="436" alt="image" src="https://github.com/user-attachments/assets/07ef3b9c-0e8a-4346-875c-8942bd1bba06" />

---

### 3. Active Twin-T 50 Hz Notch Filter Stage
Mains power line interference (50 Hz hum) can completely mask cortical signals. Standard passive notch filters introduce low Quality Factors ($Q$), resulting in unintended attenuation of the crucial 12 Hz alpha boundary. 

* **Solution:** An **Active Twin-T Notch Filter** using high-impedance op-amp feedback to sharpen the notch selectively without attenuating neighboring neural frequencies.
* **Mathematical Tuning:** $$f_0 = \frac{1}{2\pi RC} = 50\text{ Hz}$$
* **Performance:** Reaches an exceptional narrow-band attenuation of **$> -40\text{ dB}$ at exactly 50 Hz**.

<img width="684" height="487" alt="image" src="https://github.com/user-attachments/assets/0d20f9fc-4b94-4ebd-9dc1-267ac91bc175" />
 

### 4. Output Biasing and Level Shifting Stage
Because typical biomedical signals oscillate symmetrically around a 0V AC baseline, they cannot be read directly by unipolar Analog-to-Digital Converters (ADCs) found on standard microcontrollers (e.g., 0V to 3.3V ESP32/Arduino chips).

* **Function:** A summing amplifier/buffered voltage divider shifts the clean signal up to a steady reference point:
  $$V_{bias} = \frac{V_{ref}}{2} = \frac{3.3\text{ V}}{2} = 1.65\text{ V}$$
* **Optimization:** Centers the dynamic range to handle a maximum peak-to-peak signal swing of 3.3V, leveraging the maximum bit-depth of a 12-bit ADC (mapping 0V brainwaves directly to the decimal midpoint value of 2048).

---

##  Circuit Simulation & Node-by-Node Tracking
The continuous voltage progression through each processing junction (simulating a worst-case scenario input containing **$50\ \mu\text{V}$ Alpha Wave + 200mV DC Offset + 500mV Power Line Hum**) highlights the signal conditioning timeline:

| Node | Circuit Stage | Section Gain | Signal Voltage ($V_{pp}$) | Processing / Artifact Status |
| :---: | :--- | :---: | :---: | :--- |
| **0** | Raw Input Source | — | $50\ \mu\text{V}$ | 200mV DC offset + 500mV Mains Hum present. |
| **1** | Instrumentation Amp | $21\times$ | $1.05\text{ mV}$ | Common-mode rejected; DC offset amplified safely to 4.2V. |
| **2** | High-Pass Filter 1 | $1.58\times$ | $1.66\text{ mV}$ | Heavy electrode half-cell DC component completely blocked. |
| **3** | High-Pass Filter 2 | $1.58\times$ | $2.62\text{ mV}$ | Low-frequency baseline motion wander stripped ($<1\text{ Hz}$). |
| **4** | Active Twin-T Notch | $1\times$ | $2.62\text{ mV}$ | 50 Hz power line hum aggressively attenuated ($> -40\text{ dB}$). |
| **5** | Low-Pass Filter 1 | $1.58\times$ | $4.14\text{ mV}$ | High-frequency muscle noise (EMG) attenuated. |
| **6** | Low-Pass Filter 2 | $1.58\times$ | $6.54\text{ mV}$ | Sharp 4th-order roll-off active above the 12 Hz corner. |
| **7** | Final Gain & Bias Stage | $10\times$ | **$\approx 65\text{ mV}$** | Signal amplified, centered cleanly at 1.65V DC for ADC. |

<img width="1417" height="835" alt="image" src="https://github.com/user-attachments/assets/e9afd2e8-1ab6-4ba3-8dd3-7289ee223ef1" />

---

##  Test Results & Validation Matrices

### Phase 1: Ideal Performance Validation
Simulations validated the filter chain under ideal frequency environments.

* **Passband Sensitivity:** The targeted 10 Hz alpha wave achieved full operational gain without passband ripple distortion.
* **Stopband Rejection:** Sharp roll-offs successfully isolated 2 Hz motion drift and high-frequency noise.

<img width="790" height="428" alt="image" src="https://github.com/user-attachments/assets/3cd29a18-d962-49ca-a0b0-ab1ec459fbe2" />

### Phase 2: Stress Testing in Real-World Environments
To simulate strict operational environments, high-stress input vectors were introduced.

| Test Vector (Input Stress) | Design Layer Response | Project Verification Status |
| :--- | :--- | :---: |
| **300mV Constant DC Offset** | Series caps in HPF blocks isolate offset safely to 0V. | **PASS** |
| **500mV @ 50Hz Common-Mode** | Instrumentation Amplifier CMRR matching + Active Notch nulling. | **PASS**  |
| **1 Hz Baseline Wander** | 80 dB/decade high-pass roll-off prevents sensor drift. | **PASS** |
| **100 Hz High Muscle Noise** | 4th-Order Butterworth LPF pushes noise to the floor. | **PASS** |

### Phase 3: The Ultimate Real-World Scenario
A final comprehensive test combined all physiological, environmental, and DC noise sources into a single input stream. Even with an extreme noise-to-signal ratio of **10,000 : 1**, the AFE successfully isolated the raw 10 Hz alpha brainwave.

<img width="1501" height="697" alt="image" src="https://github.com/user-attachments/assets/7004e174-59ab-49b7-92e8-64a30f60332b" />

---

##  Future Work
1. **PCB Optimization:** Design an optimized, ultra-compact hardware layout using split analog/digital ground planes and shielding enclosures to further guard against localized EMI.
2. **Embedded DSP Integration:** Interface the buffered unipolar analog output with an ESP32 or STM32 MCU to execute real-time discrete Fast Fourier Transforms (FFT) and algorithmic band power mapping.
