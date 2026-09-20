# Lab 02 — Linux Networking

## Goal

Practice troubleshooting a Linux computer that says Ethernet is connected but network access is not working.

## Network

Interface: `eno1`

IPv4: `192.168.2.51/24`

Gateway: `192.168.2.1`

DNS 1: `192.168.2.1`

DNS 2: `207.164.234.193`

MTU: `1500`

## Commands Used

```bash
ip route
ping -c 4 192.168.2.1
ping -c 4 8.8.8.8
ping -c 4 google.com
resolvectl status
ip -br addr
ip neigh
journalctl -u NetworkManager
nmcli device show eno1
```

## Troubleshooting

### 1. Check routing

`ip route`

Found:

```text
default via 192.168.2.1 dev eno1
192.168.2.0/24 dev eno1
```

The computer has a default route through `192.168.2.1`.

### 2. Test the gateway

```bash
ping -c 4 192.168.2.1
```

Result: 4/4 packets received, 0% packet loss.

The computer can reach the local router.

### 3. Test outside the local network

```bash
ping -c 4 8.8.8.8
```

Result: 4/4 packets received, 0% packet loss.

The computer can reach outside the local network.

### 4. Test DNS

```bash
ping -c 4 google.com
```

Result: 4/4 packets received, 0% packet loss.

This confirmed that DNS resolution and network connectivity were working.

### 5. Check DNS configuration

```bash
resolvectl status
```

Confirmed DNS configuration for `eno1`.

### 6. Check IP addresses

```bash
ip -br addr
```

Found:

```text
lo     127.0.0.1/8
eno1   192.168.2.51/24
```

`lo` is the loopback interface.

`eno1` is the physical Ethernet interface.

### 7. Check ARP / neighbors

```bash
ip neigh
```

The routing gateway `192.168.2.1` had a MAC address associated with it.

This shows how the computer finds the MAC address needed for the next local Ethernet hop.

### 8. Read NetworkManager logs

```bash
journalctl -u NetworkManager
```

Learned to read logs by looking at:

`Time → program → level → event`

Important events included NetworkManager starting, detecting `eno1`, managing the interface, and initializing DHCP.

### 9. Check detailed Ethernet configuration

```bash
nmcli device show eno1
```

Confirmed:

- Ethernet connected
- MTU 1500
- IPv4 address `192.168.2.51/24`
- Gateway `192.168.2.1`
- DNS servers
- Local route `192.168.2.0/24`
- Default route `0.0.0.0/0`

## What I Learned

Routing is checked before ARP.

Same subnet → the computer sends directly to the destination after finding its MAC address.

Different subnet → the computer uses the gateway and finds the gateway's MAC address.

The IP packet keeps the final destination IP, while the Ethernet frame is addressed to the MAC address of the current local hop.

`ip route` shows where packets should go.

`ip neigh` shows IP-to-MAC neighbor information.

`ping` can test different parts of the network depending on the destination.

`ping 192.168.2.1` tests the local gateway.

`ping 8.8.8.8` tests connectivity beyond the local network.

`ping google.com` tests both DNS resolution and connectivity.

`resolvectl status` shows DNS configuration/status.

`nmcli` shows NetworkManager's network configuration.

`journalctl` shows service logs.

## Lab Status

Complete
