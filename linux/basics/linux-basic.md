## Table of Contents

1. [Configuring nfs](#configuring-nfs)
   1. [ `/etc/fstab` - the filesystem table](#etcfstab--the-filesystem-table)  
   1. [Mounting - attaching a filesystem](#mounting--attaching-a-filesystem)  
   1. [nfs - Network File System](#nfs--network-file-system)  
   1. [Example of use](#example-of-use)  
1. [Configuring tls for nfs](#configuring-tls-for-nfs)
  1. [Common configuration](#common-configuration)
  1. [Server](#server)
1. [Configure log rotate](#configuring-log-rotate)
   1. [Example Configuration](#example-configuration)
   1. [Directive Breakdown](#directive-breakdown)
   1. [Testing and Troubleshooting](#testing-and-troubleshooting)


# Configuring nfs

## `/etc/fstab` - the filesystem table

### What it is  
A plain-text file that tells Linux which filesystems to mount at boot (or on demand). Each line has six fields:

1. **Device or remote export** (e.g. `10.11.12.20:/path`)  
1. **Mount point** (local directory, e.g. `/mnt/nfs_logs`)  
1. **Filesystem type** (here, `nfs`)  
1. **Mount options** (comma-separated flags, e.g. `defaults,proto=tcp,port=2049`)  
1. **Dump** (usually `0`)  
1. **fsck order** (usually `0` for network filesystems)

### Why it matters  
When you run `mount -a` or on each boot, Linux reads `/etc/fstab` and automatically mounts everything it finds there.


## Mounting - attaching a filesystem

### What it means  
The `mount` command tells the kernel to attach a filesystem (local disk, CD, nfs export, etc.) and make it available under a directory in the existing directory tree.

### The mount point  
A regular directory (empty, or already a mount point) where the filesystem's contents appear. Create it with:

```bash
mkdir -p /path/to/mountpoint
```

## nfs - Network File System

### What nfs is

A protocol (and set of kernel drivers) that lets one machine share directories over the network and lets clients mount them as if they were local disks.

### How it works under the hood

1. **RPC (Remote Procedure Call):** Uses `rpcbind` to negotiate service ports.
1. **TCP/UDP on port 2049:** File operations—`read`, `write`, `lookup`—happen over TCP or UDP (here forced via `proto=tcp,port=2049`).
1. **Kernel module:** The Linux nfs client module translates local `open()`, `read()`, `write()` calls into network requests.


## Example of use

### Create the mount directory

```bash
mkdir -p /mnt/nfs_logs
```

### Add the share to `/etc/fstab`

```fstab
10.11.12.20:/home/.../log  /mnt/nfs_logs  nfs  defaults,proto=tcp,port=2049  0 0
```

### Mount all entries

```bash
mount -a
```

After this, `/mnt/nfs_logs` behaves like a local folder—its contents are served live from the nfs server.


   ### Why this matters

   * **Automation & consistency:** New VMs automatically mount the same log share.
   * **Performance & reliability:** Matching `defaults,proto=tcp,port=2049` ensures consistent transport.
   * **Seamless tooling:** Standard Linux commands (`ls`, `cp`, `tar`, etc.) work transparently on remote files.


   ### Essence

   * **`fstab`** declares what to mount where.
   * **`mount`** performs the action.
   * **nfs** provides a networked filesystem interface for remote directories.

# Configuring tls for nfs

## Common configuration

1. Install the [ktls-utils](https://github.com/oracle/ktls-utils) both in server and client

1. Launch command to download the rpm:

```bash
    sudo dnf install ktls-utils -y
```

## Server

1. Connect to the remote machine

```bash
    ssh merinod@10.10.10.10
```

### Generate private key, request and certificate

1. Create temporary configuration file

```bash
    sudo vi /tmp/nfs-openssl.cnf
```

1. Add the following content

```bash
[ req ]
distinguished_name = req_distinguished_name
req_extensions     = v3_req
prompt = no

[ req_distinguished_name ]
CN = 10.11.12.30 # IP of the nfs server

[ v3_req ]
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
IP.1 = 10.11.12.30 # IP of the nfs server
```

1. Previous steps

```bash
    sudo -i
    umask 077
    cd /etc/pki/tls/certs
```

1. Generate key and assign correct permissions

```bash
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out /etc/pki/tls/private/nfs-server.key
    chown root:root /etc/pki/tls/private/nfs-server.key
    chmod 600 /etc/pki/tls/private/nfs-server.key
```

1. Generate request and certificate
```bash
    # Create CSR
    openssl req -new -key /etc/pki/tls/private/nfs-server.key -out /tmp/nfs-server.csr -config /tmp/nfs-openssl.cnf

    # Create self signed certificate
    openssl x509 -req -in /tmp/nfs-server.csr -signkey /etc/pki/tls/private/nfs-server.key -out /etc/pki/tls/certs/nfs-server-certificate.crt -days 365 -extensions v3_req -extfile /tmp/nfs-openssl.cnf

    sudo chmod 644 /etc/pki/tls/certs/nfs-server-certificate.crt
```

1. Edit `/etc/tlshd.conf` to use these files, using your own values for x509.certificate and x509.private_key:

```bash
    sudo vi /etc/tlshd.conf
```
The content must be:

```bash
[main]
tls=1
loglevel=3

[authenticate.server]
x509.certificate= /etc/pki/tls/certs/nfs-server-certificate.crt
x509.private_key= /etc/pki/tls/private/nfs-server.key
```

1. Copy the certificate of the client

1. Add certificate to trust store 

```bash
    sudo trust anchor certificate.crt
```

1. Modify `/etc/exports` to add the `xprtsec=mtls` configuration:

```bash
    sudo vi /etc/exports
```

Previous configuration
```
    /home/project/nfsshare/sub-project/log 10.11.12.0/24(rw,sync,no_subtree_check)
```

Current configuration
```
    /home/project/nfsshare/sub-project/log 10.11.12.0/24(rw,sync,no_subtree_check,xprtsec=mtls)
```

1. Start and enable tlshd.service

```bash
    systemctl start tlshd.service
    systemctl enable tlshd.service
```


## Configuring log rotate

## Configuring logrotate for `some-app` Logs

### Table of Contents

1. [Overview](#overview)  
2. [Example Configuration](#example-configuration)  
3. [Directive Breakdown](#directive-breakdown)  
4. [nfs-Specific Considerations](#nfs-specific-considerations)  
5. [Testing and Troubleshooting](#testing-and-troubleshooting)  


## Overview

`logrotate` is a utility to manage log file growth by rotating, compressing, and pruning old logs. You place per-application configurations in `/etc/logrotate.d/` to apply project-specific rotation policies.


### Example Configuration

```bash
echo "Configuring logrotate for some-app logs..."
cat <<EOF | sudo tee /etc/logrotate.d/some-app
/opt/nfsshare/nfsshare/some-app/log/*/*.log {
    rotate 4
    size 10M
    compress
    delaycompress
    dateformat -%Y%m%d_%H%M%S
    missingok
    notifempty
    copytruncate
}
EOF
```

This creates `/etc/logrotate.d/some-app` with rules for all `.log` files under `/opt/nfsshare/nfsshare/some-app/log/*/`.


### Directive Breakdown

* **`/opt/.../*.log { … }`**
  Glob pattern matching all log files in per-component subdirectories.

* **`rotate 4`**
  Keep 4 rotated archives before deleting the oldest.

* **`size 10M`**
  Rotate when the file grows beyond 10 megabytes.

* **`compress`**
  Gzip old log files to save space (creates `.gz`).

* **`delaycompress`**
  Postpone compression until the *next* rotation cycle. Ensures the most recent rotated file remains uncompressed for tools that expect a plain `.log`.

* **`dateformat -%Y%m%d_%H%M%S`**
  Append rotation timestamp to filenames, e.g. `app.log-20250731_231045.gz`.

* **`missingok`**
  Do not error if a log file is missing. Useful when components create logs on demand.

* **`notifempty`**
  Skip rotation if the log file is empty.

* **`copytruncate`**
  Copy the current log to a rotated file and then truncate the original.
  Ideal when the application cannot be signaled to close and reopen its log descriptor.


### nfs-Specific Considerations

* **Shared Storage Consistency**
  On nfs volumes, `copytruncate` avoids issues with file‐handle reopening across the network.

* **Locking and Concurrency**
  Ensure only one node runs `logrotate` for shared logs (e.g., via a cron job on a single "log‐owner" host) to prevent conflicting rotations.

* **Permission Alignment**
  The user running `logrotate` must have write permission on the nfs share and ownership or group access to the log files.


### Testing and Troubleshooting

1. **Dry-run**

   ```bash
   sudo logrotate --debug /etc/logrotate.d/some-app
   ```

   Verifies what `logrotate` *would* do without performing actions.

2. **Manual Rotation**

   ```bash
   sudo logrotate --force /etc/logrotate.d/some-app
   ```

   Immediately applies rotation rules for testing.

3. **Check for Errors**
   Inspect `/var/lib/logrotate/status` and system logs (e.g., `/var/log/syslog` or `journalctl`) for rotation errors.

4. **Verify Archive Files**
   After rotation, you should see files like:

   ```
   /opt/nfsshare/.../app.log-20250731_231045.gz
   ```


Place this file in `/etc/logrotate.d/some-app` and ensure a system cron or timer (typically `/etc/cron.daily/logrotate`) calls `logrotate` regularly (default: once per day). Adjust schedules by editing `/etc/cron.*` or using systemd timers as needed.
