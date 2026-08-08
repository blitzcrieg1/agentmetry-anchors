# Agentmetry anchors

Merkle roots from [Agentmetry](https://github.com/blitzcrieg1/agentmetry) trails, published where the recording machine cannot rewrite them.

## What is in here

One append-only JSONL file per host. Each line is a checkpoint:

```json
{"v":1,"alg":"rfc6962-sha256","tree_size":5868,"root_sha256":"00215b14...","head_seq":5868,"head_sha256":"65deb227...","timestamp":"2026-08-08T11:20:00Z","host_id":"home-lab","trail_name":"agentmetry-trail.jsonl"}
```

That is the whole payload. A root is a hash over hashes, so it discloses nothing about the events underneath it: no commands, no file paths, no source code, no arguments. What it does disclose is how many events a trail held and when the checkpoint was taken.

## What it proves

Agentmetry chains every trail record with SHA-256 and keeps the head in a sidecar. That catches corruption, truncation, and in-place edits.

It does not catch an attacker with write access to the data directory. They can edit an event, recompute every hash after it, rewrite the sidecar, and produce a file that verifies cleanly. Every input to that check sits on the machine they control. This is the ceiling of what any self-contained file can prove about itself, not a defect in the chain.

This repository is where that ceiling gets raised. Once a root is committed here, the recording machine can no longer edit its way out of it: any change to a record below that tree size produces a different root, and the root that was published stays published.

Force-pushes and branch deletion are blocked. A commit here is durable against the workstation that produced it, which is the entire point.

## Verifying a checkpoint

Given a trail and this log:

```bash
agentmetry verify /path/to/agentmetry-trail.jsonl --trail \
  --anchors home-lab/agentmetry-trail.anchors.jsonl
```

`--anchors` matters. Checked against the anchor log sitting next to the trail, an attacker who rewrote one rewrote the other. Checked against this copy, the comparison is the one thing they could not forge.

Anchored coverage is a range, not a boolean. A checkpoint covers the records that existed when it was taken and says nothing about what was appended afterwards, so `verify` reports the anchored range and the unanchored tail separately.

## Why this is public

Force-push protection is free on public repositories and requires a paid plan on private ones. An anchor a compromised workstation can force-push is not an anchor, so the choice was between a real guarantee with visible event counts, and a private log that quietly fails against the exact threat it exists for.

The second one is worse. Anyone can now verify these roots independently, which is a property a private log would not have had either.

## Background

- [Anchoring documentation](https://github.com/blitzcrieg1/agentmetry/blob/master/docs/anchoring.md)
- [Issue #34](https://github.com/blitzcrieg1/agentmetry/issues/34), which described the gap
- Checkpoints follow [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962) tree hashing (0x00 leaf prefix, 0x01 node prefix, split at the largest power of two below n)
