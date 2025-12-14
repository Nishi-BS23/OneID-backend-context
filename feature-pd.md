# 🎤 SPEAKING NOTES - Feature Based Approach for Parkinson's Disease Detection
## Detailed Bengali Notes for Each Slide

---

# 📌 SLIDE 1: Feature Based Approach (Title Slide)

## Speaking Notes (বাংলা):

"আসসালামু আলাইকুম / নমস্কার সবাইকে।

আজকে আমি আপনাদের সাথে শেয়ার করবো আমাদের Parkinson's Disease Detection project এর Feature Based Approach নিয়ে।

এই approach এ আমরা voice recordings থেকে acoustic features extract করি এবং সেগুলো দিয়ে Machine Learning models train করি।

মূল concept টা হলো - Parkinson's রোগীদের voice এ কিছু specific changes হয় যেগুলো আমরা mathematical features দিয়ে capture করতে পারি।

চলুন দেখি কিভাবে এই পুরো pipeline কাজ করে..."

---

# 📌 SLIDE 2: Extracted Features & Feature Selection

## Speaking Notes (বাংলা):

"এই slide এ আমি দেখাচ্ছি আমাদের overall feature extraction এবং selection pipeline।

**Features কি?**
Features হলো voice signal থেকে বের করা numerical values যেগুলো voice এর বিভিন্ন characteristics describe করে।

**আমরা মোট ৫১টি features extract করেছি ৮টি category থেকে:**

1. **Jitter Features (৫টি)** - Voice pitch এর irregularity measure করে
2. **Shimmer Features (৬টি)** - Voice amplitude এর variation measure করে
3. **Harmonic Features (২টি)** - Voice clarity measure করে - কতটা noise আছে
4. **Pitch Features (৩টি)** - Fundamental frequency related measures
5. **Nonlinear Features (৬টি)** - Voice signal এর complexity measure করে
6. **Time-Domain Features (২টি)** - Energy এবং zero-crossing
7. **Spectral Features (১টি)** - Frequency distribution uniformity
8. **MFCCs (২৬টি)** - Human ear যেভাবে শোনে সেভাবে voice represent করে

**কেন Feature Selection দরকার?**
৫১টি feature অনেক বেশি! সব features useful না। কিছু features redundant, কিছু noise add করে। তাই আমরা ৯টি different methods দিয়ে best features select করেছি।

**Final Result:** ৫১ থেকে ২৪টি features select করেছি - ৫৩% reduction!"

---

# 📌 SLIDE 3: Datasets

## Speaking Notes (বাংলা):

"এবার আসি Dataset নিয়ে।

