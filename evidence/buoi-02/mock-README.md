# Mock server (Prism) — Lab 02

This file explains how to start the Prism mock server and capture 5 sample requests for evidence.

Prerequisites
- Node.js (v20+ recommended)
- npm
- curl (or use PowerShell's curl.exe)

Steps (PowerShell)
1. Install CLIs (one-time):

```powershell
cd .\fit4110-lab02-openapi-nhom-2
npm run install:cli
```

2. Start Prism mock server in a terminal (keeps running):

```powershell
cd .\fit4110-lab02-openapi-nhom-2
npm run mock
# Prism will listen on http://localhost:4010
```

3. In another terminal, run the provided test script to send 5 requests and save output:

```powershell
cd .\fit4110-lab02-openapi-nhom-2
# Run PowerShell test (recommended on Windows)
.\\scripts\\test_mock_with_curl.ps1 | Tee-Object -FilePath evidence\\buoi-02\\mock-requests.txt
```

Or on macOS / WSL / Git Bash:

```bash
cd fit4110-lab02-openapi-nhom-2
bash scripts/test_mock_with_curl.sh | tee evidence/buoi-02/mock-requests.txt
```

4. Capture screenshots of the responses or the file `evidence/buoi-02/mock-requests.txt` as evidence.

Stopping Prism
- If you started Prism in the foreground, stop it with Ctrl+C in that terminal.

If Prism fails to start
- Ensure `@stoplight/prism-cli` installed globally via `npm run install:cli`, or run via `npx @stoplight/prism-cli@5.14.2 mock openapi.yaml --port 4010`.
- If `npx` errors on Windows, try running PowerShell as Administrator and re-run install script.

What I'll do next
- If you run the commands and paste `evidence/buoi-02/mock-requests.txt` here, I'll review outputs and update `openapi.yaml` or negotiation log if responses differ from schema expectations.
