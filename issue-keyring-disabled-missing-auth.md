# CLI should reconfigure credentials when keyring is disabled

**Describe the bug**

When a provider is already configured and its credential exists only in the system keyring, starting the CLI with `GOOSE_DISABLE_KEYRING=1` exits with `Configuration value not found`. The CLI does not offer the provider setup flow that it offers when no configuration exists.

The environment variable should continue to prevent Goose from accessing the keyring. In this case, Goose should treat the credential as unavailable and prompt the user to configure the provider, storing the new credential in `secrets.yaml`.

---

**To Reproduce**

1. Remove Goose's configuration and data directories.
2. Run `goose` and configure a provider using the default keyring-backed secret storage.
3. Exit Goose.
4. Run `GOOSE_DISABLE_KEYRING=1 goose`.
5. Observe that Goose exits with `Configuration value not found: <API_KEY>`.

---

**Expected behavior**

Goose should treat the configured provider's keyring-only credential as unavailable and open provider configuration, with the existing provider selected. After the user provides a credential, it should be stored in the file-based secret store and the session should start normally.

Goose must not read or migrate secrets from the keyring while `GOOSE_DISABLE_KEYRING` is set.

---

**Screenshots**

Not applicable. Here's the output in bash:

```text
$ GOOSE_DISABLE_KEYRING=1 goose

  error: Error Configuration value not found: OPENROUTER_API_KEY.
Please check your system keychain and run 'goose configure' again.
If your system is unable to use the keyring, please try setting secret key(s) via environment variables.
For more info, see: https://goose-docs.ai/docs/troubleshooting/#keychainkeyring-errors
```

---

**Please provide the following information**

- **OS & Arch:** Nobara Linux, x86_64
- **Interface:** CLI
- **Version:** `1.53.0` and current development build
- **Extensions enabled:** None
- **Provider & Model:** OpenRouter (model not relevant)

---

**Additional context**

The configuration file still contains the selected provider and model, so the normal first-run check does not run. The failure occurs later while the session builder creates the provider.

The existing provider inventory configuration check can determine whether required credentials are available in the active secret store. The default CLI startup path should use that check before building a session and enter provider setup when the configured provider is unavailable.

**Verification plan**

1. Configure a provider with an API key in keyring-backed storage.
2. Start Goose with `GOOSE_DISABLE_KEYRING=1`.
3. Verify that Goose prompts for provider credentials instead of exiting.
4. Provide a test credential and verify it is written to `secrets.yaml` with restricted permissions.
5. Start Goose again with `GOOSE_DISABLE_KEYRING=1` and verify the provider is initialized from file-based storage.

Do not begin implementation until the issue reaches **Ready** on the [Goose Issues board](https://github.com/orgs/aaif-goose/projects/1).
