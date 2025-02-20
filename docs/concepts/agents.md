---
description: Prefect agents are like untyped workers.
tags:
    - agents
    - deployments
search:
  boost: 2
---

# Agents 
## Agent overview

Agent processes are lightweight polling services that get scheduled work from a [work pool](#work-pool-overview) and deploy the corresponding flow runs.

Agents poll for work every 15 seconds by default. This interval is configurable in your [profile settings](/concepts/settings/) with the `PREFECT_AGENT_QUERY_INTERVAL` setting.

It is possible for multiple agent processes to be started for a single work pool. Each agent process sends a unique ID to the server to help disambiguate themselves and let users know how many agents are active.

### Agent options

Agents are configured to pull work from one or more work pool queues. If the agent references a work queue that doesn't exist, it will be created automatically.

Configuration parameters you can specify when starting an agent include:

| Option                                            | Description                                                                                                                                                                                  |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--api`                                           | The API URL for the Prefect server. Default is the value of `PREFECT_API_URL`.                                                                                                               |
| `--hide-welcome`                                  | Do not display the startup ASCII art for the agent process.                                                                                                                                  |
| `--limit`                                         | Maximum number of flow runs to start simultaneously. [default: None]                                                                                                                         |
| `--match`, `-m`                                   | Dynamically matches work queue names with the specified prefix for the agent to pull from,for example `dev-` will match all work queues with a name that starts with `dev-`. [default: None] |
| `--pool`, `-p`                                    | A work pool name for the agent to pull from. [default: None]                                                                                                                                 |
| <span class="no-wrap">`--prefetch-seconds`</span> | The amount of time before a flow run's scheduled start time to begin submission. Default is the value of `PREFECT_AGENT_PREFETCH_SECONDS`.                                                   |
| `--run-once`                                      | Only run agent polling once. By default, the agent runs forever. [default: no-run-once]                                                                                                      |
| `--work-queue`, `-q`                              | One or more work queue names for the agent to pull from. [default: None]                                                                                                                     |

You must start an agent within an environment that can access or create the infrastructure needed to execute flow runs. Your agent will deploy flow runs to the infrastructure specified by the deployment.

!!! tip "Prefect must be installed in execution environments"
    Prefect must be installed in any environment in which you intend to run the agent or execute a flow run.

!!! tip "`PREFECT_API_URL` and `PREFECT_API_KEY` settings for agents"
    `PREFECT_API_URL` must be set for the environment in which your agent is running or specified when starting the agent with the `--api` flag. You must also have a user or service account with the `Worker` role, which can be configured by setting the `PREFECT_API_KEY`.

    If you want an agent to communicate with Prefect Cloud or a Prefect server from a remote execution environment such as a VM or Docker container, you must configure `PREFECT_API_URL` in that environment.

### Starting an agent

Use the `prefect agent start` CLI command to start an agent. You must pass at least one work pool name or match string that the agent will poll for work. If the work pool does not exist, it will be created.

<div class="terminal">
```bash
$ prefect agent start -p [work pool name]
```
</div>

For example:

<div class="terminal">
```bash
$ prefect agent start -p "my-pool"
Starting agent with ephemeral API...
  ___ ___ ___ ___ ___ ___ _____     _   ___ ___ _  _ _____
 | _ \ _ \ __| __| __/ __|_   _|   /_\ / __| __| \| |_   _|
 |  _/   / _|| _|| _| (__  | |    / _ \ (_ | _|| .` | | |
 |_| |_|_\___|_| |___\___| |_|   /_/ \_\___|___|_|\_| |_|

Agent started! Looking for work from work pool 'my-pool'...
```
</div>

In this case, Prefect automatically created a new `my-queue` work queue.

By default, the agent polls the API specified by the `PREFECT_API_URL` environment variable. To configure the agent to poll from a different server location, use the `--api` flag, specifying the URL of the server.

In addition, agents can match multiple queues in a work pool by providing a `--match` string instead of specifying all of the queues. The agent will poll every queue with a name that starts with the given string. New queues matching this prefix will be found by the agent without needing to restart it.

For example:

<div class="terminal">
```bash
$ prefect agent start --match "foo-"
```
</div>

This example will poll every work queue that starts with "foo-".

### Configuring prefetch

By default, the agent begins submission of flow runs a short time (10 seconds) before they are scheduled to run. This allows time for the infrastructure to be created, so the flow run can start on time. In some cases, infrastructure will take longer than this to actually start the flow run. In these cases, the prefetch can be increased using the `--prefetch-seconds` option or the `PREFECT_AGENT_PREFETCH_SECONDS` setting. Submission can begin an arbitrary amount of time before the flow run is scheduled to start. If this value is _larger_ than the amount of time it takes for the infrastructure to start, the flow run will _wait_ until its scheduled start time. This allows flow runs to start exactly on time.