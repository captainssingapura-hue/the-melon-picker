# the-melon-picker
Picking the correct melons.

Melon Picker is a version manager for estates of Java libraries and applications versioned by Line–Date Versioning.
It reads an estate's releases from its Maven repositories and checks them. It resolves each application release to
the exact jars it runs with, once, when the release is made. Then it installs and launches that application from the
record of those jars.

The name is a metaphor and stays out of the code: classes are named for what they are, such as `Catalogue`,
`Resolver`, `Store` and `Launcher`.

Its features and the work on them are kept in
[homing-graph-view-studio](https://github.com/captainssingapura-hue/homing-graph-view-studio), beside graph view's.
Nothing here depends on the studio.

## The idea

**Line–Date Versioning.** A library is numbered `BREAK.YYYYMMDD.N`:
- `BREAK` rises on an incompatible change, and only then;
- the date is the release's identity;
- `N` is its patch level.

An application is numbered `YYYYMMDD.N`, and nothing may depend on one. Within a line, any later release is a drop-in
for an earlier one. So when two releases ask for one library on the same line, the higher wins. When they ask for it
on two lines, nothing can satisfy both.

**What Melon Picker adds to Maven:**
- **The Maven repository is the catalogue.** Each release carries a small descriptor: its unit, its kind, the exact
  versions it depends on, and its jar's digest. Maven still builds and publishes.
- **A release is data, not a type.** launcher-demo's reference estate wrote each release as a Java record. Here the
  catalogue is read from descriptors, and the typed model survives only as a test kit.
- **Virtual releases are derived, not declared.** A release whose jar has the same digest as an earlier one in its
  line is a virtual release of it: the same code with newer dependencies.
- **What runs is decided once.** Each application release gets one dependency spec: generated, signed and promoted by
  its digest, never resolved again.
- **The runtime never resolves.** It installs exactly what a spec names into a store keyed by digest, where nothing is
  ever overwritten.
- **Checks guard each release.** Examples are a patch only to the newest release in a line, a spec that covers every
  class the binary uses, and security floors as sets of version ranges.

## Modules

All are empty for now. Each holds only its `package-info.java`, which says what it will hold.

| Module | Will hold | Depends on |
|---|---|---|
| `melon-picker-core` | the model as data: units, versions, releases, the catalogue, the resolver, the checker, advisories, floors, policy; no I/O | jOntology core |
| `melon-picker-json` | the descriptor, manifest, catalogue, spec and policy formats | core |
| `melon-picker-maven` | reading Maven-layout repositories, as a directory or over HTTP | core, json |
| `melon-picker-runtime` | the store, install from a spec, launch, cleanup | to be settled (see below) |
| `melon-picker-cli` | `check`, `resolve`, `explain`, `spec`, `install`, `launch`, `gc` | core, json, maven, runtime |
| `melon-picker-testkit` | an estate declared in Java types, turned into a catalogue; the reference estate as the main fixture | core, jOntology core |
| `melon-picker-explorer` | a catalogue in 3D, drawn by [homing-graph-view](https://github.com/captainssingapura-hue/homing-graph-view) | core, `homing-graph-view-viewer` |

A Maven plugin, with the goals `describe`, `check`, `spec`, `virtual` and `catalogue`, joins in phase 2. An empty
plugin module does not build.

## Phases

| Phase | Work | Done when |
|---|---|---|
| 1. Core as data | core, json, testkit; derived virtual releases; the catalogue rules; patches only to the newest release | on all 52 releases of the reference estate, the data model resolves exactly as the typed model does: the same sets, warnings and conflicts. The typed model's tests pass in their new homes |
| 2. Descriptors and catalogue | maven; the plugin's `describe`, `virtual` and `catalogue` | the catalogue read from the reference estate's repository equals the typed estate's |
| 3. Specs | the plugin's `spec`; the spec covers the binary; floors | a spec for every application release; a missing dependency fails the build |
| 4. Runtime | runtime, cli | the reference estate's apps launch from their specs, side by side, with the network off; changed bytes are refused |
| 5. Trust | signed specs; a launch token naming a spec's digest | a tampered spec, another key and an unknown digest are each refused |
| 6. Gates | members pinned, reproducible builds, line compatibility; the Java runtime pinned | each gate fails on a release broken for it |
| 7. Automation | the update bot; the curated mirror and derived lines for external libraries | the bot, started from physical releases only, makes the reference estate's virtual releases again |

## Build

Needs JDK 25, and homing-graph-view `0.1.0-SNAPSHOT` installed in the local Maven repository (`mvn install` in the
homing-graph-view repo). jOntology `0.1.0` comes from Maven Central.

```bash
mvn verify
```

## Open

- **The runtime's dependencies.** The runtime reads specs but must not have the resolver on its classpath. The spec
  format sits in json, which builds on core, where the resolver is. One of the two has to split before phase 4.
- **Its own version.** Melon Picker is a published product, so its releases follow Line–Date Versioning. It is
  `0.1.0-SNAPSHOT` until its first release.
