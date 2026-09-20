Home Lab

My hands-on home lab for learning IT, Linux, networking, virtualization, servers, Docker, and troubleshooting.

I want to learn by actually doing things, testing them, breaking things safely, and figuring out why they work.

Lab PC

Lenovo ThinkCentre M800

Intel Core i5-6400

32 GB DDR4 RAM

256 GB NVMe SSD

~180 GB SATA SSD

Intel I219-LM 1 GbE

UEFI

VT-x / VT-d enabled

Labs

Lab 01 — Linux Fundamentals

Linux terminal basics

Filesystem and directories

whoami, pwd, ls, cd

Block devices and storage

lsblk, lsblk -f

Mount points

df -h

/cow in Ubuntu Live

RAM with free -h

CPU with lscpu

Processes, PID and PPID

ps, top, pstree

Services and systemd

systemctl

journalctl

Lab 02 — Linux Networking

Network interfaces

IPv4 and IPv6

Default gateway

Routing table

ARP / neighbor table

DNS

DHCP

NetworkManager

Reading network logs

Basic network troubleshooting

Commands practiced:

ip -br addr
ip route
ip neigh
ping -c 4 192.168.2.1
ping -c 4 8.8.8.8
ping -c 4 google.com
resolvectl status
nmcli device show eno1
journalctl -u NetworkManager

How I Learn

Learn → Explain → Quiz → Lab → Troubleshoot → Explain again

I don't want to just memorize commands. I want to understand what the commands show and why I would use them.
