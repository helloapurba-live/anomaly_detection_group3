# FCDAI Anomaly Auto Detection Tool - Documentation

## 📁 Project Structure Guide

This documentation covers both Dash applications:
- **Port 8070**: Original `project_vanguard` (Apex Edition)
- **Port 8071**: New `project_vanguard_7layer` (7-Layer Edition)

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FCDAI AML Platform                          │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   app.py    │  │  config.py  │  │ pipeline.py │  │   pages/    │ │
│  │ (Dash App)  │  │ (Settings)  │  │ (ML Logic)  │  │  (UI Pages) │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│                                                                     │
│  ┌─────────────────────────────┐  ┌─────────────────────────────┐   │
│  │         layers/             │  │          utils/             │   │
│  │  L1-L7 Detection Modules    │  │  Data I/O, Logging, etc.   │   │
│  └─────────────────────────────┘  └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📄 File-by-File Documentation

### 1. `app.py` - Main Application Entry Point

```python
# =============================================================================
# IMPORTS
# =============================================================================
import dash                          # Core Dash framework
from dash import html, dcc           # HTML components and Core Components
import dash_mantine_components as dmc # Premium UI components
from dash_iconify import DashIconify # Icon library
import diskcache                     # Caching for performance
from config import APP, THEME, PATHS # Configuration settings
```

**Key Components:**

| Component | Purpose | Modify When |
|-----------|---------|-------------|
| `dash.Dash()` | App initialization | Changing app settings |
| `sidebar` | Navigation menu | Adding/removing pages |
| `app.layout` | Main page structure | Changing overall layout |
| `MantineProvider` | Theme provider | Changing dark/light mode |

**How to Modify:**

1. **Add New Navigation Link:**
```python
# In sidebar navigation section, add:
create_nav_link("mdi:icon-name", "Page Name", "/page-path"),
```

2. **Change Application Title:**
```python
# In config.py, modify:
TITLE: str = "Your New Title"
```

3. **Change Port:**
```python
# In config.py, modify:
PORT: int = 8080  # Change to desired port
```

---

### 2. `config.py` - Configuration Settings

```python
# =============================================================================
# CONFIGURATION STRUCTURE
# =============================================================================

@dataclass
class PathConfig:
    """
    PURPOSE: Define all file system paths
    MODIFY: When adding new data directories
    
    BASE: Root project directory
    DATA: Main data storage
    DATA_SOURCES: Uploaded source files (KYC, TXN, Alerts, Cases)
    MODELS: Saved ML model files
    LOGS: Application logs
    OUTPUT: Pipeline output files
    """
    BASE: Path = BASE_DIR
    DATA: Path = BASE_DIR / "data"
    # ... etc


@dataclass
class ThemeConfig:
    """
    PURPOSE: Define UI colors and styling
    MODIFY: When changing visual appearance
    
    PRIMARY: Main accent color (buttons, highlights)
    SECONDARY: Secondary accent color
    SUCCESS/WARNING/DANGER: Status colors
    DARK_BG: Page background color
    DARK_BG_CARD: Card/panel background color
    DARK_BORDER: Border colors
    """
    PRIMARY: str = "#00D4FF"      # Cyan - Change for different accent
    SECONDARY: str = "#7C3AED"   # Purple
    # ... etc


@dataclass  
class AppConfig:
    """
    PURPOSE: Application runtime settings
    MODIFY: When changing app behavior
    
    TITLE: Browser tab title
    SUBTITLE: Shown in sidebar
    VERSION: Version number
    DEBUG: Flask debug mode (False in production!)
    HOST: Server host (0.0.0.0 for network access)
    PORT: Server port number
    """
    TITLE: str = "FCDAI Anomaly Auto Detection Tool"
    PORT: int = 8071
    DEBUG: bool = False
    # ... etc
```

**Common Modifications:**

| Change | Location | Example |
|--------|----------|---------|
| App name | `AppConfig.TITLE` | `"My AML Tool"` |
| Port | `AppConfig.PORT` | `8080` |
| Theme color | `ThemeConfig.PRIMARY` | `"#FF5500"` |
| Data path | `PathConfig.DATA` | `Path("/custom/path")` |

---

### 3. `pipeline.py` - ML Pipeline Orchestration

```python
# =============================================================================
# PIPELINE STRUCTURE
# =============================================================================

class VanguardPipeline:
    """
    PURPOSE: Orchestrate all 7 detection layers
    MODIFY: When changing pipeline flow or adding layers
    
    LAYERS:
    -------
    Layer 1-2: Data ingestion + Quality scoring
    Layer 3:   Feature engineering (50+ features)
    Layer 4:   Preprocessing (4 matrix versions)
    Layer 5:   Detection methods (26 algorithms)
    Layer 6:   Ensemble fusion (score combination)
    Layer 7:   Output (alerts, narratives, audit)
    """
    
    def __init__(self):
        """
        Initialize all layer modules.
        MODIFY: Add new layer imports here
        """
        self.ingestion = DataIngestion()      # L1-2
        self.features = FeatureEngineering()   # L3
        self.preprocess = Preprocessing()      # L4
        self.detection = DetectionEngine()     # L5
        self.ensemble = EnsembleFusion()       # L6
        self.output = OutputManager()          # L7
    
    def run(self, sources, detection_methods=None, ensemble_method="weighted_average"):
        """
        Execute the full pipeline.
        
        PARAMETERS:
        -----------
        sources: dict
            {"kyc": DataFrame, "transactions": DataFrame, ...}
        detection_methods: list
            ["zscore", "isolation_forest", ...]  
            Pass None to run all 26 methods
        ensemble_method: str
            "weighted_average", "max", "voting", or "stacking"
        
        RETURNS:
        --------
        PipelineResult object with:
            - success: bool
            - records_processed: int
            - features_generated: int
            - methods_run: int
            - alerts_generated: int
            - tier_distribution: dict
            - execution_time_ms: float
        """
```

