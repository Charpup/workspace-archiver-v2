# Workspace Archiver V2

[![Version](https://img.shields.io/badge/version-2.0.1-blue.svg)](https://github.com/Charpup/workspace-archiver-v2/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-skill-purple.svg)](https://openclaw.ai)

Archive and backup OpenClaw workspace projects with multi-format compression, versioning, policy-driven auto-archiving, and storage analysis.

## V2 Highlights

| Feature | V1 | V2 |
|---------|----|----|
| Compression formats | gzip only | gzip / bzip2 / xz / zip / 7z |
| Encryption | No | Yes (AES-256-GCM) |
| Auto-archive policy | No | Yes (AutoArchivePolicy) |
| Storage analysis | No | Yes (`analyze` command) |
| Retention cleanup | Manual | Configurable |

## Installation

```bash
npm install workspace-archiver-v2
```

## Usage

```bash
# Archive entire workspace
workspace-archiver archive --all

# Archive a specific project
workspace-archiver archive ./projects/my-project

# Create a named snapshot
workspace-archiver snapshot --name "pre-release-v2"

# Restore from archive
workspace-archiver restore archive-2026-02-25.tar.gz --to ./restored

# Analyze workspace storage
workspace-archiver analyze

# Clean up archives older than 30 days, keep last 10
workspace-archiver cleanup --keep 10 --older-than 30d
```

## Supported Compression Formats

| Format | Extension | Compression | Speed | Notes |
|--------|-----------|-------------|-------|-------|
| gzip | .tar.gz | Medium | Fast | Default, best compatibility |
| bzip2 | .tar.bz2 | High | Medium | Better ratio |
| xz | .tar.xz | Highest | Slow | Maximum compression |
| zip | .zip | Medium | Fast | Windows compatible |
| 7z | .7z | High | Medium | Requires 7zip |

## License

MIT
