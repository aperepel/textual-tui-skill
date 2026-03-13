# Textual Workers — Deep Dive

Background task patterns for Textual applications.

## Contents
- Worker with progress updates
- Thread-safe UI updates
- Worker cancellation
- Worker error handling
- Multiple concurrent workers
- Async vs thread workers

## Worker with progress updates

```python
import asyncio
from textual.app import App, ComposeResult
from textual.widgets import ProgressBar

class MyApp(App):
    def compose(self) -> ComposeResult:
        yield ProgressBar(total=100, id="progress")

    def on_mount(self) -> None:
        self.run_worker(self.long_task())

    async def long_task(self) -> None:
        progress = self.query_one(ProgressBar)
        for i in range(100):
            await asyncio.sleep(0.1)
            progress.update(progress=i + 1)
```

## Thread-safe UI updates

Use `call_from_thread` when running with `thread=True`:

```python
from textual.app import App

class MyApp(App):
    def on_mount(self) -> None:
        self.run_worker(self.fetch_data, thread=True)

    def fetch_data(self) -> None:
        result = expensive_computation()
        self.call_from_thread(self.display_result, result)

    def display_result(self, result: str) -> None:
        self.query_one("#output").update(result)
```

Never modify widgets directly from thread workers.

## Worker cancellation

```python
import asyncio
from textual.app import App
from textual.worker import Worker

class MyApp(App):
    worker: Worker | None = None

    def start_task(self) -> None:
        self.worker = self.run_worker(self.long_task())

    def cancel_task(self) -> None:
        if self.worker and not self.worker.is_finished:
            self.worker.cancel()
            self.notify("Task cancelled")

    async def long_task(self) -> None:
        for i in range(1000):
            await asyncio.sleep(0.1)
            if self.worker.is_cancelled:
                return
```

## Worker error handling

```python
from textual.app import App
from textual.worker import Worker, WorkerState

class MyApp(App):
    def on_mount(self) -> None:
        worker = self.run_worker(self.risky_task())
        worker.name = "data_processor"

    async def risky_task(self) -> str:
        try:
            return await fetch_from_api()
        except Exception as e:
            self.notify(f"Error: {e}", severity="error")
            raise

    def on_worker_state_changed(self, event: Worker.StateChanged) -> None:
        if event.state == WorkerState.ERROR:
            self.log.error(f"Worker failed: {event.worker.name}")
        elif event.state == WorkerState.SUCCESS:
            self.log.info(f"Worker completed: {event.worker.name}")
```

## Multiple concurrent workers

```python
from textual.app import App

class MyApp(App):
    def on_mount(self) -> None:
        self.run_worker(self.task_one(), name="task1", group="processing")
        self.run_worker(self.task_two(), name="task2", group="processing")
        self.run_worker(self.task_three(), name="task3", group="processing")

    def cancel_all_tasks(self) -> None:
        for worker in self.workers:
            if worker.group == "processing":
                worker.cancel()
```

## Async vs thread workers

| Scenario | Worker type |
|----------|-------------|
| Network requests, async I/O | `run_worker(coro())` (default async) |
| CPU-bound computation | `run_worker(fn, thread=True)` |

```python
import httpx
from textual.app import App

class MyApp(App):
    # Async — for I/O bound
    async def fetch_data(self) -> str:
        async with httpx.AsyncClient() as client:
            response = await client.get("https://api.example.com")
            return response.text

    # Thread — for CPU bound
    def process_data(self) -> str:
        result = [i**2 for i in range(1_000_000)]
        return str(sum(result))

    def on_mount(self) -> None:
        self.run_worker(self.fetch_data())
        self.run_worker(self.process_data, thread=True)
```

## Best practices

- Use `exclusive=True` to prevent duplicate workers for the same task
- Name workers (`name="..."`) for easier debugging
- Group related workers for batch cancellation (`group="..."`)
- Always handle `WorkerState.ERROR`
- Check `worker.is_cancelled` in long loops
- Clean up resources in `finally` blocks
