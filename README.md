# GitHub Actions Technical Deep-Dive

GitHub Actions is an **event-driven orchestration engine** for ephemeral compute resources.

## The Under-the-Hood Lifecycle

1.  **Event Ingestion:** GitHub's backend architecture (Event Bus) listens for Webhook events (e.g., `git push`, `pull_request`).
2.  **Workflow Orchestration:** When an event matches your `on:` criteria, the **Actions Service** generates an execution graph based on your `jobs`.
3.  **Runner Provisioning:**
    *   For `runs-on: ubuntu-latest`, GitHub requests a Virtual Machine from an Azure pool (**Standard_DS2_v2**).
    *   This is a fresh, Hyper-V isolated environment for every job.
4.  **The Runner Agent:** A .NET-based application called `Runner.Listener` runs inside the VM. It establishes a long-poll HTTPS connection to GitHub to retrieve the **Job Message** (a JSON payload describing the steps).
5.  **Step Execution:**
    *   **`run`**: The agent spawns a sub-shell (e.g., `/bin/bash` or `pwsh`) and pipes your script string directly into it.
    *   **`uses`**: The agent downloads the specified repository, parses `action.yml`, and executes it as a Node.js process or by building/running a Docker container.