**Dataset: IPVS (Italian Parkinson's Voice and Speech) Corpus**

এটা একটা real clinical dataset - Italy র hospitals থেকে collect করা।

**Dataset এর details:**
- Total ৩৯৮টি audio samples
- দুইটা class: HC (Healthy Control) আর PD (Parkinson's Disease)
- Sampling rate: 16kHz
- Duration: 10 seconds per sample

**Data Split - এটা অনেক important!**

আমরা Patient-wise split করেছি:
- **Train:** ৩১২ samples (HC: ১৫৯, PD: ১৫৩)
- **Validation:** ৪৪ samples (HC: ২১, PD: ২৩)
- **Test:** ৪২ samples (HC: ২০, PD: ২২)

**কেন Patient-wise split?**

এটা critically important! ধরুন একজন patient এর ৫টা recording আছে। যদি আমরা randomly split করি, তাহলে হতে পারে ৩টা train এ আর ২টা test এ চলে গেলো। এতে model সেই patient এর voice 'মনে রাখতে' পারে এবং test এ cheating হয়ে যায়।

Patient-wise split এ একজন patient এর সব recordings একই set এ থাকে। এতে real-world performance accurately measure হয় - model নতুন patient দেখলে কেমন perform করবে সেটা বোঝা যায়।

এটাকে বলে 'No Data Leakage' - অনেক research paper এই mistake করে!"

---

# 📌 SLIDE 4: JITTER FEATURES (5)

## Speaking Notes (বাংলা):

"এবার আসি প্রথম feature category - Jitter।

**Jitter কি?**

আপনি যখন 'আআআআ' বলেন, আপনার vocal cords vibrate করে একটা certain frequency তে। কিন্তু প্রতিটা cycle exactly same হয় না - সামান্য variation থাকে। এই cycle-to-cycle frequency variation কে বলে Jitter।

**সহজ ভাষায়:** Jitter হলো voice pitch এর 'shakiness' বা 'instability'।

**Parkinson's এ কেন Jitter বাড়ে?**

PD রোগীদের neuromuscular control কমে যায়। তাদের vocal cords ঠিকমতো control হয় না। তাই voice এ বেশি irregularity দেখা যায় - মানে বেশি jitter।

**৫টি Jitter Features:**

1. **MDVP_Jitter(%)** - Percentage এ jitter
2. **MDVP_Jitter(Abs)** - Absolute value microseconds এ
3. **MDVP_RAP** - Relative Average Perturbation (৩-point smoothing)
4. **MDVP_PPQ** - Period Perturbation Quotient (৫-point smoothing)
5. **Jitter_DDP** - Average absolute difference of differences

**Code এ কিভাবে করেছি:**

```python
# Pitch tracking with librosa
f0, voiced_flag, voiced_probs = librosa.pyin(audio, fmin=75, fmax=500)
# Period calculation: T = 1/F0
periods = 1.0 / f0[voiced_flag]
# Jitter calculation
jitter_percent = np.mean(np.abs(np.diff(periods))) / np.mean(periods) * 100
```

**Feature Selection Result:**

🏆 **সব ৫টি Jitter feature consensus এ আছে!** এটা অসাধারণ result!

- MDVP_Jitter_percent: **৭/৯ methods** ✓
- MDVP_Jitter_Abs: **৭/৯ methods** ✓
- MDVP_PPQ: **৭/৯ methods** ✓
- MDVP_RAP: **৬/৯ methods** ✓
- Jitter_DDP: **৬/৯ methods** ✓

এর মানে Jitter features Parkinson's detection এ extremely important - প্রায় সব feature selection methods agree করেছে!"

---

# 📌 SLIDE 5: SHIMMER FEATURES (6)

## Speaking Notes (বাংলা):

"এবার আসি Shimmer নিয়ে।

**Shimmer কি?**

Jitter যেমন frequency variation measure করে, Shimmer তেমনি amplitude variation measure করে। মানে voice এর loudness কতটা stable সেটা দেখে।

**সহজ ভাষায়:** Shimmer হলো voice volume এর 'shakiness'।

**Parkinson's এ কেন Shimmer বাড়ে?**

1. **Reduced breath support** - PD রোগীদের breathing muscle weak হয়ে যায়
2. **Muscle rigidity** - Vocal folds stiff হয়ে যায়
3. **Laryngeal dysfunction** - Voice box ঠিকমতো কাজ করে না

এসব কারণে voice এর volume stable থাকে না।

**৬টি Shimmer Features:**

1. **MDVP_Shimmer(%)** - Percentage এ shimmer
2. **MDVP_Shimmer(dB)** - Decibel scale এ
3. **Shimmer_APQ3** - ৩-point Amplitude Perturbation
4. **Shimmer_APQ5** - ৫-point Amplitude Perturbation
5. **MDVP_APQ** - ১১-point Amplitude Perturbation
6. **Shimmer_DDA** - Average difference of amplitude differences

**Code এ কিভাবে করেছি:**

```python
# Peak amplitude per cycle
amplitudes = []
for i in range(len(periods)):
    start = int(sum(periods[:i]) * sr)
    end = int(sum(periods[:i+1]) * sr)
    amplitudes.append(np.max(np.abs(audio[start:end])))

# Shimmer calculation
shimmer_percent = np.mean(np.abs(np.diff(amplitudes))) / np.mean(amplitudes) * 100
shimmer_db = 20 * np.log10(shimmer_percent/100 + 1)
```

**Feature Selection Result:**

**৬টির মধ্যে ৩টি consensus এ (৫০%):**

- **Shimmer_APQ3:** **৮/৯ methods** ✓ (Top performer!)
- **Shimmer_DDA:** **৮/৯ methods** ✓ (Top performer!)
- **MDVP_Shimmer_dB:** **৬/৯ methods** ✓

এখানে দেখুন APQ3 আর DDA সবচেয়ে discriminative - ৮/৯ methods এ select হয়েছে!"

---

# 📌 SLIDE 6: NONLINEAR FEATURES (6)

## Speaking Notes (বাংলা):

"এবার একটু advanced topic - Nonlinear Features।

**Nonlinear কেন?**

আমাদের voice signal আসলে একটা nonlinear dynamical system থেকে আসে। Vocal cords, airflow, resonance - এগুলো সব complex interactions করে। Traditional linear measures (jitter, shimmer) সব capture করতে পারে না। তাই nonlinear analysis দরকার।

**৬টি Nonlinear Features:**

1. **RPDE (Recurrence Period Density Entropy)**
   - Signal এর recurrence pattern এর entropy
   - কতটা 'unpredictable' signal সেটা measure করে
   
2. **D2 (Correlation Dimension)**
   - Signal এর complexity dimension
   - Higher D2 = more complex signal
   
3. **DFA (Detrended Fluctuation Analysis)**
   - Long-range correlations detect করে
   - Self-similarity measure
   
4. **PPE (Pitch Period Entropy)**
   - Pitch variations এর entropy
   
5. **Spread1 & Spread2**
   - F0 variation এর nonlinear measures

**সহজ ভাষায় বুঝি:**

ধরুন আপনি একটা heart beat signal দেখছেন। Healthy heart এর beat 'beautifully chaotic' - একটু variation থাকে। খুব regular beat আসলে problem indicate করে। Voice ও তেমনি - healthy voice এ natural complexity থাকে।

PD রোগীদের voice এ এই natural complexity change হয়ে যায় - either খুব monotonous হয়ে যায় অথবা abnormally irregular।

**Code Example (RPDE):**

```python
from scipy.signal import find_peaks

def calculate_rpde(signal, m=10, tau=1):
    # Create delay embedding
    N = len(signal) - (m-1)*tau
    embedding = np.zeros((N, m))
    for i in range(m):
        embedding[:, i] = signal[i*tau:i*tau+N]
    
    # Calculate recurrence times
    # ... complex calculation
    
    # Calculate entropy
    rpde = -np.sum(p * np.log(p)) / np.log(len(p))
    return rpde
```

**Feature Selection Result:**

**৬টির মধ্যে মাত্র ১টি consensus এ (১৭%):**

- **RPDE:** **৭/৯ methods** ✓

মজার বিষয় - RPDE alone ৭/৯ methods এ select হয়েছে! এটা বলছে complexity measure সবচেয়ে useful। বাকিগুলো এই dataset এ discriminative না।"

---

# 📌 SLIDE 7: HARMONIC & TIME-DOMAIN FEATURES (4)

## Speaking Notes (বাংলা):

"এবার দুইটা category একসাথে দেখি - Harmonic আর Time-Domain।

## Harmonic Features (২টি):

**Voice এ Harmonics কি?**

আপনি যখন একটা note গান, সেটা শুধু একটা frequency না। Main frequency (fundamental) এর সাথে অনেকগুলো harmonic overtones থাকে। এগুলো voice এর 'quality' বা 'timbre' তৈরি করে।

**NHR (Noise-to-Harmonics Ratio):**
- Voice এ কতটা noise আছে harmonics এর তুলনায়
- বেশি NHR = breathy/hoarse voice
- PD রোগীদের NHR বেশি থাকে

**HNR (Harmonics-to-Noise Ratio):**
- NHR এর উল্টো
- বেশি HNR = cleaner voice
- PD রোগীদের HNR কম থাকে

**Code:**
```python
# Using autocorrelation
def calculate_hnr(audio, sr):
    autocorr = np.correlate(audio, audio, mode='full')
    # Find first peak after origin
    peaks = find_peaks(autocorr[len(autocorr)//2:])[0]
    if len(peaks) > 0:
        harmonic_energy = autocorr[len(autocorr)//2 + peaks[0]]
        noise_energy = autocorr[len(autocorr)//2] - harmonic_energy
        hnr = 10 * np.log10(harmonic_energy / noise_energy)
    return hnr
```

## Time-Domain Features (২টি):

**STE (Short-Time Energy):**
- প্রতিটা frame এ কতটা energy আছে
- Loudness variation capture করে

**ZCR (Zero-Crossing Rate):**
- Signal কতবার zero cross করে per second
- High ZCR = noisy/fricative sounds

**Code:**
```python
def calculate_ste(audio, frame_length=2048, hop_length=512):
    frames = librosa.util.frame(audio, frame_length, hop_length)
    ste = np.sum(frames**2, axis=0)
    return np.mean(ste), np.std(ste)

def calculate_zcr(audio):
    zcr = librosa.feature.zero_crossing_rate(audio)
    return np.mean(zcr), np.std(zcr)
```

**Feature Selection Result:**

**Harmonic:** ২টির মধ্যে ১টি consensus এ
- **NHR:** **৫/৯ methods** ✓ (HNR select হয়নি)

**Time-Domain:** ২টির মধ্যে ১টি consensus এ
- **STE:** **৫/৯ methods** ✓ (ZCR select হয়নি)

NHR আর STE just pass করেছে ৫/৯ threshold!"

---

# 📌 SLIDE 8: PITCH & SPECTRAL FEATURES (4)

## Speaking Notes (বাংলা):

"এবার Pitch আর Spectral features।

## Pitch Features (৩টি):

**Pitch কি?**
Pitch হলো voice এর fundamental frequency (F0)। এটা determine করে আপনার voice 'high' না 'low'।

- Male average: ~১২০ Hz
- Female average: ~২০০ Hz

**৩টি Pitch Features:**

1. **MDVP_Fo** - Average fundamental frequency
2. **MDVP_Fhi** - Maximum F0 (highest pitch reached)
3. **MDVP_Flo** - Minimum F0 (lowest pitch reached)

**PD তে কি হয়?**

PD রোগীদের pitch range কমে যায়। তারা monotone কথা বলে - voice এ variation কম থাকে। এটাকে বলে 'hypophonia'।

**Code:**
```python
f0, _, _ = librosa.pyin(audio, fmin=75, fmax=500, sr=sr)
f0_voiced = f0[~np.isnan(f0)]

mdvp_fo = np.mean(f0_voiced)      # Mean F0
mdvp_fhi = np.max(f0_voiced)      # Max F0
mdvp_flo = np.min(f0_voiced)      # Min F0
```

## Spectral Features (১টি):

**Spectral Entropy:**
- Frequency distribution কতটা uniform সেটা measure করে
- High entropy = flat spectrum (noise-like)
- Low entropy = peaked spectrum (tonal)

**Code:**
```python
def spectral_entropy(audio, sr):
    # FFT
    spectrum = np.abs(np.fft.rfft(audio))
    # Normalize to probability
    p = spectrum / np.sum(spectrum)
    # Entropy
    entropy = -np.sum(p * np.log2(p + 1e-10))
    return entropy
```

**Feature Selection Result:**

**Pitch:** ৩টির মধ্যে ১টি consensus এ (৩৩%)
- **MDVP_Fo:** **৫/৯ methods** ✓

**Spectral:** ১টির মধ্যে ০টি consensus এ (০%)
- Spectral_Entropy: <৫ methods

Interesting finding: Fhi আর Flo select হয়নি। কারণ হতে পারে gender variation - male আর female এর F0 range অনেক different, এটা class separation এ noise add করতে পারে।"

---

# 📌 SLIDE 9: MFCC FEATURES (26)

## Speaking Notes (বাংলা):

"এবার আসি সবচেয়ে important feature category - MFCCs!

**MFCC কি?**

MFCC = Mel-Frequency Cepstral Coefficients

এটা বুঝতে হলে আগে বুঝতে হবে:

1. **Mel Scale:** Human ear linearly শোনে না। Low frequencies এ বেশি sensitive, high frequencies এ কম। Mel scale এই human perception mimic করে।

2. **Cepstrum:** Spectrum এর spectrum। এটা vocal tract characteristics capture করে।

**সহজ ভাষায়:**

MFCCs হলো voice signal এর একটা compact representation যেটা human ear যেভাবে শোনে সেভাবে encode করে। এটা voice এর 'fingerprint' এর মতো।

**কেন MFCCs এত popular?**
- Speech recognition এ standard
- Compact representation
- Speaker independent features
- Noise robust

**আমরা কি extract করেছি:**
- ১৩টি MFCC coefficients এর **mean**
- ১৩টি MFCC coefficients এর **std (standard deviation)**
- Total = ২৬ features

**Code:**
```python
# Extract MFCCs
mfccs = librosa.feature.mfcc(y=audio, sr=sr, n_mfcc=13)

# Mean and std over time
mfcc_mean = np.mean(mfccs, axis=1)  # 13 values
mfcc_std = np.std(mfccs, axis=1)    # 13 values

# Feature names
for i in range(13):
    features[f'MFCC_{i+1}_mean'] = mfcc_mean[i]
    features[f'MFCC_{i+1}_std'] = mfcc_std[i]
```

**Feature Selection Result - এটা AMAZING!**

**২৬টির মধ্যে ১২টি consensus এ (৪৬%):**

🏆 **ALL ৯/৯ Methods (Perfect Score!):**
- MFCC_4_std
- MFCC_5_std
- MFCC_11_mean
- MFCC_13_mean

🥈 **৮/৯ Methods:**
- MFCC_2_mean

🥉 **৭/৯ Methods:**
- MFCC_5_mean
- MFCC_12_mean

**৬/৯ Methods:**
- MFCC_1_std, MFCC_6_std

**৫/৯ Methods:**
- MFCC_9_std, MFCC_11_std

**Key Insight:**

MFCCs আমাদের consensus features এর ৫০% (১২/২৪)! এবং ৪টি MFCC সব ৯টি methods এ select হয়েছে - এরা সবচেয়ে reliable features।

MFCC_4 আর MFCC_5 এর std (variation) important - মানে এই frequencies তে কতটা variation আছে সেটা PD detect করতে help করে।"

---

# 📌 SLIDE 10: FEATURE SELECTION

## Speaking Notes (বাংলা):

"এবার আসি Feature Selection methodology নিয়ে।

**কেন Feature Selection দরকার?**

1. **Curse of Dimensionality:** বেশি features = বেশি data দরকার
2. **Overfitting:** অপ্রয়োজনীয় features noise add করে
3. **Interpretability:** কম features = সহজে বোঝা যায়
4. **Efficiency:** Training এবং inference faster হয়

**আমরা কি করেছি?**

শুধু একটা method না, আমরা **৯টি different methods** ব্যবহার করেছি তিনটা category থেকে:

## Filter Methods (৪টি):

এরা model ছাড়াই feature score করে:

1. **Mutual Information:**
   - Feature আর target এর মধ্যে কতটা information share করে
   - I(X;Y) = H(Y) - H(Y|X)

2. **Chi-Squared (χ²):**
   - Statistical test - observed vs expected frequency
   - Categorical data এর জন্য design করা

3. **ANOVA F-test:**
   - Between-class variance vs within-class variance
   - F = variance_between / variance_within

4. **CFS (Correlation-based Feature Selection):**
   - High correlation with target
   - Low correlation among features (reduces redundancy)

## Wrapper Methods (২টি):

এরা model ব্যবহার করে iteratively features select করে:

5. **RFE with Logistic Regression:**
   - Recursive Feature Elimination
   - প্রতি step এ worst feature বাদ দেয়

6. **RFE with Random Forest:**
   - Same concept, different base model
   - Tree-based elimination

## Embedded Methods (৩টি):

এরা model training এর সাথেই feature importance শেখে:

7. **Lasso (L1 Regularization):**
   - L1 penalty কিছু coefficients zero করে দেয়
   - Automatic feature selection

8. **Random Forest Feature Importance:**
   - Gini importance from decision trees
   - কোন feature splits এ বেশি useful

9. **Gradient Boosting Feature Importance:**
   - Same concept, boosting based

**প্রতিটা method ২৫টি features select করেছে (৫১ থেকে)।**

**Code Example (Mutual Information):**
```python
from sklearn.feature_selection import SelectKBest, mutual_info_classif

selector = SelectKBest(score_func=mutual_info_classif, k=25)
selector.fit(X_train, y_train)
selected_features = feature_cols[selector.get_support()]
```

**Performance Comparison:**

| Method | Test F1 |
|--------|---------|
| CFS (Correlation) | **0.783** |
| Chi-Squared | 0.773 |
| ANOVA F-test | 0.773 |
| RFE (Random Forest) | 0.766 |
| RFE (Logistic) | 0.762 |
| CONSENSUS | 0.756 |
| Mutual Information | 0.727 |

**Key Finding:** Filter methods এই dataset এ Embedded methods কে beat করেছে!"

---

# 📌 SLIDE 11: CONSENSUS FEATURES

## Speaking Notes (বাংলা):

"এবার আসি আমাদের Consensus approach নিয়ে।

**Consensus কি?**

আমরা ৯টি different feature selection methods run করেছি। এখন প্রশ্ন হলো - কোন features ব্যবহার করবো?

**আমাদের strategy:**
যে features **৫টি বা তার বেশি methods** দিয়ে select হয়েছে, সেগুলোই final features।

**কেন এই approach?**

1. **Robustness:** একটা method biased হতে পারে, কিন্তু ৫টা method agree করলে সেটা reliable
2. **Method-agnostic:** কোনো specific method এর উপর depend করছি না
3. **Interpretable:** যে features সবাই important বলছে সেগুলোই নিচ্ছি

**Results:**

**Category-wise Breakdown:**

| Category | Selected/Total | Percentage |
|----------|----------------|------------|
| **Jitter** | ৫/৫ | **১০০%** 🏆 |
| Shimmer | ৩/৬ | ৫০% |
| Harmonic | ১/২ | ৫০% |
| Pitch | ১/৩ | ৩৩% |
| Nonlinear | ১/৬ | ১৭% |
| Time-Domain | ১/২ | ৫০% |
| Spectral | ০/১ | ০% |
| **MFCCs** | ১২/২৬ | ৪৬% |
| **TOTAL** | **২৪/৫১** | **৪৭%** |

**Key Insights:**

1. **Jitter সবচেয়ে consistent** - সব ৫টি feature select হয়েছে!
2. **MFCCs dominant** - ১২টি feature, consensus এর ৫০%
3. **Spectral Entropy select হয়নি** - এই dataset এ discriminative না

**Top Features by Agreement:**

🏆 **৯/৯ Methods (All agree!):**
- MFCC_4_std
- MFCC_5_std
- MFCC_11_mean
- MFCC_13_mean

🥈 **৮/৯ Methods:**
- Shimmer_APQ3
- Shimmer_DDA
- MFCC_2_mean

🥉 **৭/৯ Methods:**
- MDVP_Jitter_percent
- MDVP_Jitter_Abs
- MDVP_PPQ
- RPDE
- MFCC_5_mean
- MFCC_12_mean"

---

# 📌 SLIDE 12: CONSENSUS Results

## Speaking Notes (বাংলা):

"শেষ slide এ আমি summary দিচ্ছি আমাদের Consensus Results এর।

**Final Feature Set: ২৪ Features**

**Dimensionality Reduction:**
- Original: ৫১ features
- Final: ২৪ features
- **Reduction: ৫৩%**

**Final Features by Category:**

```
Jitter (৫):     Jitter%, Abs, RAP, PPQ, DDP
Shimmer (৩):    APQ3, DDA, dB
Harmonic (১):   NHR
Pitch (১):      MDVP_Fo
Nonlinear (১):  RPDE
Time (১):       STE
MFCCs (১২):     2m, 5m, 11m, 12m, 13m, 1s, 4s, 5s, 6s, 9s, 11s
```

**Data Shape After Selection:**
```
Original:  Train(312, 51) → Val(44, 51) → Test(42, 51)
Final:     Train(312, 24) → Val(44, 24) → Test(42, 24)
```

**Key Takeaways:**

1. ✅ **Multi-method approach** gives robust feature selection
2. ✅ **Jitter features** are most reliable for PD detection
3. ✅ **MFCCs** dominate the feature space (50% of selected)
4. ✅ **53% dimensionality reduction** without losing discriminative power
5. ✅ **Consensus F1 = 0.756** - competitive with best single method (0.783)

**Next Steps:**

এই ২৪টি features দিয়ে এখন আমরা:
1. Multiple ML models train করবো
2. Hyperparameter tuning করবো
3. Cross-validation করবো
4. Final test evaluation করবো

**Questions?**

ধন্যবাদ! কোনো প্রশ্ন থাকলে জিজ্ঞেস করতে পারেন।"

---

# 📋 QUICK REFERENCE CARD

| Slide | Duration | Key Points |
|-------|----------|------------|
| 1 | 30 sec | Title, overview |
| 2 | 2 min | 51 features, 8 categories, 9 selection methods |
| 3 | 2 min | IPVS dataset, patient-wise split, no data leakage |
| 4 | 3 min | Jitter - pitch variation, 5/5 selected |
| 5 | 3 min | Shimmer - amplitude variation, 3/6 selected |
| 6 | 3 min | Nonlinear - complexity, only RPDE selected |
| 7 | 2 min | Harmonic + Time-domain, NHR & STE selected |
| 8 | 2 min | Pitch + Spectral, only MDVP_Fo selected |
| 9 | 4 min | MFCCs - 12/26 selected, 4 features all methods agree |
| 10 | 4 min | 9 methods: Filter, Wrapper, Embedded |
| 11 | 2 min | Consensus strategy, 24 final features |
| 12 | 2 min | Summary, 53% reduction, next steps |

**Total: ~30 minutes**

---

# 🎯 TIPS FOR PRESENTING

1. **Start slow** - Give audience time to understand concepts
2. **Use analogies** - "Jitter is like voice shakiness"
3. **Show code briefly** - Don't read it, just show structure
4. **Emphasize results** - "9/9 methods agree on these 4 features!"
5. **Make eye contact** - Look at audience, not slides
6. **Pause for questions** - After complex slides
7. **Use hand gestures** - Point to graphs when explaining
8. **Summarize often** - "So far we've seen..."
