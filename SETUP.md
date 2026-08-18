# Setup guide

This guide installs the Document Redactor as an OpenWebUI Function and connects it to a private Ollama server.

## 1. Requirements

- OpenWebUI with administrator access
- Ollama reachable through loopback, Docker host networking or a trusted private LAN
- A local model capable of structured JSON output; `gemma4:12b` is the default
- Python packages `PyMuPDF`, `lxml` and `httpx` available inside the OpenWebUI container
- PDF or DOCX source documents with extractable text

Do not use a public reverse proxy, tunnel or internet-facing Ollama URL for documents containing personal information.

## 2. Start OpenWebUI

### Docker run

If Ollama is on the same Linux host:

```bash
docker run -d \
  --name open-webui \
  --restart unless-stopped \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

Open `http://SERVER_LAN_IP:3000` from a trusted LAN device.

### Docker Compose

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - open-webui:/app/backend/data

volumes:
  open-webui:
```

Start it with:

```bash
docker compose up -d
```

If OpenWebUI is already running, keep the existing container and continue.

## 3. Make Ollama reachable privately

### Ollama on the same machine

Use this Function valve:

```text
OLLAMA_BASE_URL = http://host.docker.internal:11434
```

On Linux, ensure the OpenWebUI container has the `host.docker.internal:host-gateway` mapping shown above.

### Ollama on another LAN machine

Configure Ollama to listen on its LAN interface. The exact service configuration depends on the operating system. The resulting endpoint should resemble:

```text
http://192.168.1.50:11434
```

Allow TCP port `11434` only from the OpenWebUI host or trusted private subnet. Do not expose it directly to the internet.

Test connectivity from inside the OpenWebUI container:

```bash
docker exec open-webui python -c "import urllib.request; print(urllib.request.urlopen('http://host.docker.internal:11434/api/tags').status)"
```

For a separate Ollama host, replace `host.docker.internal` with its private LAN address. A successful result prints `200`.

## 4. Pull a model

On the Ollama machine:

```bash
ollama pull gemma4:12b
```

You can use another local model, but it must follow the supplied JSON schema reliably. Keep temperature at the Function's fixed value of zero.

## 5. Install the Python dependencies

The Function header declares its requirements. Some OpenWebUI versions install them automatically when the Function is enabled.

If imports fail, install them temporarily in the running container:

```bash
docker exec -u 0 open-webui pip install --no-cache-dir PyMuPDF lxml httpx
docker restart open-webui
```

Packages installed this way disappear when the container is recreated. For a durable deployment, build a small custom image:

```dockerfile
FROM ghcr.io/open-webui/open-webui:main
RUN pip install --no-cache-dir PyMuPDF lxml httpx
```

Build and run that image using your normal Docker or Compose workflow.

## 6. Add the OpenWebUI Function

1. Sign in to OpenWebUI as an administrator.
2. Open **Admin Panel -> Functions**.
3. Choose **Create Function**.
4. Copy the complete contents of `openwebui_redactor/assessment_redactor_pipe.py`.
5. Paste it into the Function editor.
6. Save the Function.
7. Enable **Document Redactor**.
8. Open its valve/settings panel.

Recommended configuration:

```text
OLLAMA_BASE_URL = http://host.docker.internal:11434
MODEL = gemma4:12b
CANDIDATE_PREFIX = CAND
TIMEOUT_SECONDS = 240
CHUNK_CHARACTERS = 12000
MINIMUM_LLM_CONFIDENCE = 0.72
REQUIRE_LLM_PASS = true
ALLOW_NON_PRIVATE_OLLAMA_URL = false
REDACT_UNLABELLED_CONTACTS = false
MAX_FILES_PER_REQUEST = 3
```

`REDACT_UNLABELLED_CONTACTS=false` preserves unrelated publisher and service-provider contact details. Set it to `true` for a stricter document-wide contact scrub.

## 7. Limit ordinary chat features

Use the Function as a dedicated utility rather than a general chatbot. Disable unrelated features for its workspace model where available:

- Web search
- Memory
- File Context/RAG
- Knowledge collections
- External tools and integrations
- Code interpreter

The Function reads the original upload through OpenWebUI's file service. It does not need document retrieval.

## 8. Redact a document

1. Start a fresh chat.
2. Select **Document Redactor** from the model selector.
3. Upload one PDF or DOCX.
4. Send:

```text
Redact
```

The Function generates a reference such as `CAND-042817` and returns a download link.

To supply an approved non-identifying reference:

```text
Candidate ID: CAND-0042
```

Download the result and compare it page by page with the source before using it.

## 9. Mandatory review checklist

- Confirm the subject's original name is absent.
- Confirm email, phone, source ID, passport, national ID and birth date are absent where applicable.
- Confirm the generated reference is consistent.
- Confirm assessment/report dates and selected biographical values are removed.
- Confirm scores, charts, pronouns, wording and recommendations are unchanged.
- Search the output for every original identifier.
- Check images manually because embedded image OCR is not implemented.
- Confirm the output opens normally and has the same page count or document structure.

## 10. Retention and access

OpenWebUI stores uploaded and generated files in its configured data volume. After downloading and reviewing the result:

1. Delete the chat.
2. Open **Settings -> Data Controls -> Manage Files**.
3. Delete the original and redacted files.
4. Apply an approved backup and log-retention policy.
5. Keep the reference-to-person mapping outside the AI conversation and accessible only to authorised staff.

For a zero-retention workflow, place the redaction engine behind a dedicated upload/download interface that deletes temporary files automatically. This Function alone does not provide zero retention.

## 11. Troubleshooting

### The Function says the Ollama address is not private

Use loopback, `host.docker.internal`, a `.local` hostname or an RFC1918 private IP address. Leave `ALLOW_NON_PRIVATE_OLLAMA_URL=false` for real personal information.

### Ollama connection fails

- Confirm `ollama list` works on the model host.
- Confirm `/api/tags` returns HTTP 200 from inside the OpenWebUI container.
- Check the host firewall.
- Check Docker's `extra_hosts` mapping.
- Do not append `/v1`; the Function builds the native `/api/chat` URL.

### The local model pass fails

- Confirm the configured model name exactly matches `ollama list`.
- Increase `TIMEOUT_SECONDS` for slower hardware.
- Reduce `CHUNK_CHARACTERS` if the model runs out of context or memory.
- Keep `REQUIRE_LLM_PASS=true` when release must fail closed.

### A scanned PDF is blocked

This is expected. The project does not yet implement OCR with coordinate-safe redaction. Convert it through an approved OCR pipeline or review it manually; do not bypass the block for sensitive documents.

### A publisher telephone number is removed

Set `REDACT_UNLABELLED_CONTACTS=false`. Labelled subject contact fields will still be processed, while unlabelled or publisher contact details remain.
