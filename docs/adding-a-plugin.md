# Adding a plugin

## 1. Find the commit you are approving

```bash
git ls-remote https://github.com/<owner>/<repo>.git HEAD
```

Review **that commit**, not "the repo". Record the SHA; it is what you are
signing off.

## 2. Add the entry

Entries live in `plugins[]`, alpha-sorted by `name`. Three source shapes are
valid for an external plugin:

```jsonc
// whole repo is the plugin  (what diagram-design uses)
{ "source": "url", "url": "https://github.com/owner/repo.git", "sha": "<40 hex>" }

// plugin lives in a subdirectory of a larger repo
{ "source": "git-subdir", "url": "https://github.com/owner/repo.git",
  "path": "plugins/thing", "ref": "main", "sha": "<40 hex>" }

// GitHub shorthand
{ "source": "github", "repo": "owner/repo", "ref": "main", "sha": "<40 hex>" }
```

A first-party plugin kept in this repo uses a relative path instead, and needs
no `sha`:

```jsonc
{ "source": "./plugins/equinox-delivery" }
```

## 3. Validate before you push

```bash
claude plugin validate .claude-plugin/marketplace.json --strict
```

The SHA must be 40 lowercase hex characters. Uppercase or truncated fails.

## 4. After merge

Tag the marketplace, then raise the pin bump in `claude-policy`:

```bash
git tag -a mp-v$(date +%Y.%m.%d) -m "Add <plugin>"
git push origin mp-v$(date +%Y.%m.%d)
```

## Updating an existing pin

Same process, shorter: diff the upstream range you are moving across, re-run
the checklist gates that the diff touches (usually 03 and 04), update the
`sha`, note it in the approval record, tag.

```bash
git log --oneline <old-sha>..<new-sha>
```

Never bump a pin you have not diffed.
