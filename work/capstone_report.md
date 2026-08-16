# Content Refresh Opportunity Scoring — Capstone Report

**Author:** Abdul Moeed  
**Lane:** Refresh / Content Opportunity Scoring  
**Repo:** [https://github.com/moeeed2006-ops/Abdul-Moeed-flyrank-ml-work]
**Date:** August 2026  

---

### 1. Problem Framing
* **Decision Supported:** Identifies high-value search content suffering from traffic decay that requires human editorial updates.
* **Unit of Analysis:** Unique Content Page (`content_id`).
* **Output:** Ranked opportunity score (0.0 to 1.0) paired with automated Reason Codes.
* **Human Action:** FlyRank content editors prioritize updating high-scoring pages (e.g., refreshing titles, updating stats, re-optimizing metadata).
* **Cost of Wrong Call:** False Positive = wasted editorial hours on fine pages; False Negative = lost search visibility and decaying site traffic.
* **Why Data/ML Helps:** Automates prioritization across thousands of URLs, replacing manual periodic audits.

---

### 2. Data Safety
* **Tables Used:** `fact_content_daily_performance` from the FlyRank Hugging Face warehouse.
* **Excluded Columns:** Private client metadata, URLs, domains, credentials, and private search queries.
* **Leakage Control:** All input features (`X`) are calculated strictly from the **baseline time window** (`< 2026-05-01`). Recent traffic metrics (`recent_clicks`, `recent_impressions`) were used *only* to generate the ground-truth target label (`y`), preventing look-ahead leakage.
* **Pseudonymous IDs:** `content_id` was strictly used for grouping and feature joins—never as a feature.

---

### 3. Baseline
* **Rule-Based Baseline Definition:** Flags pages with `baseline_position > 10.0` and `baseline_ctr < median_ctr`.
* **Baseline Performance:**
  * Base Rate: ~20%
  * Baseline Precision: ~0.28
  * Baseline Recall: ~0.35
  * Baseline F1-score: ~0.31

---

### 4. Model / Analysis
* **Method:** Random Forest Classifier (`n_estimators=100`, `max_depth=6`).
* **Target Definition (1 sentence):** Binary flag equal to `1` if a page had >30 baseline clicks and suffered a >20% reduction in recent clicks compared to its historical baseline.
* **Feature List:**
  1. `baseline_clicks`: Historical volume.
  2. `baseline_impressions`: Historical reach.
  3. `baseline_position`: Historical average SERP rank.
  4. `baseline_ctr`: Historical click-through rate.

---

### 5. Evaluation
* **Split Strategy:** Stratified Train/Test split (80% train / 20% test) maintaining target class ratios.
* **Model vs. Baseline Metrics:**
  * **Random Forest AUC:** ~0.84 vs. Baseline AUC: ~0.53
  * **Precision:** ~0.76 (Model) vs. ~0.28 (Baseline)
  * **Recall:** ~0.70 (Model) vs. ~0.35 (Baseline)
* **Error Analysis:** False positives occur primarily on low-impression tail pages where noise distorts CTR. Adding minimum impression filters (>100) resolves most classification noise.

---

### 6. Interpretation
* **Feature Importances:** `baseline_clicks` and `baseline_position` were the most predictive signals for refresh opportunity scoring.
* **Surprises / Negative Results:** Raw impression volume alone was a poor predictor unless combined with historical position metrics.

---

### 7. Recommendation
* **Action Playbook for Editors:**
  1. Score all published pages weekly.
  2. Filter for pages with opportunity score > 0.70.
  3. Apply action based on `reason_code`:
     * `HIGH_VALUE_TRAFFIC_LOSS`: Update content stats and refresh publish date.
     * `LOW_CTR_HIGH_IMPRESSIONS`: Rewrite meta titles and snippet descriptions.
     * `RANK_DECAY_PAGE_2`: Add internal links and sub-headings to push from Page 2 to Page 1.
* **Limits:** Model outputs directional decision-support scores—it does not guarantee immediate Google ranking recovery without quality content updates.

---

### 8. Reproducibility
* **Environment:** Python 3.10+, `duckdb>=0.9.0`, `scikit-learn>=1.2.0`, `pandas>=2.0.0`.
* **Random Seed:** `random_state=42` used across train/test splits and Random Forest initialization.
* **Re-run Steps:**
  ```bash
  git clone [https://github.com/your-username/flyrank-ml-capstone.git](https://github.com/your-username/flyrank-ml-capstone.git)
  cd flyrank-ml-capstone
  pip install -r requirements.txt
  jupyter notebook work/capstone_content_refresh.ipynb
