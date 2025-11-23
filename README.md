# sshcrypt

Encrypt files using SSH Agent. No passwords, just your SSH keys.

## Why sshcrypt?

Traditional file encryption has two major problems:

**🔐 Password fatigue**: Entering passwords repeatedly is not only tedious, but also makes you vulnerable to phishing attacks. You might rush through the prompt without verifying the source.

**🗝️ Key exposure**: Storing decryption keys in plaintext on your filesystem creates a persistent security liability. If your disk is compromised, so are all your encrypted files.

**SSH Agent solves both problems.** It's a secure key manager that already exists on your system. By leveraging SSH Agent, sshcrypt lets you:

- ✅ Use biometric authentication (Touch ID via KeePassXC on macOS)
- ✅ Keep keys in memory, never on disk
- ✅ Integrate with existing security tools
- ✅ Follow the same security model as SSH itself

sshcrypt builds on the ideas from [sshovel](https://github.com/edwardspeyer/sshovel), bringing SSH Agent-based encryption to Python 3 with a clean, minimal implementation.

## Features

- 🔐 **No password prompts** - SSH Agent handles authentication
- 🔑 **Works with KeePassXC + Touch ID** - Biometric unlock on macOS
- 📦 **Self-contained encrypted files** - `.sshcrypt` format includes all metadata
- 🐍 **Pure Python 3** - Only dependency is `openssl` (pre-installed on macOS/Linux)
- 🔓 **Open format** - Simple structure, easy to understand and verify

## Install

Download the script and make it executable:

```bash
# Install to your local bin directory
curl -o ~/bin/sshcrypt https://raw.githubusercontent.com/buhoho/sshcrypt/refs/heads/main/sshcrypt
chmod +x ~/bin/sshcrypt

# Or install system-wide (requires sudo)
sudo curl -o /usr/local/bin/sshcrypt https://raw.githubusercontent.com/buhoho/sshcrypt/refs/heads/main/sshcrypt
sudo chmod +x /usr/local/bin/sshcrypt
```

**Requirements:**
- Python 3.6+
- `openssl` command (pre-installed on macOS/Linux)
- Running SSH Agent with at least one key loaded

## Usage

### Quick Start

```bash
# Make sure your SSH Agent is running and has keys
ssh-add -l

# If empty, add your key
ssh-add ~/.ssh/id_ed25519

# Encrypt a file
sshcrypt encrypt secret.txt secret.txt.sshcrypt

# Decrypt a file
sshcrypt decrypt secret.txt.sshcrypt decrypted.txt

# Edit encrypted file directly (decrypts → opens $EDITOR → re-encrypts)
sshcrypt edit secret.txt.sshcrypt
```

### Use with KeePassXC on macOS

1. Store your SSH keys in KeePassXC
2. Enable SSH Agent integration in KeePassXC settings
3. Unlock your KeePassXC database with Touch ID
4. Use sshcrypt - it will use the keys from KeePassXC automatically! 🎉

No password prompts. No keys on disk. Just Touch ID.

## How it works

1. **Encryption**: 
   - sshcrypt asks SSH Agent to sign a fixed string using your private key
   - The signature is hashed to derive an AES-256 key
   - Your file is encrypted with this key using `openssl`
   - The encrypted file includes metadata about which SSH key was used

2. **Decryption**:
   - sshcrypt reads the key metadata from the encrypted file
   - It asks SSH Agent to sign the same fixed string with that specific key
   - The signature produces the same AES-256 key
   - The file is decrypted with `openssl`

**Security properties**:
- Your SSH private key never leaves the Agent
- The signature proves you control the private key
- Without access to the SSH Agent, the file cannot be decrypted
- The encryption is as strong as your SSH key + AES-256

## File Format

```
[4 bytes: key_blob length]
[N bytes: SSH public key blob]
[remaining: AES-256-CBC encrypted data]
```

Simple, self-contained, and portable.

## Examples

```bash
# Encrypt sensitive configuration
sshcrypt encrypt ~/.aws/credentials ~/.aws/credentials.sshcrypt
rm ~/.aws/credentials

# Edit encrypted notes
export EDITOR=vim
sshcrypt edit notes.sshcrypt

# Decrypt for one-time use
sshcrypt decrypt backup.tar.gz.sshcrypt backup.tar.gz
tar xzf backup.tar.gz
rm backup.tar.gz  # Don't leave plaintext around
```

## zsh Completion

<details>
<summary>Add tab completion for .sshcrypt files</summary>

```bash
# Create completion directory
mkdir -p ~/.zsh/completions

# Download completion file
curl -o ~/.zsh/completions/_sshcrypt https://raw.githubusercontent.com/buhoho/sshcrypt/refs/heads/main/completions/_sshcrypt

# Add to .zshrc
echo 'fpath=(~/.zsh/completions $fpath)' >> ~/.zshrc
echo 'autoload -Uz compinit && compinit' >> ~/.zshrc

# Reload
source ~/.zshrc
```

</details>

## FAQ

**Q: Which SSH key does sshcrypt use?**  
A: It uses the first key returned by `ssh-add -l`. To use a specific key, make sure it's first in the agent.

**Q: Can I use the same encrypted file on different machines?**  
A: Yes, as long as both machines have access to the same SSH private key (via SSH Agent).

**Q: What if I lose my SSH key?**  
A: You cannot decrypt the files. Back up your SSH keys securely!

**Q: Is this compatible with GPG?**  
A: No, this uses SSH keys, not GPG keys. They're different systems.

**Q: Why not use `age` or `gpg`?**  
A: Both are great tools, but neither integrates with SSH Agent. If you already manage SSH keys in KeePassXC with Touch ID, sshcrypt leverages that existing infrastructure.

## Credits

Inspired by [sshovel](https://github.com/edwardspeyer/sshovel) by Edward Speyer.

## License

MIT License - see [LICENSE](LICENSE) file for details.