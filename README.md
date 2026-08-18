# OpenWebUI Document Redactor

A layout-preserving OpenWebUI Function for redacting personal information from PDF and DOCX documents. It combines deterministic matching with an optional private Ollama pass and returns a downloadable redacted copy instead of rewriting the document in chat.

It is designed for controlled document workflows where formatting, charts, scores and assessment wording must remain intact.

## Highlights

- Preserves PDF page dimensions, wording, charts and layout.
- Preserves DOCX paragraphs, tables, headers, footers, styles and text boxes.
- Replaces the subject's name and source identifiers with a generated reference such as `CAND-042817`.
- Removes common direct identifiers, exact dates and configurable biographical fields.
- Keeps gendered pronouns, scores and assessment narrative unchanged for interpretive accuracy.
- Uses true PDF redactions rather than drawing cosmetic boxes over recoverable text.
- Handles Word identifiers split across multiple styled runs.
- Removes metadata, hidden text, scripts, links, form values, attachments and cached thumbnails.
- Validates detected source values are absent before releasing the output.
- Restricts the Ollama endpoint to private or loopback addresses by default.
- Does not print source content or removed values in the chat response.
- Preserves publisher contact details by default while redacting labelled subject contact fields.

## Intended workflow

1. A trusted user uploads one PDF or DOCX in OpenWebUI.
2. Deterministic rules detect structured identifiers and sensitive fields.
3. A private local Ollama model optionally identifies additional exact strings.
4. The Function edits a copy of the original file.
5. Validation blocks release if a detected value remains.
6. The user downloads and manually reviews the output.

The generated reference and the document's substantive results remain by design. The result is **pseudonymised, not anonymous**.

## What is removed

- Subject/candidate name
- Source candidate, employee, application or test identifiers
- National ID, passport and date of birth
- Labelled subject email and phone values
- Exact assessment and report dates
- Candidate-specific organisation in headings such as `Report for Example Organisation`
- Labelled nationality, ethnicity, education, discipline, functional area and position
- Labelled colour-vision and previous-assessment fields
- Self-evaluation answers
- Identifying metadata and hidden document layers

## What is deliberately retained

- Generated `CAND-######` reference
- Gendered pronouns
- Scores, percentiles, charts and tables
- Assessment narrative and recommendations
- Standard questions
- Publisher names, addresses and contact details

Set `REDACT_UNLABELLED_CONTACTS=true` if every email address and phone-like value should be removed, including publisher or service-provider contact details.

## Quick start

See [SETUP.md](SETUP.md) for complete OpenWebUI, Docker, Ollama networking, firewall, configuration and troubleshooting instructions.

At a high level:

1. Run OpenWebUI and Ollama on a trusted machine or private LAN.
2. In OpenWebUI, open **Admin Panel -> Functions -> Create Function**.
3. Paste the complete contents of `openwebui_redactor/assessment_redactor_pipe.py`.
4. Save and enable the Function.
5. Configure its valves, especially `OLLAMA_BASE_URL` and `MODEL`.
6. Select **Document Redactor**, upload a PDF or DOCX, and send `Redact`.

## Safe default valves

```text
OLLAMA_BASE_URL = http://host.docker.internal:11434
MODEL = gemma4:12b
REQUIRE_LLM_PASS = true
ALLOW_NON_PRIVATE_OLLAMA_URL = false
REDACT_UNLABELLED_CONTACTS = false
```

The model is used only to return exact identifying substrings. It does not rewrite, summarise or interpret the document.

## Important limitations

- Scanned or image-only PDFs are blocked because OCR redaction is not implemented.
- Embedded images are retained and require visual review.
- DOCX files containing embedded OLE objects are blocked.
- Replacing Word text can cause minor line wrapping when the generated reference is longer than the original value.
- OpenWebUI stores uploaded and generated files according to its own storage configuration. Delete them after download or implement an approved retention policy.
- No automated detector can guarantee that every identifying detail has been found. Human comparison with the original remains mandatory.
- This project is a technical privacy aid, not a guarantee of legal or regulatory compliance.

## Development and tests

Install the test dependencies:

```bash
python -m pip install -r requirements-dev.txt
```

Run the suite from the repository root:

```bash
python -m unittest discover -s openwebui_redactor/tests -v
```

The tests cover PDF and DOCX structure preservation, split Word runs, metadata removal, secure identifier removal, sensitive-field scoping, narrative preservation and publisher-contact preservation.

## Security

Read [SECURITY.md](SECURITY.md) before processing real personal information. Please report security issues privately rather than opening a public issue containing sample documents or identifying data.

## Licence

MIT. See [LICENSE](LICENSE).
