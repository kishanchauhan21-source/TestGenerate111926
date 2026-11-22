# SpecPilot — BRS Test Agent

A web-based QA assistant that analyzes business requirements documents and reference material, generates traceable test cases, builds an RTM, and prepares regression scenarios from failed tests. The AI runs locally through Ollama; no OpenAI API key is required.

## Features

- Upload a BRS plus reference files (up to 12 files, 15 MB each): PDF, DOCX, TXT, PNG, JPEG, or WebP.
- Extracts text from PDF/DOCX and sends DOCX-embedded and separately uploaded images to an Ollama vision model when configured.
- Generates positive, negative, boundary, and exception tests with requirement IDs, test steps, expected results, severity, and an editable Pass/Fail status.
- Generates an RTM and exports a three-sheet Excel workbook: **Test Cases**, **RTM**, and **Regression**.
- Mark cases as **Fail** and generate focused regression scenarios; download the updated workbook.
- Upload a revised BRS again to regenerate coverage against its current contents.

> Legacy `.doc` files must first be saved as `.docx` or PDF. The download is a modern `.xlsx` Excel workbook (rather than the obsolete binary `.xls` format); it opens in Excel and other spreadsheet tools. The app does not monitor a shared drive for changes: upload each revised version manually. For linked/attached references, select and upload those files along with the BRS.

## Run locally with Ollama

Requires Node.js 20+ and the Ollama desktop app/service. No OpenAI API key or cloud AI account is needed. The model runs locally; initial model downloads require internet access and several GB of disk space. Keep Ollama running while using the app.

1. Open PowerShell or the PyCharm terminal and download a text model (once):

   ```powershell
   ollama pull llama3.2:3b
   ```

2. For analysis of embedded/uploaded images, download a vision model too (optional; large download):

   ```powershell
   ollama pull llama3.2-vision:11b
   ```

3. In the project folder, install dependencies and start the web app:

   ```powershell
   npm install
   npm start
   ```

4. Open `http://localhost:3000` in your browser.

The default text model is `llama3.2:3b`. Change it with PowerShell before starting the server, for example:

```powershell
$env:OLLAMA_MODEL="llama3.2:3b"
$env:OLLAMA_VISION_MODEL="llama3.2-vision:11b"
npm start
```

If you only downloaded the text model, use it for all requests (images will not receive vision analysis):

```powershell
$env:OLLAMA_VISION_MODEL="llama3.2:3b"
npm start
```

Ollama usually provides its local API at `http://127.0.0.1:11434`. Set `OLLAMA_BASE_URL` if yours uses a different address. Check that the model is installed with `ollama list`. The server reports helpful errors when Ollama is stopped or a selected model is missing. `GET /api/health` reports Ollama connectivity and installed models.

## Deploy online

This configuration expects Ollama to be accessible to the Node.js server at `OLLAMA_BASE_URL`. On a local computer, the app is only accessible from that computer at `http://localhost:3000`. To make the app available online, deploy the Node.js server and provide it with a securely networked Ollama server and the required models. Do not expose Ollama's unauthenticated API directly to the public internet; use a private network, VPN, or authenticated gateway. GitHub Pages alone cannot host the Node API.

## Data handling and limitations

Uploads are held in server memory for the request and are not written to disk or retained by this application. Document text and supported images are sent to the configured Ollama server for analysis. With Ollama running on the same machine, inference is local and does not require an AI API key. Model downloads themselves use internet access. Review privacy and security carefully before uploading sensitive business data or exposing the web app to other users. Add authentication, access controls, rate limits, and appropriate logging/retention policies before public deployment. The model's output is a draft and must be reviewed by a tester. Scanned PDFs without selectable text may require OCR before upload. A vision-capable model is needed to interpret document images; results depend on the selected model and available system memory.

## API

- `GET /api/health` — app status, Ollama connectivity, and installed models.
- `POST /api/generate` — multipart field `documents` (one or more supported files).
- `POST /api/regression` — JSON body `{ "failedCases": [...] }`.
