# Agent Rules

## Base Ref Policy

When a task involves rebasing, comparing, or cleaning Marlin config files against upstream, use:

`BASE_REF=latest-2.1.x`

Do not switch to another base (for example `upstream/bugfix-2.1.x`) unless the user explicitly asks.

## Required Preflight

Before editing `Marlin/Configuration.h` or `Marlin/Configuration_adv.h`, run:

```bash
git rev-list --left-right --count $BASE_REF...HEAD
git diff --stat $BASE_REF -- Marlin/Configuration.h Marlin/Configuration_adv.h
```

If another comparison base is needed, stop and ask first.

## Reporting

End the final response with:

`Compared against: <ref>`
