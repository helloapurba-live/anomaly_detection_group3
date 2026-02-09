# FCDAI Annotated Code Reference

## 📘 This document provides LINE-BY-LINE annotations for key files

---

# Part 1: `app.py` - 7-Layer Edition (Port 8071)

```python
"""
Project Vanguard 7-Layer Edition - Main Application
====================================================
Air-gapped Dash Mantine Components UI for the 7-layer pipeline.

Author: Project Vanguard Team
"""

# =============================================================================
# IMPORTS SECTION
# =============================================================================
# Line 9-14: Standard imports
import dash                          # Main Dash framework
from dash import html, dcc, page_registry, page_container
    # html: HTML components (div, p, h1, etc.)
    # dcc: Core components (graphs, dropdowns, inputs)
    # page_registry: Registry of all pages
    # page_container: Container where pages render
import dash_mantine_components as dmc # Premium UI library
    # Provides: Button, Paper, Grid, Text, Badge, etc.
from dash_iconify import DashIconify # Icon component
    # Usage: DashIconify(icon="mdi:icon-name", width=20)
import diskcache                     # Caching for callbacks
from pathlib import Path             # Cross-platform paths
import sys

# Line 17: Ensure project root is importable
sys.path.insert(0, str(Path(__file__).parent))

# Line 19: Import project configuration
from config import APP, THEME, PATHS
    # APP: Application settings (title, port, debug)
    # THEME: Color scheme (PRIMARY, DARK_BG, etc.)
    # PATHS: File system paths (DATA, LOGS, etc.)


# =============================================================================
# DASH APPLICATION INITIALIZATION
# =============================================================================
# Lines 25-34: Create the Dash app instance
app = dash.Dash(
    __name__,                           # Module name for assets
    use_pages=True,                     # Enable multi-page support
    pages_folder='pages',               # Directory containing page files
    suppress_callback_exceptions=True,  # Allow callbacks to missing components
                                        # (needed for dynamic content)
    assets_folder='assets',             # CSS/JS/images folder
    serve_locally=True,                 # CRITICAL: Serve all assets locally
                                        # Required for air-gapped operation
    title=APP.TITLE,                    # Browser tab title
    update_title=None,                  # Disable "Updating..." title text
)


# =============================================================================
# NAVIGATION LINK COMPONENT
# =============================================================================
# Lines 40-48: Reusable function to create sidebar nav links
def create_nav_link(icon: str, label: str, href: str) -> dmc.NavLink:
    """
    Create a styled navigation link.
    
    PARAMETERS:
    -----------
    icon : str
        MDI icon name, e.g., "mdi:view-dashboard"
        Find icons: https://icon-sets.iconify.design/mdi/
    label : str
        Text displayed in the sidebar
    href : str
        URL path this link navigates to
    
    RETURNS:
    --------
    dmc.NavLink component
    
    MODIFICATION NOTES:
    -------------------
    - Change "active" to "exact" for exact URL matching
    - Modify style dict to change appearance
    - Add "rightSection" for badges/notifications
    
    EXAMPLE:
    --------
    create_nav_link("mdi:cog", "Settings", "/settings")
    """
    return dmc.NavLink(
        label=label,                                    # Link text
        href=href,                                      # Destination URL
        leftSection=DashIconify(icon=icon, width=20),   # Icon on left
        active="exact",                                 # Highlight when active
        style={"borderRadius": "6px", "marginBottom": "4px"},  # Styling
    )


# =============================================================================
# SIDEBAR LAYOUT
# =============================================================================
# Lines 54-147: Define the sidebar layout
sidebar = dmc.Paper(
    [
        # -----------------------------------------------------------------
        # LOGO SECTION (Lines 57-77)
        # -----------------------------------------------------------------
        dmc.Group(
            [
                # Themed icon with gradient
                dmc.ThemeIcon(
                    DashIconify(icon="mdi:shield-lock", width=28),
                    size="xl",           # Extra large
                    radius="md",         # Medium border radius
                    variant="gradient",  # Gradient background
                    gradient={"from": "cyan", "to": "indigo"},  # Gradient colors
                    # MODIFY: Change gradient colors here
                ),
                # Text stack (title + subtitle)
                dmc.Stack(
                    [
                        dmc.Text("FCDAI", size="lg", fw=700, c="white"),
                            # fw=700: Font weight (bold)
                            # c="white": Color
                            # MODIFY: Change "FCDAI" to your brand name
                        dmc.Text("Anomaly Auto Detection", size="xs", c="dimmed"),
                            # c="dimmed": Muted gray color
                            # MODIFY: Change subtitle text here
                    ],
                    gap=0,  # No gap between elements
                ),
            ],
            gap="sm",   # Small gap between icon and text
            px="md",    # Horizontal padding: medium
            py="lg",    # Vertical padding: large
        ),
        
        dmc.Divider(color=THEME.DARK_BORDER),  # Horizontal line
        
        # -----------------------------------------------------------------
        # NAVIGATION: PIPELINE SECTION (Lines 82-92)
        # -----------------------------------------------------------------
        dmc.Stack(
            [
                # Section header
                dmc.Text("PIPELINE", size="xs", c="dimmed", fw=500, px="md", pt="md"),
                # Navigation links - MODIFY: Add/remove pages here
                create_nav_link("mdi:view-dashboard", "Command Center", "/"),
                create_nav_link("mdi:database-import", "Data Sources", "/sources"),
                create_nav_link("mdi:rocket-launch", "Pipeline Run", "/pipeline"),
                create_nav_link("mdi:layers-triple", "Layer View", "/layers"),
            ],
            gap=2,    # 2px gap between items
            px="sm",  # Padding x: small
        ),
        
        # -----------------------------------------------------------------
        # NAVIGATION: RESULTS SECTION (Lines 95-104)
        # -----------------------------------------------------------------
        dmc.Stack(
            [
                dmc.Text("RESULTS", size="xs", c="dimmed", fw=500, px="md", pt="md"),
                create_nav_link("mdi:alert-circle", "Investigation Queue", "/queue"),
                create_nav_link("mdi:file-document", "Narratives", "/narratives"),
                create_nav_link("mdi:history", "Audit Trail", "/audit"),
            ],
            gap=2,
            px="sm",
        ),
        
        # -----------------------------------------------------------------
        # NAVIGATION: ADMIN SECTION (Lines 107-114)
        # -----------------------------------------------------------------
        dmc.Stack(
            [
                dmc.Text("ADMIN", size="xs", c="dimmed", fw=500, px="md", pt="md"),
                create_nav_link("mdi:server", "System Health", "/health"),
                # ADD MORE ADMIN PAGES HERE:
                # create_nav_link("mdi:cog", "Settings", "/settings"),
            ],
            gap=2,
            px="sm",
        ),
        
        dmc.Space(h="xl"),  # Vertical spacer
        
        # -----------------------------------------------------------------
        # STATUS BADGES (Lines 120-134)
        # -----------------------------------------------------------------
        dmc.Paper(
            [
                dmc.Group(
                    [
                        # Badges showing capabilities
                        dmc.Badge("7 Layers", color="cyan", variant="light"),
                        dmc.Badge("26 Methods", color="grape", variant="light"),
                        # MODIFY: Update these when adding layers/methods
                    ],
                    gap="xs",
                ),
            ],
            p="sm",    # Padding: small
            mx="sm",   # Margin x: small
            radius="md",  # Border radius
            style={"backgroundColor": "rgba(0,0,0,0.2)"}  # Semi-transparent
        ),
    ],
    # SIDEBAR CONTAINER STYLES
    style={
        "width": "260px",                           # Fixed width
        "height": "100vh",                          # Full viewport height
        "position": "fixed",                        # Stays in place on scroll
        "top": 0,
        "left": 0,
        "backgroundColor": THEME.DARK_BG_CARD,      # Background color
        "borderRight": f"1px solid {THEME.DARK_BORDER}",  # Right border
        "overflowY": "auto",                        # Scroll if content overflows
        # MODIFY: Change width, colors here
    },
    radius=0,  # No border radius (sharp corners)
)


# =============================================================================
# MAIN LAYOUT
# =============================================================================
# Lines 153-173: Define the overall app layout
app.layout = dmc.MantineProvider(
    # MantineProvider: Required wrapper for DMC components
    # Provides theming and dark mode support
    [
        dmc.Box(
            [
                sidebar,  # Left sidebar (fixed position)
                
                dmc.Box(
                    [
                        page_container  # Dynamic page content renders here
                        # This automatically shows the current page's layout
                    ],
                    style={
                        "marginLeft": "260px",  # Same as sidebar width
                        "padding": "20px",      # Content padding
                        "minHeight": "100vh",   # Full height
                        "backgroundColor": THEME.DARK_BG,  # Page background
                        # MODIFY: Change marginLeft if sidebar width changes
                    }
                ),
            ]
        ),
    ],
    forceColorScheme="dark",  # Force dark mode
    # Options: "dark", "light", "auto"
)


# =============================================================================
# SERVER REFERENCE
# =============================================================================
server = app.server  # Flask server instance
    # Used for:
    # - Production deployment (gunicorn, waitress)
    # - Adding Flask routes
    # - Session management


# =============================================================================
# APPLICATION ENTRY POINT
# =============================================================================
if __name__ == "__main__":
    # Startup banner
    print("""
============================================================
  FCDAI ANOMALY AUTO DETECTION TOOL
  Advanced AML Detection Pipeline (7-Layer Edition)
============================================================
  🔒 AIR-GAPPED MODE: No external connections
  📊 LAYERS: 7 | METHODS: 26 | CATEGORIES: 8
  🌐 Running at: http://127.0.0.1:8071

  Press Ctrl+C to stop the server
============================================================
""")
    # Run the server
    app.run(
        debug=APP.DEBUG,  # Debug mode (auto-reload on changes)
        host=APP.HOST,    # Host (0.0.0.0 for network access)
        port=APP.PORT,    # Port number
    )
    # PRODUCTION NOTE:
    # In production, use: gunicorn app:server -b 0.0.0.0:8071
```

