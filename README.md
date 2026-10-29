# Markush FECS — Cryptographic Container Engine & CLI

**Developer:** Purn_Vadodariya (purn872008@gmail.com)  
**Version:** 3.0.0 (Production Engine)  
**License:** Proprietary Protected Cryptographic Storage Engine (Offline Evaluation)  

---

## 1. Product Description and Intended Use

### What is Markush FECS?
Markush FECS (Fragmented Encrypted Container System) is an offline cryptographic file storage engine implemented in C++17, available both as a dynamic shared library (`libfecs.so` / `libfecs.dll` with a C ABI) and as a standalone command-line application (`Markush_FECS.exe` / `Markush_FECS.AppImage` / `fecs`).

The engine packages files into fixed-slot encrypted container archives (`.fecs`). Its primary objective is to mitigate metadata and length leakage inherent in traditional file-level encryption by:
1. **Uniform Chunking (Proposition 1):** Partitioning plaintexts into uniform 512 KiB chunks so that file contents are structured into fixed-sized blocks rather than exposing arbitrary byte lengths.
2. **Decoy Dispersion:** Inserting pseudorandom decoy chunks alongside real payload chunks to obscure true payload slot counts and maintain high statistical byte entropy (~7.9999 bits/byte).
3. **Deterministic Permutation:** Shuffling physical slot positions across the container using a Fisher-Yates permutation seeded by a derived HKDF subkey.
4. **Offline Licensing Subsystem:** Enforcing a local, Ed25519-signed evaluation policy allowing up to 10 successful cryptographic operations (`encrypt` or `decrypt`) per license without internet access or telemetry.

### Intended Use
- **File Archiving:** Encrypting standalone archives, backups, or documents before uploading them to untrusted or public cloud storage providers.
- **Air-Gapped Environments:** Encrypting and decrypting data on offline workstations where network activation or cloud key management services (KMS) are unavailable or prohibited.
- **Embedded Application Storage:** Incorporating `libfecs` via its C API into local C/C++, Python, or C# applications running on supported x86-64 platforms that require fixed-slot container encryption.
- **Evaluation & Security Research:** Evaluating side-channel resistant container architectures and offline cryptographic licensing models.

---

## 2. Installation and Usage Instructions

### Direct-Use Guidelines: Single File vs. Package Archive

| Platform | Direct Single-File Execution | Instructions |
| :--- | :---: | :--- |
| **Linux (x86-64)** | **YES** (`Markush_FECS.AppImage`) | The single `.AppImage` file contains all runtime dependencies (`libcrypto.so.3`, `libssl.so.3`, `libargon2.so.1`). Make it executable (`chmod +x`) and run directly. |
| **Windows (x86-64)** | **NO** (Requires `Markush_FECS_Windows.zip`) | `Markush_FECS.exe` requires its companion DLLs (`libfecs.dll`, `libcrypto-3-x64.dll`, `libssl-3-x64.dll`, `libargon2.dll`, `libc++.dll`). You must extract the zip archive and run `Markush_FECS.exe` inside the extracted folder. |

---

### A. Windows Installation & Usage

