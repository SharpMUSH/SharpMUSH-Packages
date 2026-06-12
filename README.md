# SharpMUSH Packages

The official softcode package repository for
[SharpMUSH](https://github.com/SharpMUSH/SharpMUSH) — a modern .NET MUSH
server targeting PennMUSH compatibility.

A **package** is a declarative description of softcode: the objects that
should exist on a MUSH and the attributes, flags, locks, and parent
relationships they carry. Packages are installed, upgraded, and authored
through SharpMUSH's web admin panel (`/admin/packages`) using a plan/apply
model — the server computes a changeset against the live database, a wizard
reviews it (with three-way merge on upgrades, so local customizations
survive), and only then is anything written.

No more pasting `.mush` files and hoping the dbrefs line up.

## Using this repo

Add it as a remote in the SharpMUSH admin panel
(**Admin → Packages → Remotes**):

```
https://github.com/SharpMUSH/SharpMUSH-Packages
```

Then browse, pick a package, review the changeset, and apply. Dependencies
are resolved automatically against your installed packages, with version
constraints checked before anything is planned.

## Packages

| Package | Version | Description |
|---|---|---|
| [`hello-world/`](hello-world/) | 1.0.0 | Minimal example: one object, one `+hello` command |
| [`who-where/`](who-where/) | 1.2.0 | `+who` and `+where` commands with a shared formatting object |
| [`bbs-lite/`](bbs-lite/) | 0.9.0-beta | Tiny bulletin board; demonstrates dependencies, configure refs, well-known refs, and locks |

These currently double as the reference implementations for package authors —
each README explains which manifest features it demonstrates. The curated
default softcode experience (scene-system, bboard, chargen, events, finger,
http-hooks) will land here as those systems mature.

## Repo structure

A package is a directory containing a `package.yaml` manifest. The root
[`index.yaml`](index.yaml) lists every package for fast discovery:

```
SharpMUSH-Packages/
├── index.yaml
├── hello-world/
│   ├── package.yaml
│   └── README.md
├── who-where/
│   └── ...
└── bbs-lite/
    └── ...
```

## The manifest format, in brief

Manifests are **desired state**, not command scripts — and they never contain
dbrefs. Object identity is symbolic, resolved at install time:

```yaml
package: who-where              # id slug
version: 1.2.0                  # semver
description: "+who and +where commands."

depends:
  - package: some-dependency    # optional, with version constraints
    version: ">=1.0 <2.0"
    source:                     # where to find it if not installed
      repo: https://github.com/SharpMUSH/SharpMUSH-Packages
      path: some-dependency/

objects:
  - ref: ww_global              # abstract name — never a dbref
    type: thing
    name: Who-Where Global Object
    parent: ~ww_functions       # ~name  = another object in this package
    attributes:
      CMD_+WHO:
        value: "$+who:@pemit %#=[u(~ww_functions/WW_FN_HEADER,Who)]..."
```

Three ref sigils:

| Syntax | Resolved from |
|---|---|
| `~name` | Another object defined in the same package |
| `$name` | Server configuration (`$room_zero`, `$master_room`, `$player_start`, `$god`, `$package_manager`) |
| `?name` | The installing admin, prompted during review (declare under `configure:`) |

The full format reference lives in the SharpMUSH repo:
[`examples/packages/README.md`](https://github.com/SharpMUSH/SharpMUSH/blob/main/examples/packages/README.md).
Every manifest in this repo is validated by SharpMUSH's test suite.

## Contributing a package

1. Author your package — either by hand or with the admin panel's authoring
   tools (object picker, dbref-to-ref resolution, export).
2. Add a directory with `package.yaml` and a `README.md`.
3. Add the directory to `index.yaml`.
4. Open a pull request. Keep attribute namespaces behind a
   `convention_prefix`, never hardcode dbrefs, and declare every `?configure`
   ref with a label the installing admin will understand.

## License

Same license as SharpMUSH. Package manifests are softcode — inspect anything
you install; the admin panel's review screen flags dangerous patterns
(`@force`, `@toad`, …) to help.
