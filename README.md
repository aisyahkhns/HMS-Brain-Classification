# 🧠 HMS Harmful Brain Activity Classification — Data Analysis

This repository contains the **data-centric pipeline** for the [HMS – Harmful Brain Activity Classification](https://www.kaggle.com/competitions/hms-harmful-brain-activity-classification) competition.  
The main focus here is **data preparation**: EEG preprocessing, spectrogram generation, quality control, and label handling before training.

---

## 📌 Project Scope
The **data-centric** part of the project covers:
<img width="1920" height="1080" alt="1" src="https://github.com/user-attachments/assets/d280f584-e840-4201-8e9f-d9e3c5803c96" />

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

## 📂 Files to be used

### 1. `data_package/`
- **`meta_use.csv`** → metadata (eeg_id, patient_id, etc.)
- **`labels.npz`** → processed labels:
  - `y_soft` (soft labels, probability distribution across classes)  
  - `w_conf` (confidence weights from vote counts)  
  - `hc_mask` (high-confidence sample mask)  
  - `classes` (class names: seizure, lpd, gpd, lrda, grda, other)  
- **`splits.npz`** → 5-fold train/valid indices (GroupKFold per patient)

### 2. `spec_hr_out/`
- All processed **spectrograms** per EEG  
- Format: `<eeg_id>_hr.npz` containing:
  - `x`: spectrogram array [C=4, F, T] (0–1 normalized)  
  - `freqs`: frequency bins  
  - `eeg_id`, `patient_id`  

### 3. `wave_hr_out/` *(optional)*
- All processed **waveforms** per EEG  
- Format: `<eeg_id>_hr.npz` containing:
  - `x`: waveform [C=4, T]  
  - `fs`: sampling rate (Hz)  
  - `eeg_id`, `patient_id`  

---

## 📘 Documentation

### 1. Data Preparation
- Input: original `train.csv`, `train_eegs/`, `train_spectrograms/`  
- Preprocessing applied:
  1. **Bandpass filtering** (0.5–20 Hz) + clipping  
  2. **Montage** (Double Banana: LL, LP, RL, RP)  
  3. **Normalization** (log1p + percentile scaling for spectrograms)  
  4. **Windowing** (10s HR segments)  
- Outputs stored in: `spec_hr_out/`, `wave_hr_out/`

### 2. Labels
- Raw expert votes are rebuilt into **soft labels** (probability distribution).  
- Confidence weights (`w_conf`) scale sample contribution during training.  
- High-confidence samples (`hc_mask`) used for Stage-1 training.

### 3. Splits
- Patient-wise **GroupKFold** (5 folds, no patient overlap).  
- Saved in `data_package/splits.npz` with keys: `train_0…4`, `valid_0…4`.

### 4. Training
- **Default input**: spectrograms (`spec_hr_out/`)  
- **Optional branch**: waveforms (`wave_hr_out/`)  
- Data loading via `SpecDataset` (supports augmentation, confidence weighting).  
