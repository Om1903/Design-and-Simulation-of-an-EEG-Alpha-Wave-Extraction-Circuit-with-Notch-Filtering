# Design-and-Simulation-of-an-EEG-Alpha-Wave-Extraction-Circuit-with-Notch-Filtering


##  Team: Ohm Shanti Ohm
**Competition:** Analogverse Competition  




##  Project Overview
[cite_start]Electroencephalography (EEG) measures brain activity by recording weak electrical microvolt-level signals ($10–100\ \mu\text{V}$) from the scalp[cite: 5, 6]. [cite_start]Within the EEG spectrum, **Alpha rhythms (8–12 Hz)** serve as vital biomarkers for relaxation, cognitive load, and neural tracking[cite: 6, 33]. 

[cite_start]However, acquiring a clean EEG signal poses significant hurdles due to heavy interferences, including[cite: 7, 8]:
* [cite_start]High half-cell DC offsets from skin-electrode interfaces ($\sim 300\text{ mV}$)[cite: 8, 119].
* [cite_start]50 Hz power line/mains hum (orders of magnitude larger than the neural signal)[cite: 8, 66].
* [cite_start]Low-frequency electrode motion artifacts ($< 1\text{ Hz}$)[cite: 8, 299].
* [cite_start]High-frequency muscle noise (EMG) and radio-frequency interference (RFI)[cite: 8, 45].

[cite_start]This project presents a robust, multi-stage **Analog Front-End (AFE)** developed in **LTspice**[cite: 90, 117]. [cite_start]The circuit provides precision amplification and isolates the 8–12 Hz alpha window while attenuating out-of-band artifacts by at least **-40 dB** and providing extreme rejection of the 50 Hz power line hum[cite: 11, 139, 140].

---

## 🛠 System Architecture
[cite_start]The processing pipeline implements a strictly structured modular chain to extract the brainwaves linearly without saturating individual gain blocks[cite: 91, 142].

<img width="861" height="488" alt="image" src="https://github.com/user-attachments/assets/9f826a78-42c0-4f56-85a2-69cb2e202086" />
 

---

## 🔬 Hardware Stage Specifications

### 1. Instrumentation Amplifier (In-Amp) Stage
* **Function:** Extracts microvolt differential inputs from raw electrodes, isolates weak biological signals, and offers huge input impedance to prevent signal degradation[cite: 13, 14].
* **Rejection Mechanism:** Includes an active **Driven Right Leg (DRL) circuit** to actively sense and counteract the common-mode potential of the human body[cite: 18, 206].
* **Gain Equation:** $$\text{Gain } (G) = 1 + \frac{2R}{R_G}$$  
  *(Where $R_G$ sets the primary stage gain to $21\times$, magnifying microvolts safely without hitting voltage rail limits)[cite: 14, 103, 120].*

<img width="935" height="699" alt="image" src="https://github.com/user-attachments/assets/25217dc9-3923-4560-a1d9-76baf7e61b6f" />




### 2. Bandpass Filter Design & Topology Optimization
Isolation of the 8–12 Hz alpha range requires an optimized cascading layout of a High-Pass Filter (HPF) and a Low-Pass Filter (LPF)[cite: 33, 35]. 

#### Topology Analysis: Cascaded 2nd-Order vs. Higher-Order Single Stages
To maintain phase stability, reduce tuning complexity, and mitigate extreme sensitivity to component tolerances (where a 1% capacitor variation can alter corner frequencies by up to 15%), a **Modular Cascaded 2nd-Order block topology** was selected[cite: 41, 55].

| Feature Parameter | 1st + 3rd Order Cascade | Cascaded 2nd + 2nd Order (Chosen) |
| :--- | :---: | :---: |
| **Passband Flatness** | Moderate | **Excellent** |
| **Stopband Attenuation** | Good | **Excellent** |
| **Circuit Stability** | Moderate | **High** |
| **Component Sensitivity** | High | **Lower** |
| **Ease of Tuning** | Difficult | **Easy** |

#### A. High-Pass Filter (HPF)
* **Configuration:** 4th-Order Cascaded Sallen-Key HPF (Two 2nd-order stages)[cite: 303, 304].
* **Specification:** Target cutoff at $f_{HP} \approx 8\text{ Hz}$ with a steep **80 dB/decade roll-off**[cite: 42, 43].
* **Purpose:** Blocks baseline wander and isolates electrode half-cell DC offsets up to 300mV[cite: 36, 42].

#### B. Low-Pass Filter (LPF)
* **Configuration:** 4th-Order Cascaded active Butterworth LPF[cite: 52, 53].
* **Specification:** Cutoff corner calibrated at $12\text{ Hz}$[cite: 45].
* **Why Butterworth?** Chosen over Chebyshev to secure a maximally flat passband response with absolutely **zero gain peaking**, ensuring alpha waves aren't artificially distorted before software digitization[cite: 47, 51, 53].

<img width="883" height="436" alt="image" src="https://github.com/user-attachments/assets/07ef3b9c-0e8a-4346-875c-8942bd1bba06" />



---

### 3. Active Twin-T 50 Hz Notch Filter Stage
Mains power line interference (50 Hz hum) can completely mask cortical signals[cite: 66]. Standard passive notch filters introduce low Quality Factors ($Q$), resulting in unintended attenuation of the crucial 12 Hz alpha boundary[cite: 70]. 

