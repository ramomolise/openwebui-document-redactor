# Setup

This guide assumes OpenWebUI and Ollama are already installed and working.

## Requirements

- An existing OpenWebUI installation
- An OpenWebUI administrator account
- Ollama running locally or on a trusted private LAN
- A supported model already downloaded in Ollama, such as `gemma4:12b`
- PDF or DOCX files containing selectable text

## Add the Function to OpenWebUI

1. Open [`assessment_redactor_pipe.py`](openwebui_redactor/assessment_redactor_pipe.py).
2. Copy the complete file contents.
3. Sign in to OpenWebUI as an administrator.
4. Open **Admin Panel -> Functions**.
5. Select **Create Function**.
6. Paste the copied Python code into the editor.
7. Save the Function.
8. Enable **Document Redactor** using its toggle.
9. Open the Function's settings or valves panel.

## Configure the Function

Use the Ollama address that OpenWebUI can reach:

```text
OLLAMA_BASE_URL = http://host.docker.internal:11434
MODEL = gemma4:12b
CANDIDATE_PREFIX = CAND
REQUIRE_LLM_PASS = true
ALLOW_NON_PRIVATE_OLLAMA_URL = false
REDACT_UNLABELLED_CONTACTS = false
```

If Ollama runs on another machine, replace `host.docker.internal` with its private LAN address:

```text
http://192.168.1.50:11434
```

Do not add `/v1` to the address. Do not use a public Ollama endpoint for documents containing personal information.

`REDACT_UNLABELLED_CONTACTS=false` keeps unrelated publisher contact details. Change it to `true` only when every phone number and email address must be removed.

## Use the Function

1. Start a new OpenWebUI chat.
2. Select **Document Redactor** from the model selector.
3. Upload one PDF or DOCX.
4. Send:

```text
Redact
```

The Function generates a reference such as `CAND-042817` and returns a download link.

To choose the reference yourself, send:

```text
Candidate ID: CAND-0042
```

Download the redacted file and compare it with the original before using it.

## Recommended OpenWebUI settings

Use Document Redactor only as a file-processing utility. Disable unrelated features for this model where possible:

- Web search
- Memory
- File Context or RAG
- Knowledge collections
- External tools
- Code interpreter

## If it does not work

- **Function not listed:** confirm it is saved and enabled, then refresh OpenWebUI.
- **Ollama connection error:** check the private address, model name and network access from OpenWebUI.
- **Missing Python module:** install `PyMuPDF`, `lxml` and `httpx` in the Python environment used by OpenWebUI.
- **Scanned PDF blocked:** image-only documents are not supported because OCR redaction is not yet implemented.
- **No download released:** the Function failed closed because detection or validation did not complete safely.

## After downloading

OpenWebUI stores uploaded and generated files. Delete the chat and its files when required by your retention policy. Keep the candidate-reference mapping outside the AI conversation.
