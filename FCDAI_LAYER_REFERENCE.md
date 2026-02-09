# FCDAI 7-Layer Pipeline Reference

## Layer Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        FCDAI 7-LAYER DETECTION PIPELINE                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐   │
│  │  L1-2   │───▶│   L3    │───▶│   L4    │───▶│   L5    │───▶│   L6    │   │
│  │ Ingest  │    │Features │    │ Preproc │    │ Detect  │    │Ensemble │   │
│  │  + DQ   │    │   Eng   │    │         │    │26 Algos │    │ Fusion  │   │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘    └────┬────┘   │
│       │                                                           │         │
│       │                                                           ▼         │
│       │         ┌─────────────────────────────────────────────────────┐     │
│       │         │                      L7 OUTPUT                      │     │
│       └────────▶│  Investigation Queue │ Narratives │ Audit Trail    │     │
│                 └─────────────────────────────────────────────────────┘     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Layer 1-2: Data Ingestion + Quality

**File:** `layers/l1_l2_ingestion.py`

```python
class DataIngestion:
    """
    PURPOSE:
    --------
    Load data from multiple sources and assess quality.
    
    INPUT:
    ------
    source_files: dict
        {
            "kyc": DataFrame,
            "transactions": DataFrame,
            "alerts": DataFrame,
            "cases": DataFrame
        }
    
    OUTPUT:
    -------
    - Unified DataFrame with all sources joined
    - Data Quality (DQ) score (0-1)
    - Cleansed/validated data
    
    KEY METHODS:
    ------------
    load_sources(sources_dict) -> DataFrame
        Loads and joins all data sources
        
    calculate_dq_score(df) -> dict
        Returns:
        {
            "completeness": 0.98,  # % non-null cells
            "validity": 0.95,      # % valid values
            "consistency": 0.92,   # % consistent records
            "overall": 0.95
        }
        
    cleanse_data(df) -> DataFrame
        - Fill missing values
        - Remove duplicates
        - Standardize formats
    
    MODIFICATION:
    -------------
    To add new data source:
    1. Add to expected_sources list
    2. Add join logic in load_sources()
    3. Add validation rules in validate_source()
    """
    
    def __init__(self, config=None):
        self.config = config or {
            "completeness_threshold": 0.95,
            "validity_threshold": 0.90,
        }
        
        # Expected source types
        self.expected_sources = ["kyc", "transactions", "alerts", "cases"]
```

---

## Layer 3: Feature Engineering

**File:** `layers/l3_feature_engineering.py`

```python
class FeatureEngineering:
    """
    PURPOSE:
    --------
    Create 50+ ML features from raw data.
    
    FEATURE CATEGORIES:
    -------------------
    1. VELOCITY (5 features)
       - txn_count_1h, txn_count_24h, txn_count_7d
       - amount_velocity_1h, amount_velocity_24h
       
    2. AGGREGATES (8 features)
       - total_amount, avg_amount, max_amount, min_amount
       - std_amount, median_amount
       - unique_counterparties, unique_countries
       
    3. RATIOS (6 features)
       - amount_to_avg_ratio
       - amount_to_historical_ratio
       - txn_frequency_ratio
       - counterparty_concentration
       - country_risk_ratio
       - time_deviation_ratio
       
    4. FLAGS (4 features)
       - is_round_amount
       - is_large_txn
       - is_new_counterparty
       - is_high_risk_country
       
    5. TEMPORAL (9 features)
       - hour_of_day, day_of_week, day_of_month
       - is_weekend, is_night, is_holiday
       - days_since_first_txn, days_since_last_txn
       - txn_time_gap
       
    6. BEHAVIORAL (4+ features)
       - deviation_from_pattern
       - account_age_days
       - kyc_risk_score
       - previous_alert_count
    
    MODIFICATION:
    -------------
    To add new feature:
    1. Add method: def compute_my_feature(self, df)
    2. Register in FEATURE_REGISTRY
    3. Add to category list (if new category)
    """
    
    FEATURE_REGISTRY = {
        "velocity": ["txn_count_1h", "txn_count_24h", ...],
        "aggregate": ["total_amount", "avg_amount", ...],
        # etc.
    }
```

