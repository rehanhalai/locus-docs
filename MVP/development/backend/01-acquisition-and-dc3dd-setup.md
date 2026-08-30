# Acquisition & dc3dd Setup Notes

## What We Did
- Chose `dc3dd` as the physical drive acquisition engine for bit-stream cloning and bad sector defense (`conv=noerror,sync`).
- Decided to bundle `dc3dd` directly inside `backend/bin/` instead of asking users to install it on their host system.
- Used a **statically compiled binary** for Linux so it has zero external library dependencies and works across any Linux distro without glibc version issues.

## Binary Directory Structure
```
backend/bin/
├── linux/
│   └── dc3dd          # Static ELF binary (Linux)
└── windows/
    ├── dc3dd.exe      # Windows executable
    └── cygwin1.dll    # Required Windows DLL
```

## How It Works in Locus
- Python executes `dc3dd` as an asynchronous background subprocess.
- Communicates purely over CLI arguments and `stderr` telemetry (speed, %, ETA).
- Avoids GNU GPL licensing issues by keeping `dc3dd` as a separate standalone CLI binary rather than embedding C code into Python.

## Next Steps
- Write Python helper (`app/core/binaries.py`) to automatically locate the binary from `backend/bin/`.
- Test running acquisition and capturing progress over WebSockets.
