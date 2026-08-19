# OpenWebUI Document Redactor

An OpenWebUI Function that removes personal and identifying information from PDF and DOCX files without rebuilding or rewriting the document.

It keeps the original layout and substantive content. The primary person's name and source identifier are replaced with an external reference such as `EXT-7F92-KQ4D`; other sensitive values are physically redacted.

## Features

- Preserves the original PDF or DOCX design.
- Redacts names, IDs, contact details, birth dates and selected biographical information.
- Keeps non-sensitive wording, tables, charts, figures and pronouns unchanged.
- Uses true PDF redaction, not removable black overlays.
- Removes metadata, hidden text, attachments, scripts and form values.
- Uses a private Ollama model to find additional identifiers.
- Blocks the download if a detected identifier remains.
- Keeps publisher contact details by default.

## Install

1. Make sure OpenWebUI can reach Ollama through localhost or a private LAN address.
2. In OpenWebUI, open **Admin Panel -> Functions -> Create Function**.
3. Paste the contents of [`redactor_pipe.py`](openwebui_redactor/redactor_pipe.py).
4. Save and enable **Document Redactor**.
5. Configure these valves:

```text
OLLAMA_BASE_URL = http://127.0.0.1:11434
MODEL = gemma4:12b
REFERENCE_PREFIX = EXT
REQUIRE_LLM_PASS = true
ALLOW_NON_PRIVATE_OLLAMA_URL = false
REDACT_UNLABELLED_CONTACTS = false
```

For complete Function installation, configuration and troubleshooting, follow the [setup guide](SETUP.md).

## Use

1. Select **Document Redactor** in OpenWebUI.
2. Upload one PDF or DOCX.
3. Send `Redact`.
4. Download and manually compare the result with the original.

To choose the external reference, send:

```text
Reference ID: EXT-7F92-KQ4D
```

## Important

- The output is pseudonymised, not anonymous.
- Scanned or image-only PDFs are blocked because OCR is not yet supported.
- Embedded images must be checked manually.
- OpenWebUI stores uploaded and generated files. Delete them after download if required by your retention policy.
- Always review the output before using it.

## Test

```bash
python -m pip install -r requirements-dev.txt
python -m unittest discover -s openwebui_redactor/tests -v
```

See [SECURITY.md](SECURITY.md) for safe deployment guidance. Licensed under the [MIT License](LICENSE).
