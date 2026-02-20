---
icon: image-landscape
---

# Evidence Handling

## Evidence Requirements

### Code Blocks Over Screenshots

Wherever possible, use code blocks for terminal output rather than screenshots. Code blocks are easier to redact, neater in long reports, and allow the client to copy-paste commands for their own testing.

### Mandatory Redaction

All sensitive information, including cleartext passwords and hashes, must be redacted. Use solid colored blocks for images and placeholders like `<REDACTED>` or `<SNIP>` for text.

### Annotated Figures

Do not just provide a caption; use arrows or boxes within your screenshots to draw the reader's eye to the specific evidence of the vulnerability.
