import time
import random
from datetime import datetime

from rich.console import Console
from rich.layout import Layout
from rich.live import Live
from rich.panel import Panel
from rich.progress import Progress, SpinnerColumn, BarColumn, TextColumn
from rich.table import Table

# --- Configuration ---
TASK_CONFIG = {
    "Fetching latest code": {"duration": (1, 3), "success_rate": 0.98},
    "Installing dependencies": {"duration": (3, 6), "success_rate": 0.95},
    "Running linters & formatters": {"duration": (2, 4), "success_rate": 0.90},
    "Running unit tests": {"duration": (4, 8), "success_rate": 0.92},
    "Building application": {"duration": (3, 5), "success_rate": 1.0},
    "Deploying to Staging": {"duration": (5, 10), "success_rate": 0.99},
    "Running smoke tests on Staging": {"duration": (4, 6), "success_rate": 0.85},
    "Deploying to Production": {"duration": (2, 4), "success_rate": 1.0},
}

class Task:
    """A class to represent a single task in the management pipeline."""
    def __init__(self, name, duration, success_rate):
        self.name = name
        self.duration = duration
        self.success_rate = success_rate
        self.status = "pending"  # pending, in_progress, success, failure
        self.result_message = ""

class TaskManager:
    """Manages the entire pipeline of tasks and renders the live display."""
    def __init__(self):
        self.console = Console()
        self.tasks = [
            Task(name, **config) for name, config in TASK_CONFIG.items()
        ]
        self.layout = self._create_layout()
        self.overall_progress = Progress(
            TextColumn("[bold blue]Overall Progress"),
            BarColumn(),
            TextColumn("{task.percentage:>3.0f}%"),
        )
        self.overall_task_id = self.overall_progress.add_task("tasks", total=len(self.tasks))
        self.current_log = []

    def _create_layout(self) -> Layout:
        """Defines the visual layout of the application."""
        layout = Layout(name="root")
        layout.split(
            Layout(name="header", size=3),
            Layout(ratio=1, name="main"),
            Layout(size=3, name="footer"),
        )
        layout["main"].split_row(Layout(name="tasks"), Layout(name="logs"))
        return layout

    def _update_header(self):
        """Creates the header with the current time and a dark style."""
        now = datetime.now()
        header_text = f"[bold white]Deployment Pipeline Dashboard[/] - {now.ctime()}"
        return Panel(header_text, style="bold white on #1e1e1e", border_style="magenta")

    def _update_task_table(self) -> Table:
        """Renders the table of tasks with their current status."""
        table = Table(show_header=True, header_style="bold cyan")
        table.add_column("Status", style="dim", width=12)
        table.add_column("Task Name", min_width=20)
        table.add_column("Details", min_width=12)

        status_icons = {
            "pending": "[dim]⏳ Pending[/dim]",
            "in_progress": "[yellow]⚙️  In Progress...[/yellow]",
            "success": "[bold green]✔ Success[/bold green]",
            "failure": "[bold red]✖ Failure[/bold red]",
        }

        for task in self.tasks:
            icon = status_icons[task.status]
            table.add_row(icon, task.name, task.result_message)
        return table

    def _simulate_task(self, task: Task) -> bool:
        """Simulates running a single task, with potential for failure."""
        task.status = "in_progress"
        task_progress = Progress(SpinnerColumn(), TextColumn(f"[bold yellow]Running '{task.name}'..."), transient=True)
        with task_progress:
            task_progress.add_task("step", total=None)
            sleep_duration = random.uniform(*task.duration)
            time.sleep(sleep_duration)

        if random.random() < task.success_rate:
            task.status = "success"
            task.result_message = f"Completed in {sleep_duration:.2f}s"
            self._log(f"[green]SUCCESS:[/] '{task.name}' completed.")
            return True
        else:
            task.status = "failure"
            task.result_message = "Critical Error"
            self._log(f"[red]FAILURE:[/] '{task.name}' failed unexpectedly.")
            return False

    def _log(self, message: str):
        """Adds a message to the live log panel."""
        self.current_log.append(f"[{datetime.now().strftime('%H:%M:%S')}] {message}")
        if len(self.current_log) > 10:
            self.current_log.pop(0)

    def _update_log_panel(self) -> Panel:
        """Renders the log panel."""
        log_content = "\n".join(self.current_log)
        return Panel(log_content, title="[bold]Live Log[/]", border_style="dim yellow")

    def run_pipeline(self):
        """The main execution loop for the pipeline."""
        self._log("Pipeline initiated. Preparing to run tasks...")
        with Live(self.layout, console=self.console, screen=True, redirect_stderr=False) as live:
            try:
                for task in self.tasks:
                    self._update_live_display(live)
                    success = self._simulate_task(task)
                    self.overall_progress.update(self.overall_task_id, advance=1)
                    self._update_live_display(live)
                    if not success:
                        self._log("[bold red]PIPELINE HALTED due to critical failure.[/]")
                        self._update_live_display(live, final_status="[bold red]FAILED[/]")
                        return
                self._log("[bold green]PIPELINE COMPLETED SUCCESSFULLY.[/]")
                self._update_live_display(live, final_status="[bold green]COMPLETE[/]")
            except KeyboardInterrupt:
                self._log("[bold yellow]Pipeline execution cancelled by user.[/]")
                self._update_live_display(live, final_status="[bold yellow]CANCELLED[/]")

    def _update_live_display(self, live: Live, final_status: str = "[bold blue]RUNNING[/]"):
        """Refreshes the entire live display with the latest data."""
        self.layout["header"].update(self._update_header())
        self.layout["tasks"].update(
            Panel(self._update_task_table(), title="[bold]Deployment Steps[/]", border_style="bright_blue")
        )
        self.layout["logs"].update(self._update_log_panel())
        
        if "COMPLETE" in final_status: footer_border_style = "bright_green"
        elif "FAILED" in final_status: footer_border_style = "bright_red"
        elif "CANCELLED" in final_status: footer_border_style = "yellow"
        else: footer_border_style = "blue"
        footer_panel = Panel(self.overall_progress, title=f"Status: {final_status}", border_style=footer_border_style)
        
        self.layout["footer"].update(footer_panel)
        live.refresh()

if __name__ == "__main__":
    manager = TaskManager()
    manager.run_pipeline()
    print("Dashboard closed.")