#### Step 1: Download & Extract
1. Download `Markush_FECS_Windows.zip`.
2. Extract the archive into a folder of your choice (e.g., `C:\Tools\Markush_FECS\`).
3. Open PowerShell or Command Prompt inside that directory.

> [!NOTE]
> All required runtime DLLs are included in the zip archive. Do not move `Markush_FECS.exe` out of the folder unless the accompanying `.dll` files are moved with it or placed in your system PATH.

#### Step 2: Basic Commands
```powershell
# Check application version and developer attribution
.\Markush_FECS.exe --version

# Check current license quota and remaining slots
.\Markush_FECS.exe license-status

# Display host installation ID (for host-bound licensing)
.\Markush_FECS.exe license-bind

# Encrypt a file (consumes 1 license operation on success)
.\Markush_FECS.exe encrypt -p "YourStrongPassword123!" plain.dat vault.fecs

# Decrypt a container (consumes 1 license operation on success)
.\Markush_FECS.exe decrypt -p "YourStrongPassword123!" vault.fecs restored.dat

# Verify container authentication tags in-place (free operation, 0 license cost)
.\Markush_FECS.exe verify -p "YourStrongPassword123!" vault.fecs

# Inspect public container parameters without a password (free operation)
.\Markush_FECS.exe inspect vault.fecs
```

---

### B. Linux Installation & Usage

#### Step 1: Running the AppImage
```bash
# Grant execution permissions
chmod +x Markush_FECS.AppImage

# Check version
./Markush_FECS.AppImage --version

# Check license status
./Markush_FECS.AppImage license-status
```

#### Step 2: Encrypt, Decrypt, and Verify
```bash
# Encrypt
./Markush_FECS.AppImage encrypt -p "YourStrongPassword123!" plain.dat vault.fecs

# Decrypt
./Markush_FECS.AppImage decrypt -p "YourStrongPassword123!" vault.fecs restored.dat

# Verify integrity tags without restoring plaintext
./Markush_FECS.AppImage verify -p "YourStrongPassword123!" vault.fecs

# Inspect container header
./Markush_FECS.AppImage inspect vault.fecs
```

#### Running Without FUSE
If running on systems where `/dev/fuse` is not mounted (e.g., minimal Docker containers or WSL):
```bash
./Markush_FECS.AppImage --appimage-extract-and-run --version
```
Alternatively, extract `Markush_FECS_Linux.zip` and run the native ELF binary `fecs`:
```bash
LD_LIBRARY_PATH=. ./fecs --version
```

---

### C. C API & Application Integration

On supported x86-64 platforms, developers can integrate the engine directly into applications using the standard C ABI declared in `include/fecs_c_api.h`.

#### 1. C / C++ Integration Example
```c
#include "fecs_c_api.h"
#include <stdio.h>

int main(void) {
    // Encrypt: input, output, passphrase, decoy ratio (e.g. 0.20 = 20% decoys)
    FECS_Result res = fecs_encrypt("input.dat", "vault.fecs", "Password123!", 0.20);
    if (res.code != FECS_SUCCESS) {
        fprintf(stderr, "Encryption error: %s\n", res.message);
        return 1;
    }

    // Decrypt
    FECS_Result dec_res = fecs_decrypt("vault.fecs", "restored.dat", "Password123!");
    if (dec_res.code != FECS_SUCCESS) {
        fprintf(stderr, "Decryption error: %s\n", dec_res.message);
        return 1;
    }

    printf("Operation completed successfully.\n");
    return 0;
}
```

#### 2. Python Integration via `ctypes`
```python
import ctypes

# Load engine shared library (libfecs.so on Linux, libfecs.dll on Windows)
lib = ctypes.CDLL("./libfecs.so")

# Call fecs_encrypt
status = lib.fecs_encrypt(
    b"report.pdf", 
    b"vault.fecs", 
    b"Password123!", 
    ctypes.c_double(0.20)
)
print("Return code:", status)
```

---

## 3. Verified User-Facing Features

### Cryptographic Engine Implementation
- **Container Format v2.0 (`0x00020000`):** 32-byte container header, 28-byte slot headers, 16-byte Poly1305/GCM authentication tags, and uniform 512 KiB chunks.
- **Argon2id KDF (RFC 9106):** Configured with 64 MiB RAM, 3 iterations, and 4 parallel threads for password key derivation.
- **HKDF-SHA256 Hierarchy (RFC 5869):** Expands master secret into independent subkeys for chunk encryption, metadata, permutation seeding, and decoy PRNG generation.
- **AES-256-GCM AEAD (NIST SP 800-38D):** Authenticated encryption for all data chunks with deterministic nonces bound to slot indices.
- **Physical-Order Batch Decryption:** Reads container slots sequentially from disk in batches, caching permutation lookups to eliminate random seek thrashing.
- **Decoy Dispersion:** Inserts pseudorandom decoy slots matching payload slot size, producing near-ideal statistical entropy (~7.9999 bits/byte).
- **In-Place Verification (`verify`):** Validates all container slot tags without decrypting or writing plaintext to disk.
- **Zero-Credential Header Inspection (`inspect`):** Displays public container parameters (total slots, chunk size, version) without requiring a passphrase.

### Offline Licensing Subsystem
- **Asymmetric Signature Verification (RFC 8032):** Validates 196-byte Ed25519-signed licenses using an embedded root public key.
- **10-Operation Quota:** Enforces a limit of exactly 10 successful operations (`encrypt` or `decrypt`) per license; subsequent operations are rejected with code `107`.
- **Atomic Two-Phase Reservation:** Decrements quota only upon cryptographic success; rolls back reserved slots if an I/O or authentication error occurs.
- **Process Concurrency Safety:** Uses file locking (`flock` on Linux, `LockFileEx` on Windows) to prevent race conditions across concurrent processes.
- **Monotonic Tamper-Evident Ledger:** Local state is sealed with HMAC-SHA256; unauthorized modification locks down the license.

---

## 4. Supported Platforms and Dependencies

### Supported Platforms
- **Linux:** 64-bit x86-64 with `glibc` 2.31 or newer (tested on Ubuntu 22.04/24.04 and Debian 11/12).
- **Windows:** 64-bit x86-64 Windows 10, Windows 11, or Windows Server 2016+ with Universal C Runtime.

### Explicit Platform Exclusions
- **ARM / AArch64:** No native ARM binaries are currently provided.
- **macOS:** No Darwin / Mach-O binaries are provided.
- **32-Bit Architectures:** 32-bit x86 is not supported.

### Runtime Dependencies
- **Linux AppImage:** Completely self-contained; bundles OpenSSL 3.x, libargon2, and libc++ runtimes.
- **Windows Zip Package:** Bundles all required DLLs (`libfecs.dll`, `libcrypto-3-x64.dll`, `libssl-3-x64.dll`, `libargon2.dll`, `libc++.dll`, `libunwind.dll`).

---

## 5. Benefits and Limitations

### Practical Benefits
- **Zero Network Exposure:** Runs 100% locally with zero internet calls, telemetry, or remote key servers.
- **Length & Layout Obscurity:** Uniform 512 KiB chunking and decoy slots prevent adversaries from deducing precise file sizes or directory structures from container sizes alone.
- **Bounded Memory Profile:** Memory consumption is strictly bounded (<90 MB RSS), allowing files up to 20 GB to be processed on systems with modest RAM.
- **Cross-Platform Container Compatibility:** Containers created on Windows decrypt on Linux and vice versa, provided the passphrase is known.

### Honest Limitations & Security Boundaries
- **Offline Client Security Limits:** Offline licensing is enforced in user-space software via cryptographic signatures and HMAC state files. On user-controlled hardware, an attacker with administrative/root privileges, dynamic debuggers, or disassemblers can patch binary instructions or dump process memory. This mechanism is intended for evaluation accounting and casual tamper resistance—it is **not** an unbreakable hardware-enforced DRM (such as a hardware security module or TPM enclave).
- **Passphrase Non-Recoverability:** There are no master recovery keys, backdoors, or escrow mechanisms. If a user loses their passphrase, data recovery is mathematically impossible.
- **No Third-Party Commercial Audit:** While the implementation has been validated against published test vectors (KAT for RFC 5869, NIST SP 800-38D, and RFC 9106), fuzz testing, and AddressSanitizer, it has **not** undergone a formal external third-party security audit.
- **Archiving Model:** Markush FECS is a file container utility, not an on-the-fly virtual filesystem driver (such as VeraCrypt or BitLocker).

---

## 6. Basic Benchmark Summaries (Approved for Release)

Empirical benchmarks measured on **Tiger Lake Intel Core i5-1135G7 @ 2.40GHz (4 cores, 8 threads)**, 7.46 GB RAM, ext4 on NVMe SSD, using hardware AES-NI acceleration and Argon2id ($t=3, m=64\text{ MB}, p=4$):

| Input Size | Enc. Time | Enc. Throughput | Dec. Time | Dec. Throughput | Container Size | Expansion | Peak Resident RAM | SHA-256 Result |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1 MB** | 0.08 s | 12.08 MB/s | 0.09 s | 10.57 MB/s | 3.00 MB | +200.00%* | 88.2 MB | **100% MATCH** |
| **100 MB** | 0.37 s | 268.71 MB/s | 0.58 s | 173.84 MB/s | 121.01 MB | +21.01% | 88.5 MB | **100% MATCH** |
| **1,000 MB** (1 GB) | 4.39 s | 227.66 MB/s | 4.80 s | 208.32 MB/s | 1,201.05 MB | +20.10% | 88.6 MB | **100% MATCH** |
| **5,000 MB** (5 GB) | 27.82 s | 179.75 MB/s | 56.52 s | 88.47 MB/s | 6,001.25 MB | +20.02% | 89.1 MB | **100% MATCH** |
| **10,000 MB** (10 GB) | 44.66 s | 223.94 MB/s | 199.70 s | 50.07 MB/s | 12,001.50 MB | +20.01% | 89.2 MB | **100% MATCH** |
| **20,000 MB** (20 GB) | 83.05 s | 240.81 MB/s | 258.00 s | 77.52 MB/s | 24,002.01 MB | +20.01% | 89.5 MB | **100% MATCH** |

*\*Note: For 1 MB files, the +200% expansion represents minimal baseline padding: 1 data chunk + 1 metadata chunk + 1 decoy chunk = 3 slots (3.00 MB). Container overhead converges asymptotically to exactly +20.0% for datasets $\ge 100\text{ MB}$. Peak RAM remains strictly bounded under 90 MB RSS across all tested file sizes up to 20 GB.*

---

## 7. License, Support, and Release Downloads

### Release Downloads & Checksums

| Package / Artifact | Platform | Size | SHA-256 Checksum |
| :--- | :--- | :---: | :--- |
| [Markush_FECS.AppImage](linux/Markush_FECS.AppImage) | Linux x86-64 | 3.5 MB | `03bb9ae1ec2ec173dbce3db2b161d3bed70069e082376f67251f691175b5ec63` |
| [Markush_FECS_Linux.zip](linux/Markush_FECS_Linux.zip) | Linux x86-64 (Full) | 3.2 MB | `dc9f462ca4b68604b205800e38ecae4c581d5ea484998a2d4a6fe35a7c3b34af` |
| [Markush_FECS_Windows.zip](windows/Markush_FECS_Windows.zip) | Windows x86-64 (Full) | 3.3 MB | `e0824b5285477ec0ee2b94f88ac07ba707c020d336b323af0194241abc722bac` |
| [Markush_FECS.exe](windows/Markush_FECS.exe) | Windows x86-64 (Exe only) | 90 KB | `90fea50d43882e3546118b9db2e2594ffbb776496d333704aa3567ce54adb1c5` |

### Licensing & Support Policy
- **Developer:** Purn_Vadodariya
- **Contact & Support:** [purn872008@gmail.com](mailto:purn872008@gmail.com)
- **Copyright:** Copyright (C) 2026 Purn_Vadodariya. All rights reserved.
- **License Model:** Proprietary Protected Engine. Each distributed license (`markush.license`) grants 10 operational credits. For questions, bug reports, custom host bindings, or license renewals, contact the developer directly.
