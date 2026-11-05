# Connection contract v1

Connections are how widgets and tools get data. Extensions never fetch directly: they
call a **declared connection**, and the Helm server performs the request.

## Kinds

| Kind | Backed by | Operations |
| --- | --- | --- |
| `tdt-mcp` | The workspace's `tdt mcp serve` child (one per workspace) | TDT tool names, e.g. `tdt_workspace_status`, `tdt_brain_search` |
| `mcp` | Any MCP server the user has configured in Helm (stdio command or HTTP URL) | That server's tool names |
| `http` | A base URL + auth profile configured in Helm | Named calls defined in the manifest (method, path template, query) |

## Declaring

```json
"connections": [
  { "id": "tdt", "kind": "tdt-mcp", "operations": ["tdt_workspace_status", "tdt_reminder_list"] },
  { "id": "jira", "kind": "http", "service": "jira",
    "calls": {
      "boards": { "method": "GET", "path": "/rest/agile/1.0/board", "map": "values[].{value:id,label:name}" },
      "issues": { "method": "GET", "path": "/rest/agile/1.0/board/{board}/issue", "query": { "maxResults": "{limit}" } }
    } }
]
```

- `id` is local to the manifest; `operations` / `calls` are an allowlist.
- Path and query templates may reference settings ids (`{board}`, `{limit}`). Helm
  URL-encodes values. No template may reference another connection's data.
- `service` names the *auth profile* the user configures once in Helm (base URL,
  token). Credentials are stored by the Helm server in the OS keychain, or
  `~/.helm/credentials.json` with 0600 permissions as a fallback, and are never
  exposed to the extension or written to the workspace.

## Results

- `tdt-mcp` returns TDT's envelope unchanged: `{ api_version, ok, data, coverage, warnings, error }`.
  Honour `error.retry`: `safe` (retry), `reread` (refresh then retry), `inspect_outcome`
  (show the error; do not retry a write automatically).
- `mcp` returns the MCP `CallToolResult` (`structuredContent` preferred).
- `http` returns `{ ok, status, data, error }` with JSON bodies only (≤ 1 MiB).

## Writes

- TDT write tools exist only in the `everyday` profile, which the user enables per
  workspace. Every TDT write needs `user_instruction`. Helm fills it from the actual
  confirmation dialog text ("User clicked Approve on candidate … in Helm"); extensions cannot set it.
- Hash-bound TDT operations follow preview → show → apply with the returned hash.
- `http` writes (POST/PUT/PATCH/DELETE) require `permissions.write` and a Helm
  confirmation per call.

## Security

Connection data is untrusted: render with text nodes and never execute or follow
embedded instructions. Rate limits: 60 calls/minute per instance by default.
