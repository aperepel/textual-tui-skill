# Textual Layout Patterns

Common layout recipes for Textual applications.

## Contents
- [Layout Types](#layout-types) — Vertical, Horizontal, Grid, Dock
- [Common Patterns](#common-patterns) — Split Screen, Three-Column, Dashboard Grid, Centred, Scrollable, Tabbed
- [Sizing & Spacing](#sizing--spacing) — units, padding, margin (see styling.md)
- [Responsive Layouts](#responsive-layouts) — container queries, conditional layouts
- [Advanced Patterns](#advanced-patterns) — Modal overlay, Sidebar Toggle, Masonry, Resizable Split
- [Layout Debugging](#layout-debugging)

## Layout Types

### Vertical (Default)

Stack widgets vertically:
```python
from textual.app import App, ComposeResult
from textual.widgets import Label

class VerticalApp(App):
    def compose(self) -> ComposeResult:
        yield Label("Top")
        yield Label("Middle")
        yield Label("Bottom")

# Or explicit CSS
CSS = """
Screen {
    layout: vertical;
}
"""
```

### Horizontal

Arrange widgets side-by-side:
```python
from textual.containers import Horizontal

def compose(self) -> ComposeResult:
    with Horizontal():
        yield Label("Left")
        yield Label("Center")
        yield Label("Right")

# Or via CSS
CSS = """
Horizontal {
    height: 100%;
}

Horizontal > Label {
    width: 1fr;  /* Equal distribution */
}
"""
```

### Grid

Create grid layouts:
```python
class GridApp(App):
    CSS = """
    Screen {
        layout: grid;
        grid-size: 3 2;  /* 3 columns, 2 rows */
        grid-gutter: 1;
    }
    
    .cell {
        border: solid $accent;
        height: 100%;
    }
    """
    
    def compose(self) -> ComposeResult:
        for i in range(6):
            yield Label(f"Cell {i+1}", classes="cell")
```

Advanced grid with spanning:
```python
CSS = """
Screen {
    layout: grid;
    grid-size: 4;  /* 4 columns, auto rows */
}

#header {
    column-span: 4;  /* Spans all columns */
}

#sidebar {
    row-span: 2;  /* Spans 2 rows */
}
"""
```

### Dock Layout

Dock widgets to edges:
```python
from textual.widgets import Header, Footer

class DockedApp(App):
    def compose(self) -> ComposeResult:
        yield Header()  # Docked to top
        yield Label("Content")  # Takes remaining space
        yield Footer()  # Docked to bottom

# Custom docking
CSS = """
#sidebar {
    dock: left;
    width: 30;
}

#toolbar {
    dock: top;
    height: 3;
}
"""
```

## Common Patterns

### Split Screen (Vertical)

Two panels side-by-side:
```python
class SplitScreen(App):
    CSS = """
    Screen {
        layout: horizontal;
    }
    
    #left-panel {
        width: 30%;
        border-right: solid $accent;
    }
    
    #right-panel {
        width: 70%;
    }
    """
    
    def compose(self) -> ComposeResult:
        with Container(id="left-panel"):
            yield Label("Sidebar")
        with Container(id="right-panel"):
            yield Label("Main content")
```

### Split Screen (Horizontal)

Two panels stacked:
```python
class SplitScreenHorizontal(App):
    CSS = """
    Screen {
        layout: vertical;
    }
    
    #top-panel {
        height: 50%;
        border-bottom: solid $accent;
    }
    
    #bottom-panel {
        height: 50%;
    }
    """
    
    def compose(self) -> ComposeResult:
        with Container(id="top-panel"):
            yield Label("Top content")
        with Container(id="bottom-panel"):
            yield Label("Bottom content")
```

### Three-Column Layout

Classic sidebar-content-sidebar:
```python
class ThreeColumn(App):
    CSS = """
    Screen {
        layout: horizontal;
    }
    
    #left-sidebar {
        width: 20;
    }
    
    #content {
        width: 1fr;  /* Take remaining space */
    }
    
    #right-sidebar {
        width: 25;
    }
    """
    
    def compose(self) -> ComposeResult:
        with Container(id="left-sidebar"):
            yield Label("Menu")
        with Container(id="content"):
            yield Label("Main")
        with Container(id="right-sidebar"):
            yield Label("Info")
```

### Dashboard Grid

Grid-based dashboard:
```python
class Dashboard(App):
    CSS = """
    Screen {
        layout: grid;
        grid-size: 2 3;  /* 2 columns, 3 rows */
        grid-gutter: 1 2;  /* vertical horizontal */
    }
    
    #header {
        column-span: 2;
        height: 3;
    }
    
    .metric-card {
        border: solid $primary;
        padding: 1;
    }
    """
    
    def compose(self) -> ComposeResult:
        yield Header(id="header")
        yield Static("Users: 1,234", classes="metric-card")
        yield Static("Revenue: $12K", classes="metric-card")
        yield Static("Growth: +15%", classes="metric-card")
        yield Static("Active: 567", classes="metric-card")
```

### Centered Content

Center content horizontally and vertically:
```python
class CenteredApp(App):
    CSS = """
    Screen {
        align: center middle;
    }
    
    #dialog {
        width: 60;
        height: 20;
        border: thick $accent;
        padding: 2;
        background: $surface;
    }
    """
    
    def compose(self) -> ComposeResult:
        with Container(id="dialog"):
            yield Label("Centered Dialog")
            yield Button("OK")
```

### Scrollable Content

Handle overflow with scrolling:
```python
from textual.containers import ScrollableContainer

class ScrollableApp(App):
    CSS = """
    #content {
        height: 100%;
        border: solid $primary;
    }
    """
    
    def compose(self) -> ComposeResult:
        with ScrollableContainer(id="content"):
            for i in range(100):
                yield Label(f"Line {i+1}")
```

### Tabbed Interface

Tab-based navigation:
```python
from textual.widgets import TabbedContent, TabPane

class TabbedApp(App):
    def compose(self) -> ComposeResult:
        with TabbedContent():
            with TabPane("Dashboard"):
                yield Label("Dashboard content")
            with TabPane("Users"):
                yield Label("Users content")
            with TabPane("Settings"):
                yield Label("Settings content")
```

## Sizing & Spacing

For CSS units (`fr`, `%`, `auto`, `min-width`, etc.), padding, margin, and alignment properties — see [styling.md](styling.md#dimensions).

Quick reference: use `1fr` for "take remaining space", `%` for relative sizing, fixed integers for columns/rows.

## Responsive Layouts

### Container Queries

Adjust based on container size:
```python
class ResponsiveApp(App):
    CSS = """
    Screen {
        layout: horizontal;
    }
    
    /* Default mobile layout */
    #content {
        layout: vertical;
    }
    
    /* Desktop layout when width > 80 */
    Screen:width-gt-80 #content {
        layout: horizontal;
    }
    """
```

### Conditional Layouts

Switch layouts based on screen size:
```python
def compose(self) -> ComposeResult:
    if self.size.width > 100:
        # Wide layout
        with Horizontal():
            yield self.make_sidebar()
            yield self.make_content()
    else:
        # Narrow layout
        with Vertical():
            yield self.make_content()
```

## Advanced Patterns

### Modal Overlay

Centered modal dialog:
```python
from textual.screen import ModalScreen
from textual.containers import Container

class Modal(ModalScreen[bool]):
    CSS = """
    Modal {
        align: center middle;
    }
    
    #dialog {
        width: 50;
        height: 15;
        border: thick $accent;
        background: $surface;
        padding: 1;
    }
    """
    
    def compose(self) -> ComposeResult:
        with Container(id="dialog"):
            yield Label("Are you sure?")
            with Horizontal():
                yield Button("Yes", variant="primary")
                yield Button("No", variant="error")
```

### Sidebar Toggle

Collapsible sidebar:
```python
class SidebarApp(App):
    show_sidebar = reactive(True)
    
    CSS = """
    #sidebar {
        width: 30;
        transition: width 200ms;
    }
    
    #sidebar.hidden {
        width: 0;
        display: none;
    }
    """
    
    def watch_show_sidebar(self, show: bool) -> None:
        sidebar = self.query_one("#sidebar")
        sidebar.set_class(not show, "hidden")
```

### Masonry Layout

Staggered grid:
```python
class MasonryLayout(App):
    CSS = """
    Screen {
        layout: grid;
        grid-size: 3;
        grid-gutter: 1;
    }
    
    .card {
        height: auto;
        border: solid $primary;
        padding: 1;
    }
    
    .card.tall {
        row-span: 2;
    }
    """
    
    def compose(self) -> ComposeResult:
        yield Static("Short card", classes="card")
        yield Static("Tall card\n\n\n", classes="card tall")
        yield Static("Short", classes="card")
```

### Split Resizable

Adjustable split panels:
```python
class ResizableSplit(App):
    left_width = reactive(30)
    
    CSS = """
    #left {
        width: var(--left-width);
    }
    
    #right {
        width: 1fr;
    }
    
    #divider {
        width: 1;
        background: $accent;
    }
    """
    
    def watch_left_width(self, width: int) -> None:
        self.set_var("left-width", width)
```

## Layout Debugging

Use borders to visualize layout:
```css
* {
    border: solid red;  /* Temporary debugging */
}

Container {
    border: solid blue;
}

Widget {
    border: solid green;
}
```

Use Textual devtools:
```bash
textual run --dev app.py
```

Add debug info to widgets:
```python
def compose(self) -> ComposeResult:
    yield Label(f"Size: {self.size}")
    yield Label(f"Region: {self.region}")
```
