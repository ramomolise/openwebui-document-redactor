# Security policy

## Supported version

Security fixes are applied to the latest version on the default branch.

## Reporting a vulnerability

Use GitHub's private vulnerability reporting or Security Advisory feature for this repository. Do not open a public issue containing:

- Real source or redacted documents
- Names, contact details or identifying numbers
- Ollama or OpenWebUI credentials
- Private hostnames, IP addresses, tokens or logs containing document text

Include a minimal synthetic reproduction, the affected version and the expected versus observed behaviour.

## Threat model

The Function is intended to reduce accidental disclosure during a controlled document workflow. It assumes:

- OpenWebUI and Ollama run on trusted infrastructure.
- Only authorised users can upload and download documents.
- The Ollama endpoint is loopback or private-LAN only.
- Administrators protect OpenWebUI's database, file store, backups and logs.
- A human reviews every output before downstream use.

It does not protect against a compromised OpenWebUI host, malicious administrator, unsafe backups, an exposed Ollama endpoint or identifying content embedded inside retained images.

## Safe deployment requirements

- Keep `ALLOW_NON_PRIVATE_OLLAMA_URL=false`.
- Keep `REQUIRE_LLM_PASS=true` when the workflow must fail closed.
- Restrict OpenWebUI and Ollama with host firewalls and authentication.
- Do not route sensitive documents through a public tunnel or reverse proxy.
- Disable unrelated OpenWebUI tools, memory, web search and RAG for the redactor profile.
- Delete source and generated files according to an approved retention policy.
- Keep the generated-reference mapping outside the AI conversation.
- Inspect embedded images manually.

## Privacy limitations

The output is pseudonymised rather than anonymous when it retains a generated reference or substantive results. Deterministic and model-assisted detection can miss novel identifiers. The project therefore blocks unsupported image-only files and requires manual review.
