# Textual Testing Guide

Testing Textual apps with pytest and the Pilot API.

## Contents
- [Setup](#setup)
- [Basic tests](#basic-tests)
- [Simulating input](#simulating-input)
- [Querying state](#querying-state)
- [Workers in tests](#workers-in-tests)
- [Snapshot testing](#snapshot-testing)

## Setup

```bash
pip install pytest pytest-asyncio
```

Mark async tests:
```python
# pytest.ini or pyproject.toml
[pytest]
asyncio_mode = auto
```

## Basic tests

```python
from textual.pilot import Pilot
from my_app import MyApp

async def test_app_starts():
    app = MyApp()
    async with app.run_test() as pilot:
        assert app.screen is not None

async def test_initial_state():
    app = MyApp()
    async with app.run_test() as pilot:
        label = app.query_one("#status")
        assert label.renderable == "Ready"
```

## Simulating input

```python
async def test_button_click():
    async with MyApp().run_test() as pilot:
        await pilot.click("#submit")
        assert app.query_one("#result").renderable == "Done"

async def test_keyboard():
    async with MyApp().run_test() as pilot:
        await pilot.press("ctrl+s")   # Key combo
        await pilot.press("q")        # Single key

async def test_text_input():
    async with MyApp().run_test() as pilot:
        await pilot.click("#name-input")
        await pilot.type("Alice")
        await pilot.press("enter")
```

## Querying state

```python
from textual.widgets import Button

async def test_widget_state():
    async with MyApp().run_test() as pilot:
        # Find by ID
        button = app.query_one("#submit", Button)
        assert not button.disabled

        # Find by class
        cards = app.query(".card")
        assert len(cards) == 3

        # Check reactive value
        assert app.counter == 0
        await pilot.click("#increment")
        assert app.counter == 1
```

## Workers in tests

Pause to let workers complete:
```python
async def test_with_worker():
    async with MyApp().run_test() as pilot:
        await pilot.click("#load-data")
        await pilot.pause(0.5)  # Wait for worker
        assert app.query_one("#result").renderable != ""
```

Or wait for a specific worker state:
```python
from textual.worker import WorkerState

async def test_worker_completes():
    async with MyApp().run_test() as pilot:
        app.run_worker(app.fetch_data())
        await pilot.pause()
        workers = [w for w in app.workers if w.name == "fetch"]
        assert workers[0].state == WorkerState.SUCCESS
```

## Snapshot testing

Capture visual snapshots to catch regressions:
```python
async def test_snapshot(snap_compare):
    assert await snap_compare("my_app.py")
```

Requires `pytest-textual-snapshot`:
```bash
pip install pytest-textual-snapshot
pytest --snapshot-update  # Generate initial snapshots
pytest                    # Compare against snapshots
```

Official testing guide: https://textual.textualize.io/guide/testing/