**How to Add a New Detection Method:**

```python
# 1. In layers/l5_detection.py, add method:
def my_new_detector(self, matrix):
    """My custom detection algorithm."""
    scores = # ... your logic
    return scores

# 2. In config.py, add to DETECTION_METHODS:
"my_category": ["zscore", "my_new_method"],

# 3. Register in DetectionEngine:
self.methods["my_new_method"] = self.my_new_detector
```

---

### 4. `pages/` - UI Pages Directory

Each page follows this structure:

```python
# =============================================================================
# PAGE REGISTRATION (Required for Dash multi-page)
# =============================================================================
import dash
dash.register_page(
    __name__,               # Unique identifier
    path="/page-url",       # URL path
    title="Page Title",     # Browser tab title
    name="Sidebar Label"    # Navigation label
)


# =============================================================================
# PAGE LAYOUT
# =============================================================================
layout = dmc.Container([
    # Page content components
    dmc.Title("Page Title"),
    # ... more components
])


# =============================================================================
# CALLBACKS (Interactive behavior)
# =============================================================================
@callback(
    Output("output-id", "property"),     # What to update
    Input("input-id", "property"),       # What triggers update
    State("state-id", "property"),       # Additional data (no trigger)
)
def callback_function(input_value, state_value):
    """
    PURPOSE: Handle user interaction
    TRIGGERED BY: Changes to Input components
    RETURNS: New value for Output component
    """
    # Your logic here
    return new_value
```

**Page Files:**

| File | URL | Purpose |
|------|-----|---------|
| `command_center.py` | `/` | Dashboard KPIs |
| `data_sources.py` | `/sources` | File upload |
| `pipeline_run.py` | `/pipeline` | Execute pipeline |
| `layer_view.py` | `/layers` | Layer details |
| `investigation_queue.py` | `/queue` | Alert triage |
| `narratives.py` | `/narratives` | SHAP explanations |
| `audit_trail.py` | `/audit` | Audit logs |
| `system_health.py` | `/health` | System monitoring |

---

### 5. `layers/` - Detection Layer Modules

```python
# =============================================================================
# LAYER STRUCTURE PATTERN
# =============================================================================

class Layer:
    """
    Each layer follows this pattern:
    
    __init__: Initialize resources
    process(): Main processing method
    validate(): Input validation
    get_stats(): Return layer metrics
    """
    
    def __init__(self, config=None):
        """Setup configuration and resources."""
        self.config = config or default_config
        
    def process(self, data):
        """
        Main processing logic.
        
        PARAMETERS:
        -----------
        data: Input from previous layer
        
        RETURNS:
        --------
        Processed data for next layer
        """
        pass
```

**Layer Files:**

| File | Layer | Purpose |
|------|-------|---------|
| `l1_l2_ingestion.py` | 1-2 | Load + validate data |
| `l3_feature_engineering.py` | 3 | Create ML features |
| `l4_preprocessing.py` | 4 | Scale + transform |
| `l5_detection.py` | 5 | Run anomaly detection |
| `l6_ensemble.py` | 6 | Combine scores |
| `l7_output.py` | 7 | Generate alerts |

---

## 🔧 Common Modifications

### Add a New Page

1. **Create page file** in `pages/`:
```python
# pages/my_page.py
import dash
from dash import html
import dash_mantine_components as dmc

dash.register_page(__name__, path="/my-page", name="My Page")

layout = dmc.Container([
    dmc.Title("My New Page"),
    dmc.Text("Content here"),
])
```

2. **Add navigation link** in `app.py`:
```python
create_nav_link("mdi:star", "My Page", "/my-page"),
```

### Change Theme Colors

In `config.py`:
```python
@dataclass
class ThemeConfig:
    PRIMARY: str = "#FF5500"  # Change accent color
    DARK_BG: str = "#1a1a2e"  # Change background
```

### Add Detection Method

In `layers/l5_detection.py`:
```python
def my_detector(self, matrix):
    # Your algorithm
    scores = np.random.rand(len(matrix))  # Example
    return scores

# Register in __init__:
self.methods["my_detector"] = self.my_detector
```

In `config.py`:
```python
"my_category": ["my_detector"],
```

---

## 🚀 Running the Applications

```bash
# Port 8070 - Original Apex Edition
cd project_vanguard
python app.py

# Port 8071 - 7-Layer Edition  
cd project_vanguard_7layer
python app.py
```

---

## 📊 Comparison: 8070 vs 8071

| Feature | 8070 (Apex) | 8071 (7-Layer) |
|---------|-------------|----------------|
| Layers | Single pipeline | 7 distinct layers |
| Methods | ~10 | 26 across 8 categories |
| Features | Basic | 50+ engineered |
| UI | 7 pages | 8 pages |
| Focus | General AML | Deep anomaly detection |

---

## 📝 Quick Reference

### Callback Pattern
```python
@callback(Output, Input, State)
def fn(input_val, state_val):
    return output_val
```

### DMC Components
```python
dmc.Button("Click")           # Button
dmc.Text("Hello")             # Text
dmc.Paper([...], p="md")      # Card container
dmc.Grid([dmc.Col(...)])      # Grid layout
dmc.Badge("Status")           # Status badge
```

### Plotly Charts
```python
fig = px.bar(df, x="col", y="value")
dcc.Graph(figure=fig)
```

---

**FCDAI Team** | 2026
