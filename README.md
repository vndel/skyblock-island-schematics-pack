# skyblock-island-schematics-pack

![Minecraft](https://img.shields.io/badge/Skyblock-00C2A8?logo=minecraft&logoColor=white) ![WorldGuard](https://img.shields.io/badge/WorldGuard-regions-brightgreen) ![License](https://img.shields.io/badge/License-MIT-green)

> Island grid layout, WorldGuard flag policy and async generation settings for a
> skyblock network.

## Grid spacing cannot be changed later

```yaml
grid:
  spacing: 400
  protection-radius: 200
  island-height: 120
```

This is the one decision in the repo that is irreversible without regenerating
the world, so the reasoning matters:

- **400 spacing with a 200 radius** leaves a 200-block gap between island
  borders. Wide enough that a neighbour's mob farm does not consume your spawn
  cap; narrow enough that the world does not sprawl into millions of regions.
- **y=120** leaves room to build both up and down within vanilla height limits.
  Placing islands at y=64 wastes half the available vertical space.

A `VOID` generator is required — a normal generator would fill the gaps with
terrain, which defeats the purpose and multiplies chunk-generation cost.

## Async island creation

```yaml
generation:
  async-paste: true
  blocks-per-tick: 4000
  pregenerate-chunks: true
```

Pasting a schematic is thousands of block writes. On the main thread that stalls
the server for **every player who joins and creates an island** — the worst
possible moment, because it is a new player's first impression.

Chunks are pre-generated before the paste so the paste itself does not trigger a
synchronous chunk load.

## WorldGuard: deny by default

```yaml
__global__:
  flags:
    build: deny
    tnt: deny
    creeper-explosion: deny
    chest-access: deny
```

The global region denies everything; island regions grant selectively via
owner/member checks. The reverse ordering — allow globally, deny per-island — is
how skyblock servers end up with players TNT-ing each other's islands from
outside the protection radius.

Notable flags:

| Flag | Value | Reason |
|---|---|---|
| `tnt` | `deny` | Prevents cross-border griefing |
| `fire-spread` | `deny` | Stops island-wide fire propagation |
| `mob-spawning` | `allow` | Farms are core skyblock gameplay |
| `chest-access` | `deny` globally | Members get it through the island region |

## Per-island limits

```yaml
limits:
  hoppers-per-island: 64
  entities-per-island: 150
  spawners-per-island: 24
```

Without caps, one player's iron farm becomes everyone's TPS problem — the mob
cap is shared per-world unless per-player spawning is enabled, and hoppers tick
regardless of whether anyone is nearby.

## Permission groups

Deliberately flat: `default -> member -> vip -> mvp`. Deep hierarchies are hard
to audit, and on skyblock nearly every tier difference is a numeric limit rather
than a new capability — so limits live in group **meta**, not in permission
nodes:

```yaml
vip:
  meta:
    island-member-limit: '8'
    island-hopper-limit: '64'
    sell-multiplier: '1.15'
```

## Files

| File | Contents |
|---|---|
| `config/island-generation.yml` | Grid, async paste, per-island limits |
| `config/worldguard-regions.yml` | Global, island template and spawn flags |
| `config/permission-groups.yml` | Groups with meta-driven limits |

## License

MIT — see [LICENSE](LICENSE).
