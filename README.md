# 🧠 HMS Harmful Brain Activity Classification — Data Analysis

This repository contains the **data-centric pipeline** for the [HMS – Harmful Brain Activity Classification](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification) competition.  
The main focus here is **data preparation**: EEG preprocessing, spectrogram generation, quality control, and label handling before training.

---

## 📌 Project Scope
The **data-centric** part of the project covers:

1. **EEG Preprocessing**  
   - Bandpass filtering (0.5–20 Hz) + amplitude clipping.  
   - Removing EKG channel.  
   - Applying the **double-banana montage** → differential channels (LL, LP, RL, RP).  
   - Windowing EEG signals (e.g., 10s @ 200 Hz → 2000 samples).  

2. **Spectrogram Generation**  
   - Continuous Wavelet Transform (CWT).  
   - Normalization: `log1p` + robust scaling (1–99th percentile → range 0–1).  
   - Saved to `spec_hr_out/` as `.npz` files (`x`, `freqs`, `eeg_id`, `patient_id`).  

3. **Waveform Export (HR)**  
   - Processed EEG waveforms after montage.  
   - Saved to `wave_hr_out/` in `.npz` format with keys:  
     - `x` (shape: 4×2000)  
     - `fs`, `eeg_id`, `patient_id`.  

4. **Label Handling**  
   - Original dataset provides expert votes (`*_vote` for seizure, lpd, gpd, lrda, grda, other).  
   - Rebuilt into:  
     - `y_soft` → **soft labels** (distribution of votes).  
     - `w_conf` → **confidence weight** (function of vote count, e.g., linear/log/sqrt).  
     - `hc_mask` → boolean mask for **high-confidence** samples (≥10 votes).  
   - Saved as `labels.npz`.  

5. **Data Splits**  
   - **GroupKFold** (by patient) to ensure no patient overlap across folds.  
   - Saved as `splits.npz`.  

6. **Quality Control (QC)**  
   - Checked NaN/Inf values in spectrograms and waveforms.  
   - Inspected vote distribution (`vote_sum`), class proportions, and imbalance.  
   - Found imbalance (e.g., minority class = seizure).  