* **Solution:** An **Active Twin-T Notch Filter** using high-impedance op-amp feedback to sharpen the notch selectively without attenuating neighboring neural frequencies[cite: 71, 75].
* **Mathematical Tuning:** $$f_0 = \frac{1}{2\pi RC} = 50\text{ Hz}$$
* **Performance:** Reaches an exceptional narrow-band attenuation of **$> -40\text{ dB}$ at exactly 50 Hz**[cite: 139, 140].

<img width="684" height="487" alt="image" src="https://github.com/user-attachments/assets/0d20f9fc-4b94-4ebd-9dc1-267ac91bc175" />
 


### 4. Output Biasing and Level Shifting Stage
Because typical biomedical signals oscillate symmetrically around a 0V AC baseline, they cannot be read directly by unipolar Analog-to-Digital Converters (ADCs) found on standard microcontrollers (e.g., 0V to 3.3V ESP32/Arduino chips)[cite: 82].

* **Function:** A summing amplifier/buffered voltage divider shifts the clean signal up to a steady reference point[cite: 84]:
  $$V_{bias} = \frac{V_{ref}}{2} = \frac{3.3\text{ V}}{2} = 1.65\text{ V}$$
* **Optimization:** Centers the dynamic range to handle a maximum peak-to-peak signal swing of 3.3V, leveraging the maximum bit-depth of a 12-bit ADC (mapping 0V brainwaves directly to the decimal midpoint value of 2048)[cite: 85, 86].

---

## 📈 Circuit Simulation & Node-by-Node Tracking
The continuous voltage progression through each processing junction (simulating a worst-case scenario input containing **$50\ \mu\text{V}$ Alpha Wave + 200mV DC Offset + 500mV Power Line Hum**) highlights the signal conditioning timeline[cite: 102, 103]:

| Node | Circuit Stage | Section Gain | Signal Voltage ($V_{pp}$) | Processing / Artifact Status |
| :---: | :--- | :---: | :---: | :--- |
| **0** | Raw Input Source | — | $50\ \mu\text{V}$ | 200mV DC offset + 500mV Mains Hum present[cite: 103]. |
| **1** | Instrumentation Amp | $21\times$ | $1.05\text{ mV}$ | Common-mode rejected; DC offset amplified safely to 4.2V[cite: 103, 120]. |
| **2** | High-Pass Filter 1 | $1.58\times$ | $1.66\text{ mV}$ | Heavy electrode half-cell DC component completely blocked[cite: 103, 121]. |
| **3** | High-Pass Filter 2 | $1.58\times$ | $2.62\text{ mV}$ | Low-frequency baseline motion wander stripped ($<1\text{ Hz}$)[cite: 103, 299]. |
| **4** | Active Twin-T Notch | $1\times$ | $2.62\text{ mV}$ | 50 Hz power line hum aggressively attenuated ($> -40\text{ dB}$)[cite: 103, 140]. |
| **5** | Low-Pass Filter 1 | $1.58\times$ | $4.14\text{ mV}$ | High-frequency muscle noise (EMG) attenuated[cite: 103]. |
| **6** | Low-Pass Filter 2 | $1.58\times$ | $6.54\text{ mV}$ | Sharp 4th-order roll-off active above the 12 Hz corner[cite: 103]. |
| **7** | Final Gain & Bias Stage | $10\times$ | **$\approx 65\text{ mV}$** | Signal amplified, centered cleanly at 1.65V DC for ADC[cite: 103]. |

<img width="1417" height="835" alt="image" src="https://github.com/user-attachments/assets/e9afd2e8-1ab6-4ba3-8dd3-7289ee223ef1" />


## 🔬 Test Results & Validation Matrices

### Phase 1: Ideal Performance Validation
Simulations validated the filter chain under ideal frequency environments[cite: 117].

* **Passband Sensitivity:** The targeted 10 Hz alpha wave achieved full operational gain without passband ripple distortion[cite: 106, 108].
* **Stopband Rejection:** Sharp roll-offs successfully isolated 2 Hz motion drift and high-frequency noise[cite: 109, 110, 111].

<img width="790" height="428" alt="image" src="https://github.com/user-attachments/assets/3cd29a18-d962-49ca-a0b0-ab1ec459fbe2" />


### Phase 2: Stress Testing in Real-World Environments
To simulate strict operational environments, high-stress input vectors were introduced[cite: 117, 129].

| Test Vector (Input Stress) | Design Layer Response | Project Verification Status |
| :--- | :--- | :---: |
| **300mV Constant DC Offset** | Series caps in HPF blocks isolate offset safely to 0V[cite: 130]. | **PASS**  |
| **500mV @ 50Hz Common-Mode** | Instrumentation Amplifier CMRR matching + Active Notch nulling[cite: 130]. | **PASS**  |
| **1 Hz Baseline Wander** | 80 dB/decade high-pass roll-off prevents sensor drift[cite: 130]. | **PASS**  |
| **100 Hz High Muscle Noise** | 4th-Order Butterworth LPF pushes noise to the floor[cite: 130]. | **PASS** |

### Phase 3: The Ultimate Real-World Scenario
A final comprehensive test combined all physiological, environmental, and DC noise sources into a single input stream[cite: 131]. Even with an extreme noise-to-signal ratio of **10,000 : 1**, the AFE successfully isolated the raw 10 Hz alpha brainwave[cite: 133, 136].

<img width="1501" height="697" alt="image" src="https://github.com/user-attachments/assets/7004e174-59ab-49b7-92e8-64a30f60332b" />

