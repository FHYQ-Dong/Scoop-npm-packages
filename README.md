# Scoop npm Packages

[![Tests](https://github.com/ScoopInstaller/Main/actions/workflows/ci.yml/badge.svg)](https://github.com/ScoopInstaller/Main/actions/workflows/ci.yml)

Scoop bucket for distributing npm packages as Scoop manifests, with a shared Node.js runtime managed by Scoop.

## Design

Each npm package is installed as a Scoop app. Instead of bundling Node.js, manifests depend on Scoop's `nodejs` (from the [versions](https://github.com/ScoopInstaller/Versions) bucket) and hardcode the path to a specific Node.js version in a generated `.cmd` shim.

### Directory layout

```
scoop/apps/
├── nodejs20/                    # Scoop-managed Node.js (via versions bucket)
│   ├── 20.20.2/
│   └── current → 20.20.2
├── devcontainer/
│   └── 0.87.0/
│       ├── package/             # extract_dir: "package" (from npm tgz)
│       └── devcontainer.cmd      # generated shim → hardcoded path to node.exe
└── ...
```

### Shim strategy

Shims hardcode the version-specific path to `node.exe` (not the `current` symlink):

```cmd
@"%~dp0..\\..\\nodejs20\\20.20.2\\node.exe" "%~dp0package\\devcontainer.js" %*
```

This means each npm package can point at a different Node major version. Different packages can depend on `nodejs18`, `nodejs20`, `nodejs22`, etc.

### ⚠️ Important: Node.js version cleanup

`scoop cleanup nodejs20` removes old versions. If a shim hardcodes `20.20.2` and you run cleanup after an update to `20.20.3`, the shim breaks.

**Do not run `scoop cleanup` on Node.js manifests used by this bucket.** Reinstall the npm package if you need to update the shim's Node path.

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
4. `pre_install` generates a `.cmd` shim that hardcodes the path to `node.exe`
5. `checkver` + `autoupdate` should track the npm registry

See `bucket/devcontainer.json` for a reference implementation.
