# GitHub Actions Workflow Implementation Plan for cloudflared Fork

I will create two GitHub Actions workflows to automate upstream synchronization and MIPS architecture compilation.

## 1. Upstream Sync Workflow (`.github/workflows/sync.yml`)
**Goal**: Keep the fork synchronized with the upstream `cloudflare/cloudflared` repository daily.

### Triggers
- **Scheduled**: Runs daily (e.g., at 00:00 UTC).
- **Manual**: `workflow_dispatch` for immediate synchronization.

### Implementation Steps
1.  **Checkout**: Use `actions/checkout` with `fetch-depth: 0` (full history for merging).
2.  **Git Configuration**: Set up a bot user (e.g., `github-actions[bot]`) for commits.
3.  **Sync Logic**:
    - Add upstream remote: `https://github.com/cloudflare/cloudflared.git`.
    - Fetch upstream branches.
    - Merge `upstream/master` into the local default branch.
    - Push changes to the fork.
4.  **Error Handling**:
    - If a merge conflict occurs, the step fails.
    - **Notification**: Use `actions-ecosystem/action-create-issue` (or similar) to automatically create a GitHub Issue titled "🚨 Upstream Sync Failed" with logs, notifying the maintainer to resolve conflicts manually.

## 2. MIPS Cross-Compilation Workflow (`.github/workflows/build-mips.yml`)
**Goal**: Build static binaries for MIPS architectures using a matrix strategy.

### Triggers
- **Automatic**: `workflow_run` (triggered when `sync.yml` completes successfully).
- **Manual**: `workflow_dispatch` (allows choosing a specific branch/tag).
- **Release**: `push` on tags matching `v*` (triggers a release build).

### Build Strategy
- **Go Version**: `1.24` (matching `go.mod`).
- **CGO Policy**: **Disabled (`CGO_ENABLED=0`)**.
    - *Reason*: Cross-compiling CGO for MIPS requires complex C toolchains (gcc-mips-linux-gnu, etc.) and libc compatibility (musl vs glibc) on target routers. Pure Go builds are the industry standard for portable MIPS binaries.
- **Matrix Configuration**:
    - `goos`: `linux`
    - `goarch`: `mips`, `mipsle`, `mips64`, `mips64le`
    - `gomips`: `softfloat` (ensures maximum compatibility with older MIPS hardware).

### Implementation Steps
1.  **Environment Setup**:
    - Install Go 1.24.
    - Checkout code.
2.  **Dependencies**: Run `go mod download`.
3.  **Compilation**:
    - Run `go build` with injected `-ldflags` (Version, BuildTime) to match the official Makefile format.
    - Redirect build logs to a file (`build.log`) for archiving.
4.  **Verification**:
    - Generate SHA256 checksums for all binaries.
5.  **Artifacts**:
    - Upload binaries and `build.log` as workflow artifacts.
6.  **Release (Conditional)**:
    - If triggered by a tag, use `softprops/action-gh-release` to publish binaries and checksums to GitHub Releases.
    - Auto-generate release notes based on the commit log.
7.  **Error Handling**:
    - **Timeout**: Set `timeout-minutes: 60` to prevent stalled builds.
    - **Notification**: On failure, create an Issue notifying the maintainer.

## 3. Documentation
- I will create a `CI_README.md` or update the existing `README.md` to explain:
    - How the auto-sync works.
    - How to manually trigger builds.
    - Where to find the build artifacts.

## Verification Plan
Since I cannot run the Actions locally, I will:
1.  **Syntax Check**: Ensure YAML files follow valid GitHub Actions schema.
2.  **Build Command Verification**: Run a local cross-compilation test command (e.g., `GOOS=linux GOARCH=mips go build`) to ensure the codebase compiles for MIPS without CGO errors.
