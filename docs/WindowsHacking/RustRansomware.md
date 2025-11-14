# Proof-of-Concept Ransomware in Rust

I made a simple ransomware demo in Rust, available here: [Ransomware-Rust](https://github.com/G4sp4rCS/Ransomware-Rust) (accompanied by [this video](https://www.youtube.com/watch?v=NeAjZi9Mt64)). It is intended to run in disposable virtual machines, includes a decryptor, and its purpose is pedagogical: to show how the pieces fit together.

---

## Why this project matters

- Exposes each stage of a file-encryption workflow clearly and readably in Rust.
- Allows defensive teams to study the logic without needing to analyze obfuscated samples.
- Serves as a Rust refresher: modular design, error handling with `Result`, and safe file operations.

---

## Key components

| Module       | Function                                                                                 |
|--------------|-----------------------------------------------------------------------------------------|
| `main.rs`    | CLI parsing (`--encrypt` / `--decrypt`), logging and user prompts                       |
| `crypto.rs`  | AES-256-CBC logic with PKCS#7 padding, IV generation and ephemeral key handling         |
| `fileops.rs` | Recursive directory traversal, extension filtering, skipping `.encrypted` files and ransom note handling |

Separating responsibilities makes it easier to see where real attackers commonly add persistence, command-and-control channels, or packing techniques.

---

## How to run the lab (in a controlled environment)

1. Clone the repository inside a VM that can be discarded.
2. Build with `cargo build --release`.
3. Create a VM snapshot before testing.
4. Run `./target/release/ransomware --encrypt <test_directory>` and note the generated key.
    - Or in windows cmd: `.\target\release\ransomware.exe --encrypt <test_directory>`
5. Restore files with `--decrypt <test_directory> --key <noted_key>`.

Open source makes it possible to:
- Add logging to observe traversal and block-by-block encryption.
- Integrate events into an EDR or Sysmon stack to see how detections respond.
- Adjust filters/extensions to compare with real samples.

---

## Some extra featuring

While my current builds lean on the CLI flow above, the repository already carries extra modules I have not wired into the regular runs yet:
- `src/obfuscation.rs` bundles future hardening tricks: XOR string disguises for the command keywords and the `.encrypted` suffix, per-call random jitter to desync heuristic engines, very simple anti-sandbox checks (execution timing plus CPU count), and reversible dummy transformations meant to split work across functions. The helpers (`deobfuscate_string`, `random_jitter`, `anti_sandbox_checks`, `prepare_operation`, `xor_data`) are ready to drop in whenever I want to simulate stealthier tradecraft.
- `src/gui.rs` exposes a native dialog layer (via `native_dialog`) with warning pop-ups, key prompts, a directory picker, a faux “File Manager” menu, and even a debug-only legal waiver screen. There are also cross-platform text-input helpers so the binary could switch from plain console prompts to a social-engineering friendly GUI when desired.
