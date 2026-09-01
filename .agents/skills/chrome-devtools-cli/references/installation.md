# Installation

Install the package globally to make the `chrome-devtools` command available. You only need to do this the first time you use it.

```powershell
npm i chrome-devtools-mcp@latest -g
[Environment]::SetEnvironmentVariable('NODE_OPTIONS', '--no-experimental-webstorage', 'User')
chrome-devtools start --executablePath "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --no-usage-statistics
chrome-devtools list_pages
```

Restart terminals and agent hosts after setting `NODE_OPTIONS`. This permanently suppresses Node's irrelevant `--localstorage-file` warning for this user account.

## Troubleshooting

- **Command not found:** If `chrome-devtools` is not recognized, ensure your global npm `bin` directory is in your system's `PATH`. Restart your terminal or source your shell configuration file (e.g., `.bashrc`, `.zshrc`).
- **Permission errors:** If you encounter `EACCES` or permission errors during installation, avoid using `sudo`. Instead, use a node version manager like `nvm`, or configure npm to use a different global directory.
- **Old version running:** Run `chrome-devtools stop && npm uninstall -g chrome-devtools-mcp` before reinstalling, or ensure the latest version is being picked up by your path.
