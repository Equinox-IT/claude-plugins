# Plugin approval checklist

Seven gates. Run all of them, record the findings in
`docs/approval-record/<name>.md`, then open the PR.

This file is also the source for the automated policy-scan prompt in the next
phase, so keep it written as instructions, not prose.

## 01 Provenance

- [ ] Repo, owner and account age identified; owner is not a lookalike of a known project
- [ ] Maintainer count and bus factor recorded (one maintainer is a finding, not a rejection)
- [ ] Licence present and compatible with client redistribution
- [ ] `SECURITY.md` exists with a private disclosure route

## 02 Capability surface — read the manifest, not the README

- [ ] `hooks` — any lifecycle hook is arbitrary shell on every session. Read every one or reject
- [ ] `mcpServers` — each is new tools and a new egress path. Resolve where it connects
- [ ] `agents/` — subagents run with their own tool grants
- [ ] Every command's `allowed-tools` listed; `Bash` and `WebFetch` flagged explicitly
- [ ] Plugin `source` checked — `"./"` means the whole repo ships, so the whole repo is in scope

## 03 Code review

- [ ] Grep: `subprocess`, `os.system`, `eval`, `exec`, `__import__`, `pickle`
- [ ] Grep network: `requests`, `urllib.request`, `httpx`, `socket`, `curl`, `wget`
- [ ] Grep destructive filesystem calls and writes outside the project root
- [ ] No minified, obfuscated or base64-blob payloads anywhere in the tree
- [ ] Third-party runtime dependencies enumerated; none installed silently

## 04 Prompt content — the surface people forget

- [ ] `SKILL.md` and references read end to end for instructions that widen permissions, disable checks, or exfiltrate context
- [ ] External content the plugin ingests (files, URLs, markers) is handled as untrusted data, in writing
- [ ] No instruction to read credentials, environment variables or `.env` files

## 05 Data and egress

- [ ] Every outbound host in generated output enumerated (fonts, CDNs, telemetry)
- [ ] Deliverables leaving Equinox carry no third-party runtime call the client has not accepted
- [ ] Client-data segregation reviewed: does the workflow pull one engagement's material into another's session?

## 06 Change control

- [ ] Pinned to a commit SHA — never a floating branch
- [ ] Named owner accountable for reviewing the diff before each pin bump
- [ ] Upstream CI reviewed for supply-chain exposure: no `pull_request_target`, actions SHA-pinned, secrets unreachable from merged contributor code

## 07 Deploy and roll back

- [ ] Keys present in every managed-settings body deployed, on every platform in use
- [ ] Piloted on one machine and verified with `/plugin` before wider assignment
- [ ] Rollback tested: removing the entry actually removes the plugin on next session
