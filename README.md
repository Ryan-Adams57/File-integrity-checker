# File Integrity Checker

A simple Python tool to detect file changes in a directory using SHA-256 checksums. Useful for checking if files have been added, modified, or deleted.

> 🛡️ **Educational tool** — not meant for cryptographic or security-critical integrity enforcement.

## Features

- Creates a baseline of SHA-256 hashes of all files in a directory
- Later verifies the current state of files against the saved baseline
- Detects:
  - 🟢 Added files
  - 🔴 Removed files
  - 🟡 Modified files
- Simple CLI interface — no external dependencies

## Requirements

- Python 3.7+

## Usage

### Create baseline (initial scan)
```bash
python3 file_integrity_checker.py baseline /path/to/dir baseline.json
