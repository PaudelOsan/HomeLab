---

## 9. Command Reference — Understand Every Part

This section explains the commands and options used in Lab 04. Use it as a quick reference whenever you forget what a command or flag does.

### A. Understanding command structure

Consider this command:

```bash
journalctl -u systemd-resolved -n 20 --no-pager
```

| Part | Meaning |
|---|---|
| `journalctl` | The program we want to run. It reads system logs. |
| `-u` | An option that filters logs by a systemd unit. |
| `systemd-resolved` | The service whose logs we want to inspect. |
| `-n` | An option that specifies how many recent entries to show. |
| `20` | The number of recent log entries to display. |
| `--no-pager` | A long-form option that prints output directly instead of opening an interactive pager. |

**Remember:** Options modify how a command behaves. The value after an option often tells the command what service, number, or other setting to use.

A single-dash option such as `-n` is commonly a short option. A double-dash option such as `--no-pager` is commonly a long option.

### B. Service management options

| Command or option | Meaning |
|---|---|
| `systemctl` | Interacts with the systemd service manager. |
| `status` | Displays service state and related details. |
| `is-active` | Checks whether a service is currently active. |
| `is-enabled` | Checks whether a service is configured to start automatically. |
| `start` | Starts a service. |
| `stop` | Stops a service. |
| `restart` | Restarts a service. |
| `enable` | Configures a service to start automatically at boot. |
| `systemd-resolved` | Ubuntu's DNS resolution service. |
| `SERVICE` | A placeholder. Replace it with a real service name; do not type the word `SERVICE` literally unless that is the actual unit name. |

Examples:

```bash
systemctl status cron
systemctl is-active cron
systemctl is-enabled cron
```

These commands inspect the `cron` service without starting, stopping, or restarting it.

**Important distinction:** `is-active` describes the current state; `is-enabled` describes the startup configuration. A service can be active but disabled, or inactive but enabled.

### C. Log inspection options

| Option or term | Meaning |
|---|---|
| `journalctl` | Reads the systemd journal. |
| `-u` | Filters by a unit, such as a service. |
| `-n 10` | Shows the 10 most recent matching entries. |
| `-n 20` | Shows the 20 most recent matching entries. |
| `-p err` | Filters by error priority and more severe priorities. |
| `--no-pager` | Prints output directly without opening a pager. |
| `-p` | Specifies a log priority filter. |
| `err` | The error priority name. |

Examples:

```bash
journalctl -u cron -n 20 --no-pager
journalctl -u systemd-resolved -p err --no-pager
```

The first command shows recent logs for `cron`. The second shows error-level and more severe entries for `systemd-resolved`.

### D. Process inspection options

| Command or option | Meaning |
|---|---|
| `ps` | Displays information about processes. |
| `aux` | A common `ps` option combination that displays processes across users, including those without a controlling terminal, with user-oriented details. |
| `pgrep` | Searches for processes by name or pattern. |
| `-a` with `pgrep` | Displays the process ID and full command line for each match. |
| `cron` | The process name or search pattern in this example. |
| `PID` | Process identifier: the number used to identify a running process. |
| `%CPU` | Reported CPU usage. |
| `%MEM` | Reported memory usage. |
| `STAT` | Process state information. |
| `COMMAND` | The command associated with the process. |

Examples:

```bash
ps aux
pgrep -a cron
```

**Why is `ps aux` written without dashes?** The `ps` command supports a historical BSD-style option format where `aux` is written without a leading dash. In this particular command, `a`, `u`, and `x` have separate meanings; they are not one single option named `aux`.

### E. Network socket options

| Option | Meaning |
|---|---|
| `ss` | Displays network socket information. |
| `-l` | Shows listening sockets. |
| `-t` | Selects TCP sockets. |
| `-u` | Selects UDP sockets. |
| `-n` | Shows numeric addresses and port numbers instead of translating them into names. |
| `-p` | Shows associated process information where permitted. |
| `sudo` | Runs a command with elevated privileges, subject to system permissions. |

Examples:

```bash
ss -ltn
ss -lun
sudo ss -ltpn
sudo ss -lupn
```

Breakdown:

- `ss -ltn`: listening TCP sockets, using numeric addresses and ports.
- `ss -lun`: UDP sockets, using numeric addresses and ports.
- `sudo ss -ltpn`: listening TCP sockets, numeric output, and process information where available.
- `sudo ss -lupn`: UDP sockets, numeric output, and process information where available.

The order of these combined short options generally does not change their meaning. For example, `-ltn` combines `-l`, `-t`, and `-n`.

### F. DNS and connectivity commands

| Command or option | Meaning |
|---|---|
| `getent` | Retrieves information through configured system lookup mechanisms. |
| `hosts` | Selects the host-name lookup database. |
| `google.com` | The hostname being looked up or tested. |
| `ping` | Sends ICMP echo requests and reports replies. |
| `-c 4` | Sends four ping requests before stopping. |

Examples:

```bash
getent hosts google.com
ping -c 4 google.com
```

`getent hosts google.com` tests whether the system can resolve the hostname.

`ping -c 4 google.com` tests whether the destination responds to four ICMP probes. It also relies on hostname resolution when a hostname is supplied.

### G. Important output terms and symbols

| Term or symbol | Meaning |
|---|---|
| `active (running)` | The service is currently running. |
| `enabled` | The service is configured to start automatically at boot. |
| `LISTEN` | A TCP socket is waiting for incoming connections. |
| `UNCONN` | A UDP socket has no established connection, which is normal for UDP. |
| `127.0.0.1` | The IPv4 loopback address. |
| `127.0.0.53` | A local address commonly used by Ubuntu's DNS stub resolver. |
| `127.0.0.54` | Another local DNS stub resolver address used by systemd-resolved. |
| `lo` | The loopback network interface. |
| `:53` | Port 53, the standard DNS port. |
| `:5353` | Port 5353, commonly used for multicast DNS. |
| `0.0.0.0` | In a local socket binding, commonly indicates all local IPv4 interfaces. Its meaning depends on the context. |
| `[::]` | The IPv6 unspecified address; in a listening socket display, it commonly indicates a wildcard IPv6 binding. |
| `0% packet loss` | All transmitted ping probes received replies in that test. |
| `time=` | The reported round-trip time for a ping reply. |

### H. A simple troubleshooting sequence

When a service or network problem is reported, follow this sequence:

1. **Check status:** Is the relevant service active?
2. **Check logs:** Are there errors that explain the symptoms?
3. **Check processes:** Is the expected process present?
4. **Check sockets:** Is the expected port or socket present?
5. **Test the outcome:** Does DNS resolution work? Does the destination respond?
6. **Document the result:** Record what worked, what failed, and what remains unknown.

Do not restart or change services just because a problem has been reported. First collect evidence and make changes only when the evidence supports them.

**Lab 04 takeaway:** Learn what each command tells you, understand what it cannot prove, and use multiple pieces of evidence before drawing a conclusion.
