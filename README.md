# Markush FECS

[![Version](https://img.shields.io/badge/Version-3.0.0-blue.svg?style=flat-square)](https://github.com/purn9954/Markush_FECS/releases)
[![License](https://img.shields.io/badge/License-Proprietary%20Evaluation-orange.svg?style=flat-square)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows%20(x86--64)-green.svg?style=flat-square)](https://github.com/purn9954/Markush_FECS/releases)
[![Language](https://img.shields.io/badge/Language-C%2B%2B17%20%2F%20C%20ABI-informational.svg?style=flat-square)](include/fecs_c_api.h)
[![Developer](https://img.shields.io/badge/Developer-Purn__Vadodariya-lightgrey.svg?style=flat-square)](mailto:purn872008@gmail.com)

**Developer & Author:** Purn_Vadodariya  
**Contact:** [purn872008@gmail.com](mailto:purn872008@gmail.com)  
**Repository:** [https://github.com/purn9954/Markush_FECS](https://github.com/purn9954/Markush_FECS)  
**Release Downloads:** [GitHub Releases](https://github.com/purn9954/Markush_FECS/releases)  

---

## Table of Contents

1. [Overview](#1-overview)
2. [Why Markush FECS? (The Problem It Solves)](#2-why-markush-fecs-the-problem-it-solves)
3. [How It Works (Cryptographic Architecture)](#3-how-it-works-cryptographic-architecture)
4. [Where It Can Be Used](#4-where-it-can-be-used)
5. [Release Downloads & Package Contents](#5-release-downloads--package-contents)
6. [Installation & First Run](#6-installation--first-run)
7. [Command-Line Interface (CLI) Guide](#7-command-line-interface-cli-guide)
8. [Reusable C API Integration](#8-reusable-c-api-integration)
9. [Evaluation License & Quota Notice](#9-evaluation-license--quota-notice)
10. [Reported Test Results](#10-reported-test-results)
11. [Reported Benchmarks](#11-reported-benchmarks)
12. [Verification of Downloaded Artifacts](#12-verification-of-downloaded-artifacts)
13. [Limitations & Security Boundaries](#13-limitations--security-boundaries)
14. [License & Support](#14-license--support)

---

## 1. Overview

**Markush FECS** (Fragmented Encrypted Container System) is a native C++17 cryptographic storage engine and command-line application. It packages files into fixed-slot container archives (`.fecs`) and restores them through a sequential batch decryption pipeline.

The project is structured with dual interfaces:
- **Standalone Command-Line Tool (`Markush_FECS.exe` / `Markush_FECS.AppImage` / `fecs`):** Direct terminal usage for file archiving, verification, inspection, and scripted automation.
- **Reusable Core Engine Library (`libfecs.so` / `libfecs.dll`):** A standard C ABI (`fecs_c_api.h`) allowing applications in C, C++, Python, C#, or other languages to embed container encryption directly into their storage workflows.

---

## 2. Why Markush FECS? (The Problem It Solves)

Standard file encryption utilities (such as basic AES wrappers, OpenSSL enc, or traditional password-protected zip archives) encrypt file contents but leave critical side channels exposed:

1. **Exact Length Leakage:** A ciphertext that is 1,234,567 bytes long reveals the exact length of the original data. In many scenarios, file sizes serve as unique fingerprints that allow an observer to identify specific documents, media files, or database schemas without decrypting the data.
2. **Metadata & Directory Structure Fingerprinting:** Archival headers often expose original file extensions, timestamps, and directory structures.
3. **Traffic & Pattern Analysis:** If an encrypted backup changes by only a small fraction, an observer can infer update frequency and modification locations.

### The Markush FECS Approach:
- **Proposition 1 Uniform Chunking:** Files are fragmented into fixed 512 KiB chunks. The final chunk is padded to 512 KiB, ensuring all slots share an identical byte length.
- **Decoy Slot Dispersion:** Cryptographically indistinguishable pseudorandom decoy chunks are interspersed throughout the container. An observer cannot differentiate between real data and decoy slots.
- **Deterministic Permutation:** Physical slot locations are shuffled across the archive using a Fisher-Yates permutation keyed by a derived master secret.
- **Authenticated Encryption:** Each slot is independently protected with AES-256-GCM AEAD, binding the chunk index to its authentication tag.

---

## 3. How It Works (Cryptographic Architecture)

```text
[Input Plaintext File]
        │
        ▼
[Uniform Chunking] ──────── 512 KiB Chunks (Final chunk PKCS#7 padded)
        │
        ▼
[Decoy Generation] ──────── Pseudorandom 512 KiB decoy slots inserted
        │
        ▼
[Slot Shuffling]    ──────── Cryptographic Fisher-Yates permutation (HKDF subkey)
        │
        ▼
[AES-256-GCM AEAD] ──────── Unique IV per slot + 16-byte Poly1305/GCM tag
        │
        ▼
[Sequential Write] ──────── 32-byte header + 28-byte slot headers + slots
        │
        ▼
[.fecs Container Archive]
```

### Cryptographic Pipeline Details:
1. **Key Derivation (Argon2id):** The user passphrase and a unique 32-byte salt stored in the container header are processed by Argon2id (RFC 9106, $t=3$ iterations, $m=64\text{ MiB}$ memory, $p=4$ parallel threads) to generate a 256-bit Master Key.
2. **Key Expansion (HKDF-SHA256):** The Master Key is expanded (RFC 5869) into independent subkeys:
   - `K_meta`: Authenticates and encrypts container-level metadata.
   - `K_enc`: Encrypts individual 512 KiB data chunks.
   - `K_perm`: Seeds the deterministic Fisher-Yates permutation.
   - `K_decoy`: Seeds the ChaCha20/AES PRNG that generates decoy slots.
3. **Sequential Batch Decryption:** Decryption reads physical slots sequentially from disk in batches, lookups their logical positions in the inverted permutation table, and streams the restored data in order, avoiding random disk seek thrashing on large files.

---

## 4. Where It Can Be Used

- **Confidential Archiving & Backups:** Encrypting proprietary databases, private source repositories, financial documents, or personal records before uploading to cloud storage.
- **Air-Gapped Systems:** Securely transferring archives across air-gapped workstations without requiring network-based key management or phone-home activation.
- **Application Storage Backend:** Embedding `libfecs` into background services, document management systems, or data export pipelines on supported x86-64 platforms.
- **Research & Evaluation:** Evaluating side-channel resistant container formats and offline cryptographic licensing models.

---

## 5. Release Downloads & Package Contents

Packages are available from the **[GitHub Releases page](https://github.com/purn9954/Markush_FECS/releases)**:

| Platform | Release File | Purpose & Contents |
| :--- | :--- | :--- |
| **Linux (x86-64)** | [`Markush_FECS.AppImage`](linux/Markush_FECS.AppImage) | **Single-file portable executable.** Self-contained with bundled OpenSSL, Argon2, and C++ runtimes. |
| **Linux (x86-64)** | [`Markush_FECS_Linux.zip`](linux/Markush_FECS_Linux.zip) | **Complete Linux archive.** Contains `Markush_FECS.AppImage`, native ELF binary `fecs`, `libfecs.so`, license tool, evaluation license, and README. |
| **Windows (x86-64)** | [`Markush_FECS_Windows.zip`](windows/Markush_FECS_Windows.zip) | **Complete Windows package.** Contains `Markush_FECS.exe`, `libfecs.dll`, OpenSSL DLLs, Argon2 DLL, LLVM runtime DLLs, license tool, evaluation license, and guide. |
| **Windows (x86-64)** | [`Markush_FECS.exe`](windows/Markush_FECS.exe) | Windows executable alone (requires companion DLLs in the same folder). |
| **Both** | [`SHA256SUMS`](SHA256SUMS) | Cryptographic checksum manifest for package verification. |

---

## 6. Installation & First Run

### Linux AppImage (Direct Single-File Run)

You can download **just the single `Markush_FECS.AppImage` file** on any 64-bit Linux distribution with `glibc` 2.31 or newer. All required libraries are bundled inside.

```bash
# 1. Download Markush_FECS.AppImage from Releases
# 2. Grant execution permissions
chmod +x Markush_FECS.AppImage

# 3. Check version and developer attribution
./Markush_FECS.AppImage --version

# 4. Check license status
./Markush_FECS.AppImage license-status
```

> [!TIP]
> If running in an environment without `/dev/fuse` (e.g., minimal Docker containers or WSL), run with:  
> `./Markush_FECS.AppImage --appimage-extract-and-run --version`

---

### Windows (Extract the ZIP Package)

On Windows, download **`Markush_FECS_Windows.zip`**. The `.exe` requires its companion DLLs located in the same directory.

1. Download **`Markush_FECS_Windows.zip`**.
2. Right-click and choose **Extract All...** to an extraction folder (e.g., `C:\Tools\Markush_FECS\`).
3. Open PowerShell or Command Prompt inside that directory.
4. Run:
   ```powershell
   .\Markush_FECS.exe --version
   .\Markush_FECS.exe license-status
   ```

> [!IMPORTANT]
> Do not move `Markush_FECS.exe` by itself outside the extracted folder unless the accompanying DLLs (`libfecs.dll`, `libcrypto-3-x64.dll`, `libargon2.dll`, `libc++.dll`) are moved alongside it.

---

## 7. Command-Line Interface (CLI) Guide

All commands share the same syntax across Windows (`.\Markush_FECS.exe`) and Linux (`./Markush_FECS.AppImage`).

### 1. Encrypt a File
Creates a `.fecs` container archive. Consumes **1 license operation** upon success.
```bash
./Markush_FECS.AppImage encrypt -p "YourPassphrase123!" source.dat vault.fecs
```
*Options:*
- `-p, --passphrase`: Passphrase string (interactive prompt if omitted).
- `--passphrase-file <file>`: Read passphrase from a file.
- `-d, --decoy <ratio>`: Decoy slot ratio (default `0.20` for 20% decoys).
- `-l, --license <file>`: Specify an alternate license file.

### 2. Decrypt a Container
Restores original file from a container. Consumes **1 license operation** upon success.
```bash
./Markush_FECS.AppImage decrypt -p "YourPassphrase123!" vault.fecs restored.dat
```

### 3. Verify Container Integrity (Free Operation)
Verifies all slot authentication tags in-place without writing plaintext to disk. **Does not consume license operations.**
```bash
./Markush_FECS.AppImage verify -p "YourPassphrase123!" vault.fecs
```

### 4. Inspect Public Container Header (Free Operation)
Displays container format version, slot counts, and chunk size without requiring a passphrase.
```bash
./Markush_FECS.AppImage inspect vault.fecs
```

### 5. Check License Status (Free Operation)
Displays current license ID, operation count (e.g., `3 / 10`), and remaining quota.
```bash
./Markush_FECS.AppImage license-status
```

### 6. Display Host Installation ID (Free Operation)
Prints the local node installation ID for generating machine-bound licenses.
```bash
./Markush_FECS.AppImage license-bind
```

---

## 8. Reusable C API Integration

The core engine is built as a dynamic shared library (`libfecs.so` on Linux, `libfecs.dll` on Windows) providing a standard C ABI via `include/fecs_c_api.h`. Applications on supported x86-64 systems can integrate the engine directly.

### C / C++ Integration
```c
#include "fecs_c_api.h"
#include <stdio.h>

int main(void) {
    // Encrypt a file (0.20 = 20% decoy ratio)
    FECS_Result res = fecs_encrypt("plain.txt", "vault.fecs", "Password123!", 0.20);
    if (res.code != FECS_SUCCESS) {
        fprintf(stderr, "Encryption error: %s\n", res.message);
        return 1;
    }

    // Decrypt the container
    FECS_Result dec = fecs_decrypt("vault.fecs", "restored.txt", "Password123!");
    if (dec.code != FECS_SUCCESS) {
        fprintf(stderr, "Decryption error: %s\n", dec.message);
        return 1;
    }

    // In-place integrity verification
    FECS_Result ver = fecs_verify("vault.fecs", "Password123!");

    // Inspect container header
    FECS_ContainerInfo info;
    fecs_inspect("vault.fecs", &info);
    printf("Slots: %llu, Chunk size: %llu bytes\n", info.total_slots, info.chunk_size);

    return 0;
}
```

### Python Integration via `ctypes`
```python
import ctypes

# Load engine library (libfecs.so or libfecs.dll)
lib = ctypes.CDLL("./libfecs.so")

# Call encryption directly
status = lib.fecs_encrypt(
    b"document.pdf",
    b"vault.fecs",
    b"Password123!",
    ctypes.c_double(0.20)
)
print("Return code:", status)
```

---

## 9. Evaluation License & Quota Notice

Markush FECS includes an offline, Ed25519-signed evaluation licensing subsystem.

> [!NOTE]
> - **Evaluation Quota:** Each evaluation license (`markush.license`) permits **exactly 10 successful cryptographic operations** (`encrypt` or `decrypt`).
> - **Free Operations:** Inspection (`inspect`), verification (`verify`), status checks (`license-status`), version queries (`--version`), and binding queries (`license-bind`) are completely free and consume zero license quota.
> - **Exhaustion Behavior:** After 10 successful operations, further encrypt or decrypt operations are rejected with Exit Code `107` (`MARKUSH LICENSE EXHAUSTED`).
> - **Offline & Private:** License verification runs entirely offline using an embedded root verification public key. No network traffic, activation calls, or telemetry are ever made.
> - **Atomic Two-Phase Commit:** A license slot is reserved before the operation begins and committed only upon cryptographic success. If an error occurs (such as disk full or wrong password), the reservation is automatically rolled back.
> - **Concurreny Safe:** State is coordinated via OS-level file locking (`flock` on Linux, `LockFileEx` on Windows) to prevent race conditions during parallel execution.

---

## 10. Reported Test Results

The project's test suite records the following results for its documented test runs. These are **project-reported results** verified in the internal automated test environment; they do not represent an independent third-party commercial audit.

| Test Area | Scope & Test Method | Reported Result |
| :--- | :--- | :---: |
| **Known-Answer Tests (KAT)** | RFC 5869 (HKDF-SHA256), NIST SP 800-38D (AES-256-GCM), RFC 9106 (Argon2id) standard vectors | **100% Bit-for-Bit Match (PASS)** |
| **Round-Trip Recovery** | Boundary sizes from empty files (0B) and 1B up to 100 MB and 20 GB | **Byte-Identical SHA-256 (PASS)** |
| **Malformed Containers** | 1,000 malformed variations, invalid version tags, truncated headers, corrupted sizes | **100% Detected & Rejected (PASS)** |
| **Tamper Detection** | 19 mutation vectors targeting container headers, slot headers, ciphertexts, and tags | **19 of 19 Vectors Rejected (PASS)** |
| **Sanitizer Runs** | Clang AddressSanitizer (ASan) and UndefinedBehaviorSanitizer (UBSan) instrumented builds | **0 Memory Leaks, 0 Errors (PASS)** |
| **Licensing Security Suite** | 17 automated security scenarios (limit enforcement, 11th op rejection, rollback, recovery) | **17 of 17 Passed (PASS)** |
| **Concurrency Stress Test** | 15 parallel processes competing simultaneously for 10 evaluation license slots | **10 Succeeded, 5 Rejected (107) (PASS)** |
| **Memory Streaming Profile** | Peak RSS monitoring across file sizes up to 20 GB | **Strictly Bounded <90 MB (PASS)** |

---

## 11. Reported Benchmarks

Empirical measurements transcribed from the project's Linux test host.

### Test Environment:
- **CPU:** Intel Core i5-1135G7 @ 2.40 GHz (4 cores, 8 threads, hardware AES-NI)
- **Memory:** 7.46 GB RAM
- **Operating System:** Debian GNU/Linux (kernel 6.8.0, ext4 filesystem on NVMe SSD)
- **Cryptography Libraries:** OpenSSL 3.0.13 and libargon2
- **Parameters:** AES-256-GCM, Argon2id ($t=3, m=64\text{ MiB}, p=4$), 512 KiB slots, 20% decoy ratio

### Benchmark Results:

| Input Size | Encryption Time | Encryption Speed | Decryption Time | Decryption Speed | Container Size | Expansion Overhead | Peak RSS |
|---:|---:|---:|---:|---:|---:|---:|---:|
| **1 MB** | 0.08 s | 12.08 MB/s | 0.09 s | 10.57 MB/s | 3.00 MB | +200.00%* | 88.2 MB |
| **100 MB** | 0.37 s | 268.71 MB/s | 0.58 s | 173.84 MB/s | 121.01 MB | +21.01% | 88.5 MB |
| **1,000 MB** (1 GB) | 4.39 s | 227.66 MB/s | 4.80 s | 208.32 MB/s | 1,201.05 MB | +20.10% | 88.6 MB |
| **5,000 MB** (5 GB) | 27.82 s | 179.75 MB/s | 56.52 s | 88.47 MB/s | 6,001.25 MB | +20.02% | 89.1 MB |
| **10,000 MB** (10 GB) | 44.66 s | 223.94 MB/s | 199.70 s | 50.07 MB/s | 12,001.50 MB | +20.01% | 89.2 MB |
| **20,000 MB** (20 GB) | 83.05 s | 240.81 MB/s | 258.00 s | 77.52 MB/s | 24,002.01 MB | +20.01% | 89.5 MB |

*\*Note: For small files (e.g. 1 MB), the +200% expansion reflects minimum protocol padding: 1 data chunk + 1 metadata chunk + 1 decoy chunk = 3 slots (3.00 MB). Expansion converges to strictly +20.0% for datasets $\ge 100\text{ MB}$. Resident memory (RSS) remains strictly below 90 MB across all sizes.*

---

## 12. Verification of Downloaded Artifacts

The repository includes a [`SHA256SUMS`](SHA256SUMS) manifest. Verify your downloaded package before running:

### Linux
```bash
sha256sum Markush_FECS.AppImage
# Or verify against manifest:
sha256sum -c SHA256SUMS
```

### Windows (PowerShell)
```powershell
Get-FileHash .\Markush_FECS_Windows.zip -Algorithm SHA256
Get-FileHash .\Markush_FECS.exe -Algorithm SHA256
```

---

## 13. Limitations & Security Boundaries

- **User-Space Offline Enforcement Limits:** The offline licensing system enforces evaluation limits via asymmetric signatures and HMAC-SHA256 ledgers. On user-controlled hardware, an administrative user with dynamic debuggers (GDB/x64dbg) or reverse engineering tools can inspect process memory or modify binary instructions. This mechanism provides evaluation accounting and tamper detection against casual modification—it is **not** an unbreakable hardware-enforced DRM.
- **Passphrase Non-Recoverability:** The cryptographic design contains no backdoors, escrow keys, or master recovery bypasses. If a passphrase is lost, data recovery is mathematically impossible. Always test recovery and maintain independent backups of critical data.
- **No Third-Party Commercial Audit:** Markush FECS has undergone comprehensive internal automated test suites (KAT vectors, fuzzing, AddressSanitizer), but has **not** been independently audited by a third-party commercial security firm.
- **Archival Tool, Not an In-Kernel Filesystem:** The software is a file container utility, not a transparent on-the-fly disk driver (such as VeraCrypt or BitLocker).
- **Supported Platforms:** Precompiled binaries are provided for **x86-64 Linux (glibc 2.31+)** and **x86-64 Windows (Windows 10/11)**. Native binaries are not currently provided for ARM, macOS, or 32-bit platforms.

---

## 14. License & Support

- **Developer & Author:** Purn_Vadodariya
- **Contact:** [purn872008@gmail.com](mailto:purn872008@gmail.com)
- **Copyright:** Copyright (C) 2026 Purn_Vadodariya. All rights reserved.
- **Terms:** Proprietary Protected Cryptographic Storage Engine. Redistribution of pre-compiled release packages is permitted for evaluation and authorized testing under the terms in [`LICENSE`](LICENSE).
