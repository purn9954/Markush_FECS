# Markush FECS

**Reusable C++ cryptographic container engine and command-line application**

Markush FECS is a C++ file-encryption project designed to create and work with encrypted file containers. It provides a command-line interface (CLI) and is intended for integration into compatible applications through the project's documented interfaces.

- **Version:** 3.0.0
- **Platforms:** Linux and Windows packages are provided in the release.
- **Distribution:** Prebuilt binaries are published through GitHub Releases.
- **License:** See [`LICENSE`](LICENSE) for the applicable terms.

## Downloads

Download the package for your operating system from the **[Markush FECS Releases page](https://github.com/purn9954/Markush_FECS/releases)**.

| Platform | Download | Notes |
|---|---|---|
| Linux | `Markush_FECS.AppImage` | AppImage executable |
| Linux | `Markush_FECS_Linux.zip` | Linux ZIP package |
| Windows | `Markush_FECS.exe` | Standalone executable, if suitable for your setup |
| Windows | `Markush_FECS_Windows.zip` | Windows package; extract the ZIP before running |

Choose the latest release that is appropriate for your environment. Package contents and runtime requirements may vary by build.

## Linux

1. Download `Markush_FECS.AppImage` from the [Releases page](https://github.com/purn9954/Markush_FECS/releases).
2. Open a terminal in the directory containing the downloaded file.
3. Make it executable:

   ```bash
   chmod +x Markush_FECS.AppImage
   ```

4. Run it:

   ```bash
   ./Markush_FECS.AppImage
   ```

If you use `Markush_FECS_Linux.zip`, extract it first and follow any included instructions.

## Windows

1. Download `Markush_FECS_Windows.zip` from the [Releases page](https://github.com/purn9954/Markush_FECS/releases).
2. Extract the ZIP to a folder.
3. Open PowerShell or Command Prompt in that folder.
4. Run the executable:

   ```powershell
   .\Markush_FECS.exe
   ```

If the executable reports a missing dependency or runtime component, consult the package contents and build-specific instructions. Do not remove files included in the ZIP unless you know they are not required.

## Command-line usage

Start the application to view its available prompts or help output. Available commands and options depend on the published build. Use only the syntax supported by your version.

Avoid putting real passwords directly into command-line arguments: command history or process inspection may expose them. Use an interactive password prompt or another secure input method if your build supports one.

## Verify downloaded files

The repository includes [`SHA256SUMS`](SHA256SUMS), which lists SHA-256 checksums for the release packages. After downloading a package, compare its computed hash with the matching entry in that file.

### Linux

From the directory containing the downloaded file:

```bash
sha256sum Markush_FECS.AppImage
```

To verify files using the checksum list, download the matching package files and `SHA256SUMS`, then run the command from a directory where the paths in the checksum file resolve correctly:

```bash
sha256sum -c SHA256SUMS
```

### Windows PowerShell

For the executable:

```powershell
Get-FileHash .\Markush_FECS.exe -Algorithm SHA256
```

Compare the displayed hash with the corresponding entry in `SHA256SUMS`. A checksum comparison can detect differences from the published value; it does not by itself prove that a file is safe or independently certify who built it.

## Reusable engine and integration

Markush FECS is intended to provide a reusable native C++ engine in addition to its CLI. Integration depends on the interfaces and artifacts included in the specific distribution.

Before integrating it into another application, confirm that the release includes the required library/header files and documentation. Language bindings and platform-specific integration should be tested with the target application and runtime; availability is not guaranteed merely because a language can call a C ABI.

The downloadable CLI packages are for running the application directly. They should not be assumed to include development libraries or headers unless those files are explicitly present in the package.

## Security and operational notes

- Back up important data before encryption or decryption, and verify that you can restore files before relying on the tool for critical data.
- Keep passwords and any recovery information private.
- Protect encrypted containers and their backups according to your security requirements.
- Test the release on your target operating system before using it for important files.
- No claim of independent security certification or universal platform compatibility is made by this README.
- Review [`LICENSE`](LICENSE) for usage and redistribution terms.

## Project links

- **Repository:** https://github.com/purn9954/Markush_FECS
- **Downloads:** https://github.com/purn9954/Markush_FECS/releases
- **Checksums:** [`SHA256SUMS`](SHA256SUMS)
- **License:** [`LICENSE`](LICENSE)
