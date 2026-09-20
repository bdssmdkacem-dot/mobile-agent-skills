# Mobile Agent Skills

Reusable Agent Skills for Flutter and Android engineering workflows.

## Skills

- `android-permissions` — systematic diagnosis and verification of Android permissions from source through the final APK and runtime state.

## Structure

```
skills/
└── android-permissions/
    ├── SKILL.md
    └── references/
        └── apk-verification.md
```

## Standard

Skills in this repository follow the open Agent Skills specification:
https://github.com/agentskills/agentskills

Each skill is self-contained and can be installed or copied into a compatible agent's skills directory.

## Design principles

- Diagnose before modifying code.
- Verify the final artifact, not only source files.
- Separate manifest declaration from runtime grant state.
- Prefer the smallest safe change.
- Re-run analysis, tests, and build after changes.
- Never report success from a build alone when the user-visible behavior has not been verified.
