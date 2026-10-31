# nuphy-config

Declarative config of my NuPhy Air60 HE: key actions and per-key travel,
applied over USB with [nuphyctl](https://github.com/yoshintame/nuphyctl)
(`he` commands, a fork with Hall-effect support). The keyboard stores it
onboard, so it follows the keyboard to any computer.

## Use

```sh
mise install        # builds nuphyctl from the pinned fork revision
mise run check      # validate air60he.toml offline
mise run diff       # what apply would change
mise run apply      # write it: backup first, then read back and verify
mise run export     # pull the keyboard's current state into air60he.toml
```

Close drive.nuphy.io before `apply`: two programs on the control interface
at once can confuse each other. Every write saves a backup to
`~/.local/state/nuphyctl/he-<stamp>.bin`; undo with
`nuphyctl he restore-backup <file>`.

## The file

```toml
[m1.base]                 # mode m1..m3, layer base | fn1 | fn2 | fn3
CAPS = "FN(2)"            # momentary layer, absolute number

[m1.travel]               # millimetres; unset fields keep the keyboard's value
"*" = { actuation = 0.5, trigger = "rapid" }
ESC = { actuation = 1.5, trigger = "normal" }
```

Only listed keys and fields are touched. NuPhy IO numbers layers across
modes: m1 is FN 0-3, m2 is FN 4-7, m3 is FN 8-11, so `m3.fn3` is its FN 11.
DKS, Mod-Tap, Toggle and Snap Tap keys (raw `0x9…` values) are set in NuPhy
IO; the file keeps them as they are. Syntax, action names and the safety
rules: [nuphyctl `he` docs](https://github.com/yoshintame/nuphyctl/blob/master/docs/usage.md#hall-effect-boards).
