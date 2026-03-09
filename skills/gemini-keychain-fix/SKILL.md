---
name: gemini-keychain-fix
description: Fixes "Keychain is not available" errors in WSL, Docker, SSH, and other headless Linux environments. Configures plaintext backends for both Python-based (keyrings.alt) and Node-based (Gemini extensions) tools to bypass missing Secret Service/Gnome Keyring daemons.
---

# Gemini Keychain Fix

This skill addresses the common "Keychain is not available" error encountered when running developer tools in WSL (Windows Subsystem for Linux), Docker containers, or remote SSH sessions.

## ⚠️ SECURITY WARNING

**This fix trades encryption for functionality.** 

By default, modern CLI tools attempt to use a secure system vault (like Gnome Keyring). In headless environments, these are unavailable. This skill configures tools to store credentials (API keys, OAuth tokens) in **plain text files** in your home directory.

While these files are protected by standard Linux file permissions (readable only by your user), they are **not encrypted**. Use this only in trusted environments where you have full control over the host machine.

## The Problem

Many CLI tools—both Python-based and Node-based—attempt to store sensitive data in the system's secure credential storage. In WSL/headless sessions, the graphical D-Bus services are missing, causing tools to crash with "Keychain is not available".

## The Solution

Redirect these tools to use local, plaintext file-based storage. Ensure you set strict file permissions (`chmod 600`) on any files created.

## Fix Steps

### 1. For Python-based CLI tools

If a Python-based tool (like a login utility) fails with a keychain error:

```bash
# Install the alternative backend package
uv pip install keyrings.alt

# Configure the backend to use PlaintextKeyring
mkdir -p ~/.config/python_keyring
cat <<EOF > ~/.config/python_keyring/keyringrc.cfg
[backend]
default-keyring = keyrings.alt.file.PlaintextKeyring
EOF

# Ensure the storage directory is restricted
chmod 700 ~/.local/share/python_keyring/ 2>/dev/null || true
```

### 2. For Node-based Gemini CLI Extensions

If an extension install fails because of a "sensitive" setting (like an API Key), follow these steps to bypass the keychain requirement:

1.  **Clear Metadata:** Run `gemini extensions uninstall <name>` to remove any partial or failed installation entries from the registry. ALWAYS CHECK BEFORE DOING THIS.
2.  **Clone to Safe Location:** Manually clone the extension to a location **outside** of `~/.gemini/extensions/` (e.g., `~/localdev/<name>`). This prevents accidental deletion by the CLI's uninstall command.
3.  **Disable "Sensitive" Flag:** Flip the sensitive setting in `gemini-extension.json` using this one-liner:
    ```bash
    sed -i 's/"sensitive": true/"sensitive": false/g' gemini-extension.json
    ```
4.  **Create Plaintext .env:** Create the extension's `.env` file and set strict permissions:
    ```bash
    echo "SETTING_NAME=your_secret_here" > .env
    chmod 600 .env
    ```
5.  **Link Extension:** Run `gemini extensions link .` from within the extension directory.
6.  **Configure Interactive Settings:** If the CLI still prompts for the secret during linking, pipe the value to the config command:
    ```bash
    printf "your_secret_here\n" | gemini extensions config <name> "Setting Name"
    ```

## Verification

To confirm the fix has been applied correctly:

1.  Run `gemini extensions list`.
2.  Verify the extension is **Enabled**.
3.  Ensure the **Path** points to your manual directory.
4.  Ensure the **Source** type is **link**.

## Reverting the Fix

To return to the default (secure) behavior and clean up plaintext files:

1.  **Python Backend:** Delete the configuration: `rm ~/.config/python_keyring/keyringrc.cfg` and stored secrets: `rm -rf ~/.local/share/python_keyring/`.
2.  **Unlink Extension:** Run `gemini extensions uninstall <name>` to remove the link.
3.  **Delete Manual Extension:** Manually delete the external directory you created (e.g., `rm -rf ~/localdev/<name>`). ALWAYS CHECK BEFORE DOING THIS.
4.  **Cleanup Environment:** Remove the `GEMINI_FORCE_FILE_STORAGE` environment variable from `~/.bashrc`.
