# Cherry Studio Configuration Guide

This server has been modified to support custom configurations via environment variables, allowing seamless integration with Cherry Studio (and other MCP clients).

## Environment Variables

| Variable | Description | Default |
| :--- | :--- | :--- |
| `OPENAI_API_KEY` | Your API Key. | (Required) |
| `OPENAI_BASE_URL` | Custom API Base URL (e.g., for local endpoints or proxies). | `None` (Use Official OpenAI API) |
| `KV_EXTRACTOR_MODEL` | The model used for extraction and annotation. | `gpt-4.1-mini` |
| `KV_EXTRACTOR_EVAL_MODEL` | The model used for type evaluation and correction. | `gpt-4.1` |

## Cherry Studio Configuration

To add this MCP server to Cherry Studio, use the following configuration JSON. Replace the placeholders with your actual values.

```json
{
  "mcpServers": {
    "kv-extractor": {
      "command": "c:/path/to/your/virtualenv/Scripts/python.exe",
      "args": [
        "c:/path/to/kv-extractor-mcp-server/src/kv_extractor_mcp_server/server.py", 
        "--log=on", 
        "--logfile=c:/path/to/logs/kv-extractor.log"
      ],
      "env": {
        "OPENAI_API_KEY": "sk-...",
        "OPENAI_BASE_URL": "https://api.openai.com/v1", 
        "KV_EXTRACTOR_MODEL": "gpt-4o-mini",
        "KV_EXTRACTOR_EVAL_MODEL": "gpt-4o"
      }
    }
  }
}
```

### Notes for Cherry Studio
1.  **Python Executable**: **Crucial!** You MUST use the absolute path to the `python.exe` that has the dependencies installed (e.g., inside your virtual environment logic like `.venv/Scripts/python.exe`). Do NOT use just `python` unless your system default Python has the `pyproject.toml` dependencies installed.
2.  **AbsolutePath**: Ensure the path to `server.py` is an absolute path.
3.  **Logs**: The `--logfile` argument must also be an absolute path if enabled.
