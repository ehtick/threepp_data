# Franka Robotics FR3 — licensing notice

**Copyright 2023 Franka Robotics GmbH.**

The files in this directory come from `franka_description`, licensed under the
**Apache License 2.0** — the complete text is in [`LICENSE`](LICENSE) beside this file, and
the upstream [`NOTICE`](NOTICE) is carried forward verbatim as Apache-2.0 §4(d) requires.

Note that upstream's `LICENSE` is not a stock Apache-2.0 file: it is the Apache-2.0 text
followed by a separator and an additional **BSD 3-Clause** block naming Willow Garage, Inc.
Both blocks are reproduced here exactly as published. Do not replace this file with a
generic Apache-2.0 text.

## Provenance

| | |
|---|---|
| Source | <https://github.com/frankarobotics/franka_description> |
| Commit | `02afaae282d4a8e10d7d2f781b23b3515c303ce5` |
| Retrieved | 2026-08-04 |
| Upstream version | 2.8.1 (`package.xml`) |
| Contents | `fr3.urdf` (generated, see below), 10 visual `.dae` and 9 collision `.stl` meshes, unmodified |

## Modifications

The meshes are byte-for-byte upstream. `fr3.urdf` is **generated** and does not exist
upstream — `franka_description` ships only `.xacro`, no plain URDF anywhere in the
repository. It was produced by threepp's own xacro engine:

```
xacro_to_urdf franka_description/robots/fr3/fr3.urdf.xacro fr3.urdf \
    --package franka_description=<clone>
```

`xacro_to_urdf` is checked into threepp at `examples/loaders/xacro_to_urdf.cpp`, so this
file can be regenerated from the commit above at any time.

Two changes were then applied to the expanded output:

1. `package://franka_description/meshes/...` mesh URIs were rewritten to paths relative to
   this directory (`meshes/visual/...`, `meshes/collision/...`), matching how every other
   robot in this repository refers to its meshes. This lets the URDF load with no package
   search path configured.
2. A provenance header comment was inserted at the top of the file.

Expansion used the xacro defaults throughout — no argument overrides at all — which means
`hand:=true` and `ee_id:=franka_hand`, so the arm ships **with** the Franka Hand, including
both prismatic finger joints and the `fr3_hand_tcp` tool frame, and `gazebo:=false`, so the
robot is rooted at a plain `base` link rather than a Gazebo `world` link.

### Regenerating this file requires threepp d5e5d8cc or newer

An earlier revision of this asset was generated with `no_prefix:=` (an empty value) as a
deliberate override, because threepp's xacro engine evaluated a `$(arg)` value spelling a
boolean as a non-empty *string* — `"false"` is truthy in a Python expression — while real
xacro types it as a boolean. That single divergence broke this description twice:

- `franka_arm.xacro` derives the arm prefix as `${'' if no_prefix else arm_prefix +
  robot_type + '_'}` while the hand derives its own from `robot_type` alone. With the string
  truthy, the arm expanded unprefixed (`link8`) and the hand still expanded prefixed, then
  attached to `fr3_link8` — a link that did not exist. **The whole hand was a detached
  subtree.**
- `franka_arm.xacro` also blanks `connected_to` under `${gazebo and connected_to ==
  'base'}`. With `"false"` truthy that guard fired, and the **`base` link and
  `fr3_base_joint` were silently suppressed** — output that matched neither `gazebo:=false`
  (base link, no world link) nor `gazebo:=true` (world link, no base link).

threepp commit `d5e5d8cc` fixed the engine, and the defaults now reproduce this file exactly;
the empty-value override is dead and produces byte-identical output. Regenerating with an
older threepp will silently produce a robot with a detached gripper and no root link.

## Known divergence from upstream

Upstream couples the two fingers with `<mimic joint="fr3_finger_joint1"/>` on
`fr3_finger_joint2`. threepp does not implement `<mimic>`, so the second finger loads as an
independent prismatic DOF rather than a slave of the first. The two fingers are sibling
branches off the `fr3_hand` link rather than a closed loop, so this costs no topology — only
the equality constraint, which a caller recovers by commanding both joints to the same
target. Total gripper opening is 0 to 0.08 m (0 to 0.04 m per finger).

## Trademarks

"Franka", "Franka Robotics" and "FR3" are trademarks of Franka Robotics GmbH. Their
appearance here is nominative — this is that company's published robot description — and is
not an endorsement of, or affiliation with, threepp.

## Redistribution

Apache-2.0 permits redistribution with attribution. Anyone redistributing this directory
must carry `LICENSE` and `NOTICE` with it, and state the modifications above — which is what
this file is for.