---

## Layer 4: Preprocessing

**File:** `layers/l4_preprocessing.py`

```python
class Preprocessing:
    """
    PURPOSE:
    --------
    Transform features into 4 matrix versions for different algorithms.
    
    MATRIX VERSIONS:
    ----------------
    1. RAW (raw)
       - Original feature values
       - Use for: Trees, rules
       
    2. SCALED (scaled)
       - StandardScaler: (x - mean) / std
       - Use for: Distance-based (KNN, LOF)
       
    3. PCA (pca)
       - Dimensionality reduced
       - Use for: Visualization, Autoencoders
       
    4. ENCODED (encoded)
       - One-hot encoded categoricals
       - Use for: Neural networks
    
    KEY METHODS:
    ------------
    fit_transform(df) -> dict
        Returns:
        {
            "raw": np.array,
            "scaled": np.array,
            "pca": np.array,
            "encoded": np.array,
            "feature_names": list,
            "scaler": fitted scaler object,
            "pca_model": fitted PCA object,
        }
    
    MODIFICATION:
    -------------
    To add new matrix version:
    1. Add transformer class
    2. Add to transform_all() method
    3. Update MATRIX_VERSIONS in config
    """
```

---

## Layer 5: Detection Engine

**File:** `layers/l5_detection.py`

```python
class DetectionEngine:
    """
    PURPOSE:
    --------
    Run 26 anomaly detection algorithms across 8 categories.
    
    DETECTION METHODS BY CATEGORY:
    ------------------------------
    
    STATISTICAL (5 methods):
    ├── zscore        │ Standard score > 3σ
    ├── iqr           │ Outside 1.5×IQR
    ├── grubbs        │ Grubbs test for outliers
    ├── dixon         │ Dixon's Q test
    └── esd           │ Generalized ESD test
    
    DISTANCE (3 methods):
    ├── knn           │ K-Nearest Neighbors distance
    ├── mahalanobis   │ Mahalanobis distance
    └── lof           │ Local Outlier Factor
    
    DENSITY (4 methods):
    ├── dbscan        │ Density clustering
    ├── optics        │ Ordering points
    ├── hdbscan       │ Hierarchical DBSCAN
    └── cblof         │ Cluster-Based Local Outlier
    
    CLUSTERING (3 methods):
    ├── kmeans        │ K-Means outliers
    ├── gmm           │ Gaussian Mixture Model
    └── spectral      │ Spectral clustering
    
    TREES (2 methods):
    ├── isolation_forest    │ Isolation trees
    └── extended_isolation  │ Extended IF
    
    TIME-SERIES (3 methods):
    ├── stl           │ Seasonal decomposition
    ├── arima         │ ARIMA residuals
    └── prophet       │ Prophet anomalies
    
    GRAPH (4 methods):
    ├── pagerank      │ PageRank scores
    ├── hits          │ HITS algorithm
    ├── community     │ Community detection
    └── centrality    │ Node centrality
    
    DEEP LEARNING (2 methods):
    ├── autoencoder   │ Reconstruction error
    └── vae           │ Variational AE
    
    OUTPUT:
    -------
    dict: {
        "zscore": np.array (anomaly scores 0-1),
        "isolation_forest": np.array,
        "autoencoder": np.array,
        # ... all 26 methods
    }
    
    MODIFICATION:
    -------------
    To add new detection method:
    
    1. Add method in l5_detection.py:
       def my_method(self, matrix, **kwargs):
           # Your algorithm
           scores = compute_anomaly_scores(matrix)
           return self.normalize_scores(scores)
    
    2. Register in __init__:
       self.methods["my_method"] = self.my_method
    
    3. Add to config.py:
       "my_category": ["my_method"],
    """
    
    def __init__(self):
        # Register all methods
        self.methods = {
            # Statistical
            "zscore": self.zscore_detector,
            "iqr": self.iqr_detector,
            # ... etc
        }
```

---

## Layer 6: Ensemble Fusion

