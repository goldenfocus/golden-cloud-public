# Secrets

Everything in this folder is **SOPS-encrypted with age**. Commit ciphertext, never plaintext.

## Quickstart

```bash
# Edit (decrypts in your $EDITOR, re-encrypts on save)
sops secrets/my-secret.yaml

# Decrypt to stdout
sops -d secrets/my-secret.yaml

# Decrypt into an env var for a one-off command
eval "$(sops -d secrets/prod.env | sed 's/^/export /')" && ./run.sh
```

## Adding a new laptop / person / AI device

1. On the new device:
   ```bash
   age-keygen -o ~/.config/sops/age/keys.txt
   chmod 600 ~/.config/sops/age/keys.txt
   grep '^# public key:' ~/.config/sops/age/keys.txt
   ```
2. Send the **public key only** to an existing recipient.
3. Existing recipient adds it to `.sops.yaml` under `age:` and runs:
   ```bash
   sops updatekeys secrets/*
   git commit -am "sops: add <device/person>"
   git push
   ```

## Revoking a recipient

1. Remove their pubkey from `.sops.yaml`.
2. `sops updatekeys secrets/*`
3. **Rotate the secrets.** A revoked recipient has history access — they could have saved ciphertexts and may still hold the old age key for them. Treat revocation as "change the password," not just "remove the lock."

## Rules

- Never commit plaintext. The pre-commit hook (gitleaks) blocks obvious leaks but isn't foolproof.
- Never decrypt into the working tree. Decrypt to `/tmp/`, stream to stdin, or use `sops exec-env`.
- Never put secrets in `golden-cloud-public`. Ever.
