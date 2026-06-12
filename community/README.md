# Community Package Repos

This folder is the curated directory of **accepted community package repos**.
SharpMUSH games ping this folder (via the admin panel's package browser) to
discover community sources beyond the official packages — each listing here
shows up in every game's **Admin → Packages → Community** view, with the
community trust tier and a rendered view of the repo's README.

Listing here is what "accepted" means: trust levels are never self-declared
by repos (SharpMUSH decision 20.8) — they come from this curated directory or
from a game admin's own remote configuration.

## Getting your repo listed

1. Make your repo a valid package repo: one or more package directories with
   `package.yaml` manifests (format reference:
   [SharpMUSH examples/packages](https://github.com/SharpMUSH/SharpMUSH/blob/main/examples/packages/README.md)),
   ideally with an `index.yaml`, release tags (`<pkg>/v<version>`), and a
   root `README.md` — that README is what games render when admins inspect
   your listing.
2. Copy [`_template.yaml`](_template.yaml) to `community/<your-repo-slug>.yaml`
   and fill it in.
3. Open a pull request. CI validates the listing file (required fields,
   valid URL, no duplicate URLs).

Acceptance criteria, beyond CI passing:

- The repo is publicly clonable and contains at least one package that
  validates against the current manifest format.
- Release tags are used for versions and are never moved or deleted
  (games verify this and will show trust warnings if a tag moves).
- A README.md exists at the repo root describing what the packages do.

## Listing format

```yaml
name: Volund's MUSH Suite            # display name (required)
url: https://github.com/volund/mush-suite   # git URL (required)
description: "Core, BBS, jobs, and mail."   # one line (required)
branch: stable                        # optional — recommended branch
maintainers: [Volund]                 # optional
homepage: https://volund.example.com  # optional
```

Files starting with `_` or `.` are ignored by games (templates, dotfiles).

Removal works the same way: a PR deleting the file. Games re-read this folder
on browse, so removals propagate on their next refresh.
