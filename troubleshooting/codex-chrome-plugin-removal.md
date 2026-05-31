## Source

Originally reported publicly in openai/codex issue #21670:

https://github.com/openai/codex/issues/21670

## Summary

On Windows Codex Desktop, the bundled Chrome plugin failed to uninstall and browser-use setup became unstable. The uninstall failed with Windows `os error 5`, indicating access denied / file-lock behavior.

## Symptoms

- Codex Chrome plugin uninstall failed.
- UI showed plugin uninstall failure.
- Browser setup calls hung until timeout.
- Logs showed IPC instability.
- Plugin cache update/install failed because of Windows file locks.

## Key error

```text
failed to uninstall plugin: failed to remove existing plugin cache entry: 拒绝访问。 (os error 5)
