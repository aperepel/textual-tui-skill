---
name: textual-tui
description: Builds modern, interactive terminal user interfaces with Textual (Python). Use when creating CLI dashboards, monitoring tools, data viewers, interactive terminal apps, or any Python TUI. Covers widgets, layouts, styling, events, reactive programming, and background workers.
---

# Textual TUI Development

## Quick start

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Button

class MyApp(App):
    BINDINGS = [("q", "quit", "Quit")]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Button("Click me!", id="click")
        yield Footer()

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.exit()

if __name__ == "__main__":
    MyApp().run()
```

Install: `pip install textual textual-dev`
Dev mode with hot reload: `textual run --dev app.py`

## Core patterns

### Layouts

Use container classes — see [references/layouts.md](references/layouts.md) for recipes.

```python
from textual.app import ComposeResult
from textual.containers import Container, Horizontal, Vertical, ScrollableContainer
from textual.widgets import Header, Footer, Label

def compose(self) -> ComposeResult:
    with Vertical():
        yield Header()
        with Horizontal():
            with Container(id="sidebar"):
                yield Label("Menu")
            with ScrollableContainer(id="content"):
                yield Label("Content")
        yield Footer()
```

CSS in `App.CSS` string or `CSS_PATH = "app.tcss"` — see [references/styling.md](references/styling.md).

### Reactive attributes

```python
from textual.reactive import reactive
from textual.widget import Widget

class Counter(Widget):
    count = reactive(0)

    def watch_count(self, value: int) -> None:
        self.refresh()  # auto-called when count changes
```

### Events and bindings

```python
from textual.app import App
from textual.widgets import Input, Log

class MyApp(App):
    BINDINGS = [("q", "quit", "Quit"), ("d", "toggle_dark", "Dark mode")]

    def action_toggle_dark(self) -> None:
        self.dark = not self.dark

    def on_input_submitted(self, event: Input.Submitted) -> None:
        self.query_one(Log).write(event.value)
```

Handler names follow `on_{widget_class}_{message_name}` (snake_case).

### Workers — CRITICAL for blocking operations

**Never block the event loop.** Use workers for anything > 100ms (network, file I/O, DB, computation).

```python
from textual.app import App
from textual.widgets import Button
from textual.worker import Worker, WorkerState

class MyApp(App):
    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.run_worker(self.fetch_data(), exclusive=True)

    async def fetch_data(self) -> str:
        return await some_async_call()  # async for I/O; thread=True for CPU-bound

    def on_worker_state_changed(self, event: Worker.StateChanged) -> None:
        if event.state == WorkerState.ERROR:
            self.notify(f"Failed: {event.worker.name}", severity="error")
```

**Thread safety:** Use `self.call_from_thread(fn, args)` to update UI from thread workers.
For progress, cancellation, and multiple workers — see [references/workers.md](references/workers.md).

### Screens and modals

```python
from textual.app import ComposeResult
from textual.containers import Container
from textual.screen import ModalScreen
from textual.widgets import Button, Label

class ConfirmDialog(ModalScreen[bool]):
    def compose(self) -> ComposeResult:
        with Container():
            yield Label("Are you sure?")
            yield Button("Yes", id="yes")
            yield Button("No", id="no")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.dismiss(event.button.id == "yes")

# In an async method within your App or Screen:
# result = await self.push_screen_wait(ConfirmDialog())
```

### Custom messages

```python
from textual.app import App
from textual.message import Message
from textual.widget import Widget

class MyWidget(Widget):
    class Selected(Message):
        def __init__(self, value: str) -> None:
            self.value = value
            super().__init__()

    def on_click(self) -> None:
        self.post_message(self.Selected("item"))

class MyApp(App):
    def on_my_widget_selected(self, msg: MyWidget.Selected) -> None:
        self.log(msg.value)
```

## Testing

See [references/testing.md](references/testing.md) for full Pilot API patterns. Quick example:

```python
from my_app import MyApp

async def test_button_click():
    async with MyApp().run_test() as pilot:
        await pilot.click("#click")
        # assert expected state
```

## Key pitfalls

- **Blocking event loop**: Never `time.sleep()` in async — use `await asyncio.sleep()`; use workers for long tasks
- **Thread safety**: Never modify widgets from threads — use `call_from_thread()`
- **Worker leaks**: Workers outlive screens; cancel them or store references
- **Message handler names**: Must exactly match `on_{widget_class}_{message_name}` (snake_case)
- **CSS specificity**: Use IDs (`#id`) and classes (`.cls`) rather than element selectors for targeted styling

## References

| Topic | Resource |
|-------|----------|
| Widget examples (DataTable, Tree, Input, etc.) | [references/widgets.md](references/widgets.md) |
| Layout patterns | [references/layouts.md](references/layouts.md) |
| CSS/styling | [references/styling.md](references/styling.md) |
| Workers (progress, cancellation, multiple) | [references/workers.md](references/workers.md) |
| Testing with Pilot API | [references/testing.md](references/testing.md) |
| Official Textual docs index | [references/official-guides-index.md](references/official-guides-index.md) |
| Example apps | [assets/README.md](assets/README.md) |
