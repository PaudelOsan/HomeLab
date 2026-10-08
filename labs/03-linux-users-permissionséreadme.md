# Lab 03 — Linux Users & Permissions

## Goal

Learn how Linux users, groups, ownership, and permissions work.

## What I practiced

### Users & Groups

```bash
whoami
id
groups
```

* UID = user ID
* GID = group ID
* Users can belong to multiple groups.

### File Permissions

```bash
ls -l
ls -la
```

Permissions are divided into:

* Owner
* Group
* Others

```text
r = read
w = write
x = execute
```

Example:

```text
-rw-rw-r--
```

Linux checks one permission class:

* Owner permissions if I own the file
* Group permissions if I'm not the owner but belong to the file's group
* Others permissions otherwise

### Numeric Permissions

```text
r = 4
w = 2
x = 1
```

```text
644 = rw-r--r--
750 = rwxr-x---
640 = rw-r-----
```

### Changing Permissions

```bash
chmod u+w file
chmod g-w file
chmod o+x file
chmod 750 file
```

### Ownership

```bash
chown user:group file
```

`chown` changes the owner and/or group.

### Detailed File Information

```bash
stat file
```

Used it to check:

* UID
* GID
* Owner
* Group
* Permissions

### Directory Permissions

```text
r = list contents
w = create/delete/rename entries
x = access/traverse
```

`w` and `x` are generally needed together to create, delete, or rename entries.

### Permission Troubleshooting

```bash
namei -l /path/to/file
```

Used to check permissions along the entire path.

Basic troubleshooting flow:

```text
ls -l
↓
Check owner/group
↓
Check my user/groups
↓
Check permissions
↓
namei -l
↓
chmod / chown if needed
```

## Lab Status

Completed.
