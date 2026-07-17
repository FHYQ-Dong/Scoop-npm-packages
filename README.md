# Scoop npm Packages

[![Tests](https://github.com/ScoopInstaller/Main/actions/workflows/ci.yml/badge.svg)](https://github.com/ScoopInstaller/Main/actions/workflows/ci.yml)

Scoop bucket for distributing npm packages as Scoop manifests, with a shared Node.js runtime managed by Scoop.

## Design

Each npm package is installed as a Scoop app. Instead of bundling Node.js, manifests depend on a Scoop-managed Node.js runtime (from the [versions](https://github.com/ScoopInstaller/Versions) bucket) and generate a `.cmd` shim that points to `node.exe` through Scoop's `current` symlink.

### Directory layout

```
scoop/apps/
├── nodejs20/                    # Scoop-managed Node.js (via versions bucket)
│   ├── 20.20.2/
│   ├── 20.20.3/                 # after update
│   └── current → 20.20.3
├── devcontainer/
│   └── 0.87.0/
│       ├── package/             # extract_dir: "package" (from npm tgz)
│       └── devcontainer.cmd      # generated shim → node.exe via current
└── ...
```

### Shim strategy

Shims use the `current` symlink, not a hardcoded patch version:

```cmd
@"%~dp0..\\..\\nodejs20\\current\\node.exe" "%~dp0package\\devcontainer.js" %*
```

This has two advantages:

1. **Survives `scoop update`**: When Node.js updates (e.g. `20.20.2` → `20.20.3`), Scoop repoints `current` and the shim follows automatically — no reinstall needed.
2. **Survives `scoop cleanup`**: Cleanup only removes old version directories; `current` and the latest version are untouched.

Each npm package can point at a different Node major version. Different packages can depend on `nodejs18`, `nodejs20`, `nodejs22`, etc. The only failure case is uninstalling the Node.js dependency entirely.

## Setup

```powershell
# Add the versions bucket (provides nodejs18, nodejs20, nodejs22, etc.)
scoop bucket add versions

# Add this bucket
scoop bucket add npm-packages https://github.com/<your-username>/Scoop-npm-packages

# Install
scoop install npm-packages/devcontainer
```

## Available packages

- `devcontainer` — [@devcontainers/cli](https://github.com/devcontainers/cli) reference implementation

## Contributing

Add a new npm package:

1. Create `<package>.json` in `bucket/`
2. Use `depends` to specify the Node.js version from the `versions` bucket
3. `extract_dir: "package"` — npm tgz files have their content under `package/`
4. `pre_install` generates a `.cmd` shim that points to `node.exe` via the `current` symlink
5. `checkver` + `autoupdate` should track the npm registry

See `bucket/devcontainer.json` for a reference implementation.
