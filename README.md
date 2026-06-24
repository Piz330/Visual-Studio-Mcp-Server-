# VisualStudio MCP Bridge

VisualStudio MCP Bridge is a Visual Studio VSIX extension that exposes the current Visual Studio IDE session through a local MCP server.

The server binds only to `127.0.0.1` and requires a local bearer token.

## Features

- List running Visual Studio instances.
- Read the current solution, projects, active document, and open documents.
- Open files in Visual Studio.
- Read, replace, insert, and save text documents through Visual Studio.
- Trigger a Visual Studio solution build.

## Installation

1. Download `VisualStudioMcpExtension.vsix` from GitHub Releases.
2. Close Visual Studio.
3. Install the VSIX by double-clicking it, or run:

```powershell
& "C:\Program Files\Microsoft Visual Studio\18\Community\Common7\IDE\VSIXInstaller.exe" `
  "C:\Path\To\VisualStudioMcpExtension.vsix"
```

4. Open Visual Studio.
5. Start the bridge:

```text
Tools > VisualStudio MCP Bridge > Connect to Local MCP Server
```

## Connection

The extension writes a per-user config file:

```text
%LOCALAPPDATA%\VisualStudioMcpBridge\config.txt
```

It contains:

```text
endpoint=http://127.0.0.1:32177/mcp
token=<local bearer token>
```

Use these values in your MCP client:

```text
URL: http://127.0.0.1:32177/mcp
Authorization: Bearer <token>
```

Do not share or commit your token.

## Test With PowerShell

```powershell
$config = Get-Content "$env:LOCALAPPDATA\VisualStudioMcpBridge\config.txt"
$token = ($config | Where-Object { $_ -like "token=*" }) -replace "^token=", ""

$body = @{
  jsonrpc = "2.0"
  id = 1
  method = "tools/list"
  params = @{}
} | ConvertTo-Json -Depth 10

Invoke-RestMethod `
  -Uri "http://127.0.0.1:32177/mcp" `
  -Method Post `
  -Headers @{ Authorization = "Bearer $token" } `
  -ContentType "application/json" `
  -Body $body
```

## MCP Tools

- `visualstudio_instances`
- `visualstudio_solution`
- `visualstudio_active_document`
- `visualstudio_open_documents`
- `visualstudio_build_solution`
- `visualstudio_open_file`
- `visualstudio_read_document`
- `visualstudio_replace_selection`
- `visualstudio_insert_text`
- `visualstudio_replace_document`
- `visualstudio_save_document`

## Build From Source

Publish the bridge first:

```powershell
dotnet publish .\src\VisualStudioMcpBridge -c Release -o .\src\VisualStudioMcpBridge\publish
```

Build the VSIX:

```powershell
dotnet build .\src\VisualStudioMcpExtension -c Release
```

The generated VSIX is:

```text
src\VisualStudioMcpExtension\bin\Release\net472\VisualStudioMcpExtension.vsix
```

Upload the `.vsix` file to GitHub Releases. Do not commit generated build outputs.
