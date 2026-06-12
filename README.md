# SharpMUSH Packages

The official softcode package repository for
[SharpMUSH](https://github.com/SharpMUSH/SharpMUSH) — a modern .NET MUSH
server targeting PennMUSH compatibility.

A **package** is a declarative description of softcode: the objects that
should exist on a MUSH and the attributes, flags, locks, and parent
relationships they carry. Packages are installed, upgraded, and authored
through SharpMUSH's web admin panel (`/admin/packages`) using a plan/apply
model — the server computes a changeset against the live database, a wizard
reviews it, and only then is anything written. Upgrades three-way-merge
against stored baselines so your local customizations survive, every apply is
recorded as a revision, and installs can be rolled back.

No more pasting `.mush` files and hoping the dbrefs line up.

## Using this repo

Add it as a remote in the SharpMUSH admin panel
(**Admin → Packages → Remotes**):

```
https://github.com/SharpMUSH/SharpMUSH-Packages
```

Then browse, pick a package and version, review the changeset, and apply.
Dependencies are resolved against your installed packages, with version
constraints checked before anything is planned.

## Packages

| Package | Version | Description |
|---|---|---|
| [`hello-world/`](hello-world/) | 1.0.0 | Minimal example: one object, one `+hello` command |
| [`who-where/`](who-where/) | 1.2.0 | `+who` and `+where` commands with a shared formatting object |
| [`starter-area/`](starter-area/) | 1.0.0 | Two rooms and linked exits |
| [`bbs-lite/`](bbs-lite/) | 0.9.0-beta | Tiny bulletin board; dependencies, typed configure params, cross-package refs |

These currently double as the reference implementations for package authors —
each README explains which manifest features it demonstrates. The curated
default softcode experience (scene-system, bboard, chargen, events, finger,
http-hooks) will land here as those systems mature.

## Repo structure & releases

A package is a directory containing a `package.yaml` manifest; the root
[`index.yaml`](index.yaml) lists every package with its id, latest version,
and description.

**Releases are git tags** named `<package-dir>/v<version>` — e.g.
`who-where/v1.2.0`. The admin panel's version list is the tag list; installing
a version checks out its tag; `main` is the development channel. Release tags
are **immutable**: never move or delete one — publish a fix as a new version.
Installers record the exact commit they applied and warn loudly if a release
tag later points elsewhere.

## The manifest format, in brief

Manifests are **desired state**, not command scripts — and they never contain
dbrefs. Object identity is symbolic, resolved at install time:

```yaml
format: 1
package: who-where              # id slug
version: 1.2.0                  # semver
description: "+who and +where commands."
license: MIT

depends:
  - package: some-dependency    # optional, with version constraints
    version: ">=1.0 <2.0"
    source:                     # where to find it if not installed
      repo: https://github.com/SharpMUSH/SharpMUSH-Packages
      path: some-dependency/

configure:
  storage:
    label: "Object that stores data"
    type: dbref                 # dbref | string | number | boolean

objects:
  - ref: ww_global              # abstract name — never a dbref
    type: thing                 # thing | room | exit | player
    name: Who-Where Global Object
    parent: "{{ww_functions}}"
    attributes:
      CMD_+WHO: |-
        $+who:@pemit %#=[u({{ww_functions}}/WW_FN_HEADER,Who)]...
```

Ref tokens (case-insensitive; every token must resolve or the manifest is
rejected; escape a literal `{{` as `{{{{`):

| Syntax | Resolved from |
|---|---|
| `{{name}}` | Another object defined in the same package |
| `{{pkg/name}}` | An object owned by `pkg` (must be listed under `depends:`) |
| `{{$name}}` | Server configuration (`room_zero`, `master_room`, `player_start`, `god`, `package_manager`) |
| `{{?name}}` | The installing admin, prompted during review (declare under `configure:`) |

**Write all MUSHcode as block scalars (`|-`)** — YAML plain and double-quoted
strings corrupt softcode (`\` escapes, ` #` comment truncation, `&token`
anchors). Exits declare `location:` and `destination:`; rooms declare
neither.

The full format reference lives in the SharpMUSH repo:
[`examples/packages/README.md`](https://github.com/SharpMUSH/SharpMUSH/blob/main/examples/packages/README.md).
Every manifest here is validated by CI (`tools/PackageValidator`, strict
mode: warnings fail).

## Contributing a package

1. Author your package — by hand or with the admin panel's authoring tools
   (object picker, dbref-to-ref resolution, export).
2. Add a directory with `package.yaml` and a `README.md`; declare a
   `license`.
3. Add the directory to `index.yaml` (path, id, version, description).
4. Open a pull request. Keep attribute namespaces behind a
   `convention_prefix`, never hardcode dbrefs, declare every `{{?configure}}`
   ref with a label and type, and use block scalars for all code.
5. Once merged, a maintainer tags the release `<package>/v<version>`.

Package ids are first-come within this repo, must not collide with an
existing id even with punctuation stripped (`who-where` blocks `whowhere`),
and released versions are never rewritten — bump the version instead.

## License

Same license as SharpMUSH unless a package declares otherwise in its
manifest. Package manifests are softcode — inspect anything you install; the
admin panel's review screen flags dangerous patterns (`@force`, `@toad`, …)
to help.
