---
name: keychain-fix
description: Fixes "Keychain is not available" errors in WSL, Docker, SSH, and other headless Linux environments. Configures a plaintext keyring backend (keyrings.alt) for nlm/Python and manual file-based storage for gemini extensions to bypass missing Secret Service/Gnome Keyring daemons.
---

# Keychain Fix

This skill addresses the common "Keychain is not available" error encountered when running the Gemini CLI or NotebookLM (nlm) tools in WSL (Windows Subsystem for Linux), Docker containers, or remote SSH sessions.

## ⚠️ SECURITY WARNING

**This fix trades encryption for functionality.** 

By default, these tools attempt to use a secure system vault (like Gnome Keyring). In headless environments, these are unavailable. This skill configures tools to store credentials (API keys, OAuth tokens) in **plain text files** in your home directory.

While these files are protected by standard Linux file permissions (readable only by your user), they are **not encrypted**. Use this only in trusted environments where you have full control over the host machine.

## The Problem

Many Python-based and Node-based tools (like `keyring` in Python or `keytar` in Node.js) attempt to store sensitive data in the system's secure credential storage. In WSL/headless sessions, the graphical D-Bus services are missing, causing tools to crash with "Keychain is not available".

## The Solution

Redirect tools to use local, plaintext file-based storage. Ensure you set strict file permissions (`chmod 600`) on any files created.

## Fix Steps

### 1. For Python-based tools (like `nlm`)

Ensure the `keyrings.alt` package is installed and configure the keyring backend:

```bash
# Install the alternative backend package
uv pip install keyrings.alt

# Configure the backend to use PlaintextKeyring
mkdir -p ~/.config/python_keyring
cat <<EOF > ~/.config/python_keyring/keyringrc.cfg
[backend]
default-keyring = keyrings.alt.file.PlaintextKeyring
EOF

# Ensure the storage file (once created) is restricted
# Default location is often ~/.local/share/python_keyring/keyring_pass.cfg
chmod 700 ~/.local/share/python_keyring/ 2>/dev/null || true
```

### 2. For Node-based tools (like `gemini` extensions)

If an extension install fails because of a "sensitive" setting:

1. Manually clone the extension to `~/.gemini/extensions/<name>`.
2. Edit `gemini-extension.json` and change `"sensitive": true` to `"sensitive": false` for the problematic setting.
3. Manually create the extension's `.env` file at `~/.gemini/extensions/<name>/.env`.
4. **CRITICAL:** Run `chmod 600 ~/.gemini/extensions/<name>/.env` immediately.
5. Run `gemini extensions link ~/.gemini/extensions/<name>`.

### 3. Global WSL/Headless Environment Variable

Add this to `~/.bashrc` to force file storage globally where supported:

```bash
export GEMINI_FORCE_FILE_STORAGE=true
```

## Reverting the Fix

To return to the default (secure) behavior and clean up plaintext files:

1. Delete the configuration: `rm ~/.config/python_keyring/keyringrc.cfg`
2. Delete stored Python secrets: `rm -rf ~/.local/share/python_keyring/`
3. Remove the environment variable from `~/.bashrc`.
4. For extensions, revert the `sensitive` flag and delete the `.env` files.
