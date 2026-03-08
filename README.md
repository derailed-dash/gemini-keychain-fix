# Gemini CLI Extension: WSL Keychain Fix

This extension fixes "Keychain is not available" errors when running Gemini CLI or NotebookLM (`nlm`) in headless Linux environments like WSL (Ubuntu), Docker containers, or remote SSH sessions.

## ⚠️ IMPORTANT SECURITY WARNING

**This fix replaces encrypted OS-level storage (Keychain) with plain text file storage.**

While the files are protected by standard Linux file permissions (readable only by your user), they are **not encrypted**. Use this only in trusted environments where you have full control over the host machine.

## How to use

Install directly via URL (once published):

```bash
gemini extensions install https://github.com/derailed-dash/gemini-skill-wsl-keychain-fix
```

After installation, the agent will have the **`wsl-keychain-fix`** skill active. If you encounter a "Keychain is not available" error, simply ask the agent to fix it, and it will use the skill to configure the correct backends.

## Contents
*   **wsl-keychain-fix skill:** Procedural expertise for configuring Python (`keyrings.alt`) and Node.js (`.env`) backends.
*   **Auto-fix capability:** The agent can now automatically diagnose and repair these issues in your environment.
