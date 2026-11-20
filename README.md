# SpecPilot — BRS Test Agent

A web-based QA assistant that analyzes business requirements documents and reference material, generates traceable test cases, builds an RTM, and prepares regression scenarios from failed tests.

## Features

- Upload a BRS plus reference files (up to 12 files, 15 MB each): PDF, DOCX, TXT, PNG, JPEG, or WebP.
- Extracts text from PDF/DOCX and analyzes DOCX-embedded and separately uploaded images with a vision-capable model.
- Generates positive, negative, boundary, and exception tests with requirement IDs, test steps, expected results, severity, and an editable Pass/Fail status.
- Generates an RTM and exports a three-sheet Excel workbook: **Test Cases**, **RTM**, and **Regression**.
- Mark cases as **Fail** and generate focused regression scenarios; download the updated workbook.
- Upload a revised BRS again to regenerate coverage against its current contents.

> Legacy `.doc` files must first be saved as `.docx` or PDF. The download is a modern `.xlsx` Excel workbook (rather than the obsolete binary `.xls` format); it opens in Excel and other spreadsheet tools. The app does not monitor a shared drive for changes: upload each revised version manually. For linked/attached references, select and upload those files along with the BRS.

## Run locally

Requires Node.js 20+ and an OpenAI API key.

```bash
npm install
OPENAI_API_KEY=your_key_here npm start
```

Open `http://localhost:3000`. To select a different supported model, set `OPENAI_MODEL` (default: `gpt-4o-mini`). On Windows PowerShell, set the environment variable with `$env:OPENAI_API_KEY="your_key_here"` before running `npm start`.

The API key is read only by the server and must never be placed in the browser or committed to the repository. Without a configured key, the app returns a setup message instead of generating results.

## Deploy online

Deploy this Node.js application to a host that supports Node 20+ (for example, Render, Railway, Fly.io, or a VM). Configure the start command as `npm start`, set `OPENAI_API_KEY` as a private environment secret, and use the provider's HTTPS URL. GitHub Pages alone cannot host the Node API. Keep uploaded documents private and deploy only to a workspace whose access policy is appropriate for the requirements data.

## Data handling and limitations

Uploads are held in server memory for the request and are not written to disk or retained by this application. Document text and supported images are sent to the configured OpenAI API for analysis; review your provider's privacy and retention terms before uploading sensitive business data. Apply authentication, access controls, rate limits, and appropriate logging/retention policies before using this app with confidential documents or a public deployment. The model's output is a draft and must be reviewed by a tester. Scanned PDFs without selectable text may require OCR before upload. The app does not automatically monitor shared drives, read arbitrary embedded file attachments, or synchronize version changes; upload the BRS and its reference documents together.

## API

- `GET /api/health` — app status and whether an AI key is configured.
- `POST /api/generate` — multipart field `documents` (one or more supported files).
- `POST /api/regression` — JSON body `{ "failedCases": [...] }`.
