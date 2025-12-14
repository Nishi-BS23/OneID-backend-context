# 📚 Feature Selection Methodology - Complete Technical Documentation
## Parkinson's Disease Detection from Voice Signals

---

# 📑 TABLE OF CONTENTS

1. [Why Feature Selection?](#1-why-feature-selection)
2. [Our Approach Overview](#2-our-approach-overview)
3. [Filter Methods (4 Methods)](#3-filter-methods)
   - 3.1 Mutual Information
   - 3.2 Chi-Squared (χ²)
   - 3.3 ANOVA F-Statistic
   - 3.4 Correlation-Based Feature Selection (CFS)
4. [Wrapper Methods (2 Methods)](#4-wrapper-methods)
   - 4.1 RFE with Logistic Regression
   - 4.2 RFE with Random Forest
5. [Embedded Methods (3 Methods)](#5-embedded-methods)
   - 5.1 Lasso (L1 Regularization)
   - 5.2 Random Forest Feature Importance
   - 5.3 Gradient Boosting Feature Importance
6. [Consensus Strategy](#6-consensus-strategy)
7. [Final Results & Analysis](#7-final-results--analysis)

---

# 1. WHY FEATURE SELECTION?

## 1.1 আমাদের Problem Statement

আমাদের কাছে **51টি features** আছে 398টি audio samples থেকে extract করা। কিন্তু সব features কি সমান important? না!

### সমস্যাগুলো:

1. **Curse of Dimensionality (মাত্রার অভিশাপ)**
   ```
   আমাদের data: 398 samples × 51 features
   
   Rule of thumb: n_samples >= 10 × n_features
   আমাদের দরকার: 51 × 10 = 510 samples
   আমাদের আছে: 398 samples ❌
   
   তাই feature কমাতে হবে!
   ```

2. **Overfitting Problem**
   - বেশি features = model noise শেখে
   - Training accuracy বাড়ে, test accuracy কমে
   - Generalization খারাপ হয়

3. **Redundancy (Repeated Information)**
   - কিছু features একই information দেয়
   - Example: MFCC_1_mean আর MFCC_1_std correlated হতে পারে
   - দুইটা রাখলে computation বাড়ে, benefit বাড়ে না

4. **Noise (শুধু confusion)**
   - কিছু features তে PD vs HC তে কোনো difference নেই
   - এগুলো model কে confuse করে

## 1.2 আমাদের Goal

```
Original: 51 features
Target: ~25 features (50% reduction)
Strategy: Multiple methods → Consensus
```

---

# 2. OUR APPROACH OVERVIEW

## 2.1 তিন ধরনের Feature Selection Methods

```
┌─────────────────────────────────────────────────────────────┐
│                    FEATURE SELECTION                         │
├─────────────────┬─────────────────┬─────────────────────────┤
│  FILTER         │  WRAPPER        │  EMBEDDED               │
│  (Model-free)   │  (Model-based)  │  (Training-integrated)  │
├─────────────────┼─────────────────┼─────────────────────────┤
│ • Fast          │ • Accurate      │ • Automatic             │
│ • Simple        │ • Slow          │ • Model-specific        │
│ • Independent   │ • Iterative     │ • Efficient             │
└─────────────────┴─────────────────┴─────────────────────────┘
```

## 2.2 আমাদের 9টি Methods

| Category | Method | কি করে? |
|----------|--------|---------|
| **Filter** | Mutual Information | Feature-target dependency |
| | Chi-Squared | Statistical association |
| | ANOVA F-test | Class variance ratio |
| | CFS (Correlation) | Target correlation - redundancy |
| **Wrapper** | RFE (Logistic) | Iterative elimination |
| | RFE (Random Forest) | Tree-based elimination |
| **Embedded** | Lasso (L1) | Coefficient shrinkage |
| | Random Forest | Gini importance |
| | Gradient Boosting | Boosting importance |

## 2.3 আমাদের Configuration

```python
K_FEATURES = 25  # প্রতিটা method 25টি feature select করবে

# Data splits
X_train: (312, 51)  # Training set
X_val: (44, 51)     # Validation set
X_test: (42, 51)    # Test set
```

---

# 3. FILTER METHODS

Filter methods model ছাড়াই statistical measures ব্যবহার করে features score করে। এরা fast এবং model-agnostic (যেকোনো classifier এর সাথে কাজ করে)।

---

## 3.1 MUTUAL INFORMATION (MI)

### 3.1.1 Concept (সহজ ভাষায়)

**Mutual Information** measure করে দুইটা variable এর মধ্যে কতটা "shared information" আছে।

**Analogy:** ধরুন আপনি একজন মানুষকে চেনেন না। শুধু তার height জানলে কি তার weight guess করতে পারবেন? পারবেন কিছুটা! কারণ height আর weight এর মধ্যে "mutual information" আছে।

```
MI(X; Y) = "X জানলে Y সম্পর্কে কতটা জানা যায়"

High MI: Feature জানলে class predict করা সহজ
Low MI: Feature class এর সাথে unrelated
```

### 3.1.2 Mathematical Definition

$$I(X; Y) = H(Y) - H(Y|X)$$

Where:
- $H(Y)$ = Entropy of Y (target class এর uncertainty)
- $H(Y|X)$ = Conditional entropy (feature জানলে target এর uncertainty)

**Entropy:**
$$H(Y) = -\sum_{y} P(y) \log P(y)$$

### 3.1.3 আমাদের Implementation

```python
from sklearn.feature_selection import SelectKBest, mutual_info_classif

# SelectKBest ব্যবহার করেছি
mi_selector = SelectKBest(
    score_func=mutual_info_classif,  # Scikit-learn এর MI function
    k=K_FEATURES                      # Top 25 features select করবে
)
mi_selector.fit(X_train_orig, y_train)

# Results extract করা
mi_scores = mi_selector.scores_           # প্রতিটা feature এর MI score
mi_selected_mask = mi_selector.get_support()  # Boolean mask: selected/not
mi_selected_features = [feature_cols[i] for i in range(len(feature_cols)) 
                        if mi_selected_mask[i]]
```

### 3.1.4 কেন MI ব্যবহার করলাম?

| Advantage | Explanation |
|-----------|-------------|
| **Non-linear relationships ধরতে পারে** | Correlation শুধু linear দেখে, MI nonlinear ও দেখে |
| **No assumptions** | Distribution assume করে না |
| **Scale-invariant** | Feature scaling এ result change হয় না |

### 3.1.5 Limitations

- Estimation এ কিছু randomness থাকে
- Small samples এ inaccurate হতে পারে
- Computationally expensive (large data তে)

---

## 3.2 CHI-SQUARED (χ²) TEST

### 3.2.1 Concept (সহজ ভাষায়)

**Chi-Squared test** দেখে observed frequencies আর expected frequencies এর মধ্যে কতটা difference।

**Analogy:** ধরুন আপনি dice roll করছেন। Fair dice হলে প্রতিটা number সমান সম্ভাবনায় আসা উচিত। কিন্তু যদি 6 বেশি আসে, তাহলে dice "biased"। Chi-squared এই "bias" measure করে।

```
High χ²: Feature class এর সাথে strongly associated
Low χ²: Feature আর class independent
```

### 3.2.2 Mathematical Definition

$$\chi^2 = \sum \frac{(O - E)^2}{E}$$

Where:
- $O$ = Observed frequency
- $E$ = Expected frequency (if independent)

### 3.2.3 আমাদের Implementation

```python
from sklearn.feature_selection import chi2
from sklearn.preprocessing import MinMaxScaler

# ⚠️ CRITICAL: Chi-squared requires NON-NEGATIVE values!
# আমাদের features negative হতে পারে (e.g., MFCCs)
# তাই MinMaxScaler ব্যবহার করে [0, 1] range এ আনলাম

scaler_minmax = MinMaxScaler()
X_train_positive = scaler_minmax.fit_transform(X_train_orig)

chi2_selector = SelectKBest(
    score_func=chi2,    # Chi-squared function
    k=K_FEATURES        # Top 25
)
chi2_selector.fit(X_train_positive, y_train)

chi2_scores = chi2_selector.scores_
chi2_selected_mask = chi2_selector.get_support()
chi2_selected_features = [feature_cols[i] for i in range(len(feature_cols)) 
                          if chi2_selected_mask[i]]
```

### 3.2.4 কেন MinMaxScaler দরকার?

```python
# Problem:
MFCC_1_mean = -50.23  # Negative value!
chi2() → ERROR! χ² formula তে (O-E)²/E, E negative হলে problem

# Solution:
MinMaxScaler: X_scaled = (X - X_min) / (X_max - X_min)
Result: সব values 0 থেকে 1 এর মধ্যে
```

### 3.2.5 কেন Chi-Squared ব্যবহার করলাম?

| Advantage | Explanation |
|-----------|-------------|
| **Simple & Fast** | Direct calculation, no iteration |
| **Well-established** | Statistical test since 1900 |
| **Works with discrete** | Originally for categorical data |

### 3.2.6 Limitations

- Requires non-negative values
- Assumes features are positive counts (আমরা scaling করে সমস্যা solve করলাম)
- Only captures linear dependencies

---

## 3.3 ANOVA F-STATISTIC

### 3.3.1 Concept (সহজ ভাষায়)

**ANOVA (Analysis of Variance)** দেখে different groups এর means কতটা different।

**Analogy:** ধরুন দুইটা school এর exam results দেখছেন। School A তে average 80, School B তে average 60। কিন্তু প্রশ্ন হলো - এই difference কি significant না just random variation?

```
                    Variance BETWEEN classes
F-statistic = ────────────────────────────────
                    Variance WITHIN classes

High F: Classes are well separated by this feature
Low F: Feature doesn't discriminate between classes
```

### 3.3.2 Mathematical Definition

$$F = \frac{\text{Between-group variance}}{\text{Within-group variance}} = \frac{MS_{between}}{MS_{within}}$$

Where:
$$MS_{between} = \frac{\sum n_i(\bar{x_i} - \bar{x})^2}{k-1}$$
$$MS_{within} = \frac{\sum\sum (x_{ij} - \bar{x_i})^2}{N-k}$$

### 3.3.3 আমাদের Implementation

```python
from sklearn.feature_selection import f_classif

anova_selector = SelectKBest(
    score_func=f_classif,  # ANOVA F-test
    k=K_FEATURES           # Top 25
)
anova_selector.fit(X_train_orig, y_train)

anova_scores = anova_selector.scores_
anova_selected_mask = anova_selector.get_support()
anova_selected_features = [feature_cols[i] for i in range(len(feature_cols)) 
                           if anova_selected_mask[i]]
```

### 3.3.4 Visual Example

```
Feature: MFCC_4_std

           HC (Healthy)        PD (Parkinson's)
           ────────────        ────────────────
Values:    [2.1, 2.3, 2.0]     [4.5, 4.8, 4.2]
Mean:      2.13                 4.5
           
Between-class variance: HIGH (means are different: 2.13 vs 4.5)
Within-class variance: LOW (values close to their means)

F-statistic: HIGH → Good feature! ✓
```

### 3.3.5 কেন ANOVA ব্যবহার করলাম?

| Advantage | Explanation |
|-----------|-------------|
| **Direct class separation** | Explicitly measures how well feature separates classes |
| **No model needed** | Pure statistical test |
| **Handles multiple classes** | Works for more than 2 classes too |

### 3.3.6 Limitations

- Assumes normal distribution
- Sensitive to outliers
- Only captures linear relationships

---

## 3.4 CORRELATION-BASED FEATURE SELECTION (CFS)

### 3.4.1 Concept (সহজ ভাষায়)

CFS আলাদা কারণ এটা শুধু target correlation না, **feature redundancy** ও consider করে।

**Analogy:** ধরুন আপনি cricket team বানাচ্ছেন। আপনি 5 জন batsman চাইলেন। কিন্তু যদি 5 জনই opening batsman হয়, তাহলে team unbalanced। আপনি চাইবেন different skills - opener, middle order, finisher।

```
CFS Strategy:
1. High correlation with target ✓
2. Low correlation with other selected features ✓

"Good features are highly correlated with target but 
 uncorrelated with each other"
```

### 3.4.2 CFS Merit Formula

$$Merit = \frac{k \cdot \overline{r_{cf}}}{\sqrt{k + k(k-1) \cdot \overline{r_{ff}}}}$$

Where:
- $k$ = number of selected features
- $\overline{r_{cf}}$ = average feature-class correlation
- $\overline{r_{ff}}$ = average feature-feature correlation

### 3.4.3 আমাদের Implementation (Custom Function)

```python
def correlation_based_selection(X, y, feature_names, k=25):
    """
    Select features with:
    - High correlation with target
    - Low inter-feature correlation (redundancy)
    """
    
    # Step 1: Calculate feature-target correlation
    # প্রতিটা feature target এর সাথে কতটা correlated
    feature_target_corr = np.array([
        np.abs(np.corrcoef(X[:, i], y)[0, 1]) 
        for i in range(X.shape[1])
    ])
    feature_target_corr = np.nan_to_num(feature_target_corr, 0)
    
    # Step 2: Calculate inter-feature correlation matrix
    # Features নিজেদের মধ্যে কতটা correlated
    feature_corr_matrix = np.abs(np.corrcoef(X.T))
    np.fill_diagonal(feature_corr_matrix, 0)  # Diagonal = self-correlation = 0
    
    # Step 3: Greedy forward selection
    selected_indices = []
    remaining = list(range(X.shape[1]))
    
    for _ in range(k):  # k features select করবো
        best_score = -np.inf
        best_idx = None
        
        for idx in remaining:
            if len(selected_indices) == 0:
                # First feature: just use target correlation
                score = feature_target_corr[idx]
            else:
                # CFS merit formula
                k_selected = len(selected_indices) + 1
                
                # Sum of target correlations
                sum_target_corr = feature_target_corr[idx] + \
                                  sum(feature_target_corr[i] for i in selected_indices)
                
                # Sum of inter-feature correlations (redundancy penalty)
                sum_feature_corr = sum(feature_corr_matrix[idx, i] 
                                       for i in selected_indices)
                
                # Merit formula
                avg_target = sum_target_corr / k_selected
                avg_feature = sum_feature_corr / max(1, len(selected_indices))
                
                denominator = np.sqrt(k_selected + k_selected * (k_selected - 1) * avg_feature)
                score = (k_selected * avg_target) / max(denominator, 1e-10)
            
            if score > best_score:
                best_score = score
                best_idx = idx
        
        # Add best feature to selected set
        if best_idx is not None:
            selected_indices.append(best_idx)
            remaining.remove(best_idx)
    
    selected_features = [feature_names[i] for i in selected_indices]
    scores = dict(zip(feature_names, feature_target_corr))
    
    return selected_features, scores

# Call the function
cfs_selected_features, cfs_scores = correlation_based_selection(
    X_train_orig, y_train, feature_cols, k=K_FEATURES
)
```

### 3.4.4 Step-by-Step Example

```
Iteration 1:
  - No features selected yet
  - Select feature with highest target correlation
  - Winner: MFCC_4_std (corr = 0.45)

Iteration 2:
  - Already have: MFCC_4_std
  - For each remaining feature, calculate merit
  - MFCC_5_std has:
    - High target corr: 0.42 ✓
    - Low corr with MFCC_4_std: 0.15 ✓
    - Merit = high
  - MFCC_4_mean has:
    - High target corr: 0.40
    - High corr with MFCC_4_std: 0.85 ✗ (redundant!)
    - Merit = low
  - Winner: MFCC_5_std

...continue for 25 iterations...
```

### 3.4.5 কেন CFS সবচেয়ে ভালো perform করলো?

| Reason | Explanation |
|--------|-------------|
| **Handles redundancy** | অন্য methods শুধু target correlation দেখে, CFS redundancy ও দেখে |
| **Diverse feature set** | Different information capture করে |
| **No overfitting** | Fewer correlated features = less noise |

**আমাদের Result:** CFS achieved **Test F1 = 0.783** (Best among all methods!)

---

# 4. WRAPPER METHODS

Wrapper methods আসল ML model ব্যবহার করে features evaluate করে। Accurate কিন্তু slow।

---

## 4.1 RFE WITH LOGISTIC REGRESSION

### 4.1.1 Concept (সহজ ভাষায়)

**RFE (Recursive Feature Elimination)** একটা backward elimination process।

**Analogy:** ধরুন আপনি 51 জন candidate থেকে 25 জন select করবেন interview তে। আপনি কি করবেন?

Option 1: সবার score দেখে top 25 নেবেন (Filter method)
Option 2: প্রতি round এ worst performer বাদ দেবেন (RFE method)

```
RFE Process:
Round 1: 51 features → Train model → Remove worst feature → 50 features
Round 2: 50 features → Train model → Remove worst feature → 49 features
...
Round 26: 26 features → Train model → Remove worst feature → 25 features ✓
```

### 4.1.2 কেন Logistic Regression?

Logistic Regression এ প্রতিটা feature এর একটা **coefficient** থাকে।

$$P(y=1|X) = \frac{1}{1 + e^{-(\beta_0 + \beta_1 x_1 + \beta_2 x_2 + ... + \beta_n x_n)}}$$

- High |coefficient| = Important feature
- Low |coefficient| = Unimportant feature

### 4.1.3 আমাদের Implementation

```python
from sklearn.feature_selection import RFE
from sklearn.linear_model import LogisticRegression

# Base estimator: Logistic Regression
base_estimator = LogisticRegression(
    max_iter=1000,           # Maximum iterations
    random_state=42,         # Reproducibility
    class_weight='balanced'  # Handle class imbalance (HC ≠ PD count)
)

# RFE selector
rfe_selector = RFE(
    estimator=base_estimator,
    n_features_to_select=K_FEATURES,  # Final 25 features
    step=1,                           # Remove 1 feature per iteration
    verbose=0
)
rfe_selector.fit(X_train_orig, y_train)

# Results
rfe_selected_mask = rfe_selector.support_    # Boolean mask
rfe_rankings = rfe_selector.ranking_         # Ranking (1 = selected)
rfe_selected_features = [feature_cols[i] for i in range(len(feature_cols)) 
                         if rfe_selected_mask[i]]

# Convert rankings to scores (lower rank = higher score)
rfe_scores = {feat: 1.0 / rank for feat, rank in zip(feature_cols, rfe_rankings)}
```

### 4.1.4 Ranking System

```
rfe_rankings array:
[1, 1, 1, 3, 5, 1, 2, ...]

Meaning:
- Rank 1 = Selected (in final 25)
- Rank 2 = 26th best (eliminated 2nd to last)
- Rank 27 = Worst (eliminated first)
```

### 4.1.5 Complexity Analysis

```
Total iterations = 51 - 25 = 26 rounds
Each round: Train Logistic Regression on training data

Time complexity: O(n_rounds × training_time)
```

---

## 4.2 RFE WITH RANDOM FOREST

### 4.2.1 Concept

Same RFE process কিন্তু **Random Forest** ব্যবহার করে feature importance calculate করা হয়।

### 4.2.2 কেন Random Forest?

Logistic Regression শুধু linear relationships দেখে। কিন্তু Random Forest:
- Nonlinear relationships ধরতে পারে
- Feature interactions capture করে
- More robust to outliers

### 4.2.3 আমাদের Implementation

```python
# Base estimator: Random Forest (lighter version for speed)
rf_estimator = RandomForestClassifier(
    n_estimators=50,    # 50 trees (less for speed)
    random_state=42,
    max_depth=5,        # Limit depth (prevent overfitting, faster)
    n_jobs=-1           # Use all CPU cores
)

rfe_rf_selector = RFE(
    estimator=rf_estimator,
    n_features_to_select=K_FEATURES,
    step=2,              # Remove 2 features per iteration (faster)
    verbose=0
)
rfe_rf_selector.fit(X_train_orig, y_train)

rfe_rf_selected_mask = rfe_rf_selector.support_
rfe_rf_rankings = rfe_rf_selector.ranking_
rfe_rf_selected_features = [feature_cols[i] for i in range(len(feature_cols)) 
                            if rfe_rf_selected_mask[i]]
```

### 4.2.4 Key Differences from RFE-Logistic

| Parameter | RFE-Logistic | RFE-RF |
|-----------|--------------|--------|
| `step` | 1 | 2 (faster) |
| `n_estimators` | N/A | 50 |
| `max_depth` | N/A | 5 |
| Feature importance | Coefficients | Gini importance |

---

# 5. EMBEDDED METHODS

Embedded methods feature selection টা model training এর সাথেই করে - separate step না।

---

## 5.1 LASSO (L1 REGULARIZATION)

### 5.1.1 Concept (সহজ ভাষায়)

**Lasso** regression এ **L1 penalty** add করে যেটা coefficients কে zero বানিয়ে দেয়।

**Analogy:** ধরুন আপনার budget limited। আপনি খরচ কমাতে চান। Lasso বলে - "যেসব জিনিস না হলেও চলবে, সেগুলোতে একদম খরচ করো না (coefficient = 0)।"

```
Regular Regression Loss:
Loss = Sum(errors²)

Lasso Loss:
Loss = Sum(errors²) + α × Sum(|coefficients|)
                       ↑
                   L1 Penalty
```

### 5.1.2 কেন Lasso Feature Selection করে?

L1 penalty এর geometry র কারণে কিছু coefficients exactly zero হয়ে যায়:

```
                    L2 (Ridge)              L1 (Lasso)
                    
Shape:              Circle                  Diamond
                      ⬤                       ◇
                                              
Effect:             Coefficients            Coefficients can be
                    shrink but              exactly ZERO
                    never zero              (automatic selection!)
```

### 5.1.3 আমাদের Implementation

```python
from sklearn.linear_model import LassoCV

# LassoCV automatically finds best alpha using cross-validation
lasso = LassoCV(
    cv=5,              # 5-fold cross-validation
    random_state=42,
    max_iter=5000      # More iterations for convergence
)
lasso.fit(X_train_orig, y_train)

# Get coefficients (non-zero = important)
lasso_coefs = np.abs(lasso.coef_)

# Select top K features by coefficient magnitude
lasso_ranking = np.argsort(lasso_coefs)[::-1]  # Descending order
lasso_selected_indices = lasso_ranking[:K_FEATURES]  # Top 25
lasso_selected_features = [feature_cols[i] for i in lasso_selected_indices]

# Count truly non-zero coefficients
n_nonzero = np.sum(lasso_coefs > 1e-10)

print(f"Optimal alpha: {lasso.alpha_:.6f}")
print(f"Non-zero coefficients: {n_nonzero}")
```

### 5.1.4 Alpha Parameter

```
α (alpha) controls regularization strength:

α = 0:      No penalty → All coefficients free → Overfitting
α = small:  Weak penalty → Few zeros → More features
α = large:  Strong penalty → Many zeros → Fewer features
α = ∞:      All coefficients = 0

LassoCV finds optimal α automatically!
```

### 5.1.5 আমাদের Result

```
Optimal alpha: 0.001234
Non-zero coefficients: 38 (out of 51)
Selected top 25 by |coefficient|
```

---

## 5.2 RANDOM FOREST FEATURE IMPORTANCE

### 5.2.1 Concept (সহজ ভাষায়)

Random Forest 100 টা decision trees তৈরি করে। প্রতিটা tree তে যে feature ভালো split করে (classes separate করে), সেটা important।

**Analogy:** আপনি 100 জন judge কে বললেন PD vs HC distinguish করতে। যে feature কে সবচেয়ে বেশি judges গুরুত্ব দিলো, সেটাই important।

### 5.2.2 Gini Importance

$$Importance(f) = \sum_{nodes \text{ using } f} \frac{n_{node}}{n_{total}} \times \Delta Gini$$

Where:
- $\Delta Gini$ = Impurity reduction from the split
- $n_{node}$ = Samples reaching that node

### 5.2.3 আমাদের Implementation

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.feature_selection import SelectFromModel

rf_model = RandomForestClassifier(
    n_estimators=100,        # 100 trees
    random_state=42,
    max_depth=10,            # Limit depth
    class_weight='balanced', # Handle imbalance
    n_jobs=-1                # Parallel
)
rf_model.fit(X_train_orig, y_train)

# Get feature importances
rf_importances = rf_model.feature_importances_

# Select top K features using SelectFromModel
rf_selector = SelectFromModel(
    rf_model, 
    prefit=True,             # Already trained
    threshold=-np.inf,       # Don't threshold, just use max_features
    max_features=K_FEATURES  # Top 25
)
rf_selected_mask = rf_selector.get_support()
rf_selected_features = [feature_cols[i] for i in range(len(feature_cols)) 
                        if rf_selected_mask[i]]
```

### 5.2.4 Random Forest vs Single Decision Tree

```
Single Tree:        Unstable, varies with data
Random Forest:      Averages over 100 trees → Stable importance scores
```

---

## 5.3 GRADIENT BOOSTING FEATURE IMPORTANCE

### 5.3.1 Concept (সহজ ভাষায়)

**Gradient Boosting** sequentially trees build করে, প্রতিটা tree আগের tree এর mistakes correct করে।

**Analogy:** 
- Random Forest: 100 judges independently vote করে
- Gradient Boosting: Judge 2 tries to correct Judge 1's mistakes, Judge 3 corrects Judge 2's mistakes...

### 5.3.2 Difference from Random Forest

| Aspect | Random Forest | Gradient Boosting |
|--------|---------------|-------------------|
| Trees | Independent (parallel) | Sequential |
| Mistakes | Averaged out | Corrected iteratively |
| Speed | Faster | Slower |
| Overfitting | Less prone | More prone |

### 5.3.3 আমাদের Implementation

```python
from sklearn.ensemble import GradientBoostingClassifier

gb_model = GradientBoostingClassifier(
    n_estimators=100,    # 100 boosting rounds
    learning_rate=0.1,   # Step size
    max_depth=5,         # Shallow trees (boosting তে shallow better)
    random_state=42
)
gb_model.fit(X_train_orig, y_train)

gb_importances = gb_model.feature_importances_

# Select top K features
gb_ranking_idx = np.argsort(gb_importances)[::-1][:K_FEATURES]
gb_selected_features = [feature_cols[i] for i in gb_ranking_idx]
```

---

# 6. CONSENSUS STRATEGY

## 6.1 কেন Consensus?

আমরা 9টা different methods run করলাম। এখন কোনটা বিশ্বাস করবো?

**Problem:**
- কোনো method perfect না
- Different methods different assumptions করে
- কিছু methods certain features কে overvalue করে

**Solution:** **Ensemble/Voting approach!**

```
যদি একটা feature 5+ methods দ্বারা selected হয়,
সেটা probably genuinely important।

"Wisdom of the crowd"
```

## 6.2 আমাদের Implementation

```python
# Count how many methods selected each feature
all_selected = {}
for method, results in feature_selection_results.items():
    for feat in results['selected_features']:
        if feat not in all_selected:
            all_selected[feat] = []
        all_selected[feat].append(method)

feature_counts = {feat: len(methods) for feat, methods in all_selected.items()}

# Strategy: Select features chosen by at least 5 out of 9 methods
MIN_METHODS = 5
consensus_features = [f for f, c in feature_counts.items() if c >= MIN_METHODS]

print(f"Consensus features: {len(consensus_features)}")
# Result: 24 features
```

## 6.3 কেন Threshold = 5?

```
9 methods:
- 5/9 = 55.6% (majority vote)
- This is the democratic threshold

Too low (e.g., 3): Too many features, includes unreliable ones
Too high (e.g., 8): Too few features, may miss good ones
```

## 6.4 Consensus Results

```
Perfect Agreement (9/9 methods): 4 features
  - MFCC_4_std
  - MFCC_5_std
  - MFCC_11_mean
  - MFCC_13_mean

Strong Agreement (8/9): 3 features
  - Shimmer_APQ3
  - Shimmer_DDA
  - MFCC_2_mean

Good Agreement (7/9): 7 features
  - MDVP_Jitter_percent, MDVP_Jitter_Abs, MDVP_PPQ
  - RPDE
  - MFCC_5_mean, MFCC_12_mean
  - (and more)

Total Consensus (5+/9): 24 features
```

---

# 7. FINAL RESULTS & ANALYSIS

## 7.1 Performance Comparison

| Method | Type | Test F1 |
|--------|------|---------|
| **CFS (Correlation)** | Filter | **0.783** 🥇 |
| Chi-Squared | Filter | 0.773 🥈 |
| ANOVA F-test | Filter | 0.773 🥈 |
| RFE (Random Forest) | Wrapper | 0.766 |
| RFE (Logistic) | Wrapper | 0.762 |
| **CONSENSUS** | Hybrid | 0.756 |
| Random Forest | Embedded | 0.750 |
| Gradient Boosting | Embedded | 0.743 |
| Mutual Information | Filter | 0.727 |
| Lasso (L1) | Embedded | 0.714 |

## 7.2 Key Findings

### Finding 1: Filter Methods Won!
```
Top 3 all Filter methods!
Why? Our dataset is small (398 samples)
- Filter methods are more stable
- Wrapper/Embedded can overfit
```

### Finding 2: CFS is Best
```
CFS considers BOTH:
1. Feature-target correlation ✓
2. Feature-feature correlation (redundancy) ✓

Other methods only consider #1
```

### Finding 3: Consensus is Robust
```
Consensus F1 = 0.756 (4th best)
But more reliable because:
- Multiple methods agree
- Less likely to overfit
- More generalizable
```

## 7.3 Final Feature Set

**51 features → 24 features (53% reduction)**

```
Category-wise:
┌─────────────────────────────────────────────────┐
│ Jitter:      5/5 = 100% selected 🏆             │
│ MFCCs:       12/26 = 46% selected               │
│ Shimmer:     3/6 = 50% selected                 │
│ Harmonic:    1/2 = 50% selected                 │
│ Time-Domain: 1/2 = 50% selected                 │
│ Pitch:       1/3 = 33% selected                 │
│ Nonlinear:   1/6 = 17% selected                 │
│ Spectral:    0/1 = 0% selected                  │
└─────────────────────────────────────────────────┘
```

## 7.4 Conclusion

আমাদের multi-method feature selection approach:

1. ✅ **Robust:** 9 different perspectives থেকে features evaluate
2. ✅ **Interpretable:** যে features সবাই important বলে সেগুলোই select
3. ✅ **Efficient:** 53% dimensionality reduction
4. ✅ **Clinical relevance:** Jitter (voice shakiness) সবচেয়ে important - এটা PD র known symptom!

---

# 📚 REFERENCES

1. Scikit-learn Feature Selection: https://scikit-learn.org/stable/modules/feature_selection.html
2. Hall, M. A. (1999). Correlation-based Feature Selection for Machine Learning. PhD Thesis.
3. Guyon, I., & Elisseeff, A. (2003). An introduction to variable and feature selection. JMLR.
4. Tibshirani, R. (1996). Regression shrinkage and selection via the lasso. JRSS.

---

*Document created for Parkinson's Disease Detection Project*
*Feature Selection Methodology Documentation*
*Date: December 2024*
