# CI/CD Workflows for cloudflared Fork

This fork includes automated GitHub Actions workflows to keep the code synchronized with upstream and provide pre-compiled binaries for MIPS architectures.

## 1. Upstream Sync (`sync.yml`)

**Goal**: Automatically keep this fork up-to-date with `cloudflare/cloudflared`.

- **Schedule**: Runs daily at 00:00 UTC.
- **Trigger**: Can also be triggered manually via the "Actions" tab.
- **Behavior**:
  - Fetches changes from `cloudflare/cloudflared`.
  - Merges `upstream/master` into the local `master` branch.
  - **Conflict Handling**: If a merge conflict occurs, the workflow fails and automatically opens a GitHub Issue to notify maintainers.

## 2. MIPS Cross-Compilation (`build-mips.yml`)

**Goal**: Build static binaries for MIPS routers and devices.

- **Trigger**: 
  - Automatically runs after a successful Upstream Sync.
  - Runs on any tag push matching `v*` (Release build).
  - Can be triggered manually via "Actions" tab (allows building any branch).

- **Supported Architectures (Matrix)**:
  - `mips` (32-bit, softfloat)
  - `mipsle` (32-bit, softfloat)
  - `mips64` (64-bit)
  - `mips64le` (64-bit)

- **Build Details**:
  - **Go Version**: 1.24
  - **CGO**: Disabled (`CGO_ENABLED=0`) for maximum portability on embedded devices.
  - **FPU Support**: 32-bit builds use `GOMIPS=softfloat` to ensure compatibility with devices lacking hardware floating-point units.

## 3. Artifacts & Releases

- **Workflow Artifacts**: Every build produces binaries and SHA256 checksums, available in the "Summary" page of the workflow run.
- **Releases**: Pushing a tag (e.g., `v2024.1.0`) will automatically:
  - Build all MIPS variants.
  - Create a GitHub Release.
  - Upload binaries and checksums to the release.
  - Generate release notes based on the changelog.
