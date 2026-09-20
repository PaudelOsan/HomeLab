# Lab 01 — Linux Fundamentals

## Goal

Learn basic Linux commands, processes, services, storage, and networking.

## Lab PC

* Lenovo ThinkCentre M800
* i5-6400 | 4 cores / 4 threads
* 32 GB RAM
* 256 GB NVMe — lab drive
* 180 GB SATA SSD — Windows 11
* Intel I219-LM 1 GbE

## Ubuntu Live

Booted Ubuntu 26.04.1 LTS from USB using UEFI.
Used **Try Ubuntu** only. Did not install Ubuntu on the internal drives.

## Linux Basics

`whoami` → shows current user
`pwd` → shows current directory
`ls` → lists files/directories
`cd` → changes directory
`cd ..` → goes to parent directory
`cd /` → goes to root directory `/`

Important directories:

* `/home` → user files
* `/etc` → configuration
* `/var` → logs/data that change
* `/tmp` → temporary files
* `/dev` → devices
* `/proc` → process/kernel information
* `/boot` → boot-related files

## Storage

`lsblk` → shows block devices
`lsblk -f` → shows filesystem and mount information
`df -h` → shows mounted filesystem space usage

Storage found:

* `sda` → USB (~477 GB)
* `sdb` → Windows SATA SSD (~179 GB)
* `nvme0n1` → NVMe lab SSD (~256 GB)
* `sr0` → DVD drive
* `loop*` → virtual loop devices used by Ubuntu Live/Snap

Mount point = directory where a filesystem is attached to Linux's directory tree.

`/cdrom` → Ubuntu Live USB
`/cow` → temporary writable layer for the Live session
`tmpfs` → temporary filesystem backed by RAM

## Memory

`free -h` → RAM and swap usage

My system:

* ~31 GiB RAM available to Linux
* Swap: 0

`buff/cache` is RAM Linux uses for caching and can generally reclaim when applications need it.

## CPU

`lscpu` → CPU information

* Intel Core i5-6400
* 1 socket
* 4 physical cores
* 4 hardware threads
* VT-x virtualization
* L1/L2/L3 cache information

## Processes

`top` → live process/resource view
`ps aux` → process list
`ps -p $$ -o user,pid,ppid,comm` → information about my current shell
`pstree -p 1` → process hierarchy

PID = Process ID
PPID = Parent Process ID

Example:
`bash` → child process of terminal
terminal → child of another process
`systemd` → PID 1, root of the normal userspace process tree

## Services

`systemctl --type=service --state=running` → running services
`systemctl status NetworkManager` → service status
`systemctl show NetworkManager -p MainPID` → main process ID
`ps -p 2024 -o user,pid,ppid,stat,%cpu,%mem,comm,args` → inspect NetworkManager process
`journalctl -u NetworkManager -n 10 --no-pager` → recent NetworkManager logs

Learned:

* **Service** = function provided by the system
* **Daemon** = background process providing that function
* **systemd** = manages services/process startup
* `enabled` = starts automatically
* `active/running` = running right now

## Networking

`ip -br addr` → interfaces and IP addresses
`ip route` → routing table
`ip neigh` → ARP/neighbour information

My network:

* Interface: `eno1`
* IPv4: `192.168.2.51/24`
* Gateway: `192.168.2.1`

Same subnet:
`192.168.2.x` → local network → ARP finds destination MAC → Ethernet frame sent directly.

Different subnet:
→ routing chooses gateway → ARP finds gateway MAC → frame goes to router → router forwards packet toward destination.

## Network Troubleshooting

Scenario: Ethernet connected but network access not working.

`ping -c 4 192.168.2.1`
→ tested connection to local router
→ 4/4 packets received

`ping -c 4 8.8.8.8`
→ tested connectivity beyond the local network
→ 4/4 packets received

This confirmed:
PC → Ethernet → Router → Internet path was working.

## Key Things I Learned

* Linux uses one directory tree starting at `/`.
* A mount point is a directory where a filesystem is attached.
* `lsblk` shows storage devices; `df -h` shows mounted filesystem usage.
* Processes have PIDs and parent/child relationships.
* systemd manages services.
* Routing decides where packets go.
* ARP finds the MAC address for the next local Ethernet hop.
* Same subnet = direct local delivery.
* Different subnet = send to the gateway.
* A successful gateway ping does not automatically prove Internet access.

## Lab Status

**Complete**
