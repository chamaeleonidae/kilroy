---
name: auto-update
description: Use when the user wants to pull the latest Kilroy source from the upstream remote (https://github.com/danshapiro/kilroy.git) and build/install the new binary.
---

# Auto-Update Kilroy

Pull the latest changes from the upstream public repo and rebuild the binary.

## Upstream Remote

The upstream source of truth is `https://github.com/danshapiro/kilroy.git`.
The local `origin` remote points to `git@github.com:chamaeleonidae/kilroy.git` (the fork).

## Steps

### 1. Ensure upstream remote exists

```bash
git remote get-url upstream 2>/dev/null || git remote add upstream https://github.com/danshapiro/kilroy.git
```

### 2. Fetch upstream

```bash
git fetch upstream
```

### 3. Show what's new

```bash
# Compare current HEAD with upstream/main
git log HEAD..upstream/main --oneline --no-merges
```

If there are no new commits, tell the user and stop — nothing to update.

### 4. Check for uncommitted local changes

```bash
git status --short
```

If the working tree is dirty, stop and tell the user to commit or stash before updating.

### 5. Record the current version

```bash
cat internal/version/version.go
```

### 6. Merge upstream into the current branch

```bash
git merge --ff-only upstream/main
```

If `--ff-only` fails (branches have diverged), present options to the user:
- `git merge upstream/main` — creates a merge commit
- `git rebase upstream/main` — rewrites local commits on top of upstream

Do not proceed without user confirmation if fast-forward fails.

### 7. Build the binary

```bash
go build -o ./kilroy ./cmd/kilroy
```

If the build fails, show the error and stop. Do not attempt to fix build errors without the user's direction.

### 8. Run the test suite

```bash
go test ./...
```

If tests fail, report them to the user. Do not install a broken binary.

### 9. Report the new version

```bash
./kilroy --version
```

Show the user the old and new version strings.

### 10. Install (optional — ask the user)

The built binary is `./kilroy` in the repo root. Ask the user where they want it installed:

- **Repo-local use**: leave it in place — invoke as `./kilroy`
- **PATH install**: copy to a directory in `$PATH`, e.g.:

```bash
# Example: install to /usr/local/bin (may need sudo)
cp ./kilroy /usr/local/bin/kilroy

# Or go install (installs to $GOPATH/bin or $HOME/go/bin)
go install ./cmd/kilroy
```

Confirm the destination with the user before copying.

## Safety

- Never force-push or modify `origin/main` as part of an update.
- If the merge produces conflicts, stop immediately and present the conflict list to the user.
- If the upstream fetch fails (network issue, auth), report the error and stop.
- If `go test ./...` has failures, tell the user before installing anything.