---

# Part 2: Page Structure Pattern

```python
"""
PAGE TEMPLATE
=============
Copy this to create new pages.
"""

import dash
from dash import html, dcc, callback, Input, Output, State
import dash_mantine_components as dmc
from dash_iconify import DashIconify

# =============================================================================
# STEP 1: REGISTER THE PAGE
# =============================================================================
dash.register_page(
    __name__,                    # Unique identifier (use __name__)
    path="/my-page",             # URL path (must start with /)
    title="My Page Title",       # Browser tab title
    name="My Page",              # Displayed in page_registry
    order=5,                     # Order in navigation (optional)
)


# =============================================================================
# STEP 2: DEFINE THE LAYOUT
# =============================================================================
# The 'layout' variable is REQUIRED - Dash looks for this
layout = dmc.Container(
    [
        # Page Header
        dmc.Group(
            [
                dmc.Title("My Page", order=2),  # order=1 is largest
                dmc.Badge("Status", color="cyan"),
            ],
            justify="space-between",  # Space elements apart
            mb="lg",  # Margin bottom: large
        ),
        
        dmc.Divider(mb="lg"),
        
        # Page Content
        dmc.Grid(
            [
                dmc.GridCol(
                    dmc.Paper(
                        [
                            dmc.Text("Column 1"),
                            dmc.Button("Click Me", id="my-button"),
                        ],
                        p="md",  # Padding
                        radius="md",  # Border radius
                        withBorder=True,
                    ),
                    span=6,  # 6 out of 12 columns
                ),
                dmc.GridCol(
                    dmc.Paper(
                        [
                            dmc.Text("Column 2"),
                            html.Div(id="output-div"),  # Callback target
                        ],
                        p="md",
                        radius="md",
                        withBorder=True,
                    ),
                    span=6,
                ),
            ],
        ),
    ],
    fluid=True,  # Full width
)


# =============================================================================
# STEP 3: ADD CALLBACKS (Interactive Behavior)
# =============================================================================
@callback(
    # OUTPUT: What to update
    Output("output-div", "children"),  # component-id, property
    
    # INPUT: What triggers the update
    Input("my-button", "n_clicks"),    # Fires on click
    
    # STATE: Additional data (doesn't trigger)
    # State("some-input", "value"),
    
    prevent_initial_call=True,  # Don't run on page load
)
def handle_button_click(n_clicks):
    """
    Callback function.
    
    TRIGGERED BY: Button click
    
    PARAMETERS:
    -----------
    n_clicks : int
        Number of times button was clicked
    
    RETURNS:
    --------
    str or dash component
        New content for output-div
    
    NOTES:
    ------
    - Return must match Output property type
    - For "children", return str, list, or components
    - For "value", return the value type
    - For "figure", return plotly figure
    """
    if n_clicks is None:
        return "Not clicked yet"
    return f"Clicked {n_clicks} times!"
```

