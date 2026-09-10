# RustOverlay Contracts

The wire schema between the RustOverlay agent and the relay. This is the canonical definition —
both sides generate from it, neither hand-maintains a copy.

Published so anyone running the [agent](https://github.com/RustOverlay/rustoverlay-agent) can see exactly what
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

| Target | Package | Status |
|---|---|---|
| C# (agent) | NuGet `RustOverlay.Contracts` | published by CI on tag |
| Python (relay) | PyPI `rustoverlay-contracts` | **not built yet** — the backend commits `_gen/` and pins `contracts.lock` to a sha256 of this tree |

Do not generate by hand and commit the output.

## Releasing

The tag is the only place the version is recorded, so a release cannot disagree with its own name:

```
git tag v1.0.0 && git push origin v1.0.0
```

That packs `csharp/RustOverlay.Contracts` at `1.0.0`, pushes it to nuget.org and opens a GitHub
release with the `.nupkg` attached. Requires the `NUGET_API_KEY` repository secret.

`buf breaking` runs against **the last released tag**, not the previous commit — what matters is
whether a change breaks an agent already installed on someone's machine, and only tagged versions
ever reach one.

## Licence

Apache-2.0. Deliberately permissive rather than matching the agent's GPL-3.0: the relay is a
separate, closed codebase that imports the generated Python package, and a copyleft schema would
pull it in. Copyleft belongs on the application, not the schema.
