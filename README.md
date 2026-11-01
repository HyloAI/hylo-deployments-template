# \<lab\>-deployments (template)

Template for a lab's own, private repository of what runs on its robots: one workcell config per robot or
simulator, with its calibration, safety limits and keep-out zones, and how it reaches its policy server. It holds
configuration only; components are named by what the worldcurrent repositories register.

| File | Workcell |
|---|---|
| `workcells/sim_arm.yaml` | a simulated arm (no hardware): bring-up and operator flow |
| `workcells/so101.yaml` | an SO-101 arm through LeRobot's Robot interface |
| `workcells/libero10_sim.yaml` | LIBERO-10 in simulation, with the robot-side runtime |

```bash
wc serve workcell --config workcells/sim_arm.yaml --port 8102
wc serve workcell --config workcells/so101.yaml --host 0.0.0.0 --port 8100 --token $WC_TOKEN
```

Configs name runtimes (`remote: {runtime: robot}`), never paths: each machine's execution profile says where the
runtime's venv is and which environment variables it needs (`wc profile set-runtime robot --python ... --env ...`;
bench's LIBERO setup registers `libero-sim` and `lerobot`). Keep tokens out of the repository (environment
variables).

Episodes recorded on the robot computer reach the workstation through the store (RFC-0005): register the
directory the workcell writes to (`wc store init runs` on the robot, where `output_dir: runs/workcell` lands), then
on the workstation `wc store pull robot-host:/path/to/runs` copies the runs it does not have yet, and
`wc store push robot-host:/path/to/runs artifact:<name>@<digest>` sends an artifact (a judge, a memory) the
other way. Sealed artifacts are never overwritten; the same path holding different content is a conflict.
