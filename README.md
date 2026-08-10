# sstatus
`sstatus` shows a quick overview of the current allocation status of a SLURM cluster with some colored bars per partition or node. It also shows some info about pending jobs and upcoming reservations. When a node is unavailable (based on reason codes), it will be marked with an asterisk `*`. Data is collected from `sinfo`, `squeue`, and `scontrol`. Column alignment scales dynamically to the length of node and partition names as well as the current terminal window size (best effort).

## Requirements
Just python3 and its default packages. A slurm cluster, obviously.

## Usage
```
$ sstatus -h
usage: sstatus [-h] [-n] [-c | -w COLUMNS] [-t] [--no-colour]

Displays a SLURM cluster allocation summary by partition or per node.
https://github.com/KasperSkytte/sstatus

options:
  -h, --help            show this help message and exit
  -n, --nodes           show per-node status instead of by partition.
  -c, --compact         don't stretch the table to fill the window, only print it as wide as
                        its contents require.
  -w COLUMNS, --width COLUMNS
                        fill this many columns instead of the width of the window.
  -t, --table-only      print only the table, without the surrounding job, queue and
                        reservation summaries.
  --no-colour, --no-color
                        don't colourize the bars.

Every option can also be set with an environment variable, so preferences can be exported
from ~/.bashrc: SSTATUS_NODES, SSTATUS_COMPACT, SSTATUS_WIDTH, SSTATUS_TABLE_ONLY and
SSTATUS_COLOUR. Booleans are true for 1, true, yes or on. Options given on the command line
take precedence.
```

By default the bars stretch to fill the terminal window. With `-c`/`--compact` the table is only as wide as it has to be: enough for the longest node and partition names, 10 characters per bar (8 for GPUs) or more if the allocated value needs it, plus the numbers printed on either side. That minimum always applies.

## Environment variables
Every option can also be set with an environment variable, so preferences can be exported from `~/.bashrc` instead of being typed on every run:

```bash
export SSTATUS_NODES=1       # same as -n
export SSTATUS_COMPACT=1     # same as -c
export SSTATUS_WIDTH=120     # same as -w 120
export SSTATUS_TABLE_ONLY=1  # same as -t
export SSTATUS_COLOUR=0      # same as --no-colour (SSTATUS_COLOR works too)
```

Booleans are true for `1`, `true`, `yes` or `on`, and false for anything else. Options given on the command line always win.

## Oversubscription
Partitions configured with `OverSubscribe=FORCE:n` in `slurm.conf` run up to `n` jobs per CPU, so their CPU totals are multiplied by `n` and the partition name gets an `(nx)` suffix. A partition holding a single 40 core node with `OverSubscribe=FORCE:4` reports 160 CPUs:

```
Partition        | Nodes |        CPUs        |   Memory (GB)
================================================================
interactive (4x) | 0/0/1 | 148 12        /160 | 31 31        /62
```

The per-node table (`-n`) scales a node's CPU total the same way, and a node in several oversubscribed partitions is scaled by the largest of their factors. Memory and GPUs are never scaled, since neither is oversubscribed. Neither is the `Nodes` column, which counts a node as allocated once its physical cores are taken.

Only `FORCE:n` is picked up. `OverSubscribe=YES:n` is left at 1x, because there the sharing only happens for jobs that explicitly ask for it, so the extra capacity isn't something the partition can be counted on to provide.

## Screenshots
### Per partition
![partition](www/sstatus_partition.png)

### Per node
![node](www/sstatus_node.png)