---

# Part 3: Callback Patterns Reference

```python
# =============================================================================
# PATTERN 1: Simple Input -> Output
# =============================================================================
@callback(
    Output("display", "children"),
    Input("text-input", "value"),
)
def update_display(text):
    return f"You typed: {text}"


# =============================================================================
# PATTERN 2: Multiple Inputs
# =============================================================================
@callback(
    Output("result", "children"),
    Input("input-a", "value"),
    Input("input-b", "value"),
)
def combine_inputs(a, b):
    return f"A={a}, B={b}"


# =============================================================================
# PATTERN 3: Multiple Outputs
# =============================================================================
@callback(
    Output("output-1", "children"),
    Output("output-2", "style"),
    Input("button", "n_clicks"),
)
def multiple_outputs(n):
    text = f"Clicked {n}"
    style = {"color": "green" if n % 2 == 0 else "red"}
    return text, style  # Return tuple matching Output order


# =============================================================================
# PATTERN 4: Using State (No Trigger)
# =============================================================================
@callback(
    Output("result", "children"),
    Input("submit-btn", "n_clicks"),    # Triggers callback
    State("data-input", "value"),       # Only reads value
)
def use_state(n_clicks, input_value):
    # n_clicks triggers, input_value is just read
    return f"Submitted: {input_value}"


# =============================================================================
# PATTERN 5: Prevent Initial Call
# =============================================================================
@callback(
    Output("output", "children"),
    Input("button", "n_clicks"),
    prevent_initial_call=True,  # Skip on page load
)
def no_initial(n):
    return f"Clicked {n}"


# =============================================================================
# PATTERN 6: Chart Updates
# =============================================================================
import plotly.express as px

@callback(
    Output("my-graph", "figure"),
    Input("dropdown", "value"),
)
def update_chart(selected_value):
    # Filter data based on selection
    filtered_df = df[df["column"] == selected_value]
    
    # Create figure
    fig = px.bar(filtered_df, x="x", y="y")
    
    # Apply theme
    fig.update_layout(
        template="plotly_dark",
        paper_bgcolor="rgba(0,0,0,0)",
        plot_bgcolor="rgba(0,0,0,0)",
    )
    
    return fig


# =============================================================================
# PATTERN 7: Loading State
# =============================================================================
from dash import dcc

# In layout:
# dcc.Loading(
#     id="loading",
#     children=[html.Div(id="output")],
#     type="circle",  # or "default", "dot", "cube"
# )

@callback(
    Output("output", "children"),
    Input("button", "n_clicks"),
)
def slow_operation(n):
    import time
    time.sleep(2)  # Shows loading spinner
    return "Done!"
```

