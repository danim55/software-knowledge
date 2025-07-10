## Table of Contents

1. [Configuring Nfs](#configuring-nfs)
   1. [ `/etc/fstab` - the filesystem table](#etcfstab--the-filesystem-table)  
   1. [Mounting - attaching a filesystem](#mounting--attaching-a-filesystem)  
   1. [NFS - Network File System](#nfs--network-file-system)  
   1. [Example of use](#example-of-use)  
      1. [Why this matters](#why-this-matters)  
      1. [Essence](#essence)  

---

# Configuring NFS

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

---

## Mounting - attaching a filesystem

### What it means  
The `mount` command tells the kernel to attach a filesystem (local disk, CD, NFS export, etc.) and make it available under a directory in the existing directory tree.

### The mount point  
A regular directory (empty, or already a mount point) where the filesystem's contents appear. Create it with:

```bash
mkdir -p /path/to/mountpoint
```

---

## NFS - Network File System

### What NFS is

A protocol (and set of kernel drivers) that lets one machine share directories over the network and lets clients mount them as if they were local disks.

### How it works under the hood

1. **RPC (Remote Procedure Call):** Uses `rpcbind` to negotiate service ports.
1. **TCP/UDP on port 2049:** File operations—`read`, `write`, `lookup`—happen over TCP or UDP (here forced via `proto=tcp,port=2049`).
1. **Kernel module:** The Linux NFS client module translates local `open()`, `read()`, `write()` calls into network requests.

---

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

After this, `/mnt/nfs_logs` behaves like a local folder—its contents are served live from the NFS server.

---

   ### Why this matters

   * **Automation & consistency:** New VMs automatically mount the same log share.
   * **Performance & reliability:** Matching `defaults,proto=tcp,port=2049` ensures consistent transport.
   * **Seamless tooling:** Standard Linux commands (`ls`, `cp`, `tar`, etc.) work transparently on remote files.

   ---

   ### Essence

   * **`fstab`** declares what to mount where.
   * **`mount`** performs the action.
   * **NFS** provides a networked filesystem interface for remote directories.
