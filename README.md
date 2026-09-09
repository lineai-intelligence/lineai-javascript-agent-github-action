## About

GitHub Action to scan JavaScript/TypeScript source into a Lineai server using the Lineai JavaScript Agent (jscape-cli).

The action analyzes source directly (Babel-based; no build step required) and uploads the resulting scan to your Lineai instance.


### Example

```yaml
name: lineai-scan

on:
  push:
    branches: [ "integration" ]
  pull_request:
    branches: [ "integration" ]
  workflow_dispatch:

jobs:
  lineai-scan:
    name: Perform Lineai Scan
    environment: Lineai Scan Env
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v4
      - name: Run the Lineai Scan
        uses: lineai-intelligence/lineai-javascript-agent-github-action@v1
        with:
          lineai_host: ${{ vars.LINEAI_HOST }}
          agent_uuid: ${{ vars.AGENT_UUID }}
          agent_password: ${{ secrets.AGENT_PASSWORD }}
          application_name: MyApplication
          scan_space: default
```


## Customizing

| Name                    | Type    | Description                                                                                                                                                                                             |
|-------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `lineai_host`           | String  | The host address of the Lineai instance (e.g. https://mycompany.app.lineai.net), with no path suffix — the agent appends the API mount point (/api) itself.                                             |
| `agent_uuid`            | String  | The UUID of the Agent in Lineai.                                                                                                                                                                        |
| `agent_password`        | String  | The password for the agent.                                                                                                                                                                             |
| `application_name`      | String  | The Application node to create that will be the parent of all objects found in the scan.                                                                                                                |
| `scan_space`            | String  | The name of the scan space that the data will be saved to. If specified, a ScanSpace with this name will be created if not found. If not specified, information will be saved to the default ScanSpace. |
| `scan_path`             | String  | The path of the file or directory to scan. Must start with /github/workspace/. Defaults to /github/workspace.                                                                                           |
| `entrypoint_type`       | String  | How scan entry points are discovered: `recursive`, `package`, or `file`. Defaults to `recursive`.                                                                                                       |
| `dependency_depth`      | String  | How many levels of node_modules dependencies to follow (default 3; 0 = root package only, -1 = unlimited).                                                                                              |
| `expand_imports`        | boolean | Expand imported files before analysis (heavier, more nodes per file). Defaults to false.                                                                                                                |
| `force_rescan`          | boolean | Forces the agent to rescan already scanned artifacts. Defaults to false.                                                                                                                                |
| `expunge_scan_sessions` | boolean | Instruct the server to delete all other scan sessions created by this agent and its configuration after the current scan session has completed successfully. Defaults to false.                         |


## Exit codes

| Code | Meaning                                                                                                                                            |
|------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `0`  | Scan and server upload succeeded.                                                                                                                   |
| `1`  | Scan failed.                                                                                                                                        |
| `2`  | Scan artifacts were written inside the container, but the upload to the Lineai server failed. The step fails; re-run once connectivity is restored. |

Note that dependencies under `node_modules` are typically absent in CI checkouts (no
install step is required for the scan); references to them surface in Lineai as
unresolved references that resolve automatically when those packages are scanned.