---

# Part 4: Common DMC Components

```python
# =============================================================================
# TEXT & HEADINGS
# =============================================================================
dmc.Title("Heading", order=1)  # order: 1-6 (h1-h6)
dmc.Text("Normal text", size="lg", fw=500, c="dimmed")
    # size: "xs", "sm", "md", "lg", "xl"
    # fw: font weight (100-900)
    # c: color ("dimmed", "red", hex, etc.)


# =============================================================================
# BUTTONS
# =============================================================================
dmc.Button("Click", id="btn-id", variant="filled", color="cyan")
    # variant: "filled", "outline", "light", "subtle"
    # color: any mantine color

dmc.Button("With Icon", leftSection=DashIconify(icon="mdi:save"))


# =============================================================================
# INPUTS
# =============================================================================
dmc.TextInput(id="input", label="Label", placeholder="Enter...")
dmc.Select(id="select", data=["A", "B", "C"], value="A")
dmc.MultiSelect(id="multi", data=["A", "B"], value=["A"])
dmc.NumberInput(id="num", value=10, min=0, max=100)
dmc.Slider(id="slider", min=0, max=100, value=50)


# =============================================================================
# LAYOUT
# =============================================================================
dmc.Grid([
    dmc.GridCol(content, span=6),  # 6/12 width
    dmc.GridCol(content, span=6),
])

dmc.Group([a, b, c], gap="md")  # Horizontal group
dmc.Stack([a, b, c], gap="md")  # Vertical stack

dmc.Paper(content, p="md", radius="md", withBorder=True)
dmc.Card(content, shadow="sm", padding="lg")


# =============================================================================
# FEEDBACK
# =============================================================================
dmc.Badge("Status", color="green", variant="light")
dmc.Alert("Message", title="Alert", color="red")
dmc.Progress(value=75, size="lg")
dmc.Loader(size="lg", variant="dots")


# =============================================================================
# DATA DISPLAY
# =============================================================================
dmc.Table(
    data={
        "head": ["Name", "Value"],
        "body": [["A", 1], ["B", 2]],
    }
)

dmc.Accordion([
    dmc.AccordionItem(
        value="item-1",
        children=[
            dmc.AccordionControl("Section 1"),
            dmc.AccordionPanel("Content"),
        ]
    ),
])

dmc.Tabs([
    dmc.TabsList([
        dmc.TabsTab("Tab 1", value="1"),
        dmc.TabsTab("Tab 2", value="2"),
    ]),
    dmc.TabsPanel("Panel 1", value="1"),
    dmc.TabsPanel("Panel 2", value="2"),
], value="1")
```

---

**FCDAI Team** | 2026
