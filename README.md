[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/scriptedalchemy-devtools-debugger-mcp-badge.png)](https://mseep.ai/app/scriptedalchemy-devtools-debugger-mcp)

# Node.js Debugger MCP

An MCP server that provides comprehensive Node.js debugging capabilities using the Chrome DevTools Protocol. This server enables AI assistants to debug Node.js applications with full access to breakpoints, stepping, variable inspection, call stacks, expression evaluation, and source maps.

## Why use this MCP server?
This MCP server is useful when you need AI assistance with debugging Node.js applications. It provides programmatic access to all the debugging features you'd find in Chrome DevTools or VS Code, allowing AI assistants to help you set breakpoints, inspect variables, step through code, and analyze runtime behavior. 

## Features

- **Full Node.js debugger**: Set breakpoints, conditional breakpoints, logpoints, and pause-on-exceptions
- **Stepping controls**: Step over/into/out, continue to location, restart frame
- **Variable inspection**: Explore locals/closure scopes, `this` preview, and drill down into object properties
- **Expression evaluation**: Evaluate JavaScript expressions in the current call frame with console output capture
- **Call stack analysis**: Inspect call stacks and pause-state information
- **Source map support**: Debug TypeScript and other transpiled code with full source map support
- **Console monitoring**: Capture and review console output during debugging sessions

## Installation

```bash
npm install devtools-debugger-mcp
```

## Configuration

Add the server to your MCP settings configuration:

```json
{
  "devtools-debugger-mcp": {
    "command": "node",
    "args": ["path/to/devtools-debugger-mcp/dist/index.js"]
  }
}
```

Alternatively, if installed globally, you can use the CLI binary:

```json
{
  "devtools-debugger-mcp": {
    "command": "devtools-debugger-mcp"
  }
}
```

## Node.js Debugging

This MCP server can debug Node.js programs by launching your script with the built‑in inspector (`--inspect-brk=0`) and speaking the Chrome DevTools Protocol (CDP).

How it works
- `start_node_debug` spawns `node --inspect-brk=0 your-script.js`, waits for the inspector WebSocket, attaches, and returns the initial pause (first line) with a `pauseId` and top call frame.
- You can then set breakpoints (by file path or URL regex), choose pause-on-exceptions, and resume/step. At each pause, tools can inspect scopes, evaluate expressions, and read console output captured since the last step/resume.
- When the process exits, the server cleans up the CDP session and resets its state.

Quickstart (from an MCP-enabled client)
1) Start a debug session
```json
{ "tool": "start_node_debug", "params": { "scriptPath": "/absolute/path/to/app.js" } }
```
2) Set a breakpoint (file path + 1-based line)
```json
{ "tool": "set_breakpoint", "params": { "filePath": "/absolute/path/to/app.js", "line": 42 } }
```
3) Run to next pause (optionally include console/stack)
```json
{ "tool": "resume_execution", "params": { "includeConsole": true, "includeStack": true } }
```
4) Inspect at a pause
```json
{ "tool": "inspect_scopes", "params": { "maxProps": 15 } }
{ "tool": "evaluate_expression", "params": { "expr": "user.name" } }
```
5) Step
```json
{ "tool": "step_over" }
{ "tool": "step_into" }
{ "tool": "step_out" }
```
6) Finish
```json
{ "tool": "stop_debug_session" }
```

Node.js tool reference (summary)
- `start_node_debug({ scriptPath, format? })` — Launches Node with inspector and returns initial pause.
- `set_breakpoint({ filePath, line })` — Breakpoint by file path (1-based line).
- `set_breakpoint_condition({ filePath?, urlRegex?, line, column?, condition, format? })` — Conditional breakpoint or by URL regex.
- `add_logpoint({ filePath?, urlRegex?, line, column?, message, format? })` — Logpoint via conditional breakpoint that logs and returns `false`.
- `set_exception_breakpoints({ state })` — `none | uncaught | all`.
- `blackbox_scripts({ patterns })` — Ignore frames from matching script URLs.
- `list_scripts()` / `get_script_source({ scriptId? | url? })` — Discover and fetch script sources.
- `continue_to_location({ filePath, line, column? })` — Run until a specific source location.
- `restart_frame({ frameIndex, pauseId?, format? })` — Re-run the selected frame.
- `resume_execution({ includeScopes?, includeStack?, includeConsole?, format? })` — Continue to next pause or exit.
- `step_over|step_into|step_out({ includeScopes?, includeStack?, includeConsole?, format? })` — Stepping with optional context in the result.
- `evaluate_expression({ expr, pauseId?, frameIndex?, returnByValue?, format? })` — Evaluate in a paused frame; defaults to top frame.
- `inspect_scopes({ maxProps?, pauseId?, frameIndex?, includeThisPreview?, format? })` — Locals/closures and `this` summary.
- `get_object_properties({ objectId, maxProps?, format? })` — Drill into object previews.
- `list_call_stack({ depth?, pauseId?, includeThis?, format? })` — Top N frames summary.
- `get_pause_info({ pauseId?, format? })` — Pause reason/location summary.
- `read_console({ format? })` — Console messages since the last step/resume.
- `stop_debug_session()` — Kill process and detach.

Notes
- File paths are converted to `file://` URLs internally for CDP compatibility.
- `line` is 1-based; CDP is 0-based internally.
- The server buffers console output between pauses; fetch via `includeConsole` on step/resume or with `read_console`.
- Use `set_output_format({ format: 'text' | 'json' | 'both' })` to set default response formatting.



## Available Tools

This MCP server provides the following Node.js debugging tools. All tools support optional `format` parameter (`'text'` or `'json'`) to control response formatting.

### Session Management
- **`start_node_debug`** - Launch a Node.js script with debugging enabled
- **`stop_debug_session`** - Terminate the debugging session and clean up

### Breakpoint Management  
- **`set_breakpoint`** - Set a breakpoint at a specific file and line
- **`set_breakpoint_condition`** - Set a conditional breakpoint or breakpoint by URL regex
- **`add_logpoint`** - Add a logpoint that logs messages when hit
- **`set_exception_breakpoints`** - Configure pause-on-exception behavior

### Execution Control
- **`resume_execution`** - Continue execution to the next breakpoint or completion
- **`step_over`** - Step over the current line
- **`step_into`** - Step into function calls
- **`step_out`** - Step out of the current function
- **`continue_to_location`** - Run until reaching a specific location
- **`restart_frame`** - Restart execution from a specific call frame

### Inspection and Analysis
- **`inspect_scopes`** - Examine local variables, closures, and `this` context
- **`evaluate_expression`** - Evaluate JavaScript expressions in the current context
- **`get_object_properties`** - Drill down into object properties
- **`list_call_stack`** - View the current call stack
- **`get_pause_info`** - Get information about the current pause state

### Utilities
- **`list_scripts`** - List all loaded scripts
- **`get_script_source`** - Retrieve source code for scripts
- **`blackbox_scripts`** - Configure scripts to skip during debugging
- **`read_console`** - Read console output captured during debugging

For detailed usage examples and parameter descriptions, see the "Node.js Debugging" section above.

## License

MIT
