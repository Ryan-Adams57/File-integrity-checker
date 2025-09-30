# File-integrity-checker

File Integrity Checker (detects tampering)

#!/usr/bin/env python3
"""
file_integrity_checker.py

Usage:
  # Create baseline hashes
  python3 file_integrity_checker.py baseline /path/to/dir baseline.json

  # Verify current files against baseline
  python3 file_integrity_checker.py verify /path/to/dir baseline.json

Notes:
  - Excludes directories from hashing (only files).
  - Uses SHA-256. Designed for learning and small directories.
"""

import os
import sys
import json
import hashlib
from pathlib import Path

def hash_file(path, chunk_size=65536):
    h = hashlib.sha256()
    with open(path, "rb") as f:
        while chunk := f.read(chunk_size):
            h.update(chunk)
    return h.hexdigest()

def build_baseline(directory):
    baseline = {}
    directory = Path(directory)
    for root, _, files in os.walk(directory):
        for fname in files:
            fpath = Path(root) / fname
            try:
                rel = str(fpath.relative_to(directory))
                baseline[rel] = hash_file(fpath)
            except (PermissionError, OSError):
                print("Skipped (no permission):", fpath)
    return baseline

def save_baseline(baseline, outpath):
    with open(outpath, "w", encoding="utf-8") as f:
        json.dump(baseline, f, indent=2)
    print("Baseline saved to", outpath)

def load_baseline(path):
    with open(path, "r", encoding="utf-8") as f:
        return json.load(f)

def verify(directory, baseline):
    directory = Path(directory)
    current = {}
    for root, _, files in os.walk(directory):
        for fname in files:
            fpath = Path(root) / fname
            try:
                rel = str(fpath.relative_to(directory))
                current[rel] = hash_file(fpath)
            except (PermissionError, OSError):
                print("Skipped (no permission):", fpath)

    added = [f for f in current if f not in baseline]
    removed = [f for f in baseline if f not in current]
    modified = [f for f in current if f in baseline and current[f] != baseline[f]]

    return {"added": added, "removed": removed, "modified": modified}

def main():
    if len(sys.argv) < 4:
        print("Usage:")
        print("  python3 file_integrity_checker.py baseline <directory> <baseline.json>")
        print("  python3 file_integrity_checker.py verify <directory> <baseline.json>")
        sys.exit(1)

    action = sys.argv[1].lower()
    directory = sys.argv[2]
    baseline_file = sys.argv[3]

    if action == "baseline":
        baseline = build_baseline(directory)
        save_baseline(baseline, baseline_file)
    elif action == "verify":
        baseline = load_baseline(baseline_file)
        result = verify(directory, baseline)
        print("Verification results:")
        print(f"  Added files:   {len(result['added'])}")
        for f in result["added"]:
            print("    +", f)
        print(f"  Removed files: {len(result['removed'])}")
        for f in result["removed"]:
            print("    -", f)
        print(f"  Modified files:{len(result['modified'])}")
        for f in result["modified"]:
            print("    *", f)
        if not (result["added"] or result["removed"] or result["modified"]):
            print("No changes detected. Files match the baseline.")
    else:
        print("Unknown action:", action)

if __name__ == "__main__":
    main()

