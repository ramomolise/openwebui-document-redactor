# Contributing

Contributions that improve detection accuracy, document fidelity, validation and test coverage are welcome.

## Ground rules

- Use only synthetic test data.
- Never commit real documents or personal information.
- Preserve document wording and layout unless a change is necessary for secure redaction.
- Add regression tests for every detector or document-processing change.
- Keep private-endpoint enforcement enabled by default.
- Document any new false-positive or false-negative trade-off.

## Local checks

```bash
python -m pip install -r requirements-dev.txt
python -m unittest discover -s openwebui_redactor/tests -v
python -m py_compile openwebui_redactor/redactor_pipe.py
```

Submit focused pull requests with a clear description of the privacy or fidelity problem being addressed.
