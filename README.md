# Automaticity Trainer releases

This public repository contains release metadata and application-status information for **Automaticity Trainer**. The application source code is maintained separately in a private repository.

## Files

- [`VERSION`](VERSION) contains the latest published application version.
- [`status.json`](status.json) controls whether published application versions are active.
- GitHub Releases may contain installers and release notes when packaged builds are published.

## Application status

`status.json` has an overall `active` switch and optional version-specific overrides:

```json
{
  "schemaVersion": 1,
  "product": "automaticity-trainer",
  "latestVersion": "1.2.0",
  "active": true,
  "message": "Automaticity Trainer is active.",
  "versions": {
    "1.2.0": {
      "active": true,
      "message": "Version 1.2.0 is active."
    }
  },
  "updatedAt": "2026-07-13T00:00:00Z"
}
```

The overall switch is applied first. A version is usable only when the overall switch and its version-specific switch are both active. A version without an override follows the overall switch.

To suspend every version, set the root `active` value to `false`. To suspend one version, keep the root active and set that version's `active` value to `false`. Update its message and `updatedAt` value in the same commit.

Once an installation successfully reads an inactive status, it remembers that state locally and stops at its startup status screen. On a later startup, it will become usable again only after successfully reading an active status for its installed version. An unavailable or malformed status file does not newly disable an installation, but it also does not unlock an installation that previously observed an inactive status.

Keep this repository public: installed copies read the raw `status.json` file without GitHub credentials.

## Ownership

Copyright (c) 2026 Wilco Ludikhuize. This metadata repository does not publish or license the private application source code.
