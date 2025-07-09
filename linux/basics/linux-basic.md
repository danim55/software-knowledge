## Linux Basics Documentation

This directory (`linux/basics`) contains foundational topics on Linux system administration. Each topic is stored in its own Markdown file.

### Sections

- [Filesystem Management][fstab]  
- [User and Group Management][users]  
- [Package Management][packages]  

---

## 1. `/etc/fstab` – the filesystem table

**What it is**  
A plain‑text file that tells Linux which filesystems to mount at boot (or on demand). Each line has six fields:

1. **Device or remote export** (e.g. `10.11.12.20:/path`)  
2. **Mount point** (local directory, e.g. `/mnt/nfs_logs`)  
3. **Filesystem type** (here, `nfs`)  
4. **Mount options** (comma‑separated flags, e.g. `defaults,proto=tcp,port=2049`)  
5. **Dump** (usually `0`)  
6. **fsck order** (usually `0` for network filesystems)

**Why it matters**  
When you run `mount -a` or reboot, Linux reads `/etc/fstab` and automatically mounts every entry.

---

## 2. Mounting – attaching a filesystem

**What it means**  
The `mount` command tells the kernel to attach a filesystem (local disk, CD, NFS export, etc.) and make it available under a directory in the existing directory tree.

**The mount point**  
A regular directory (empty, or already a mount point) where the filesystem’s contents appear. Create it with:

```bash
mkdir -p /path/to/mountpoint
````

---

## 3. NFS – Network File System

**What NFS is**
A protocol (and set of kernel drivers) that lets one machine share directories over the network, and lets clients mount them as if they were local disks.

**How it works under the hood**

1. **RPC (Remote Procedure Call):** Uses `rpcbind` to negotiate service ports.
2. **TCP/UDP on port 2049:** File operations—`read`, `write`, `lookup`—happen over TCP or UDP (here forced via `proto=tcp,port=2049`).
3. **Kernel module:** The Linux NFS client module translates local `open()`, `read()`, `write()` calls into network requests.

---

## 4. Putting it all together

1. **Create the mount directory**

   ```bash
   mkdir -p /mnt/nfs_logs
   ```

2. **Add the share to `/etc/fstab`**

   ```fstab
   10.11.12.20:/home/.../log  /mnt/nfs_logs  nfs  defaults,proto=tcp,port=2049  0 0
   ```

3. **Mount all entries**

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

**Essence**

* **`fstab`** declares what to mount where.
* **`mount`** performs the action.
* **NFS** provides a networked filesystem interface for remote directories.

---