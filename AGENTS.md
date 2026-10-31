# nuphy-config — agent rules

`air60he.toml` is the source of truth for the NuPhy Air60 HE: key actions,
per-key travel and macros, written to the keyboard with `nuphyctl he` (pinned
in `mise.toml`). The keyboard is used mainly on Windows and reaches the Mac
only through Deskflow, so layout logic lives in the keyboard itself, not in
Karabiner or AutoHotkey.

## Changing the keyboard

1. Edit `air60he.toml`. Touch only what the request names; the file lists
   only what differs from factory, plus the full travel table. Deleting a
   line changes nothing on the keyboard: `apply` touches only listed keys, so
   to undo a binding set it back to its factory value (the `# factory …`
   comment, `NO` on an Fn layer, `[]` for a macro), apply, then `export`.
2. `mise run check` — offline validation. Fix every error before going on.
3. `mise run diff` — read the planned changes; they must match the request
   and nothing else.
4. Ask the user to close drive.nuphy.io, then `mise run apply`. It backs up,
   writes, reads back and fails unless the keyboard matches.
5. Commit `air60he.toml`.

If the user changed something in NuPhy IO, `mise run export` first and
commit the result before editing.

To see every layer of a mode with position names and factory values:
`nuphyctl he dump --mode m1` (add `--changed` for only what differs).
Position names, assignable actions and what the keyboard cannot do from this
file (tap-hold, layer toggles) are listed in the nuphyctl `he` docs and in
the user's vault guide `knowledge/nuphy-air60he-config.md`.

## Rules

- Commands that open the keyboard (`diff`, `apply`, `export`, `nuphyctl he …`)
  need the agent sandbox off: hidapi cannot reach USB from inside it.
- Never write while drive.nuphy.io is open.
- Do not hand-edit raw `0x9…` actions (DKS, Mod-Tap, Toggle, Snap Tap pairs):
  they point into tables nuphyctl does not manage yet.

## When nuphyctl cannot express a binding

The keyboard can do more than nuphyctl writes (Mod-Tap, DKS, Toggle, Snap
Tap, mode settings). If a binding needs one of these, extend the fork in the
same session instead of setting it in NuPhy IO for good: the fork is
`~/Development/yoshintame/nuphyctl` (module `src/he/`). Reverse the format on
the free mode m3 (the user sets a sample in NuPhy IO, you read and diff),
implement it behind the same safety rules (command whitelist, backup,
read-back, write tests), check on m3 with a restore, push the fork, bump the
rev in `mise.toml`, then add the binding here. The full procedure is the
user's vault note `knowledge/nuphyctl-he-extending.md`; never write to the
keyboard with `nuphyctl raw send`.
- Keys holding `FACTORY_RESET` or `FIRMWARE_LOADER` (m1 `fn1` `LBRC` holds
  factory reset) cannot be reassigned by `apply`: no command writes those
  actions, so a backup could not put them back. Move them in NuPhy IO first.
- Modes are `m1`..`m3` (NuPhy IO's M1..M3). Layers are numbered across modes
  exactly as NuPhy IO shows them: `base` is a mode's first layer, then
  `fn1`..`fn3` in m1, `fn5`..`fn7` in m2, `fn9`..`fn11` in m3. `FN(n)` takes
  the same absolute number.
- Macros are `[mN.macros]`, numbered 1..32 like NuPhy IO's MACRO 1..32 and
  bound with `MACRO(n)`: a chord `"LCTL+C"` or explicit steps
  `["LCTL down 1", "C down 1", "C up 1", "LCTL up"]` (trailing number: delay
  in ms). Macros are numbered per mode: m1's MACRO(1) and m2's are different.
- `MACRO(n)` plays once per press. Do not bind `MACRO(n, toggle)` (repeat
  until pressed again) to modifier chords: stopping it can leave Ctrl held
  until the keyboard is unplugged. `MACRO(n, hold)` repeats while held.
- If `apply` or `export` refuses because a previous write did not finish,
  run the `nuphyctl he restore-backup <file>` it names before anything else.
- To undo a write: `nuphyctl he restore-backup ~/.local/state/nuphyctl/he-<stamp>.bin`
  (the path `apply` printed).