**File:** `layers/l6_ensemble.py`

```python
class EnsembleFusion:
    """
    PURPOSE:
    --------
    Combine 26 detection scores into final risk score.
    
    FUSION METHODS:
    ---------------
    1. WEIGHTED_AVERAGE (default)
       - Weights based on method performance
       - final = Σ(weight_i × score_i) / Σ(weight_i)
       
    2. MAX
       - Take maximum score across methods
       - final = max(all scores)
       
    3. VOTING
       - Count methods flagging as anomaly
       - final = count(score > threshold) / total_methods
       
    4. STACKING
       - Meta-model trained on method scores
       - final = meta_model.predict(all_scores)
    
    RISK TIER MAPPING:
    ------------------
    Score >= 0.9  →  CRITICAL (immediate action)
    Score >= 0.7  →  HIGH (same-day review)
    Score >= 0.4  →  MEDIUM (weekly review)
    Score <  0.4  →  LOW (monthly sampling)
    
    MODIFICATION:
    -------------
    To change tier thresholds:
    1. Edit TIER_THRESHOLDS in config.py
    
    To add fusion method:
    1. Add method: def my_fusion(self, scores_dict)
    2. Register in FUSION_METHODS
    """
    
    TIER_THRESHOLDS = {
        "Critical": 0.9,
        "High": 0.7,
        "Medium": 0.4,
        "Low": 0.0,
    }
```

---

## Layer 7: Output Manager

**File:** `layers/l7_output.py`

```python
class OutputManager:
    """
    PURPOSE:
    --------
    Generate alerts, narratives, and audit trail.
    
    COMPONENTS:
    -----------
    1. INVESTIGATION QUEUE
       - Prioritized list of alerts
       - Capacity: 1000 (configurable)
       - Sorted by risk score
       
    2. NARRATIVE GENERATOR
       - SHAP-style explanations
       - Natural language summary
       - Contributing factors
       
    3. AUDIT TRAIL
       - All decisions logged
       - Timestamps
       - User actions
       - Compliance-ready
    
    ALERT STRUCTURE:
    ----------------
    {
        "alert_id": "ALT-000001",
        "timestamp": "2026-02-07T12:00:00",
        "entity_id": "CUST-12345",
        "risk_score": 0.95,
        "risk_tier": "Critical",
        "narrative": "High-risk transaction detected...",
        "contributing_factors": [
            {"factor": "amount_zscore", "contribution": 0.35},
            {"factor": "velocity_anomaly", "contribution": 0.25},
        ],
        "recommended_action": "Immediate review required",
        "status": "Open",
    }
    
    MODIFICATION:
    -------------
    To customize narratives:
    1. Edit NARRATIVE_TEMPLATES in l7_output.py
    
    To change queue capacity:
    1. Edit INVESTIGATION_QUEUE_CAPACITY in config.py
    """
```

---

## Pipeline Orchestrator

**File:** `pipeline.py`

```python
class VanguardPipeline:
    """
    PURPOSE:
    --------
    Orchestrate all 7 layers in sequence.
    
    USAGE:
    ------
    from pipeline import VanguardPipeline
    
    pipeline = VanguardPipeline()
    result = pipeline.run(
        sources={"transactions": df, ...},
        detection_methods=None,  # None = all 26
        ensemble_method="weighted_average"
    )
    
    if result.success:
        print(f"Alerts: {result.alerts_generated}")
        pipeline.save_results()
        pipeline.export_alerts("xlsx")
    
    EXECUTION FLOW:
    ---------------
    1. L1-2: Ingest → DQ Score → Cleanse
    2. L3:   Generate 50+ features
    3. L4:   Create 4 matrix versions
    4. L5:   Run detection methods
    5. L6:   Ensemble → Risk tiers
    6. L7:   Generate alerts
    
    RESULT OBJECT:
    --------------
    PipelineResult:
        success: bool
        records_processed: int
        features_generated: int
        methods_run: int
        dq_score: float
        alerts_generated: int
        tier_distribution: dict
        execution_time_ms: float
    """
```

---

**FCDAI Team** | 2026
