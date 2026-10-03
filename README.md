# GRU-Drive

A distributed network drive built in **C++20 for Linux**, using machines you already own as storage nodes.
The master exposes a Linux Network Block Device (NBD), which you can format and mount as a normal folder.
Built as a final project at Infinity Labs (ILRD).

## How it works

```text
Linux filesystem -> /dev/nbd0 -> Master -> UDP -> Minions (backing files)
```

- **Master:** routes block requests, tracks responses, retries unanswered requests, and aggregates results.
- **Minions:** independent storage processes that serve reads, writes, and flushes from fixed-size backing files.
- **Replication:** a RAID10-style ring distributes 4 KiB stripes across nodes and mirrors each stripe onto the next node; a single minion supports a simpler local setup.
- **Framework:** a reusable reactor and thread pool dispatch commands through a factory, with timers for asynchronous work and support for loading command plugins at runtime.

Linux handles files and directories; GRU-Drive handles the blocks underneath.

## Quick start

Requires Linux, GNU Make, and a C++20-capable `g++`.
Run these commands from the repository root.

Try a local write/read simulation without root or an NBD device:

```bash
make minion master-sim
./scripts/run_single_machine_sim.sh
```

To run a mounted-drive demo with three local minions, use an unused NBD device:

```bash
make minion master-nbd
./scripts/setup_nbd.sh --nbd-device /dev/nbd0
./scripts/run_single_machine_nbd.sh --nbd-device /dev/nbd0 --minion-count 3
```

The NBD setup and master require `sudo`.
The script prints the formatting and mounting commands to run in another terminal; unmount the filesystem before stopping the demo.
Both local demo scripts recreate their backing files on each run, so demo data does not persist between runs.
See the [NBD demo guide](docs/current/nbd-step13-demo.md) for details.

Run the protocol, master, and minion tests with `make test`.
The framework tests are separate from that target.
For a 32-bit ARM minion build, use `make minion-rpi` with the `arm-linux-gnueabihf-g++` cross-compiler.

## Project layout

| Directory | Contents |
| --- | --- |
| [`framework/`](framework/) | Reactor, thread pool, command factory, scheduling, and plugin loading |
| [`concrete/common/`](concrete/common/) | Versioned wire protocol, serialization, UUIDs, and task types |
| [`concrete/master/`](concrete/master/) | NBD integration, stripe placement, replication, and request coordination |
| [`concrete/minion/`](concrete/minion/) | UDP request handling and backing-file storage |
| [`scripts/`](scripts/) | Build and demo helpers |
| [`docs/`](docs/) | Architecture, flows, diagrams, and development notes |

## Current status

This is a learning project intended for a trusted network: transport authentication and encryption are not implemented.
Replica selection and degraded writes are implemented for nodes marked inactive, but automatic failure detection and recovery are still planned.
Overlapping writes do not yet have ordering guarantees.
See the [known issues](docs/known-issues.md), [roadmap](docs/architecture/roadmap.md), and [framework architecture](docs/architecture/framework-architecture.md) for more detail.
