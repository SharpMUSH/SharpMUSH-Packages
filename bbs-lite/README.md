# bbs-lite

A deliberately tiny bulletin board that exercises every manifest feature
beyond the basics:

- **Dependency with a range constraint and a source hint** — requires
  `who-where` at any 1.x version (`>=1.0 <2.0`). Install is blocked until it's
  present, but because the dependency declares a `source:` (repo, directory
  path, branch), the installer can offer to fetch it from there rather than
  just reporting it missing.
- **`?configure` ref** — `?bbs_storage` is declared under `configure:` with a
  label; the installing admin is prompted to supply the storage object during
  review, and every occurrence is substituted at apply time.
- **`$well-known` ref** — the lounge room is parented to `$room_zero`,
  resolved from server configuration on the target game.
- **Lock with an `~internal` ref** — the function object's use-lock points at
  `~bbsl_global`, so only the global object can call its functions.
- **Attribute flags** — `BBSL_FN_LIST` carries `no_clone`.
- **Prerelease version** — `0.9.0-beta` sorts before `0.9.0`.

Posts are stored as `POST_<timestamp>` attributes on the configured storage
object; `+bbread` lists them via `lattr()` with a `firstof()` fallback when
the board is empty.
