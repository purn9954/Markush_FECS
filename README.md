# Markush FECS

**Reusable C++ cryptographic container engine and command-line application**

Markush FECS is a native C++ project for encrypting files into a container format and restoring them through the corresponding decryption workflow. It includes a command-line interface (CLI); the project also documents a C API intended to support integration with compatible applications.

This README describes the user-facing purpose, release packages, basic usage, project-reported tests, and benchmark conditions. It deliberately does not publish private implementation details or claim that the software has been independently certified.

- **Version documented here:** 3.0.0
- **Platforms packaged:** Linux and Windows
- **Downloads:** [GitHub Releases](https://github.com/purn9954/Markush_FECS/releases)
- **License and usage terms:** See [`LICENSE`](LICENSE)

## Contents

- [Why use a file-container encryption tool?](#why-use-a-file-container-encryption-tool)
- [What Markush FECS provides](#what-markush-fecs-provides)
- [Downloads](#downloads)
- [Installation and first run](#installation-and-first-run)
- [Reusable engine and integration](#reusable-engine-and-integration)
- [How the workflow is used](#how-the-workflow-is-used)
- [Test results](#test-results)
- [Benchmarks](#benchmarks)
- [Verify downloaded files](#verify-downloaded-files)
- [Limitations and security notes](#limitations-and-security-notes)
- [Project files](#project-files)

## Why use a file-container encryption tool?

Files can contain personal, business, research, or other confidential information. Encrypting them before storing or transferring them can reduce the risk of disclosure if an unauthorized person obtains the encrypted copy. A container can also provide a single artifact to manage instead of handling the original file as plain data.

Encryption does not replace safe password handling, backups, endpoint security, or access controls. Protection depends on the implementation, how it is used, and the environment in which it runs.

## What Markush FECS provides

The project documentation and release packages describe the following capabilities:

- **File encryption and decryption:** create an encrypted container from input data and restore the original data through the corresponding decryption process.
- **Native C++ implementation:** the core engine is implemented in C++.
- **Command-line application:** run supported operations from a terminal.
- **C API:** a documented C-facing interface is intended for use by compatible applications. Check the source and package contents for the exact interface available in your build.
- **Container integrity checks:** the test report describes authentication and rejection tests for modified or malformed containers.
- **Streaming and larger-file testing:** the project test report includes memory and performance measurements across multiple input sizes.
- **Linux and Windows packages:** downloadable builds are provided through GitHub Releases.
- **Checksums:** `SHA256SUMS` is provided to help compare downloaded files with the published hashes.

These are descriptions of the project and its reported tests, not a guarantee that every possible input, operating system, or deployment has been tested.

## Downloads

Download packages from the **[Markush FECS Releases page](https://github.com/purn9954/Markush_FECS/releases)**. Release assets are hosted there rather than linked through local filesystem paths in this README.

| Platform | Release asset | Purpose |
|---|---|---|
| Linux | `Markush_FECS.AppImage` | Linux AppImage executable |
| Linux | `Markush_FECS_Linux.zip` | Linux ZIP package |
| Windows | `Markush_FECS.exe` | Windows executable |
| Windows | `Markush_FECS_Windows.zip` | Windows ZIP package |
| Both | `SHA256SUMS` | Published SHA-256 hashes for package verification |

Use the assets attached to the release you intend to install. Package contents, runtime requirements, and compatibility may vary by build.

## Installation and first run

### Linux AppImage

1. Download `Markush_FECS.AppImage` from the [Releases page](https://github.com/purn9954/Markush_FECS/releases).
2. Open a terminal in the download directory.
3. Allow execution:

   ```bash
   chmod +x Markush_FECS.AppImage
   ```

4. Launch the application:

   ```bash
   ./Markush_FECS.AppImage
   ```

If you use `Markush_FECS_Linux.zip`, extract it and follow any instructions included in the package.

### Windows

1. Download `Markush_FECS_Windows.zip` from the [Releases page](https://github.com/purn9954/Markush_FECS/releases).
2. Extract the ZIP to a directory.
3. Open PowerShell in that directory.
4. Run the executable:

   ```powershell
   .\Markush_FECS.exe
   ```

You can also download `Markush_FECS.exe` directly if that package is appropriate for your setup.

### Finding supported commands

Start the application and follow its prompts or use its help option if the build provides one. Exact commands and options depend on the version and package. This README intentionally does not invent command flags that may not exist in a given build.

Avoid entering real passwords directly as command-line arguments if the application supports a safer input method. Shell history or process inspection may expose command-line values.

## Reusable engine and integration

Markush FECS is intended to be useful both as a CLI application and as a reusable native engine.

- **CLI use:** a person or script launches the packaged application and supplies supported input and output options.
- **Native integration:** a compatible application can use the documented C API when the required headers, library artifacts, and build instructions are available.
- **Other languages:** integration depends on that language's ability to call the available interface and on a tested binding or wrapper. Do not assume that every language or runtime is supported automatically.

The downloadable executable packages should not be assumed to contain development headers, import libraries, or integration examples unless those files are explicitly included. Check the package contents and project files before planning an integration.

## How the workflow is used

At a high level, the expected workflow is:

1. **Choose an input file** and a destination for the encrypted container.
2. **Run the encryption operation** using the options supported by your build.
3. **Store or transfer the encrypted container** with appropriate access controls.
4. **Keep the required password or recovery information safe.** Losing required credentials may make recovery impossible.
5. **Run the matching decryption operation** when you need to restore the file.
6. **Verify the restored data and keep a separate backup** before relying on the tool for important information.

The exact user prompts, arguments, output naming, and recovery behavior should be confirmed against the build you download. Always test with non-critical sample files first.

## Test results

The project's `03_TEST_RESULTS.md` report records the following results for its documented test runs. These are **project-reported results**; this README does not represent an independent third-party audit or reproduce every test harness and log.

| Test area | Reported test scope | Reported result |
|---|---|---|
| Cryptographic known-answer tests | Argon2id, HKDF-SHA256, and AES-256-GCM test vectors referenced by the report | Report states the tested vectors matched |
| Round-trip and boundary tests | Input sizes from empty files and one byte through 100 MB | Report states restored-file SHA-256 hashes matched the originals |
| Malformed-container handling | 1,000 malformed container variations, including truncated headers and invalid size values | Report states malformed inputs were rejected with error results |
| Tamper tests | 19 listed mutations affecting container headers, slots, ciphertext, tags, slot order, truncation, and appended bytes | Report states 19 of 19 test vectors were detected and rejected |
| Sanitizer runs | AddressSanitizer and UndefinedBehaviorSanitizer in the reported test setup | Report states no issues were detected in those runs |
| Memory profiling | Reported input sizes from 1 MB to 20 GB | Report records peak resident memory between 88.2 MB and 89.5 MB on its test host |
| Statistical checks | Entropy, chi-square, serial correlation, and compression comparisons for the tested samples | Report records results within its selected test thresholds |

### How to interpret these results

- Passing a finite test suite is evidence about the tested cases, not proof that software is bug-free or secure against every attack.
- Statistical tests on sample bytes do not prove that data is “indistinguishable” in every setting.
- Sanitizer results apply to the instrumented builds and exercised paths; they do not guarantee the absence of all memory-safety defects.
- The test report describes a Debian/Linux test environment. The results shown here should not be treated as Windows test results.
- The test report and its supporting test scripts/logs should be reviewed when independently reproducing any result.

## Benchmarks

The project benchmark report records the following measurements on its Linux test host. Values are transcribed from the project's `03_TEST_RESULTS.md` report and `benchmark_results.csv`; they are not independently reproduced here.

### Reported file-processing measurements

| Input size | Encryption time | Encryption speed | Decryption time | Decryption speed | Reported container size | Peak RSS |
|---:|---:|---:|---:|---:|---:|---:|
| 1 MB | 0.08 s | 12.08 MB/s | 0.09 s | 10.57 MB/s | 3.00 MB | 88.2 MB |
| 100 MB | 0.37 s | 268.71 MB/s | 0.58 s | 173.84 MB/s | 121.01 MB | 88.5 MB |
| 1,000 MB | 4.39 s | 227.66 MB/s | 4.80 s | 208.32 MB/s | 1,201.05 MB | 88.6 MB |
| 5,000 MB | 27.82 s | 179.75 MB/s | 56.52 s | 88.47 MB/s | 6,001.25 MB | 89.1 MB |
| 10,000 MB | 44.66 s | 223.94 MB/s | 199.70 s | 50.07 MB/s | 12,001.50 MB | 89.2 MB |
| 20,000 MB | 83.05 s | 240.81 MB/s | 258.00 s | 77.52 MB/s | 24,002.01 MB | 89.5 MB |

### Reported test environment

- **CPU:** Intel Core i5-1135G7 @ 2.40 GHz
- **Memory:** 7.46 GB reported available physical memory
- **Operating system:** Debian GNU/Linux, x86-64; report lists kernel 6.8.0
- **Storage:** NVMe SSD, ext4 filesystem
- **Cryptography libraries:** OpenSSL 3.0.13 and libargon2
- **Compiler:** GCC 13.2.0 with the flags recorded in the report
- **Harnesses:** Python, C ABI tests, pytest, AddressSanitizer, and UndefinedBehaviorSanitizer

### Benchmark caveats

- Results depend on the machine, filesystem, build flags, library versions, input data, and system load.
- The report's 1 MB case has a much larger container relative to the input than the larger cases. Do not infer a constant percentage of overhead for very small files.
- Encryption and decryption speeds differ by input size in the report; neither speed should be assumed for another computer.
- The reported 20 GB result is a project benchmark, not a promise of a particular speed or memory ceiling for every release or operating system.
- No Windows performance figures are presented in this table.

## Verify downloaded files

The repository's [`SHA256SUMS`](SHA256SUMS) file lists SHA-256 hashes for the packages. Download the checksum file and the package you want to verify, then compare the computed hash with the matching entry.

### Linux

For example:

```bash
sha256sum Markush_FECS.AppImage
```

To use the checksum list directly, the paths recorded in `SHA256SUMS` must resolve from the directory where you run the command. If the checksum file lists paths such as `linux/Markush_FECS.AppImage`, download or arrange the files so those paths exist, then run:

```bash
sha256sum -c SHA256SUMS
```

### Windows PowerShell

For the executable:

```powershell
Get-FileHash .\Markush_FECS.exe -Algorithm SHA256
```

Compare the displayed value with the entry for that file in `SHA256SUMS`. A matching hash shows that the file matches the published checksum; it does not by itself prove that the program is safe or independently establish who built it.

## Limitations and security notes

- Back up important data before encryption or decryption, and test recovery before using the tool for critical files.
- Keep passwords and recovery information private. A lost password may prevent recovery.
- Protect the original files, decrypted outputs, and backups as well as the encrypted container.
- Test the release on your intended operating system before relying on it.
- Do not assume that an AppImage runs on every Linux distribution or that a Windows executable works on every Windows version.
- Do not assume that the CLI packages include the development artifacts needed to embed the engine in another program.
- No independent security certification, universal compatibility, or absolute protection against attack is claimed here.
- Review [`LICENSE`](LICENSE) for permitted use and redistribution.
- For security-sensitive use, inspect the source available to you, reproduce the tests where practical, and obtain an independent review appropriate to your risk.

## Project files

The public repository and release page are the authoritative places to check current project files and downloads:

- **Repository:** https://github.com/purn9954/Markush_FECS
- **Releases and binaries:** https://github.com/purn9954/Markush_FECS/releases
- **License:** [`LICENSE`](LICENSE)
- **Checksum list:** [`SHA256SUMS`](SHA256SUMS)

