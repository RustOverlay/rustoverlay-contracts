# RustOverlay Contracts

The wire schema between the RustOverlay agent and the relay. This is the canonical definition —
both sides generate from it, neither hand-maintains a copy.

Published so anyone running the [agent](https://github.com/<org>/rustoverlay) can see exactly what
leaves their machine.

```
proto/rustoverlay/v1/team.proto
```

## Compatibility rules

Agents are installed desktop software and cannot be force-updated. An agent released today will be
talking to a much newer relay a year from now, so:

- **Never renumber a field. Never reuse a number.** Mark removals `reserved`.
- **Additive changes only.** New fields are optional by construction in proto3.
- **Never repurpose a field's meaning.** Add a new one instead — a consumer on an old build cannot
  know the meaning changed.
- Angles and coordinates travel in the game's own convention (degrees `[0, 360)`, positive pitch
  downward, Y-up, north +Z). Converting is the consumer's job, so a change of convention downstream
  cannot silently reinterpret archived data.

## Generated packages

| Target | Package |
|---|---|
| C# (agent) | NuGet `RustOverlay.Contracts` |
| Python (relay) | PyPI `rustoverlay-contracts` |

Both are built and published by CI on tag. Do not generate by hand and commit the output.

## Licence

Apache-2.0. Deliberately permissive rather than matching the agent's GPL-3.0: the relay is a
separate, closed codebase that imports the generated Python package, and a copyleft schema would
pull it in. Copyleft belongs on the application, not the schema.
