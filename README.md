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

## How consumers take it

There is no package registry in the loop. Each consumer vendors this tree and its own CI holds the
copy byte-identical to `main`, so drift fails a build rather than surfacing as a wire bug:

| Consumer | Vendors | Checked by |
|---|---|---|
| [agent](https://github.com/RustOverlay/rustoverlay-agent) | `proto/` — protoc runs at build time via `Grpc.Tools` | `diff -ru` against this repo in agent CI |
| backend | generated Python in `_gen/`, plus `contracts.lock`, a sha256 of this tree | `task proto && git diff --exit-code` in backend CI |

The two differ because their vendored artifacts differ. The agent keeps the `.proto` itself, so a
diff against this repo is the complete check. The backend keeps *generated output*, whose
relationship to the schema is not visible by eye, so it locks a hash as well.

A registry was considered and dropped: a version number nobody was reading, in exchange for a
publish step, an account and an API key. The compatibility rules above are enforced by
`buf breaking` in CI, which is what actually stops a field being renumbered — a package version
never did.

Do not generate by hand and commit the output into this repo.

## Changing the schema

An edit here is not live until both consumers have re-vendored it. `buf breaking` runs against the
previous commit on `main`, so the rules above are enforced on the way in.

## Licence

Apache-2.0. Deliberately permissive rather than matching the agent's GPL-3.0: the relay is a
separate, closed codebase that imports the generated Python package, and a copyleft schema would
pull it in. Copyleft belongs on the application, not the schema.
